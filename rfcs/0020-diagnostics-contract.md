# 0020-diagnostics-contract

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: diagnostics の最低契約、error category、required fields、negative test 接続
- Feature Name: `diagnostics_contract`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は diagnostics を UX の飾りではなく、仕様と実装の一致を検証するための契約として扱う。少なくとも type、borrow、trait、visibility、effect、error-boundary、boundary verification の主要 error category を区別し、それぞれが required field を持つ構造化診断を返すことを要求する。現行プロファイルでは `boundary verification` は独立 category とする。

## Motivation

- Moti では conformance と negative test を最終的に支える必要がある。
- error category ごとの required fields が曖昧だと、実装差を許しすぎて drift が見えない。
- borrow や trait solver の保守的制約は、diagnostics の説明責任があって初めて読み手に受け入れられる。
- verified extern のような境界制度も、diagnostics がなければ原因を局所化しにくい。
- use case 1 は、negative test で wording ではなく contract を照合したい implementer である。
- use case 2 は、trait / borrow / boundary failure の理由を category と span で追いたい user である。

## Guide-level explanation

この RFC の mental model は単純である。Moti は「親切なメッセージ」を標準化したいのではない。少なくとも何の category の失敗か、どこで起きたか、何と何が衝突したかを、実装が共通に返せるようにしたいのである。

読み手は、compiler が違っても「失敗の種類」と「最低限の説明責任」は同じだと期待できる。

## Reference-level explanation

- diagnostics は major error category を区別する。
- 少なくとも type、borrow、trait、visibility、effect、error-boundary、boundary verification の category を持つ。
- 各 diagnostic は primary span を持つ。
- 必要に応じて related span / note を持つ。
- type: `expected_type`、`actual_type`、`primary_span` を必須とする。
- borrow: `conflicting_place`、`borrow_kind`、`previous_use_span`、`primary_span` を必須とする。
- trait: `obligation`、`failure_mode`、`primary_span` を必須とする。`failure_mode` は no-solution / ambiguity / invalid-program を区別する。
- visibility: `declared_visibility`、`blocking_dependency`、`primary_span` を必須とする。
- effect: `required_effect`、`actual_effect` か `unhandled_effect`、`primary_span` を必須とする。
- error-boundary: `leaked_failure_form`、`required_surface_form`、`primary_span` を必須とする。
- boundary verification: `verification_phase`、`disposition`、`failed_checks`、`primary_span` を必須とする。
- negative test は wording ではなく category と required field を基準に照合する。
- exact wording、color、IDE UI は契約に含めない。
- sample payload:
  - type mismatch: `{ category: type, expected_type, actual_type, primary_span }`
  - borrow conflict: `{ category: borrow, conflicting_place, borrow_kind, previous_use_span, primary_span }`
  - verified extern rejection: `{ category: boundary_verification, verification_phase, disposition, failed_checks, primary_span }`

## Drawbacks

- 実装者に追加コストが掛かる。
- wording 自由度を残しつつ contract を決める設計が必要になる。
- 診断面まで仕様化することに抵抗が出やすい。

## Rationale and alternatives

- 代替案 1 は diagnostics を完全に実装自由にする案だったが、conformance を支えられないため採らない。
- 代替案 2 は wording まで固定する案だったが、UX まで標準化しすぎるため採らない。
- 代替案 3 は type error だけ先に固定する案だったが、borrow / trait / effect を含めた全体 mental model を作りにくい。
- 何もしない場合、規則はあっても failure surface が実装ごとにばらつく。

## Prior art

- Rust や GHC の diagnostics は category ごとの説明責任の参考になる。
- language server / compiler の structured diagnostic format も比較対象になる。
- lesson は「wording ではなく structured field を contract にすること」である。

## Unresolved questions

- required field をどこまで細かく標準化するか。
- schema versioning をどの段階で導入するか。

## Future possibilities

- machine-readable diagnostics schema。
- teaching mode / expert mode のような presentation profile。
