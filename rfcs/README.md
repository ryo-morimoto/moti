# RFC 文書

このディレクトリは、**まだ accepted ではないが、review に耐える粒度まで育った仕様提案 RFC** の置き場所である。

## 役割

ここに置く文書は、正準言語仕様ではない。
ここに置く文書は、次を説明するためにある。

- いま何を提案しているか
- どの圧力がその提案を必要にしているか
- どの不変条件を守ろうとしているか
- どの代替案を検討したか
- どの open question が残っているか

accepted な意味論は `docs/language/` などの正準文書へ移す。
cross-cutting な根拠は必要に応じて `docs/decisions/` に残す。

## このディレクトリに置く資料型

- 提案文書 (`Proposal`)
  - `NNNN-<slug>.md`
- レビュー記録 (`ReviewRecord`)
  - `NNNN-<slug>.review.md`

## ここに置く状態

- `Framed`
- `Draft`
- `InReview`
- `NeedsRevision`
- `ValidatedProfile`
- `Deferred`（proposal 作成後に止めた場合）
- `Rejected`

`Question` と、proposal をまだ持たない `Deferred` は既定では `docs/questions/Q-<slug>.md` に置く。
`CanonicalRule` と `Superseded` は正準文書側に置く。

## 命名規則

- 形式は `NNNN-<slug>.md`
- `NNNN` は 4 桁の安定した提案番号
- `&lt;slug&gt;` は topic を表す短い kebab-case
- `ReviewRecord` は `NNNN-<slug>.review.md` を使う
- 新規作成時は [`_template.md`](./_template.md) と [`_review_template.md`](./_review_template.md) を起点にする
- 新しい `Proposal` は既存最大番号 + 1 を使い、欠番は再利用しない
- state が変わってもファイル名は変えない

例:

- `0001-trait-solver-formalization.md`
- `0002-verified-extern-evidence-model.md`
- `0002-verified-extern-evidence-model.review.md`

## 必須セクション

1. 状態
2. 日付
3. 対象範囲
4. 背景と問題
5. 提案
6. 守る不変条件
7. 非目標
8. 例
9. 検討した代替案
10. 未解決点

## 運用ルール

1. `Framed` になった時点で、問い文書だけに閉じ込めず proposal file を作る。
2. `InReview` に入った時点で `ReviewRecord` を作る。
3. `Draft`、`InReview`、`NeedsRevision` は同じ `Proposal` を育て、レビュー所見は `ReviewRecord` に蓄積する。
4. `ValidatedProfile` では proposal file を消さず、canonicalization の根拠として残す。accepted 本文は `docs/language/` に移し始める。
5. `Deferred` でも既存の `Proposal` と `ReviewRecord` は消さず、停止理由だけを残す。
6. `Rejected` でも proposal file は残し、なぜ却下したかを記録する。
7. 構造を変えたいときは、先に template とこの README を更新する。
