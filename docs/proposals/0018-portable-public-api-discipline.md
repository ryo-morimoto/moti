# 0018-portable-public-api-discipline

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: portable public API の規律、`Nat` / `Fin[n]` / `BoundsError`、公開 contract の正規形
- Feature Name: `portable_public_api_discipline`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の portable public API は、利用者が backend や boundary の詳細を知らなくても読める contract を持たなければならない。長さは `Nat`、添字は `Fin[n]`、境界違反は `Result[..., BoundsError]` で表し、公開面の shape を portable semantics に正規化する。boundary-only 非露出と platform-layer numerics の禁止は、それぞれ `0003` と `0017` の規律を参照して従う。

## Motivation

- Moti は型で隔離された trust boundary と portable semantics を目標にしている。
- raw pointer や raw handle を public surface へ出すと、boundary discipline が崩れる。
- `Nat` / `Fin[n]` を採ることで、length/index semantics を public contract に埋め込める。
- `ISize` / `USize` や backend-only detail を public API に出すと portability が壊れる。

## Guide-level explanation

Moti の public API は「内部でどう実装しているか」ではなく、「利用者が何を安全に期待できるか」を型で示す。配列長は `Nat`、有効添字は `Fin[n]`、境界違反は `BoundsError` で表し、raw な表現や capability を直接見せない。

```moti
pub fn check_index(xs: Vec[T], i: Nat): Result[Fin[len(xs)], BoundsError]
pub fn get(xs: Vec[T], i: Fin[len(xs)]): T
```

読み手は、unchecked な index をまず `Fin[n]` に正規化し、その後 total API を使うと考えればよい。range や subview のような事前検証しにくい操作だけが `Result[..., BoundsError]` を返す。

## Reference-level explanation

- public signature は portable semantics に正規化された shape を持つ。
- length は `Nat` で表す。
- valid index は `Fin[n]` で表す。
- out-of-bounds は `Result[..., BoundsError]` で表す。
- prevalidated index を受け取る API は `Fin[n]` を受ける。
- range や subview のように事前検証できない API は `Result[..., BoundsError]` を返す。
- mutation-capable subview でも portable contract は `Nat` / `Fin` / `Result` 規律を守る。
- raw `extern`、boundary-only capability、platform-layer numerics の非露出は `0003` と `0017` の規律に従う。
- implementation-local representation と public contract を分離する。

## Drawbacks

- API が一般的な index ベース設計より厳格に見える。
- `Fin[n]` の導入で学習コストが増える。
- low-level API や FFI binding では wrapper を厚く書く必要がある。

## Rationale and alternatives

- 代替案 1 は conventional integer index を public API に使う案だったが、境界条件が型から消えるため採らない。
- 代替案 2 は raw handle を package surface までは許す案だったが、portable reasoning を壊すため採らない。
- 代替案 3 は language ではなく library guideline に留める案だったが、Moti の中心不変条件に対して弱すぎる。
- 何もしない場合、public API は safe fragment と portable semantics の橋渡しになれない。

## Prior art

- dependently typed languages の length-indexed collection は直接の参考になる。
- Rust の `usize` index は practical だが、Moti が public portability を優先する点では異なる。
- lesson は「public contract に境界条件を埋め込むこと」、divergence は「Moti が boundary-only と platform-layer numerics を public shape へ出さないこと」である。

## Unresolved questions

- `Fin[n]` ergonomics をどこまで library で補うか。
- mutable subview や slice API の最終 surface。

## Future possibilities

- richer range proof API。
- collection-family 全体への一貫した canonical API 展開。
