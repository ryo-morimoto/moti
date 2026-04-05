# 0038-pattern-coverage-and-exhaustiveness

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: exhaustive match、coverage checking、wildcard の位置づけ、guard 不採用
- Feature Name: `pattern_coverage_and_exhaustiveness`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の pattern matching は exhaustive でなければならず、partial match を runtime failure へ落としてはならない。coverage check は data constructor、literal、record pattern、wildcard を対象に行い、guard による coverage 補正は導入しない。pattern match は convenience syntax ではなく typed case analysis の公開面である。

## Motivation

- partial match は hidden error path を生み、Moti の typed error surface 目標と衝突する。
- guard を許すと coverage reasoning が局所式から外れてしまう。
- use case 1 は、`match` を見た時点で分岐の全体像を把握したい user である。
- use case 2 は、compiler が exhaustiveness diagnostics を安定して返せるようにしたい implementer である。
- no-op のままだと、pattern match が dependent refinement の基盤ではなく convenience の集合になる。

## Guide-level explanation

Moti の `match` は全分岐を列挙する。`_` は「未処理を runtime へ流す穴」ではなく、残りの被覆を明示的にまとめる pattern である。guard は導入しない。

```moti
match result {
  Ok(x) => use(x)
  Err(e) => handle(e)
}
```

読み手は、これで `result` の場合分けが完結していると期待できる。別の hidden path を探す必要はない。

## Reference-level explanation

- `match` は exhaustive でなければならない。
- wildcard pattern は coverage 上の残余集合を 1 つの枝で受け取る pattern として扱う。
- guard は導入しない。
- literal pattern、record pattern、enum constructor pattern は coverage tree 上で展開する。
- partial function を pattern 抜けや runtime trap で表してはならない。
- coverage check は dependent refinement を考慮しない base layer と、refinement-aware layer の二段階に分けてよいが、観測可能結論は同じ exhaustive requirement に従う。
- diagnostics は missing pattern family または redundant branch を示せなければならない。

## Drawbacks

- guard に慣れた利用者には冗長に見える。
- wildcard を多用すると explicit constructor 列挙より意図が薄く見えることがある。
- coverage checker 実装は pattern tree を必要とする。

## Rationale and alternatives

- 代替案 1 は partial match を warning に留める案だったが、typed error surface を弱めるため採らない。
- 代替案 2 は guard 付き match を導入する案だったが、coverage reasoning を複雑化するため採らない。
- 代替案 3 は functional core では exhaustive、script layer では partial 可と分ける案だったが、言語の mental model を二重化するため採らない。
- 何もしない場合、pattern match は失敗経路を隠す構文に戻る。

## Prior art

- ML、Rust の exhaustive match は基礎的な先行例である。
- Agda や Idris の coverage discipline は dependent layer と base layer の分離の lesson を与える。
- Moti は guard を採らず、coverage と refinement を分離して説明可能性を優先する。

## Unresolved questions

- OR pattern を導入する場合の coverage wording。
- warning と error の境界を redundant branch にどう割り当てるか。

## Future possibilities

- coverage witness を diagnostics に表示する。
- pattern synonym を入れる場合の coverage integration。
