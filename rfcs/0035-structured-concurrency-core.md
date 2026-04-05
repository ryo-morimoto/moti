# 0035-structured-concurrency-core

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: task tree、scope ownership、detached task 不採用、join 規律
- Feature Name: `structured_concurrency_core`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の async model は structured concurrency を核に持つ。task は必ず親 scope に属し、detached task は導入しない。親 scope は child task が完了または cancel+join される前に終了できず、task lifecycle は tree に沿って追跡される。

## Motivation

- async safety の本質は syntax よりも lifecycle ownership にある。
- detached task を許すと resource cleanup、error propagation、cancellation ownership が分散する。
- use case 1 は、親処理が終わるとき child cleanup まで保証したい API author である。
- use case 2 は、async code の lifetime を tree として読めるようにしたい programmer である。
- no-op のままだと、Moti の async は explicit でも ownership が曖昧な制度になる。

## Guide-level explanation

Moti では task は必ず scope の子である。scope を抜ける前に child は片付いていなければならない。バックグラウンドへ投げっぱなしの task はない。

```moti
with_task_group {
  let a = async fetch_a()
  let b = async fetch_b()
  await a
  await b
}
```

読み手は、この block を抜ける時点で `a` と `b` が scope の責任下で処理されると理解すればよい。task は tree から浮遊しない。

```moti
with_task_group {
  let child = async watch_stream()
  cancel child
  await child
}
```

この例では cancel だけで終わらず、scope を抜ける前に `await child` で join する。parent は `cancel+join` を含めて child の後始末責任を持つ。

## Reference-level explanation

- task 生成は構造化 scope 内でのみ許す。
- 各 task は一意の親 scope を持つ。
- detached task は language feature として導入しない。
- parent scope は child task が完了、または policy に従って cancel+join される前に終了できない。
- task handle は parent-child 所有関係を反映し、join responsibility を隠してはならない。
- lexical borrow、await-affine capability、domain-local capability の持ち込み可否は別 RFC の規律に従う。
- `Async` は same-domain suspension を、`Parallel` は cross-domain 並行実行を表すが、両者とも task tree owner を持つ。
- runtime profile は task tree を実装できなければ standard profile を満たさない。

## Drawbacks

- detached background work を簡単に書けない。
- 一部 ecosystem 由来 API をそのまま輸入しにくい。
- task scope を explicit に持つため、短い script でも構造が見える。

## Rationale and alternatives

- 代替案 1 は detached task を許す案だったが、resource ownership が散るため採らない。
- 代替案 2 は structured concurrency を library convention に留める案だったが、言語の async safety story として弱すぎるため採らない。
- 代替案 3 は `Async` と `Parallel` の差を隠す案だったが、same-domain / cross-domain 規律を曖昧にするため採らない。
- 何もしない場合、Moti の async は「typed だが片付け責任が不透明」になる。

## Prior art

- Kotlin、Swift などの structured concurrency は task tree ownership の lesson を与える。
- Go の goroutine は表現力が高いが lifecycle ownership を別 discipline に任せる。
- Moti は detached convenience を捨て、tree ownership を language law に置く。

## Unresolved questions

- task group syntax の最終表面形。
- root task の entrypoint API をどこに置くか。

## Future possibilities

- actor / stream / channel を task tree 上に乗せる上位 RFC。
- distributed structured concurrency profile。
