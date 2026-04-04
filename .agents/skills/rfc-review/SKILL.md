---
name: rfc-review
description: Rust RFC template に従って RFC を書く・レビューするための skill。Use when RFC の新規作成、既存 RFC の section review、論点の split/merge 判定、motivation と reference の整合確認、代替案や未解決点の不足検出、Moti 向け言語設計 RFC の修正方針整理を行うとき。
---

# RFC Review

Rust RFC の一次ソースに沿って、RFC を section ごとに書ける状態と、
review で差し戻せる状態を作るための skill。

この skill の仕事は次の 2 つです。

1. RFC を template 通りに埋めること
2. RFC の弱い section と、修正すべき具体点を findings-first で返すこと

この skill は「RFC の体裁確認」だけには使わない。
主目的は、読み手が安定したメンタルモデルを作れる RFC にすること。

## Grounding

1. Rust 側の一次ソースを確認する。
   - `references/rust-rfc-template.md`
   - `references/rust-rfc-source-notes.md`
2. Moti 側の一次ソースを確認する。
   - `docs/purpose.md`
   - `AGENTS.md`
   - 対象 RFC と、その隣接 RFC / question / proposal / draft
3. 仕様 RFC の場合は、関連する `language_spec_v0_13_full.md` の節を読む。
4. review では `references/revision-checklist.md` を使う。
5. section ごとの執筆・修正では `references/section-writing-guide.md` を使う。

## When To Use

次のどれかに当てはまるときに使う。

1. RFC を新規作成するとき
2. 既存 RFC を section ごとにレビューするとき
3. RFC が broad すぎて split すべきか迷うとき
4. 複数 RFC が同じ判断を重複していないか確認するとき
5. Motivation / Guide / Reference / Alternatives の整合が怪しいとき
6. Moti の言語設計上の不変条件を RFC に落とし込めているか確認するとき

## Output Modes

### Review mode

最終出力は次の順にする。

1. Findings
2. Section status
3. Revision plan
4. Residual risks

Findings は severity 順に並べる。
各 finding は必ず次を含める。

1. 問題がある section
2. 何が足りないか、何が矛盾しているか
3. どの section へ何を足す・削る・移すべきか
4. 可能ならファイルパスと行番号

### Author mode

最終出力は次の順にする。

1. RFC の decision sentence
2. section ごとの要点
3. 例
4. unresolved questions
5. split / merge が必要ならその判断

## Review Process

1. RFC の対象を classify する。
   - 言語意味
   - 境界 / 安全性
   - runtime / standard library
   - 実装方針
   - process / policy
2. その RFC が本当に 1 つの decision を扱っているか確認する。
3. `Summary` が「何を決める RFC か」を一文で言えているか確認する。
4. `Motivation` が pressure と use case を持っているか確認する。
5. `Guide-level explanation` が読み手のメンタルモデルを作れているか確認する。
6. `Reference-level explanation` が規則・境界・相互作用・角ケースを持っているか確認する。
7. `Drawbacks` が本当のコストを書いているか確認する。
8. `Rationale and alternatives` が rejected alternatives と非採用理由を書いているか確認する。
9. `Prior art` が precedent の列挙ではなく lesson の抽出になっているか確認する。
10. `Unresolved questions` が欠陥の隠し場所になっていないか確認する。
11. `Future possibilities` が現行 RFC に入れるべき内容を先送りしていないか確認する。
12. Moti 固有レンズで再評価する。

## Moti-Specific Review Lenses

以下の観点で弱い RFC は差し戻す。

1. safe fragment と trust boundary が曖昧
2. proof obligation の所在が曖昧
3. import-independent resolution を壊す
4. implicit prelude / implicit import を導入する
5. canonical surface を崩す
6. typed error surface を曖昧にする
7. runtime-backed なものを pure core に混ぜる
8. implementation convenience を将来の単調増加より優先する
9. invariant / current profile / reserved extension point の区別がない

## Split / Merge Rules

### Split すべきとき

1. 1 つの RFC が複数の independent decision を含む
2. Motivation が 2 つ以上の別 pressure を扱っている
3. Guide-level explanation の読者が 2 種類以上に分かれ、別々の mental model を要求する
4. Reference-level explanation で別々の invariant を守っている
5. Alternatives が RFC 全体ではなく節ごとに別々に存在する

### Merge すべきとき

1. 2 つの RFC の Summary が同じ decision を別の言い方で繰り返している
2. 片方の RFC が他方の Motivation を前提にしないと成立しない
3. 片方の Reference が他方の rule を直接内包している
4. 読者が両方を読まないと 1 つの mental model を作れない

## Section Status Labels

各 section を次のどれかで判定する。

1. `ok`
2. `expand`
3. `rewrite`
4. `split`
5. `merge`
6. `move`
7. `delete`

## Writing Rules

1. 日本語で書く
2. section 名は Rust RFC template に合わせる
3. Summary は 1 段落で終える
4. Motivation は use case を持つ
5. Guide-level explanation は例から始める
6. Reference-level explanation は規則と境界を書く
7. Drawbacks は実コストを書く
8. Alternatives は「なぜ今の案が勝つか」を示す
9. Unresolved questions は本当に未解決のものだけを書く
10. Future possibilities は現行設計を曖昧にする逃がし場所にしない

## Default Deliverables

この skill を使った review の既定 deliverables は次です。

1. section ごとの status
2. blocking findings
3. 修正対象の section と追加内容
4. split / merge 判定
5. Moti 固有の不変条件に照らした残リスク

詳しい section guide は [references/section-writing-guide.md](references/section-writing-guide.md)、
review checklist は [references/revision-checklist.md](references/revision-checklist.md)、
一次ソースの読み取り結果は [references/rust-rfc-source-notes.md](references/rust-rfc-source-notes.md)、
原典 template は [references/rust-rfc-template.md](references/rust-rfc-template.md) を参照する。
