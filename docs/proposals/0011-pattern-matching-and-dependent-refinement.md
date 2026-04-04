# 0011-pattern-matching-and-dependent-refinement

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: exhaustive match、guard 不採用、inaccessible pattern、absurd pattern、refinement
- Feature Name: `pattern_matching_and_dependent_refinement`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の pattern matching は exhaustive であり、partial match と guard に依存しない。dependent refinement は inaccessible pattern と absurd pattern で表し、codata は projection-only のため `match` 対象にしない。pattern matching は制御分岐の convenience ではなく、型で追跡可能な case analysis として設計する。

## Motivation

- Moti は追跡されないエラーパスと暗黙制御フローを減らすことを目標にしている。
- partial match や guard は local convenience を与える一方で、網羅性と reasoning を弱くする。
- dependent typing を持つ以上、pattern match は refinement を伴う主要な証明手段になる。
- codata を `match` で観測させると kernel safety と衝突する。

## Guide-level explanation

Moti の `match` は「とりあえず書いておいて抜けは runtime で落ちる」構文ではない。全分岐を列挙し、必要なら型の側が不可能な分岐を inaccessible / absurd として示す。

```moti
match value {
  Ok(x) => ...
  Err(e) => ...
}
```

読み手は、`match` を見れば失敗や分岐の全体像を追えるべきであり、guard の後ろに隠れた別経路を探す必要がない。

## Reference-level explanation

- pattern matching は exhaustive を要求する。
- partial function を `match` の抜けで表してはならない。
- guard は導入しない。
- inaccessible pattern と absurd pattern を refinement 表現として許す。
- equation style は canonical core へ一方向 elaboration される場合に限って許す。
- codata は projection-only とし、`match` 対象にしない。
- record pattern と literal pattern は正規化規則を持つ。

## Drawbacks

- guard がないため、一部の書き慣れた分岐記法を失う。
- inaccessible / absurd pattern は学習コストがある。
- exhaustiveness のために冗長な分岐が必要になることがある。

## Rationale and alternatives

- 代替案 1 は guard 付き match だったが、control flow の局所推論を難しくするため採らない。
- 代替案 2 は wildcard で未処理分岐を逃がす案だったが、typed exhaustiveness に反するため採らない。
- 代替案 3 は codata matching を許す案だったが、projection-only の kernel rule と衝突する。
- 何もしない場合、pattern match は dependent refinement の核になれず、ただの convenience syntax に留まる。

## Prior art

- Agda / Idris / Lean の inaccessible / absurd pattern は直接の参考になる。
- Rust や ML の exhaustive pattern match は重要な基礎だが、dependent refinement までは持たない。

## Unresolved questions

- case tree elaboration を RFC 本文でどこまで formal に記述するか。
- equation style の admission criteria をどこまで厳密にするか。

## Future possibilities

- pattern synonym 的な表面 sugar。
- diagnostics で refinement 情報をどう表示するかの詳細化。
