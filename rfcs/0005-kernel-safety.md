# 0005-kernel-safety

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: strict positivity、codata、guarded corecursion、definitional equality、bypass 不可
- Feature Name: `kernel_safety`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は trusted kernel を小さく保つため、strict positivity、projection-only codata、guarded corecursion、小さい definitional equality を採用する。この RFC は kernel safety の総論と境界だけを固定し、inductive / coinductive formation の詳細は `0026`、definitional equality と kernel conversion の詳細は `0027` で精密化する。

## Motivation

- Moti の安全性主張は kernel に escape hatch がないことを前提にしている。
- 依存型を導入しても、positivity や productivity の境界が崩れると証明安全性は失われる。
- codata や FFI を便利に扱うために equality を膨らませると、どこまでが trusted か分からなくなる。
- verified extern を入れる場合でも、kernel は拡張せず、外部 checker へ責務を押し出す必要がある。

## Guide-level explanation

この RFC は「Moti で安全だと言ってよい最小核はどこまでか」を固定する。読み手は、次のものが kernel に入らないと理解すればよい。

- raw FFI contract
- runtime の実装知識
- convenience のための特殊 rewrite
- codata を `match` で分解する操作

kernel が小さく閉じているからこそ、外側の制度を強くしても安全境界が読める。

## Reference-level explanation

- strict positivity を要求する。
- codata は projection-only とし、pattern match 対象にしない。
- corecursion は guarded 条件を要求する。
- definitional equality は小さく保ち、外部実装事情を入れない。
- `extern fn` と `verified extern fn` は definitional equality に参加しない。
- runtime shortcut や optimizer rewrite を kernel law として扱わない。
- bypass 不可を原則とし、kernel 安全条件は implementation convenience で緩めない。
- positivity、codata、guarded corecursion の詳細規則は `0026` を正本とする。
- definitional equality と kernel conversion の詳細規則は `0027` を正本とする。

## Drawbacks

- 表現力が保守的になる。
- 一部の便利な equational reasoning や codata ergonomics を諦める必要がある。
- 実装者にとっては elaboration と kernel の境界維持が厳しい。

## Rationale and alternatives

- 代替案 1 は definitional equality を広くして実装を楽にする案だったが、trusted kernel を肥大化させるため採らない。
- 代替案 2 は codata matching を許す案だったが、productivity と観測規律が曖昧になるため採らない。
- 代替案 3 は verified extern の証跡を kernel へ埋め込む案だったが、boundary ownership の原則に反する。
- 何もしない場合、後続 RFC が kernel を都合よく拡張し始め、全体の安全性主張が壊れる。

## Prior art

- Lean や Coq 系の positivity / guardedness discipline は重要な参考になる。
- Rust の unsafe boundary は kernel ではなく言語周辺制度に置かれる。Moti も FFI は kernel 外へ保つ。
- total languages の経験は、少ない trusted kernel を守る重要性を示している。

## Unresolved questions

- guarded corecursion の check を surface RFC とどこまで結ぶか。
- equality の mechanized presentation をどこまで formal に書くか。

## Future possibilities

- kernel を変えない範囲での equality hint。
- codata ergonomics の表面 sugar。
