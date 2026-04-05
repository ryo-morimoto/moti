# 0034-effect-abstraction-and-limited-polymorphism

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: effect abstraction、named effect alias、limited polymorphism、public surface 制約
- Feature Name: `effect_abstraction_and_limited_polymorphism`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は `0032` が固定する closed-row surface を前提に、effect abstraction を named closed-row alias と module-private helper で行う。v1 では user-facing な open-row polymorphism を導入せず、標準ライブラリが必要とする限定的 polymorphism がある場合も、それは surface syntax ではなく elaboration と predefined combinator に閉じ込める。

## Motivation

- library author は effect を共通化したいが、open row variable を公開面へ出すと読む負荷が急増する。
- effect system を導入した目的は solver の自由度ではなく visible control flow の固定である。
- use case 1 は、複数関数で同じ effect set を共有したい package author である。
- use case 2 は、標準 combinator では polymorphic behavior が欲しいが、一般利用者には closed row を見せたい language designer である。
- no-op のままだと、effect abstraction の必要が every RFC で ad-hoc に議論される。

## Guide-level explanation

Moti の v1 では、利用者が `e` のような row variable を public signature に書くことはしない。共通 effect は alias でまとめ、必要なら combinator 実装側でだけ一般化を使う。

```moti
effectalias FileIO[E] = {IO, Exn[E]}

pub fn read(path: Path): Bytes / FileIO[ReadError]
```

読み手は alias 展開後も closed row を得られる。これは open-ended solver への入口ではなく、surface readability を保つための抽象化である。

## Reference-level explanation

- public signature は `0032` の規則に従い、closed effect row または closed effect alias のみを許す。
- effect alias は closed row を名前付けする mechanism であり、open row parameter を露出してはならない。
- 一般 user code での effect polymorphism は v1 では導入しない。
- 標準ライブラリや compiler intrinsic が限定的 polymorphism を必要とする場合、その surface contract は closed row へ正規化されなければならない。
- trait solver、type inference、export closure は effect polymorphism を前提に拡張してはならない。
- alias expansion 後の row は row 正規形規則に従う。
- diagnostics は alias 展開前後の両方を必要に応じて示せるが、contract basis は展開後の closed row に置く。

## Drawbacks

- advanced effect abstraction を望む利用者には制約が強い。
- alias だけでは HKT 的な再利用を表しにくい。
- 標準ライブラリ側に限定的な special case を置く設計に見えやすい。

## Rationale and alternatives

- 代替案 1 は full row polymorphism を user-facing に導入する案だったが、surface readability と diagnostics を悪化させるため採らない。
- 代替案 2 は effect abstraction 自体を禁じる案だったが、library ergonomics を不必要に損なうため採らない。
- 代替案 3 は macro や documentation convention に任せる案だったが、contract 面が曖昧になるため採らない。
- 何もしない場合、effect alias と hidden polymorphism の議論が別 RFC に散る。

## Prior art

- Koka や row-polymorphic effect systems は表現力の方向性を示す。
- OCaml effect handler 周辺の経験は、surface に何を出しすぎないかの lesson を与える。
- Moti は v1 で closed-row readability を優先し、open-row surface を意図的に遅らせる。

## Unresolved questions

- effect alias declaration の最終 keyword。
- 標準 combinator の限定的 polymorphism をどこまで spec に書くか。

## Future possibilities

- closed-row readability を壊さない higher-order effect abstraction。
- module signature 上での effect alias export rules の追加明文化。
