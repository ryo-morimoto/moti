# 0031-suspend-boundaries-and-await-affine-capabilities

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: suspend boundary、`await` 越し保持禁止、await-affine capability、same-domain 例外
- Feature Name: `suspend_boundaries_and_await_affine_capabilities`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は `await` を明示的な suspend boundary として扱い、lexical borrow、`Borrow`、`BorrowMut`、`LocalRoot`、`RuntimeToken` などの await-affine capability をこの境界の向こうへ保持させない。`OwnedRoot` の same-domain 継続のみ current profile で限定的に許し、cross-domain への持ち込みは禁じる。

## Motivation

- `await` を単なる関数呼び出しとみなすと、scheduler と lifetime の責任が混ざる。
- borrow core が明確でも、suspend boundary を別規則にしないと async safety を語れない。
- use case 1 は、borrow 中の値を `await` 越しに保持してよいかを型だけで判断したい user である。
- use case 2 は、domain-local capability を scheduler 跨ぎで漏らしたくない runtime designer である。
- no-op のままだと、unsafe ではない async code が scheduler timing に依存して壊れる余地が残る。

## Guide-level explanation

Moti では `await` は「いったん制御が外へ戻る場所」だと考える。そのため、その間に他の処理が走ってはいけないことを前提にした capability は持ち越せない。

```moti
let r = &mut buf
await io_task()
use(r)
```

これは許されない。`r` は suspend boundary を越えているからである。読み手は `await` を見た時点で「borrow と local capability はここで切る」と考えればよい。

## Reference-level explanation

- 分類: `await` を suspend boundary として扱い、lexical borrow と await-affine capability を越境禁止にすることは恒久不変である。
- 分類: `OwnedRoot` の same-domain 例外は current profile である。
- 分類: capability migration helper の追加余地は reserved extension point である。
- `await` は same-domain suspension を表す suspend boundary である。
- lexical borrow、`Borrow`、`BorrowMut`、`LocalRoot`、`RuntimeToken` は await-affine capability とする。
- await-affine capability は `await` をまたいで live range を持ってはならない。
- same-domain の `OwnedRoot` は current profile で例外的に保持を許せるが、domain 越境は許さない。
- capability capture を行う closure / async block も同じ規則に従う。
- `await` 前に capability を解放、consume、または owned value へ変換できるなら継続してよい。
- diagnostics は capability kind と crossing suspend point を示さなければならない。
- `Parallel` spawn は `await` より強い境界であり、本 RFC の許可は `Parallel` へ持ち込まれない。

## Drawbacks

- async code で一時束縛の分割が増える。
- same-domain 例外が current profile に留まるため、完全一律ではない。
- coroutine 風 API の一部はより明示的な ownership 変換を要する。

## Rationale and alternatives

- 代替案 1 は `await` 越し borrow を広く許す案だったが、scheduler interaction を読み手へ押し付けるため採らない。
- 代替案 2 は same-domain と cross-domain を区別しない案だったが、必要以上に表現力を失うため採らない。
- 代替案 3 は library lint に委ねる案だったが、安全規則として弱すぎるため採らない。
- 何もしない場合、async safety story は borrow core と別の例外集になる。

## Prior art

- Rust async の `!Send` future や borrow across await 制限は直接の先行例である。
- structured concurrency 系言語は suspension point が resource lifetime を切る重要性を示す。
- Moti は await-affine capability をより明示的に naming することで mental model を固定する。

## Unresolved questions

- generator / stream syntax を導入する場合の suspend boundary の wording。

## Future possibilities

- same-domain capability migration を安全に行う標準 helper。
- suspend-boundary diagnostics の teaching mode。
