# 0044-verified-extern-wrapper-obligations-and-logical-safe

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: wrapper obligations、`logical-safe` 昇格条件、公開 surface 正規化、boundary proof ownership
- Feature Name: `verified_extern_wrapper_obligations_and_logical_safe`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti では checker が `verified` を返しただけでは public API は `logical-safe` にならない。boundary author は wrapper obligations を満たし、raw foreign contract、evidence handle、boundary-only capability を公開面へ漏らさず、failure、ownership、domain affinity を型付き契約へ正規化しなければならない。`logical-safe` 昇格は checker result と wrapper contract の両方が揃った場合にのみ許す。

## Motivation

- verified extern の最大の失敗モードは「検証済み」という語だけが独り歩きして raw contract が漏れることにある。
- checker interface と wrapper disciplineを分けないと、誰が何を引き受けるかが曖昧になる。
- use case 1 は、検証済み C binding を safe wrapper として package user に見せたい boundary author である。
- use case 2 は、domain-local / cancellation-sensitive foreign API を public surface へ安全に正規化したい runtime integrator である。
- no-op のままだと、verified extern は checker に責任を押し付ける名札に留まる。

## Guide-level explanation

`verified extern` は「この raw declaration の外部契約が検証された」ことを示すだけで、利用者向け wrapper が安全だという意味ではない。safe になるには wrapper が失敗・所有権・domain affinity を public contract に反映しなければならない。

```moti
verified extern fn raw_hash(...): Digest

pub fn hash(data: Bytes): Result[Digest, HashError] {
  ...
}
```

利用者が信じてよいのは `hash` の contract である。`raw_hash` の checker result だけではない。読み手は `logical-safe` を checker verdict ではなく wrapper-level property として理解するべきである。

## Reference-level explanation

- `logical-safe` 昇格は checker result が required safety axis で `verified` であり、かつ wrapper obligations が満たされた場合にのみ許す。
- wrapper obligations は少なくとも failure normalization、ownership / borrowing precondition の反映、domain affinity surface の保持、unchecked panic / cancellation の非露出、boundary-only metadata の非露出を含む。
- public / package surface に raw foreign contract、evidence handle、verification metadata、boundary-only capability を出してはならない。
- wrapper は `Result`、effect row、capability trait、safe facade type などを用いて公開 contract を正規化する。
- `logical-safe` は kernel equality や trait solver への shortcut を与えない。
- wrapper obligation を満たさない verified extern は verified boundary declaration ではあっても public logical-safe facade ではない。
- non-`logical-safe` な verified extern は boundary-only declaration に留まり、package / public surface へ出してはならない。
- diagnostics は checker result failure と wrapper obligation failure を区別できなければならない。

## Drawbacks

- wrapper 実装責任が重くなる。
- `verified` と `logical-safe` の二段階を教える必要がある。
- 一部 simple-looking FFI でも facade 設計が要求される。

## Rationale and alternatives

- 代替案 1 は checker `verified` をそのまま public safe と見なす案だったが、proof ownership が wrapper から消えるため採らない。
- 代替案 2 は wrapper obligations を guideline に留める案だったが、boundary discipline として弱すぎるため採らない。
- 代替案 3 は public API へ evidence handle を出して user に選ばせる案だったが、safe fragment を利用者責任へ戻すため採らない。
- 何もしない場合、verified extern の安全性主張は raw contract の再包装に終わる。

## Prior art

- Rust の safe wrapper over unsafe FFI は wrapper obligation の lesson を与える。
- proof-carrying interface の研究は checker verdict と public contract を分ける重要性を示す。
- Moti は `logical-safe` を wrapper-level property として明示する点で、検証済み宣言と公開安全性を切り分ける。

## Unresolved questions

- `logical-safe` を surface keyword にするか documentation-only class にするか。
- wrapper obligation を negative test へどう接続するか。

## Future possibilities

- wrapper obligation template の自動生成。
- boundary audit report との連携。
