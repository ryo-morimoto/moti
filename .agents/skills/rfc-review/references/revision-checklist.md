# Revision Checklist

## Section-by-Section Checks

### Summary

1. decision が 1 文で言えるか
2. scope creep がないか
3. Motivation とズレていないか

### Motivation

1. 実在する pressure があるか
2. use case があるか
3. no-op の痛みがあるか

### Guide-level explanation

1. 読者がどう考えるべきか示しているか
2. 例があるか
3. 読みや保守の負担に触れているか

### Reference-level explanation

1. terminology は定義済みか
2. interaction が書かれているか
3. corner case があるか
4. guide の例を説明できるか

### Drawbacks

1. 実コストか
2. single-sided advocacy になっていないか

### Rationale and alternatives

1. alternatives が 2 つ以上あるか
2. 却下理由が property ベースか
3. library / policy / profile 代替を見たか

### Prior art

1. lesson があるか
2. divergence があるか
3. authority appeal になっていないか

### Unresolved questions

1. 欠陥の隠し場所になっていないか
2. future RFC に出すべき項目が分離されているか

### Future possibilities

1. 現行 RFC の未完成さを隠していないか
2. extension point と core design hole が区別されているか

## Cross-Section Failure Modes

1. `Summary` では A を決めると言っているのに `Reference` は B を規定している
2. `Motivation` の pressure が `Alternatives` に反映されていない
3. `Guide` の mental model と `Reference` の境界条件がズレている
4. `Drawbacks` にしか現れない重要概念がある
5. `Prior art` が `Rationale` に接続していない
6. `Unresolved questions` が多すぎて RFC の中心 decision がまだ分離できていない

## Moti-Specific Failure Modes

1. safe fragment を守る規則が書かれていない
2. FFI や boundary の trust ownership が曖昧
3. import-independent resolution を壊す余地がある
4. implicit prelude / implicit import を許してしまう
5. typed error surface を曖昧にする
6. runtime-backed 概念を `core` に混ぜている
7. implementation convenience を main rationale にしている
8. invariant / current profile / reserved extension point が区別されていない

## Severity Guide

### blocking

1. RFC の decision が 1 つに切れていない
2. Motivation が弱く、なぜ今やるのかが読めない
3. Reference が曖昧で規則として使えない
4. 重要な interaction が欠けている
5. 同じ判断を別 RFC が持っている

### major

1. Guide が mental model を作れていない
2. alternatives が弱い
3. drawbacks が薄い
4. prior art が lesson になっていない

### minor

1. 例が少ない
2. 用語順が悪い
3. future possibilities の切り方が甘い

## Repair Actions

### Expand

section の方向は正しいが情報が足りない。

### Rewrite

section の役割と内容がずれている。

### Split

1 RFC 1 decision を超えている。

### Merge

同一判断が複数 RFC に分散している。

### Move

書くべき内容が section をまたいで誤配置されている。

### Delete

内容が重複、または本 RFC の scope 外。
