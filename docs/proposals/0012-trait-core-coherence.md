# 0012-trait-core-coherence

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: trait の役割、coherence、orphan、overlap、不採用拡張
- Feature Name: `trait_core_coherence`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の trait core は、ad-hoc polymorphism を import-independent かつ coherence-first に解決するための制度である。trait は implicit value injection や overloaded syntax の入口ではなく、有限で決定的な制約解決のために使う。overlap、specialization、implementation-local defaulting は採らない。

## Motivation

- Moti では trait は単なる便利機能ではなく、解決規律の中心である。
- import によって impl 候補が変わると、名前解決と型推論の mental model が壊れる。
- overlap や specialization は表現力を増やすが、local reasoning を著しく弱くする。
- future extension を考えても、まず小さい core を守らないと solver の formalization が不可能になる。

## Guide-level explanation

Moti の trait は「何でも overloading するための制度」ではない。読み手は、trait を見たときに「この制約は package graph と宣言だけで一意に解決される」と期待できなければならない。

```moti
trait Eq[T] {
  fn eq(x: T, y: T): Bool
}
```

重要なのは、どの impl が使われるかを import の偶然で変えないことだ。Moti では trait core 自体を小さく保ち、solver が読み手の予想を裏切らないようにする。

## Reference-level explanation

- trait core は static constraint solving 制度である。
- impl 候補は import によって増減しない。
- orphan rule と overlap 禁止を採る。
- specialization は採らない。
- overloaded syntax は trait core へ直接入れない。
- implicit dictionary passing を user-facing mechanism として露出しない。
- associated type は trait core に含むが、solver formalization は別 RFC で精密化する。

## Drawbacks

- Rust や Haskell に慣れた利用者には表現力が保守的に見える。
- 一部の ergonomic sugar を採りにくい。
- trait の使いどころを意図的に狭めるため、library design 側で工夫が要る。

## Rationale and alternatives

- 代替案 1 は typeclass 的柔軟性を広く許す案だったが、import-dependent resolution を避けられないため採らない。
- 代替案 2 は specialization を入れる案だったが、coherence-first を崩すため採らない。
- 代替案 3 は operator overloading を trait に乗せる案だったが、canonical surface と衝突するため採らない。
- 何もしない場合、trait は便利だが予測不能な制度となり、Moti の設計目標と逆行する。

## Prior art

- Rust の coherence と orphan rule は強い参考になる。
- Haskell の typeclass は expressive だが、Moti が避けたい import / solver entanglement を持つ部分がある。
- Scala 系の implicits / givens は Moti が明示的に採らない方向の教材になる。

## Unresolved questions

- trait core に残す associated item の最小集合。
- derive と trait core の接続の normative 範囲。

## Future possibilities

- coherence-first を崩さない closed structural traits の拡張。
- solver を変えずに追加できる trait-side convenience surface。
