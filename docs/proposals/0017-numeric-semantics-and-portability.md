# 0017-numeric-semantics-and-portability

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: 整数・浮動小数の意味論、platform-layer numerics、portable semantics
- Feature Name: `numeric_semantics_and_portability`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、数値意味論を portable semantics として保守的に固定する。固定幅整数は modular arithmetic を基本とし、整数除算は checked に扱い、浮動小数は IEEE ベースの意味論を持つ。`ISize` / `USize` は platform-layer numerics と位置づけ、portable public API の基礎型とはしない。

## Motivation

- Moti の要件にはクロスプラットフォーム実行、予測可能な挙動、小さな成果物が含まれる。
- 数値意味論が target 依存の暗黙知に落ちると、portable API と conformance が崩れる。
- `ISize` / `USize` をどこでも便利に使える型として扱うと、platform leak が public surface へ出る。
- float semantics は optimizer や target quirks と衝突しやすいため、保守的な基準が必要である。
- use case 1 は、同じ numeric code が target ごとに静かに意味を変えないことを望む application author である。
- use case 2 は、portable public API に platform word size を漏らしたくない library author である。

## Guide-level explanation

Moti の数値は「その target だとこうなることが多い」ではなく、「portable に何を期待してよいか」を先に決める。整数と浮動小数の convenience よりも、挙動の予測可能性を優先する。

読み手は、`Int32` や `UInt64` のような固定幅を基本にし、`ISize` / `USize` は platform layer の都合を表す特殊な型だと考えればよい。

## Reference-level explanation

- 固定幅整数は modular arithmetic を基本とする。
- 固定幅整数の overflow 振る舞いは恒久不変とする。
- 整数除算は checked で扱う。
- 浮動小数は IEEE ベースの意味論を採るが、portable semantics を文書化する。
- 浮動小数の current profile は次とする:
  - 算術演算は IEEE value semantics を基準にする
  - `NaN` の payload と符号は portable contract に含めない
  - `NaN` は比較で unordered とする
  - `-0.0` と `+0.0` の区別は IEEE semantics に従って観測可能とする
  - rounding は IEEE の round-to-nearest-even を基準にする
  - `to_bits` / `from_bits` は bit-level representation を明示的に観測・構成する escape hatch とする
- quiet/signaling NaN の詳細、payload 伝播、target-specific caveat は現行プロファイルとして文書化し、public portable API の契約からは外す。
- `ISize` / `USize` は platform-layer numerics とし、portable surface の基底型とはしない。
- target-specific caveat は profile と docs に明示的に落とす。

## Drawbacks

- 一部 target 特化最適化を標準意味論に取り込みにくい。
- platform-native style に慣れた利用者には厳格に見える。
- float semantics の完全 formalization は長くなりやすい。

## Rationale and alternatives

- 代替案 1 は target-native semantics 優先だったが、portable reasoning を壊すため採らない。
- 代替案 2 は `ISize` / `USize` を一般 public API へ広く許す案だったが、platform leak を招くため採らない。
- 代替案 3 は float semantics を曖昧に保つ案だったが、conformance と predictability が弱くなるため採らない。
- 何もしない場合、portable API discipline が型設計だけでなく数値意味論の段階で崩れる。

## Prior art

- Rust の fixed-width integer と float semantics 議論は直接の比較対象である。
- C 系の `size_t` / machine word dependence は Moti が public portable surface で避けたい例になる。
- lesson は「platform-native convenience より portable contract を先に固定すること」、divergence は「Moti が public surface で `ISize` / `USize` を抑制すること」である。

## Unresolved questions

- quiet/signaling NaN の wording をどこまで厳密にするか。
- checked integer operations の surface naming。

## Future possibilities

- fixed-point / decimal numerics の追加。
- fast-math 的 opt-in profile の検討。
