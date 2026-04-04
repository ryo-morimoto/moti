# Rust RFC 一次ソースメモ

このメモは `rust-lang/rfcs` の一次ソースを、Moti の RFC review に使うために
圧縮したもの。

参照元:

1. `/tmp/rust-lang-rfcs/0000-template.md`
2. `/tmp/rust-lang-rfcs/README.md`
3. `/tmp/rust-lang-rfcs/lang_changes.md`
4. `/tmp/rust-lang-rfcs/compiler_changes.md`
5. `/tmp/rust-lang-rfcs/libs_changes.md`
6. `/tmp/rust-lang-rfcs/text/3391-result_ffi_guarantees.md`
7. `/tmp/rust-lang-rfcs/text/3484-unsafe-extern-blocks.md`
8. `/tmp/rust-lang-rfcs/text/3509-prelude-2024-future.md`
9. `/tmp/rust-lang-rfcs/text/3514-float-semantics.md`
10. `/tmp/rust-lang-rfcs/text/3355-rust-spec.md`

## 原典 template から取るべきこと

1. `Summary` は 1 段落で proposal の decision を言う。
2. `Motivation` は problem と use case を書く。
3. `Guide-level explanation` は teaching 用であり、例中心。
4. `Reference-level explanation` は interaction と corner case を扱う。
5. `Drawbacks` は実コストを書く。
6. `Rationale and alternatives` は alternatives の空欄を許さない。
7. `Prior art` は precedent の列挙ではなく lesson を扱う。
8. `Unresolved questions` は未決着の列挙。
9. `Future possibilities` は extension を置くが、acceptance 根拠にはしない。

## README から取るべきこと

1. RFC は substantial change のための controlled path である。
2. 低品質 RFC は motivation 不足、impact 理解不足、drawbacks / alternatives の不誠実さで弱くなる。
3. RFC は PR 上で改訂されることが普通で、議論の要約と trade-off の整理が重要。
4. accepted RFC は implementation の rubber stamp ではない。
5. 大きな変更は amendment ではなく新しい RFC にすべきことがある。

## 言語 RFC 方針から取るべきこと

1. 言語変更はほぼすべて RFC を要する。
2. non-local change、motivating use case を変える変更、複数解がある変更は新 RFC に寄せる。

## compiler / libs 方針から取るべきこと

1. すべてを RFC にするのではなく、substantial か user-facing かで分ける。
2. RFC は重いので、 trivial な変更まで RFC に押し込まない。
3. accepted RFC と implementation は 1 対 1 に完全一致しなくてよいが、大きな逸脱は説明が要る。

## 実例 RFC から取るべきこと

### `3391-result_ffi_guarantees`

1. 小さい RFC では `Guide-level explanation` を短くしてもよい。
2. Reference は表と具体型で規則を一発で読める形にすると強い。
3. `Future possibilities` は scope を広げすぎず extension だけを置く。

### `3484-unsafe-extern-blocks`

1. Motivation で proof obligation の所在を明確にしている。
2. Guide で誰が何を unsafe に引き受けるかを教えている。
3. Alternatives が豊富で、なぜ別案がだめかが分かる。
4. migration story を alternatives の中でも評価している。

### `3509-prelude-2024-future`

1. Guide は migration 前後の短い例で mental model を作っている。
2. Reference は最小差分だけを書く。
3. Rationale は選定基準そのものを明示する。

### `3514-float-semantics`

1. semantics RFC は Motivation で open issue pressure を示すと強い。
2. Guide は「普通の読み方」と「例外条件」を分けて説明する。
3. Reference は terminology を先に定義し、後でルールを与える。
4. Alternatives では spec strength の別水準を比較している。

### `3355-rust-spec`

1. process / policy RFC では goal / non-goal / current state を丁寧に分ける。
2. scope を deliberately left open として残すときは、何を今決めないかを明示する。
3. implementation ではなく coordination の RFC でも Motivation は具体的に書ける。

## Moti へ持ち込む lesson

1. メンタルモデル構築を優先する RFC では `Guide-level explanation` が特に重要。
2. 仕様 RFC では `Reference-level explanation` に invariant、boundary、interaction、failure mode を置く。
3. split / merge 判定は use case と protected invariant を基準にする。
4. alternatives を書けない RFC は decision がまだ分離できていない可能性が高い。
5. prior art は authority ではなく、pressure と lesson の比較材料として使う。
