# 0021-evaluation-strategy-and-order-of-evaluation

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: call-by-value、式の評価順、`await` を含む sequencing、effect 発火順序
- Feature Name: `evaluation_strategy_and_order_of_evaluation`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の評価戦略は strict な call-by-value であり、観測可能な式の評価順序は source 順の left-to-right で固定する。関数呼び出し、`let`、record / tuple 構築、match の scrutinee、effect operation の引数、`await` 前の評価はすべてこの規律に従う。これにより、effect、borrow、async をまたいでも hidden control flow を増やさない。

## Motivation

- Moti は宣言的デフォルトを採るが、評価順が曖昧だと hidden control flow が残る。
- borrow、effect handler、`await` が共存する以上、いつ何が評価されるかを固定しないと diagnostics と reasoning が揺れる。
- use case 1 は、複数の effectful 引数を持つ関数呼び出しで副作用順を読みたい programmer である。
- use case 2 は、`await` を含む式で suspension 前にどこまで評価されるかを把握したい async API author である。
- no-op のままだと、実装ごとに argument evaluation order や suspension timing がずれ、portable semantics を損なう。

## Guide-level explanation

Moti では「見た順に評価される」と考えてよい。関数に引数を 3 つ渡すなら、左から右へ評価される。`match` はまず scrutinee を評価し、分岐選択後に branch body を評価する。`await` は、その式の被演算子を先に評価してから suspend する。

```moti
let x = log_a()
let y = log_b()
f(x, y)
```

この例では `log_a()` が先、`log_b()` が後である。`f` の本体に入る前に両方の引数評価が終わる。読み手は optimizer や backend の気分ではなく、surface の並び順だけを見ればよい。

```moti
let tmp = make_ref(buf)
await io_task()
```

ここで本 RFC が固定するのは `make_ref(buf)` が `await` より前に評価されることまでである。borrow の live range、temporary の drop scope、await-affine capability の禁止規則そのものは borrow / async owner RFC を正本とする。

## Reference-level explanation

- 分類: strict な call-by-value と left-to-right sequencing は恒久不変である。
- 評価戦略は call-by-value である。
- 観測可能な subexpression の評価順は left-to-right である。
- 関数呼び出しでは callee expression を先に評価し、その後に各引数を左から右へ評価する。
- `let x = e1; e2` は `e1` を先に評価し、値を束縛してから `e2` を評価する。
- tuple、record、enum payload の各要素は source 順で評価する。
- `match e { ... }` は scrutinee `e` を先に評価し、branch 選択後に選ばれた branch body だけを評価する。
- pattern 自体は値を評価しない。pattern checking に伴う refinement は branch 選択後に行う。
- effect operation は operation 引数を先に left-to-right で評価してから発火する。
- `await e` は `e` を値まで評価し、その結果が awaitable なら suspend boundary に入る。`e` の内部 subexpression は suspension 前に評価される。
- handler の `resume` は resumption argument を先に評価してから継続を再開する。
- 本 RFC が固定するのは観測可能な subexpression sequencing までである。temporary drop scope、live borrow、await-affine capability の禁止規則は `0030`、`0031`、`0035` を正本とする。
- 評価順序の最適化は、typed error surface、effect surface、borrow conflict の観測可能結果を変えてはならない。
- 実装は内部 IR で順序を並べ替えてよいが、外部観測可能な evaluation order を変えてはならない。

## Drawbacks

- 一部 backend 最適化が制約される。
- non-strict evaluation による遅延計算の利点は既定では得られない。
- すべての式位置で順序が固定されるため、実装者は IR に sequencing を保持する必要がある。

## Rationale and alternatives

- 代替案 1 は未規定の evaluation order を許す案だったが、effect と borrow の相互作用が実装依存になるため採らない。
- 代替案 2 は lazy evaluation を既定にする案だったが、space leak と effect surface の説明が重くなるため採らない。
- 代替案 3 は call-by-push-value を surface semantics に直接出す案だったが、読者の mental model を増やしすぎるため採らない。
- 何もしない場合、`await` 前後の責任、argument side effect、borrow rejection の局所説明が壊れる。

## Prior art

- ML、Rust、Go の strict evaluation からは、source 順をそのまま読み筋にできる lesson を取る。
- Koka や effectful calculi からは、effect 発火順序を semantics に残さないと hidden control flow が戻ることを学ぶ。
- Haskell の lazy evaluation は強い表現力を示すが、Moti は hidden control flow 抑制と予測可能な cost model を優先し、その既定は輸入しない。

## Unresolved questions

- 短絡評価演算子を導入する場合の precise wording。
- pure subexpression のみを対象にした順序無害最適化の conformance wording。

## Future possibilities

- strict default を壊さない明示的 lazy block。
- evaluation trace を diagnostics / pedagogy に使う補助仕様。
