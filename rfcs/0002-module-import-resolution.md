# 0002-module-import-resolution

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: パッケージ、モジュール、import、再公開、名前解決
- Feature Name: `module_import_resolution`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の名前解決は import-independent でなければならない。パッケージは canonical import path を 1 つだけ持ち、implicit prelude、implicit import、wildcard import は導入しない。import は module 単位で行い、名前解決の結果が import の仕方や scope injection で変わらないように設計する。

## Motivation

- Moti は hidden mechanism を減らす言語であり、名前解決が import の偶然に依存してはいけない。
- trait core でも import-dependent resolution を禁止する以上、module resolution 側も同じ規律を持つ必要がある。
- package / module / import が明示的であれば、エージェント生成コードでも読み手が依存関係を局所的に理解できる。
- implicit prelude を認めると、標準ライブラリ境界や portability の議論まで不透明になる。

## Guide-level explanation

Moti では「使いたいものを明示的に開く」が基本である。import するときは module を名前付きで取り込み、その module 配下の item を参照する。

```moti
import core.result as result

fn ok(x: Int): result.Result[Int, Never] {
  result.Ok(x)
}
```

重要なのは、import は「見え方」を変えても「意味」を変えないことだ。ある impl や item が見つかるかどうかが、どの module をどの順序で import したかに依存してはならない。

## Reference-level explanation

- package は canonical import path を 1 つ持つ。
- import は module 単位で行う。
- implicit prelude、implicit import、wildcard import、scope injection は導入しない。
- 再公開は明示的に行う。
- 名前解決は lexical かつ import-independent である。
- trait candidate の集合は import によって増減しない。
- facade module は明示的に利用できるが、自動 open しない。

## Drawbacks

- 記述量は短期的には増える。
- 既存言語の prelude に慣れた利用者には冗長に感じられる。
- import の省略記法や shortcut を求める圧力が継続的に生じる。

## Rationale and alternatives

- 代替案 1 は implicit prelude だったが、見えている依存と実際の依存がずれるため採らない。
- 代替案 2 は wildcard import だったが、名前衝突と review 困難性を増やすため採らない。
- 代替案 3 は Rust 風の trait import で candidate を増やす案だったが、trait coherence-first の目標と衝突するため採らない。
- 何もしない場合、module system は便利だが不透明な層となり、Moti 全体の明示性が崩れる。

## Prior art

- Rust は explicit import を基本にしつつ prelude を持つ。Moti はそこからさらに暗黙性を削る。
- Go は import を明示し、unused import を嫌う点で近い。
- Haskell の qualified import は読み手の局所性を高めるが、typeclass resolution の import 依存性は Moti が避けたい方向である。

## Unresolved questions

- alias 必須化をどこまで強くするか。
- facade module の naming convention を仕様へ含めるか。

## Future possibilities

- IDE や formatter が canonical import form を自動整形するための補助規則。
- package graph 表示や doc generation と統合した import visualization。
