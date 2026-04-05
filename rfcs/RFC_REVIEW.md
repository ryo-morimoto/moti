# RFC Review

- 対象: `0001` から `0020` までの RFC 群
- 日付: 2026-04-05
- review basis:
  - `.agents/skills/rfc-review/SKILL.md`
  - `.agents/skills/rfc-review/references/section-writing-guide.md`
  - `.agents/skills/rfc-review/references/revision-checklist.md`

## 解消した主要所見

- `0019` で証跡 attach 単位を declaration ごとに固定した。
- `0019` で checker result の最小 shape と wrapper obligations を追加した。
- `0020` で required fields を category ごとに具体化した。
- `0016` で runtime profile の `must / should / may` を実際に割り当てた。
- `0017` で数値 portability の最小 contract を具体化した。
- `0015` で relational constraints の最小 model を追加した。
- `0018` を public surface の正規形へ寄せ、`0003` / `0017` の規律参照で責務重複を減らした。
- `RFC_MAP.md` に diagnostics と runtime profile の依存辺を追加した。

## 意図的に残したこと

- `0008`、`0009`、`0016` は breadth-first の都合でやや広い RFC のまま維持した。
- ただし本文中で中心決定を明文化し、current profile と恒久不変の区別を補った。

## 残リスク

- `0008`、`0009`、`0016` は将来、実装やレビューの深まりに応じて split した方が読みやすくなる可能性がある。
- `0017` の quiet/signaling NaN wording は今後さらに formal に詰める余地がある。
- `0020` の schema versioning は今後の conformance 体制次第で追加設計が必要になる。
