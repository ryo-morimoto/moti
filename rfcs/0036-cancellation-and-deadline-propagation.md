# 0036-cancellation-and-deadline-propagation

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: cancellation context、deadline、propagation policy、typed result との接続
- Feature Name: `cancellation_and_deadline_propagation`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は cancellation と deadline を task context に組み込みで持たせ、structured task tree に沿って伝播する。cancellation は unchecked exception としては扱わず、typed result と policy hook を通じて回収する。deadline も同じ context で伝播し、runtime profile は timer と cancellation wakeup を提供しなければならない。

## Motivation

- cancellation を library 慣習へ委ねると、async surface の意味が runtime ごとに揺れる。
- deadline と cancel を別制度にすると、task API の責任境界が読みにくくなる。
- use case 1 は、親 task の cancel が child へどう届くかを型と policy で読みたい user である。
- use case 2 は、timeout 付き API を backend 非依存に提供したい runtime author である。
- no-op のままだと、Moti の structured concurrency は tree を持っても failure / timeout story が曖昧なまま残る。

## Guide-level explanation

Moti では task は cancel されうるし、deadline を持ちうる。だが、その伝播は hidden throw ではなく task context の一部である。API author は sibling failure policy や timeout surface を明示できる。

```moti
with_deadline(5s) {
  fetch_remote()
}
```

この block は 5 秒の deadline を持つ。期限切れや親 cancellation は task tree に沿って流れ、結果は typed surface に回収されると理解すればよい。

## Reference-level explanation

- task context は cancellation token と optional deadline を持つ。
- parent cancellation は child task へ伝播する。
- deadline 超過は corresponding cancellation reason として扱う。
- sibling への cancellation 伝播は task group policy に従う。
- cancellation は unchecked exception として surface へ漏らしてはならない。
- async API の failure surface は `Result` などの typed result を通じて cancellation / timeout を表現しなければならない。
- runtime profile は conforming implementation として deadline timer、wake-up、cancel propagation hook を提供しなければならない。must / should / may の割り当て自体は `0042` を正本とする。
- `Diverge`、`Exn`、`Async` との interaction で cancellation reason を隠してはならない。

## Drawbacks

- API が policy parameter を持ちやすくなり、簡単さを損なう場面がある。
- runtime 実装に timer / wake-up 責務が増える。
- cancellation を型付き surface へ回収するため、従来の throw-style より記述量が増える。

## Rationale and alternatives

- 代替案 1 は cancellation を unchecked throw にする案だったが、typed error surface を壊すため採らない。
- 代替案 2 は deadline を library helper に委ねる案だったが、runtime contract が不明瞭になるため採らない。
- 代替案 3 は sibling cancel policy を固定しきる案だったが、use case 差が大きいため policy hook を残す。
- 何もしない場合、timeout / cancel の責任が ecosystem 慣習に流れ、portable async contract が崩れる。

## Prior art

- Kotlin、Swift の structured cancellation は親子伝播の重要な lesson を与える。
- Rust ecosystem の cancel story の多様さは、言語側で contract を早く固定する必要を示す。
- Moti は typed result を中心に置き、exception-like cancellation を採らない点で意図的に分岐する。

## Unresolved questions

- cancellation reason の標準 taxonomy。
- default sibling policy をどこまで標準化するか。

## Future possibilities

- virtual time test runtime との tighter integration。
- distributed deadline propagation。
