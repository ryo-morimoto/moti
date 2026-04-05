# 0026-inductive-and-coinductive-formation

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: strict positivity、inductive family、codata、projection-only、guarded corecursion
- Feature Name: `inductive_and_coinductive_formation`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は inductive family の定義に strict positivity を要求し、coinductive data は projection-only で観測する。codata への pattern match は許さず、coinductive constructor 的操作は guarded corecursion の下でのみ導入する。これにより、trusted kernel を膨らませずに inductive / coinductive structure を扱う。

## Motivation

- dependent typing を持つ以上、data formation の規律が弱いと kernel safety が最初に崩れる。
- codata を自由に match させると productivity と観測規則が曖昧になる。
- use case 1 は、length-indexed collection や証明対象の inductive family を安全に定義したい user である。
- use case 2 は、stream のような coinductive object を観測はしたいが kernel escape hatch にはしたくない designer である。
- no-op のままだと、positivity と codata discipline が実装依存になり、proof safety が壊れる。

## Guide-level explanation

Moti では普通の data は constructor と match で扱うが、codata は field projection でしか観測しない。stream から head と tail を取ることはできるが、enum のように分解することはしない。

```moti
codata Stream[T] {
  head: T
  tail: Stream[T]
}
```

この `Stream` は `match` で壊すのではなく、`.head` と `.tail` で読む。読み手は「inductive は case analysis、coinductive は observation」と分けて考える。

```moti
corec fn repeat(x: T): Stream[T] {
  Stream { head = x, tail = repeat(x) }
}
```

この形は accept できる。各再帰呼び出しが `Stream` producer の field 構成の下に現れているからである。

```moti
corec fn bad_tail(s: Stream[T]): Stream[T] {
  bad_tail(s)
}
```

この形は reject する。再帰呼び出しが guard の下に入っていないからである。guarded corecursion では、coinductive producer の正規形が projection で観測可能な field を構成した後にだけ再帰を置ける。

## Reference-level explanation

- 分類: strict positivity と projection-only codata は恒久不変である。
- inductive declaration は strict positivity を満たさなければならない。
- positivity check は parameter / index の出現位置を検査し、negative position の自己参照を禁止する。
- inductive family は usual constructor introduction と pattern-matching elimination を持つ。
- coinductive declaration は projection signature を持つ。
- codata の elimination は projection-only であり、pattern match や absurd elimination の対象にしてはならない。
- codata value の生成は guarded corecursion に限る。
- guardedness check は recursive call が coinductive producer の field 構成の下にあることを要求する。
- guard として数える構文は、当該 codata producer の field 構成だけである。
- plain `let`、既存 codata への projection、helper 呼び出しだけでは guard を作らない。
- projection は codata の観測であって guard ではない。既存 codata を projection して再帰呼び出しへ渡すだけの式は guard 条件を満たさない。
- accepted / rejected の判定は、producer 正規形、guard として数える構文、recursive call の位置関係で決まる。
- inductive / coinductive formation の可否は kernel judgment の対象であり、FFI evidence や optimizer rewrite に依存してはならない。
- effectful observation や boundary-backed codata facade は library layer で包み、kernel rule と混ぜない。

## Drawbacks

- codata ergonomics は ML 風の ADT より保守的になる。
- positivity / guardedness check の説明コストが高い。
- 一部の convenient encoding は reject される。

## Rationale and alternatives

- 代替案 1 は codata matching を許す案だったが、kernel 観測規則が肥大化するため採らない。
- 代替案 2 は positivity check を緩める案だったが、proof safety を壊すため採らない。
- 代替案 3 は codata を library encoding に委ねる案だったが、language-level productivity story が欠けるため採らない。
- 何もしない場合、coinduction を導入するたびに kernel 境界が揺れる。

## Prior art

- Coq、Agda、Lean からは positivity / guardedness を kernel rule として小さく保つ lesson を取る。
- Idris2 からは coinductive structure を surface に出すときの ergonomic lesson を学ぶ。
- Haskell の lazy stream は表現力を示すが、Moti は productivity を implicit laziness ではなく explicit rule で固定する。

## Unresolved questions

- guardedness check で許す syntactic sugar の範囲。
- coinductive record の surface syntax を record 系とどこまで共通化するか。

## Future possibilities

- sized type と guarded recursion を組み合わせる拡張。
- codata projection を使う teaching-oriented diagnostics の強化。
