# 0019-verified-extern-evidence-model

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `verified extern`、証跡 attach 単位、checker interface、`logical-safe` 昇格条件
- Feature Name: `verified_extern_evidence_model`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は `verified extern` を、boundary module 内の各宣言単位に証跡を attach して安全境界を扱う制度として導入する。この RFC は verified extern の総論を固定する。abstract checker interface は `0043`、wrapper obligations と `logical-safe` 昇格条件は `0044` で精密化する。公開境界の一般規律は `0003-visibility-and-boundary-modules.md` を正本とする。

## Motivation

- FFI 境界は Moti の safe fragment に残る最後の大きな穴の一つである。
- raw foreign contract を直接公開すると、利用者に boundary proof obligation を押し付けてしまう。
- しかし、証跡 format や backend pipeline まで言語仕様へ固定すると柔軟性を失う。
- よって、proof ownership は外部 checker に置きつつ、言語側では checker interface と昇格条件だけを固定する必要がある。
- use case 1 は、C ABI を包む boundary author が安全責任を declaration ごとに示したいケースである。
- use case 2 は、利用者に raw foreign contract を見せず safe wrapper だけを expose したい package author である。

## Guide-level explanation

`verified extern` は「FFI が安全になった」という魔法ではない。正しくは、boundary module の作者が checker を通して外部契約を検証し、その結果を safe wrapper の公開契約へ正規化する制度である。利用者は raw contract を見ない。見るのは型付き wrapper だけである。

```moti
verified extern fn raw_hash(...): Digest

pub fn hash(data: Bytes): Result[Digest, HashError] {
  ...
}
```

読み手は、`verified extern` を kernel の新機能ではなく、trust boundary を明示するための boundary discipline として理解するべきである。ここで attach される証跡は `raw_hash` 宣言に結び付く。利用者が触れるのは `hash` の wrapper contract だけであり、証跡ハンドルや checker result は public surface に出ない。

## Reference-level explanation

- `verified extern` は boundary module 内で扱う。
- 証跡 attach 単位は verified extern 宣言ごととする。
- 言語仕様は concrete proof format を定義しない。
- 言語仕様は abstract checker interface を定義する。
- checker result は少なくとも `verified`、`rejected`、`inconclusive` の disposition と、capability safety、memory safety、panic / cancellation safety、scheduler / domain affinity の各判定欄を持つ。
- `logical-safe` 昇格は、各判定欄が `verified` であり、かつ wrapper obligations が満たされる場合にのみ許す。
- wrapper obligations は少なくとも次を含む:
  - failure を `Result` か同等の型付き公開契約へ正規化する
  - raw foreign contract、evidence handle、boundary-only capability を公開面へ漏らさない
  - domain / scheduler affinity 制約を wrapper の surface で保持する
  - panic / cancellation を unchecked failure として漏らさない
  - ownership / borrowing 前提を wrapper の入力契約へ反映する
- checker result、evidence handle、verification metadata は boundary-only とし、public/package surface に露出しない。
- raw foreign contract と boundary-only capability の非露出は `0003-visibility-and-boundary-modules.md` の規律に従う。
- `verified extern` は definitional equality に参加しない。
- checker interface と invalidation protocol の正本は `0043` とする。
- wrapper obligations と `logical-safe` 昇格条件の正本は `0044` とする。

## Drawbacks

- checker interface の設計が難しい。
- safe wrapper の記述量が増える。
- 「verified」という名前が過剰な安心感を与える危険がある。

## Rationale and alternatives

- 代替案 1 は raw `extern` のまま wrapper discipline に任せる案だったが、証拠責任が曖昧なまま残る。
- 代替案 2 は module pack 単位で attach する案だったが、declaration ごとの invalidation と diagnostics を粗くしすぎるため採らない。
- 代替案 3 は concrete proof format を仕様へ入れる案だったが、backend 依存性が強すぎるため採らない。
- 代替案 4 は checker 結果を kernel law へ埋め込む案だったが、小さい trusted kernel の原則に反するため採らない。
- 何もしない場合、FFI safety story は「外部実装を信じる」に留まり、Moti の差別化が崩れる。

## Prior art

- Rust の unsafe FFI は境界責任の所在を示すが、外部検証との接続は言語外にある。
- verified boundary や proof-carrying code の発想は参考になるが、Moti は abstract checker interface に留める。

## Unresolved questions

- checker result と diagnostics contract の接続詳細。
- checker の invalidation protocol をどこまで本文で規定するか。

## Future possibilities

- backend-specific verifier profile。
- verified extern wrapper の自動生成支援。
