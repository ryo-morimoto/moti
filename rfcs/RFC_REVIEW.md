# RFC Review

- 対象:
  - 分割元 RFC 更新: `0003` `0004` `0005` `0007` `0008` `0009` `0011` `0016` `0019`
  - 新規 / 分割 RFC: `0021` から `0044`
  - 補助文書: `RFC_INDEX.md` `RFC_MAP.md` `index.md` `rfc_restructuring_state.json`
- 日付: 2026-04-05
- review basis:
  - `.agents/skills/rfc-review/SKILL.md`
  - `.agents/skills/rfc-review/references/section-writing-guide.md`
  - `.agents/skills/rfc-review/references/revision-checklist.md`

## 解消した主要所見

- `0003` `0004` `0005` `0007` `0008` `0009` `0011` `0016` `0019` に successor RFC を明示し、分割後の owner を固定した。
- `0021` で strict call-by-value と left-to-right sequencing を独立 RFC として切り出した。
- `0022` `0023` `0027` の責務境界を整理し、kernel judgment / algorithmic typing / conversion owner を分離した。
- `0026` で positivity / projection-only codata / guarded corecursion の最小 accepted / rejected 例を追加した。
- `0032` `0033` `0034` を分離し、closed row、handleable / latent effect、effect abstraction の owner を整理した。
- `0035` `0036` `0037` を分離し、task tree、cancellation / deadline、`Send` / `Share` / domain affinity の owner を整理した。
- `0043` `0044` で checker interface と wrapper obligations / `logical-safe` を分離し、disposition と admission rule を明文化した。
- `RFC_INDEX.md` と `RFC_MAP.md` を 44 本体制へ更新し、`rfc_restructuring_state.json` を正本として整合させた。

## レビュー結果

- `0021` から `0027` は 3 回の review pass 後に findings なし。
- `0028` から `0034` は 3 回の review pass 後に findings なし。
- `0035` から `0044` は 3 回の review pass 後に findings なし。
- 分割元 RFC、索引、状態 JSON の coherence review でも findings なし。

## 意図的に残したこと

- 正準言語文書化はまだ行っていない。accepted semantics の置き場は引き続き `docs/language/` であり、この作業では RFC 層までに留めた。
- `0008`、`0009`、`0016`、`0019` は総論 RFC として残し、詳細 owner を後続 RFC に分配した。

## 残リスク

- 次の大きなリスクは RFC 本文そのものではなく、accepted 後に `docs/language/` と関連 decision / question 文書へ同期しないことによる drift である。
- `0042` の profile 拡張、`0044` の `logical-safe` 表現、`0037` の将来 capability 拡張は、後続 RFC の入口として残っている。
- 追加実装や conformance 体制の整備に入る際は、`0020` の diagnostics contract と合わせて再確認が必要である。
