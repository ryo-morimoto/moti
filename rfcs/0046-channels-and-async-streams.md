# 0046-channels-and-async-streams

- 状態: `Draft`
- 日付: 2026-04-06
- 対象範囲: channels、async streams、bounded back-pressure、task-context ownership、typed shutdown
- Feature Name: `channels_and_async_streams`
- Start Date: 2026-04-06
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は channels と async streams を `std` の structured abstraction として提供し、core language primitive にはしない。これらは task tree と task context の上に構成され、cancellation、deadline、end-of-stream を typed result として表面へ出し、producer-side `close` / `finish` 自体は owner operation として扱う。v1 の canonical default は bounded back-pressure を持つ single-producer / single-consumer 形であり、detached producer や unbounded growth を既定にしない。

## Motivation

- `0009` と `0035` は structured concurrency を固定したが、その上位 abstraction である producer / consumer surface は未分離である。
- `0042` は runtime profile に scheduler、timer、blocking bridge を要求するが、channels / streams がどの層で何を要求するかはまだ固定していない。
- use case 1 は、pipeline を detached task なしで表したい concurrent API author である。
- use case 2 は、consumer が遅いときの pressure と cancellation を channel / stream surface で読めるようにしたい user である。
- no-op のままだと、ecosystem は unbounded queue、detached callback、hidden background task をばらばらに持ち込み、Moti の structured async story が崩れる。

## Guide-level explanation

Moti の channel / stream は「構造化 task の中で使う、pressure を隠さないデータ運搬路」である。producer と consumer は task tree の責務から外れず、slow consumer は bounded capacity を通じて producer 側へ戻ってくる。

```moti
with_task_group {
  let ch = std/channel.bounded[Int](64)
  let produce_task = async produce(ch.sender())
  let consume_task = async consume(ch.receiver())

  await produce_task
  await consume_task
}
```

読み手は、`bounded(64)` を見た時点で back-pressure があると分かり、`with_task_group` を見た時点で producer / consumer が detached でないと分かる。それで十分である。

```moti
match await ch.receiver().next() {
  Ok(Some(value)) => use(value)
  Ok(None) => finish()
  Err(ChannelCancelled) => recover()
}
```

この `Ok(None)` は normal shutdown であり、cancel や deadline は error 側で読む。producer-side `close` は owner operation であり、consumer が観測するのはその結果としての `Ok(None)` である。読み手は「producer が正常終了した」のか「task context が打ち切られた」のかを区別できる。

```moti
with_task_group {
  let ch = std/channel.bounded[Int](64)
  let child = async consume(ch.receiver())

  await child
}
```

receiver を child task へ move した後の `recv` / `next` は、作成時の context を凍結して使うのではなく、child task の current context を使う。読み手は handle owner と task context を分けて考えればよい。

```moti
with_task_group(group) {
  let s = std/stream.from_task_group(group, source)
  await consume_stream(s)
}
```

task-owning builder が許される場合でも、その owner は `group` のように surface で読めなければならない。hidden helper task を内部で生やす builder は v1 に入れない。

## Reference-level explanation

