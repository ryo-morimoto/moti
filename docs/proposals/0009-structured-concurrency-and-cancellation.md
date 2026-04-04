# 0009-structured-concurrency-and-cancellation

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: structured concurrency、`Async` / `Parallel` 分離、cancellation、deadline
- Feature Name: `structured_concurrency_and_cancellation`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の非同期は structured concurrency を前提とし、task lifecycle を構造化 scope の中へ閉じ込める。`Async` と `Parallel` を分け、task は scope 外へ浮遊しない。現行プロファイルでは cancellation と deadline を task context に組み込みで伝播し、cross-domain 実行には `Send` / `Share` 規律を要求する。

## Motivation

- Moti はシンプルに使える構造化非同期を要件に持つ。
- detached task を許すと、所有権、cancel、error propagation、resource cleanup の責任が散る。
- `Async` と `Parallel` を分けないと、same-domain suspension と cross-domain send/share 制約が混ざる。
- deadline と cancellation を library 慣習へ委ねると、公開 API の mental model が揺れる。
- use case 1 は、親 task を抜けるときに child cleanup まで保証したい concurrent API author である。
- use case 2 は、same-domain `await` と cross-domain parallel spawn を別物として読める必要がある programmer である。

## Guide-level explanation

Moti の async では、task は必ずどこかの scope に属する。親 scope は child を見失わない。`await` は same-domain の suspension であり、cross-domain の並行実行は `Parallel` によって明示される。

読み手は次だけ覚えればよい。

- detached task はない
- cancel は scope に沿って伝播する
- `Async` と `Parallel` は別 effect である
- `Parallel` へ送る値には `Send` / `Share` 規律が掛かる

```moti
with_task_group {
  let a = async fetch_a()
  let b = async fetch_b()
  await a
  await b
}
```

この RFC の中心決定は、「task lifecycle を構造化 scope に閉じ込め、非同期の責任帰属を tree に沿って読む」ことである。`Async` / `Parallel` 分離、cancel / deadline、`Send` / `Share` はその decision を成立させる制度として位置付ける。

## Reference-level explanation

- 分類: structured concurrency と detached task 不採用は恒久不変である。
- 分類: `Async` / `Parallel` 分離は恒久不変である。
- 分類: cancellation / deadline の組み込み伝播は現行プロファイルである。
- 分類: `Send` / `Share` の導出方式は現行プロファイルである。
- task 生成は構造化 scope 内でのみ許す。
- parent scope は child 完了または cancel+join 前に終了できない。
- `Async` は same-domain suspension を表す。
- `Parallel` は cross-domain 並行実行を表す。
- cancellation と deadline は task context で伝播する。
- sibling cancellation は typed result / policy に従って行い、暗黙例外で伝播しない。
- `Send` / `Share` は user-defined manual impl を許さず、構造導出を採る。
- lexical borrow と await-affine capability は `Parallel` へ持ち込めない。

## Drawbacks

- detached background work の記法的な手軽さを失う。
- 一部の既存 async API 慣習と相性が悪い。
- cancellation policy を明示する分、API surface がやや重くなる。

## Rationale and alternatives

- 代替案 1 は detached task を許す案だったが、resource lifetime と error propagation を曖昧にするため採らない。
- 代替案 2 は `Async` と `Parallel` を統合する案だったが、same-domain / cross-domain の安全条件が混ざるため採らない。
- 代替案 3 は cancel / deadline を純 library convention に任せる案だったが、mental model が揺れるため採らない。
- 何もしない場合、Moti の async は「型は強いが実行責務が不透明」な制度になる。

## Prior art

- Go の goroutine は表現力が高いが、構造化を別 discipline に任せる。
- Kotlin / Swift などの structured concurrency は task tree と cancellation 伝播の重要性を示す。
- Rust async は explicit だが、runtime / cancellation story が ecosystem 依存になりやすい。

## Unresolved questions

- task group API の最終 surface shape。
- cancellation の policy hook をどこまで標準化するか。

## Future possibilities

- stream / channel / actor の上位 abstraction。
- distributed runtime profile への拡張。
