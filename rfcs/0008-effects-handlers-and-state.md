# 0008-effects-handlers-and-state

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: closed effect row、built-in effects、handler、handler state
- Feature Name: `effects_handlers_and_state`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、状態変化と制御効果を explicit effect discipline の下へ集約する。この RFC は effect system の総論を固定する。closed effect row と built-in effect は `0032`、handler と handler state は `0033`、effect abstraction と limited polymorphism は `0034` で精密化する。

## Motivation

- Moti の目標には effect と可変状態の追跡が含まれている。
- しかし、row variable や複雑な handler discipline を surface へそのまま出すと、学習負荷が高い。
- 一般局所可変を消した以上、状態をどこへ押し出すかを明確にしなければならない。
- `Exn` と `Async` を同じ effect space で扱うことで、error model と concurrency model を接続できる。
- use case 1 は、純粋関数と状態を持つ処理を型だけで見分けたい library author である。
- use case 2 は、内部では effect を使うが、公開面では hidden control flow を残したくない API designer である。

## Guide-level explanation

Moti では「この関数が何をし得るか」は型に出る。状態を書き換えるなら `State`、例外的制御を使うなら `Exn`、非同期なら `Async` が見える。source では `/ {A, B}` のような closed row だけを見ればよく、推論のための内部変数を読み手に押し付けない。

handler は「effect を捕まえて局所的に意味付ける」制度だが、まずは deep-only / one-shot に限定する。これで mental model を増やしすぎずに済む。

```moti
handle counter() with {
  state = 0
  on Count() k => resume k(state + 1) with state + 1
}
```

この RFC の中心決定は、「状態や例外的制御を、closed effect row と保守的 handler discipline の中でだけ表す」ことである。built-in effect 集合と handler state は、その decision を成立させる現行プロファイルとして位置付ける。

## Reference-level explanation

- 分類: closed effect row の surface 露出は恒久不変である。
- 分類: built-in effect の最小集合は現行プロファイルである。
- 分類: deep-only / one-shot handler は現行プロファイルである。
- source で露出する effect row は closed row のみとする。
- built-in effect は `IO`、`Diverge`、`Async`、`Parallel`、`State[S]`、`Exn[E]` を基礎集合とする。
- row は順序非依存・重複なしの正規形を持つ。
- handler は deep-only を既定とし、shallow handler は導入しない。
- 継続再開は one-shot とし、複数回 resume を許さない。
- 一般局所可変は持たず、handler state を専用構文で表す。
- `Exn` は effect space の一員だが、公開境界への正規化規則は `0010-error-model-and-result-boundaries.md` を正本とする。
- closed effect row と built-in effect 集合の正本は `0032` とする。
- handler discipline と handler state の正本は `0033` とする。
- effect abstraction と limited polymorphism の正本は `0034` とする。

## Drawbacks

- effect と handler の概念を理解するコストがある。
- closed row only により、一部の polymorphism 表現が冗長になる。
- handler の保守的制約で、表現力より predictability を優先する。

## Rationale and alternatives

- 代替案 1 は例外と状態を構文糖に埋め込む案だったが、追跡されない制御フローを増やすため採らない。
- 代替案 2 は shallow / multi-shot handler を許す案だったが、mental model と runtime cost model を重くするため採らない。
- 代替案 3 は `var` を復活させる案だったが、宣言的デフォルトと衝突するため採らない。
- 何もしない場合、effect system はあるが state discipline が曖昧な言語になる。

## Prior art

- Koka は effect-first の mental model において強い参考になる。
- algebraic effects / handlers 系の研究は豊富だが、Moti は表現力を狭めてでも予測可能性を優先する。
- Rust の `Result` / `async` は explicit だが、effect row までは持たない。
- lesson は「surface へ internal solver detail を出さないこと」、divergence は「Moti が deep-only / one-shot を current profile として先に固定すること」である。

## Unresolved questions

- built-in effect の最小集合をどこまで strict にするか。
- handler state の surface syntax をどこまで固定するか。

## Future possibilities

- closed row discipline を壊さない範囲での surface sugar。
- limited effect polymorphism の表層露出。
