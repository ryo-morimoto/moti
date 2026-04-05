# 0007-borrowing-lifetimes-and-suspend-boundaries

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: place model、lifetime、reborrow、suspend boundary
- Feature Name: `borrowing_lifetimes_and_suspend_boundaries`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の borrow は flow-sensitive かつ place-sensitive であり、`await` は suspend boundary として特別に扱う。この RFC は borrow / suspend interaction の総論を固定する。canonical place と reborrow の詳細は `0030`、`await` 越し capability 禁止の詳細は `0031` で精密化する。

## Motivation

- Moti のメモリ安全は borrow が flow-sensitive であることに依存する。
- place model を曖昧にすると、diagnostics も async 制約も説明できない。
- `await` をまたいだ borrow や capability の保持は、安全性と scheduler reasoning を複雑化する。
- public API では lifetime の意味を読める必要がある一方、call-site まで lifetime syntax を押し付けたくはない。

## Guide-level explanation

Moti では borrow は「変数」ではなく「place」に対して起きる。したがって、同じ container の一部を触る操作でも、place が衝突するとみなされれば借用は拒否される。複雑な alias analysis を読み手に要求しないためである。

また、`await` は単なる関数呼び出しではなく suspend boundary である。そこをまたいで borrow や `BorrowMut` を保持してはならない。

## Reference-level explanation

- borrow 判定は canonical place に対して行う。
- field-sensitive だが、container-whole projection と index projection は保守的に扱う。
- reborrow は明示的な規則を持つ。
- two-phase borrow は導入しない。
- temporary lifetime extension に依存する規則は導入しない。
- public signature では lifetime 明示を許すが、call-site syntax は増やさない。
- lexical borrow、`Borrow`、`BorrowMut`、`LocalRoot`、`RuntimeToken` は `await` 越し保持不可とする。
- `OwnedRoot` は same-domain `await` のみ保守的に許すが、domain 越境は許さない。
- canonical place、競合判定、reborrow の正本は `0030` とする。
- suspend boundary と await-affine capability の正本は `0031` とする。

## Drawbacks

- 一部の安全そうに見える借用パターンも reject される。
- borrow 由来の冗長な一時束縛が増える可能性がある。
- async と borrow の相互作用を強く制限するため、表現力の圧力が継続する。

## Rationale and alternatives

- 代替案 1 は index/key 非衝突の積極推論だったが、predictability を損なうため採らない。
- 代替案 2 は two-phase borrow だったが、mental model と diagnostics を難しくするため採らない。
- 代替案 3 は `await` 越し borrow を広く許す案だったが、capability safety と domain safety を崩すため採らない。
- 何もしない場合、borrow は「賢いが読めない」制度になり、Moti の低認知負荷目標に反する。

## Prior art

- Rust の borrow checker は重要な参考だが、Moti は async と capability 境界をより厳しく整える。
- region-based / affine type system は place と lifetime を明示する設計の参考になる。

## Unresolved questions

- lifetime elision の詳細をどこまで RFC 本文で固定するか。
- `OwnedRoot` の same-domain 例外を profile 制約として維持するか。

## Future possibilities

- capability-aware diagnostics の強化。
- container-specific safe split API を library side で増やす余地。
