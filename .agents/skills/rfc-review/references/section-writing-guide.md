# Section Writing Guide

この文書は Rust RFC template の各 section に何を書くか、何が修正対象かを
具体化するための guide。

## Header

### 役割

RFC の識別子と追跡先を固定する。

### 必ず書くこと

1. `Feature Name`
2. `Start Date`
3. RFC の参照先
4. 実装や追跡先が未定なら、その旨を空欄ではなく明記

### 修正対象

1. 名前が広すぎて 2 つ以上の decision を含む
2. 名前が mechanism だけで pressure が読めない
3. 追跡先がなく、後続作業が追えない

## Summary

### 役割

RFC が何を決めるかを 1 段落で確定する。

### 必ず書くこと

1. 決定そのもの
2. 対象範囲
3. 読者が得る新しい mental model の核

### 良い状態

1. 1 段落で読める
2. 「何を変える RFC か」が迷わない
3. Motivation を読まなくても decision の輪郭が分かる

### 修正対象

1. 背景説明だけで decision がない
2. scope が広すぎる
3. 参考案の列挙になっている

## Motivation

### 役割

なぜ今この RFC が必要かを示す。

### 必ず書くこと

1. 解く問題
2. その問題が実際にどこで痛いか
3. 具体 use case
4. 何を守りたいか

### 良い状態

1. 問題が user / implementer / spec maintainer の言葉で見える
2. use case が 2 つ以上ある
3. 後段の design がこの pressure から自然に導かれる

### 修正対象

1. 「あったら便利」止まり
2. use case がない
3. 既存案の欠点を書かず、新案の自慢だけになっている
4. protected invariant が見えない

## Guide-level explanation

### 役割

読み手のメンタルモデルを作る。

### 必ず書くこと

1. 新しい用語の導入
2. 最小の例
3. 読み手がどう考えるべきか
4. 既存コードや周辺機能への影響

### 良い状態

1. 例から入る
2. 仕様を知らない読者でも使い方の像が持てる
3. 境界や unsafe obligation の所在が直感的に分かる

### 修正対象

1. reference の写経になっている
2. 例がない
3. 新用語が未定義
4. mental model ではなく条文しかない

## Reference-level explanation

### 役割

厳密な規則・境界・相互作用を定義する。

### 必ず書くこと

1. 規則
2. 適用範囲
3. 他機能との interaction
4. corner case
5. implementation / mechanization の見通し

### 良い状態

1. guide の例を reference で説明し直せる
2. 読者が境界条件を予測できる
3. failure mode と rejection mode が読める

### 修正対象

1. guide の例と reference の規則が一致しない
2. terminology 未定義のまま進む
3. interaction が省略されている
4. corner case がない
5. 実装不能な曖昧語が多い

## Drawbacks

### 役割

その案の実コストを認める。

### 必ず書くこと

1. teachability cost
2. implementation cost
3. migration cost
4. spec complexity cost
5. expressiveness cost

### 修正対象

1. 形だけある
2. 反対意見の strawman 化
3. 実コストが alternatives 側へ追い出されている

## Rationale and alternatives

### 役割

なぜこの案が他案より良いかを示す。

### 必ず書くこと

1. 採用案が守る invariant
2. 却下案
3. 各却下案の非採用理由
4. 何もしない場合の影響

### 良い状態

1. 2 案以上を比較している
2. 却下理由が pressure に結び付いている
3. 「この案が最善」と言う理由が mechanism ではなく property ベース

### 修正対象

1. 「他案は複雑」だけで終わる
2. no-op の影響がない
3. library / macro / profile 制約などの代替レイヤが検討されていない

## Prior art

### 役割

他言語や既存実践から lesson を取る。

### 必ず書くこと

1. 類似機構
2. どの圧力に対する設計か
3. 何を学ぶか
4. 何を輸入しないか

### 修正対象

1. precedent の列挙だけ
2. どこが違うかがない
3. authority appeal になっている

## Unresolved questions

### 役割

今の RFC で閉じない点を明示する。

### 必ず書くこと

1. RFC merge 前に詰めるもの
2. 実装で詰めるもの
3. future RFC に回すもの

### 修正対象

1. core design hole を未解決扱いしている
2. scope 外と未解決が混ざっている
3. 「None」だが alternatives や reference に穴がある

## Future possibilities

### 役割

現行決定を壊さない extension を整理する。

### 必ず書くこと

1. 自然な拡張口
2. 現行設計との関係
3. 将来 RFC に分ける理由

### 修正対象

1. 本来 current RFC に入れるべき核心設計を逃がしている
2. 方向性が広すぎて何も言っていない
3. 現行設計を不安定に見せる
