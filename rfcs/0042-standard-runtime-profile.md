# 0042-standard-runtime-profile

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: standard runtime profile、must/should/may、scheduler、timer、blocking bridge、test runtime
- Feature Name: `standard_runtime_profile`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の standard runtime profile は language feature を成立させる最低 runtime 契約を `must` / `should` / `may` で定義する。`must` には structured task tree を実行する scheduler、cancellation 伝播、deadline timer、blocking bridge、runtime-backed `std` API を支える driver を含める。`should` と `may` は testability と observability を current profile として切り分ける。

## Motivation

- runtime profile を informal convention にすると、async と `std` の meaning が backend ごとに変わる。
- must / should / may を分けないと、conformance と ecosystem flexibility の両立が難しい。
- use case 1 は、runtime implementer と library author が同じ最低契約を参照したいケースである。
- use case 2 は、virtual time test runtime や deterministic scheduling hook を profile 上で位置付けたい maintainer である。
- no-op のままだと、standard runtime という語が marketing だけで終わる。

## Guide-level explanation

runtime profile は「Moti の language feature を動かすために runtime が最低限何を提供するか」を書く契約である。利用者は detail を全部読む必要はなく、`must` を満たす runtime なら `std` の基本機能が動くと理解すればよい。

```txt
must: scheduler, cancellation, deadline timer, blocking bridge
should: virtual time test runtime, deterministic scheduling hook
may: profiling hook, backend-specific reactor optimization
```

## Reference-level explanation

- standard runtime profile は `must`、`should`、`may` の三段階を持つ。
- `must` は structured task tree を実行する scheduler、cancellation context 伝播、deadline timer、blocking bridge、runtime-backed `std` API driver を含む。
- blocking bridge は、blocking operation を scheduler 進行と分離して実行し、call site へ typed result で結果を返せなければならない。
- runtime-backed `std` API driver は、少なくとも timer / I/O / wakeable operation の基礎能力を提供し、unchecked trap ではなく typed failure surface へ接続しなければならない。
- `should` は virtual time test runtime、deterministic scheduling hook、runtime capability diagnostics を含む。
- `may` は profiling hook、backend-specific reactor optimization、observability extension を含む。
- runtime profile は `core` surface の意味を変えてはならない。
- profile 差分は `std` と boundary module の API contract に反映しなければならない。
- conformance は `must` 準拠を最小基準とする。

## Drawbacks

- runtime 実装者への要求が明確に増える。
- `should` と `must` の線引きに議論が継続する。
- 言語仕様と runtime 契約の両文書を読む必要がある。

## Rationale and alternatives

- 代替案 1 は runtime profile を informal にする案だったが、language feature 成立条件が曖昧になるため採らない。
- 代替案 2 はすべて `must` に上げる案だったが、実装負担が大きすぎるため採らない。
- 代替案 3 は runtime ごとに独自契約を持たせる案だったが、standard profile の意味が失われるため採らない。
- 何もしない場合、`std` と async の portability は説明不能になる。

## Prior art

- Rust ecosystem の runtime 多様性は profile 契約の不在が library surface を難しくする例である。
- Go は言語と runtime が一体だが、その contrast が Moti に explicit profile の必要を教える。
- Moti は flexibility を残しつつ must / should / may で minimum contract を先に固定する。

## Unresolved questions

- virtual time test runtime を将来 `must` に引き上げる条件。
- driver を monolithic に書くか capability ごとに分解するか。

## Future possibilities

- 複数 standard profile の併存。
- build / package system と profile selection の接続。
