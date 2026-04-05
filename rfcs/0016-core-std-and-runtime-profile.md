# 0016-core-std-and-runtime-profile

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `core` / `std` の責務、standard runtime profile、test runtime
- Feature Name: `core_std_and_runtime_profile`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、標準提供物の ownership を `core` と `std` に分け、その境界を standard runtime profile で支える。`core` は pure、runtime-independent、portable な API のみを持ち、`std` は runtime-backed API を担う。現行プロファイルでは standard runtime profile が language feature を成立させる最低契約を must / should / may で表す。

## Motivation

- Moti では runtime-backed feature と pure core を混ぜると portability が崩れる。
- implicit prelude を持たない以上、`core` / `std` の役割分担は明示的に決める必要がある。
- async、timer、blocking bridge、test runtime などは言語機能を支えるが、どこまでが必須 runtime かは曖昧になりやすい。
- library surface の mental model を安定させるため、runtime contract と optional library を分けたい。
- use case 1 は、runtime が変わっても pure API を同じ意味で読みたい library user である。
- use case 2 は、language feature を成立させる最低 runtime を固定したい runtime implementer である。

## Guide-level explanation

Moti では「言語にある」と「標準で使える」と「ある runtime profile が提供する」は同じ意味ではない。`core` は pure でどこでも同じように使える。`std` は runtime を前提にした層であり、そこには scheduler、timer、I/O bridge などが属する。

読み手は、まず `core` が純粋で portable かを見る。runtime が必要なら `std` と runtime profile の契約を見る。それで十分である。

この RFC の中心決定は、「標準提供物の owner を pure core と runtime-backed std に分け、そのあいだを runtime profile の契約で橋渡しする」ことである。`must / should / may` はその契約を現行プロファイルとして表す道具である。

## Reference-level explanation

- 分類: `core` の pure / runtime-independent / portable は恒久不変である。
- 分類: `std` が runtime-backed API を担うことは恒久不変である。
- 分類: standard runtime profile の must / should / may 境界は現行プロファイルである。
- `core` は pure、runtime-independent、portable の三条件を満たす。
- `std` は runtime-backed API を担う。
- implicit prelude を導入しない。
- standard runtime profile は must / should / may の境界を持つ。
- must:
  - structured task tree を実行する scheduler
  - cancellation context の伝播
  - deadline 伝播と timer
  - `std` の同期 I/O を支える blocking bridge
  - `std` が公開する runtime-backed API を成立させる driver
- should:
  - virtual time test runtime
  - deterministic scheduling hook
  - runtime capability の診断補助
- may:
  - profiling hook
  - backend-specific reactor 最適化
  - runtime-specific observability 拡張
- optional library は runtime contract と分ける。

## Drawbacks

- runtime profile の定義が保守的になりやすい。
- ecosystem 側には additional runtime abstraction の圧力が生じる。
- `core` に何を入れないかの判断がしばしば議論を生む。

## Rationale and alternatives

- 代替案 1 は `core/std` を分けない案だったが、portable semantics と runtime dependency が混ざるため採らない。
- 代替案 2 は runtime profile を informal convention にする案だったが、language feature の成立条件が曖昧になるため採らない。
- 代替案 3 は広い prelude を使って差を隠す案だったが、Moti の explicitness と衝突する。
- 何もしない場合、runtime-dependent feature が純粋ライブラリの顔をしてしまう。

## Prior art

- Rust の `core` / `std` 分離は参考になるが、Moti は runtime contract をより前面に出す。
- Go や Deno は runtime の前提が強い。Moti はそれを explicit profile として切り出す。

## Unresolved questions

- virtual time test runtime を `should` から `must` へ上げる条件。
- `driver` を monolithic contract で書くか、network / fs / timer に分解するか。

## Future possibilities

- 複数 runtime profile の並立。
- profile selection を build system へ接続する後続 RFC。
