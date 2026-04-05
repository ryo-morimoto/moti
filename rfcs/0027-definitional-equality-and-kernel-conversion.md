# 0027-definitional-equality-and-kernel-conversion

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: definitional equality、conversion、透明 unfolding、kernel に入る rewrite 範囲
- Feature Name: `definitional_equality_and_kernel_conversion`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の definitional equality は小さく閉じた kernel conversion として定義する。beta、必要最小限の iota、transparent local definition unfolding、universe cumulativity だけを含み、`extern fn`、`verified extern`、runtime shortcut、implementation-specific rewrite は含めない。conversion は typing judgment の補助であり、境界外の知識を取り込む入口にしない。

## Motivation

- dependent typing では conversion が広すぎると trusted kernel が肥大化する。
- FFI や runtime の知識を equality に入れると、safe fragment の境界が読めなくなる。
- use case 1 は、型検査でどの同値が自動で使われるかを予測したい user である。
- use case 2 は、kernel を小さく保ったまま elaboration と optimization を分離したい implementer である。
- no-op のままだと、convenience rewrite が後から kernel law に混入し、proof obligation の所在が崩れる。

## Guide-level explanation

Moti では「自明に同じ」と扱う範囲を小さく固定する。beta reduction や `match` による明らかな分解は自動で使われるが、runtime がたまたま同じ結果を返すからといって型検査上も同じにはならない。

```moti
(fn (x: T) => x)(y)
```

この式は conversion で `y` と同じとみなせる。一方、`verified extern fn` の checker が安全だと返したからといって、その呼び出しを kernel equality に入れることはない。

## Reference-level explanation

- 分類: small kernel conversion と extern / runtime rewrite の不参加は恒久不変である。
- definitional equality は symmetric / transitive / congruent な conversion relation として定義する。
- conversion に入るのは alpha-renaming、beta reduction、inductive data に対する iota reduction、local binding に対する zeta reduction、transparent かつ total で effect-free な定義に対する delta unfolding、universe cumulativity だけである。
- transparent unfolding は language law で透明と定義された local binding / named definition に限る。
- unfolding へ参加できるのは total で effect-free な定義だけである。
- opaque boundary、`extern fn`、`verified extern fn`、runtime primitive、optimizer-introduced rewrite は conversion に入らない。
- `Diverge` を持つ定義、partial 定義、effectful 定義は transparent でも conversion に入らない。
- primitive projection shortcut、checker verdict、boundary metadata、backend 固有等式は conversion に入らない。
- proof erasure は conversion の前提ではなく elaboration / runtime phase の責務である。
- conversion は typing judgment の checking phase でのみ利用し、trait candidate discovery や boundary verification の代替にしてはならない。
- normalization procedure が必要な場合でも、それは上記の小さい relation に対してのみ定義する。
- 実装は追加の simplification を内部で使ってよいが、失敗を成功へ変えるような extra equality を仕様化してはならない。

## Drawbacks

- 便利に見える equational reasoning の一部は手動 lemma や explicit proof を要求する。
- implementation 上は fast path と kernel conversion の二重管理が必要になる。
- 利用者には「runtime で同じ」と「kernel で同じ」の差を教える必要がある。

## Rationale and alternatives

- 代替案 1 は runtime knowledge を equality に広く入れる案だったが、trusted boundary を崩すため採らない。
- 代替案 2 は conversion を最小 beta のみに絞る案だったが、依存パターンや basic ergonomics を損なうため採らない。
- 代替案 3 は equality hint を user-extensible にする案だったが、proof obligation の所在が曖昧になるため採らない。
- 何もしない場合、kernel conversion は convenience のたびに拡張され、review 不能な中心制度になる。

## Prior art

- Coq、Lean、Agda からは conversion set を小さく閉じる lesson を取る。
- Rust の trait / const evaluation 周辺は convenience と soundness の緊張関係を示す contrast になる。
- Moti は proof-carrying boundary や runtime knowledge を equality に混ぜず、trust boundary を kernel 外へ保つ点で意図的に分岐する。

## Unresolved questions

- transparent definition の precise admission criteria を totality check とどう接続するか。
- normalization by evaluation を許す場合の conformance wording。

## Future possibilities

- kernel を変えない equality hint 用の補助証明機構。
- conversion trace を示す diagnostics support。
