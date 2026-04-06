# 0047-canonical-formatter-contract

- 状態: `Draft`
- 日付: 2026-04-06
- 対象範囲: canonical formatter、surface normalization、qualified form、hole preservation、tooling 境界
- Feature Name: `canonical_formatter_contract`
- Start Date: 2026-04-06
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は formatter を単なる補助ツールではなく language contract の一部として扱う。well-formed な module には package-selected formatter version の下で一つの canonical surface form だけがあり、formatter は import ordering、effect row、qualified trait call、whitespace を正規化する。formatter は意味を変える rewrite、hidden import、annotation 補完、core への不可逆な desugar を行ってはならない。

## Motivation

- `docs/purpose.md` は低エントロピーで一意なコードスタイルを根本要件に置いている。
- `0001` は canonical surface を固定したが、formatter が language law としてどこまで normative かは未決である。
- use case 1 は、agent が大量生成したコードを diff と review だけで読めるようにしたい maintainer である。
- use case 2 は、compiler、formatter、language server が同じ canonical surface を共有したい tooling author である。
- no-op のままだと、構文は一つに絞っても print 形が複数に分岐し、import ordering、effect row、qualified form の drift が残る。
- formatter version skew を無制限に許すと、package ごとの canonical surface contract が崩れ、低エントロピー目標そのものが弱くなる。

## Guide-level explanation

Moti では「意味が同じなら formatter が一つの見た目へ寄せる」。書き方の好みを競うのではなく、読み手が最短でメンタルモデルへ入れることを優先する。

```moti
import core/result as Result
import std/fs as FS

Eq.eq(&x, &y)
fn parse(path: Path): Ast / {IO, Exn[ParseError]}
```

この形が canonical なら、formatter は別の空白、別の import 並び順、別の effect row 順序へ揺らさない。読み手は「Moti ではこう見える」と覚えればよい。

## Reference-level explanation

- 分類: formatter を language contract の一部とすることは恒久不変である。
- 分類: package-selected formatter version の下で canonical surface form を一つに定めることは恒久不変である。
- 分類: package 単位で formatter version を選ぶこと、formatter major 境界でのみ canonical rewrite を許すこと、comment anchoring の細部は現行プロファイルである。
- formatter は whitespace、indentation、import ordering、effect row ordering / deduplication、qualified trait call の正規形を決める。
- import の canonical order は canonical import path の辞書順とし、formatter は import alias の spelling 自体を再命名しない。alias の可否と明示性は `0002` の owner である。
- current profile では、旧統合仕様 `language_spec_v0_13_full.md` の「alias は formatter により canonical に正規化される」は、import list の順序と owner 既定済みの canonical import form を表示面でそろえるという意味に限って読む。alias presence の決定自体は `0002` の owner であり、この RFC では先取りしない。
- `package-selected formatter version` とは、その package の build / tooling 設定が選ぶ formatter version を指す。この RFC は選択方法そのものではなく、選ばれた後の surface contract だけを定める。
- current profile では、各 package は全 module に対して一つの formatter version だけを選び、file-local override を許さない。
- canonical output を変更する formatter rewrite は formatter major 境界でのみ導入してよい。patch / minor では whitespace bugfix や comment anchoring fix の範囲を超えてはならない。
- package-selected formatter version が出力する canonical form と異なる source も parse 自体はされうるが、formatting contract 上は non-conforming とみなす。
- formatter は hidden import、implicit prelude、型 annotation の推論挿入、effect inference の結果挿入、core への不可逆な desugar を行ってはならない。
- `?` hole、visibility marker、boundary marker、effect annotation、qualified path の意味は保持されなければならない。
- formatter は `0001`、`0002`、`0032`、`0040` が定める canonical surface を表示面で強制するが、それらの意味論 owner にはならない。
- formatter が強制するのは、各 owner RFC がすでに固定した canonical surface の表示面だけである。未決の import alias policy や将来 sugar の admission 自体を formatter が先に決めてはならない。
- formatter と canonical printer は標準ツールチェイン上で同じ surface contract を共有し、別々の正規形を持ってはならない。
- linter や language server は追加提案を出してよいが、formatter と競合する別 canonical form を定義してはならない。
- holes、`--verbose` 出力、inlay hint、IDE UI はこの RFC の必須標準化対象ではない。

## Drawbacks

- style の自由度をさらに捨てるため、既存言語の個人流儀を持ち込みにくい。
- formatter の変更が source diff と教育コストに直結するため、versioning の責任が重い。
- lossless syntax と stable print を両立する実装負担が増える。
- formatter major 更新時には package 全体で大きな diff churn が起こりやすい。
- package 単位で version を固定するため、formatter 移行を file 単位で段階的に進めにくい。

## Rationale and alternatives

- 代替案 1 は formatter を non-normative tool に留める案だったが、agentic code generation 時代の surface drift を防げないため採らない。
- 代替案 2 は複数 style profile を許す案だったが、低エントロピー目標と直接衝突するため採らない。
- 代替案 3 は formatter ではなく linter の autofix に任せる案だったが、canonical form が複数生まれるため採らない。
- 代替案 4 は formatter が import alias spelling まで rename する案だったが、`0002` が alias naming law を owner していない段階で新しい命名規則を持ち込むため採らない。
- 代替案 5 は version selection / distribution governance の細部まで本 RFC で固定する案だったが、build / tooling governance の owner を越えるため採らない。この RFC は package-selected version の下での最小 surface contract と formatter-major 境界だけに限定する。
- 何もしない場合、言語本体が canonical surface を掲げても、実際の source は print 形の揺れで再び分岐する。

## Prior art

- `gofmt` は canonical formatter を language culture の中核へ置く実例である。
- `zig fmt` は language-owned formatter が explicitness と tooling coherence を支える lesson を与える。
- `rustfmt` は実用性が高いが、Moti は style option を減らし language law により近づける。

## Unresolved questions

- comment anchoring をどこまで stable contract にするか。

## Future possibilities

- hole-driven assist と canonical formatter の接続。
- machine-readable formatting diff contract。
- canonical printer を diagnostics snippet rendering と共有する補助 RFC。
