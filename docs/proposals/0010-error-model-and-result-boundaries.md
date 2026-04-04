# 0010-error-model-and-result-boundaries

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `Result`、`Exn`、公開境界の失敗表現、panic / abort 不在
- Feature Name: `error_model_and_result_boundaries`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、公開境界の失敗を必ず `Result[T, E]` で表し、`panic` や `abort` を通常の意味論へ持ち込まない。`Exn[E]` は内部制御用の effect としてのみ許し、module boundary、public API、FFI wrapper、async surface へそのまま露出させない。

## Motivation

- Moti の要件には型付きで網羅的なエラーハンドリングが含まれている。
- 追跡されない失敗経路を残すと、safe fragment と低認知負荷の両方を壊す。
- `Exn` を user-facing error model として許すと、public API の contract が不透明になる。
- async、FFI、portable API を一貫した mental model で読むためには、公開境界で failure as data を固定する必要がある。

## Guide-level explanation

Moti で外に見える失敗は `Result` だけだと思ってよい。内部では `Exn` を使って control transfer を表してもよいが、それは handler で回収されて `Result` へ正規化される。

```moti
pub fn parse(text: Text): Result[Ast, ParseError] {
  ...
}
```

この mental model により、読み手は「どこで失敗しうるか」を型だけで追える。`panic` か `throw` かを別途探し回る必要がない。

## Reference-level explanation

- 公開 API の失敗は `Result[T, E]` で表す。
- `panic` と `abort` を通常の language law として持たない。
- `Exn[E]` は内部制御 effect としてのみ許す。
- `Exn[E]` は module boundary、public surface、portable public API、FFI wrapper、async API へそのまま露出しない。
- async task の失敗も typed result として回収する。
- negative path は exhaustiveness と diagnostics の対象になる。

## Drawbacks

- 失敗表現が verbose になりやすい。
- 一部の内部制御パターンでは `Exn` と `Result` の往復が必要になる。
- 既存言語の unchecked exception に慣れた利用者には窮屈に見える。

## Rationale and alternatives

- 代替案 1 は unchecked exception を公開面まで許す案だったが、error path を隠すため採らない。
- 代替案 2 は panic を非型付き failure として残す案だったが、typed exhaustiveness 目標に反する。
- 代替案 3 は `Exn` と `Result` を同じ surface で並立させる案だったが、利用者の mental model を二重化するため採らない。
- 何もしない場合、Moti の failure surface は強い型と裏腹に不透明なものになる。

## Prior art

- Rust の `Result` 中心文化は重要な参考になる。
- checked exception 言語は型付き失敗の意義を示すが、公開境界と internal control の分離は別途必要である。
- algebraic effects は `Exn` を内部制度として扱う構成の参考になる。

## Unresolved questions

- `Exn` の internal-only rule を surface syntax でどこまで示すか。
- large error enum の ergonomic support をどこまで標準化するか。

## Future possibilities

- richer error context の標準構文。
- domain-specific error taxonomy の標準ライブラリ支援。