- 分類: channels / async streams を `std` に置き、core language primitive にしないことは恒久不変である。
- 分類: task tree と task context に従属させることは恒久不変である。
- 分類: bounded back-pressure を v1 の canonical default にすることは現行プロファイルである。
- 分類: v1 の endpoint cardinality を single-producer / single-consumer に制限することは現行プロファイルである。
- 分類: v1 の endpoint / handle を same-domain owner に制限し、`Parallel` 越し move を許さないことは現行プロファイルである。
- 分類: bounded channel の full 状態を immediate status として露出せず、capacity が空くまで suspend で表すことは現行プロファイルである。
- 分類: unbounded channel、broadcast、actor mailbox、sync primitive 全般は reserved extension point である。
- 分類: cross-domain channel wrapper と stream fan-out は reserved extension point である。
- passive な channel object の生成は structured scope 内で行う。これは task を生成しない state object の導入である。
- producer task を内部に持つ stream builder を導入する場合、その builder は task group などの scope owner を surface 引数で明示しなければならない。hidden helper task を内部で起動する builder は認めない。
- v1 の channel は一つの sender owner と一つの receiver owner を持つ。endpoint clone と multi-producer / multi-consumer coordination は v1 の規則に含めない。
- v1 の async stream は単一 consumer owner を持ち、複数 consumer への fan-out は別 abstraction へ分離する。
- v1 の `Sender`、`Receiver`、stream handle は `Send` / `Share` を得ず、same-domain child task への move だけを許す。`Parallel` 越しの cross-domain move は後続 RFC が domain-safe wrapper を定めるまで認めない。
- channel endpoint と stream handle は scope owner を持つが、cancellation / deadline context 自体を作成時に凍結して保持しない。
- `send` / `recv` / `next` / `close` の各 operation は、その operation を実行する current task の context を観測する。handle を child task へ move した後は、child 側の current context が使われる。
- child task が親より短い deadline を持つ場合、moved endpoint 上の operation はその短い deadline に従う。parent 側 deadline を暗黙に復元してはならない。
- send-like operation の canonical return shape は `Result[Unit, SendError]` とし、`closed`、`cancelled`、`deadline_exceeded` を error 側で表す。
- recv / next-like operation の canonical return shape は `Result[Option[T], ReceiveError]` とし、`Ok(Some(value))` は値、`Ok(None)` は normal end-of-stream、`cancelled` と `deadline_exceeded` は error 側で表す。
- send / recv / next の canonical signature family は `/ {Async}` を持つ。suspension 可能性は effect row に、close / cancel / deadline の観測結果は `Result` 側に現れる。
- producer-side の `close` / `finish` は current profile では non-suspending owner operation とし、effect row を持たない。double-close は no-op として扱ってよく、typed failure surface の主体にしない。
- producer-side の explicit close / finish は cancel と別の正常終了 signal であり、consumer 側では `Ok(None)` に対応しなければならない。v1 では唯一の producer owner だけが EOS を確定できる。
- producer owner の explicit close / finish、または producer owner の正常終了後の drop により channel / stream は end-of-stream へ遷移してよい。cancelled task や deadline timeout は EOS を発生させず、error 側で観測する。
- bounded channel の `send` は capacity が空くまで suspend する。v1 では full 状態を separate immediate status として露出しない。
- receiver owner が drop した時点で、pending および future の `send` は `SendError.closed` で wake / 完了しなければならない。
- producer owner が `close` / `finish` した時点、または正常終了後に drop した時点で、pending および future の `recv` / `next` は `Ok(None)` で wake / 完了しなければならない。
- cancel、deadline、receiver drop、producer close は、待機中 operation を対応する result で wake しなければならない。runtime は wakeable operation を hidden trap に落としてはならない。
- reserved extension として domain-safe channel wrapper や cross-domain stream abstraction を導入する場合、それらは `0037` の `Send` / `Share` 規律に従わなければならない。
- channel / stream operation は suspend point を含みうるため、capture と borrow は `0031` の suspend-boundary 規律に従う。
- channel / stream 実装は detached helper task を hidden に起動してはならない。必要な task は user が読む scope owner を持たなければならない。

## Drawbacks

- Go 風の「とりあえず channel を投げる」体験より保守的で、表面がやや重く見える。
- bounded default により、buffer sizing を考える場面が増える。
- stream と channel の close / end-of-stream / cancel を typed に分けるため、API 設計が慎重になる。

## Rationale and alternatives

- 代替案 1 は channels を core primitive にする案だったが、task tree・runtime profile・typed failure との境界が language core に混ざるため採らない。
- 代替案 2 は unbounded queue を既定にする案だったが、back-pressure を hidden にし、予測可能なメモリ挙動へ反するため採らない。
- 代替案 3 は async streams を detached producer task の sugar とみなす案だったが、structured concurrency の責任帰属を壊すため採らない。
- 代替案 4 は end-of-stream を error 側へ入れる案だったが、normal shutdown と cancellation を同じ failure surface へ混ぜるため採らない。
- 代替案 5 は `close` を `cancel` と統合する案だったが、producer の正常終了と task context の打ち切りを区別できなくなるため採らない。
- 代替案 6 は full 状態を必ず immediate status で返す案だったが、bounded back-pressure の canonical mental model を弱めるため v1 では採らない。
- 何もしない場合、pipeline abstraction は ecosystem ごとの慣習に流れ、cancel / deadline / memory pressure の mental model が統一されない。

## Prior art

- C# の `IAsyncEnumerable<T>` と `Channel<T>` は pull / push の二面性と cancellation 接続の lesson を与える。
- Go の channel は構成力を示すが、Moti は detached goroutine 的運用を許さない点で意図的に分岐する。
- Kotlin Flow や Trio nursery は structured scope と back-pressure を abstraction 側へ持ち込む重要性を示す。

## Unresolved questions

- `recv` / `next` / `close` / `finish` の最終 naming をどう揃えるか。

## Future possibilities

- broadcast、watch、actor mailbox を別 RFC として分離する。
- fan-out / fan-in と multi-producer / multi-consumer coordination を別 RFC として分離する。
- sync primitive を channel / stream と分けて扱う後続 RFC。
- distributed runtime profile 上の channel / stream 契約。
