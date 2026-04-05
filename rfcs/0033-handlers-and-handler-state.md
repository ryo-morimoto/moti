# 0033-handlers-and-handler-state

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: deep-only handler、one-shot resumption、handler state、resume 規律
- Feature Name: `handlers_and_handler_state`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の handler は deep-only かつ one-shot である。handler は operation を捕まえて局所的に意味付けでき、必要なら専用構文の handler state を持つ。shallow handler、multi-shot resume、一般局所可変の代替としての自由な mutable cell は導入しない。

## Motivation

- effect row だけでは stateful / control-flowful program の実行規律が閉じない。
- 一般局所可変を復活させずに状態を扱うには、handler state の owner を明示する必要がある。
- use case 1 は、局所 state machine を effect handler で包みたい library author である。
- use case 2 は、resume 回数や control transfer を保守的に読みたい user である。
- no-op のままだと、effect system は存在しても operational story が library 慣習に流れる。

## Guide-level explanation

handler は「effect を別の意味に置き換える局所境界」である。Moti ではまず deep-only / one-shot に限るので、継続は一度だけ再開され、state は handler ごとに明示的に持つ。

```moti
handle counter() with {
  state = 0
  on Count() k => resume k(state + 1) with state + 1
}
```

読み手は、この handler が `Count` operation を捕まえ、state を 1 つ持ち、継続を一度だけ再開すると理解すればよい。一般 `var` の隠れた別名ではない。

## Reference-level explanation

- handler は deep-only とする。
- resumption は one-shot であり、同一 continuation を複数回 `resume` してはならない。
- handler clause は handled operation、bound continuation、optional handler state を持つ。
- handler state は handler owner に閉じた mutable slot であり、一般 local mutable binding ではない。
- unhandled effect は外側へ伝播する。
- current profile で user-level handler が扱える built-in effect は `State[S]`、`Exn[E]`、`Async` に限る。
- `IO`、`Diverge`、`Parallel` は latent tracked effect であり、通常の user-level handler はこれらを除去しない。
- `Exn` を handler で回収して `Result` へ正規化する規律は error-boundary RFC に従う。
- `Async` を handler で扱う場合でも、suspend boundary や domain affinity を破ってはならない。
- one-shot violation は compile error とする。dynamic check を言語意味の本体には置かない。
- diagnostics は double-resume と unhandled effect を区別できなければならない。

## Drawbacks

- shallow handler や multi-shot continuation の表現力を失う。
- stateful handler の理解コストがある。
- 実装は continuation representation を制限付きで設計する必要がある。

## Rationale and alternatives

- 代替案 1 は shallow handler を同時導入する案だったが、mental model が二重化するため採らない。
- 代替案 2 は multi-shot resume を許す案だったが、cost model と safety reasoning が重くなるため採らない。
- 代替案 3 は local `var` を復活させる案だったが、canonical surface と effect-first 方針に反するため採らない。
- 何もしない場合、state discipline は effect system の外で局所例外化する。

## Prior art

- algebraic effects and handlers の研究は直接の理論背景である。
- Koka は effect-first mental model と practical lesson を与える。
- Moti は deep-only / one-shot を current profile として早く固定し、表現力を intentionally 絞る。

## Unresolved questions

- handler state の surface syntax を record update 系とどこまで共有するか。

## Future possibilities

- restricted shallow forwarding。
- handler-local resource protocol を表す library pattern。
