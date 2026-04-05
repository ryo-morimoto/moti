# 0028-effective-visibility-and-export-closure

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: effective visibility、公開シグネチャ閉包、re-export、blocking dependency diagnostics
- Feature Name: `effective_visibility_and_export_closure`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は `private`、`package`、`pub` の可視性を持ち、宣言の effective visibility はそのシグネチャに現れる型、effect、trait requirement、associated projection、const parameter より広くなれない。Moti の re-export は module alias に限られ、その alias 先が露出する contract も同じ closure rule を満たさなければならない。公開境界は参照先も含めて読める contract で閉じる。

## Motivation

- `pub fn` の戻り型が private 型を含むような宣言を許すと、公開面の mental model が壊れる。
- 境界や effect をシグネチャから読めるようにするには、visibility を token だけでなく依存閉包で見なければならない。
- use case 1 は、公開 API の契約が型依存まで含めて self-contained であってほしい library user である。
- use case 2 は、compiler が export error を declaration-site で説明してほしい implementer である。
- no-op のままだと、wrapper は `pub` でも内部依存が漏れた半公開 API が残る。

## Guide-level explanation

Moti では「名前が `pub` なら公開」ではなく、「その名前が参照するものまで公開可能なら公開」である。戻り値、effect、trait requirement のどれかに private な要素が含まれていれば、その宣言は `pub` にできない。`pub import foo as bar` のような module alias re-export でも、`bar` 越しに見える contract が同じ closure rule を満たさなければならない。

```moti
private type Secret

pub fn leak(): Secret {
  ...
}
```

この宣言は不正である。読み手は、公開宣言を見たときに別ファイルの private detail を追わずとも契約を理解できなければならない。

## Reference-level explanation

- 可視性は `private`、`package`、`pub` の三段階とする。
- 宣言 `d` の effective visibility は、そのシグネチャに現れる依存宣言の可視性との meet で計算する。
- 依存宣言には型、effect alias、trait、associated item、const parameter、publicly named module path を含む。
- `pub` 宣言の effective visibility が `package` または `private` になる場合、その宣言は reject する。
- `package` 宣言も同様に、`private` 依存を package surface へ漏らしてはならない。
- module alias re-export は、再公開先の visibility と alias 先 module が露出する contract の effective visibility の両方を満たす場合にのみ許す。
- diagnostics は `declared_visibility` と `blocking_dependency` を示さなければならない。
- boundary-only type の詳細規律は別 RFC に委ねるが、この RFC では「見えてはいけない dependency があれば export できない」という closure rule を固定する。

## Drawbacks

- API refactoring 時に visibility error が増える。
- effect や projection を公開面に出す設計では closure 計算がやや重い。
- 一部の convenience re-export が明示 wrapper を要求される。

## Rationale and alternatives

- 代替案 1 は token-level visibility のみ見る案だったが、公開契約の自己完結性を壊すため採らない。
- 代替案 2 は warning に留める案だったが、safe fragment と boundary contract の説明責任を果たせないため採らない。
- 代替案 3 は package surface までは緩める案だったが、internal accidental leak を増やすため採らない。
- 何もしない場合、公開面と実際の依存面がずれ、boundary review が困難になる。

## Prior art

- Rust の effective visibility 概念は直接の参考になる。
- ML module signature の ascription は surface closure の重要性を示す。
- Moti はここに effect / projection / boundary discipline を含める点でより厳格に進む。

## Unresolved questions

- generated API doc が blocking dependency をどう表示するか。

## Future possibilities

- export closure を可視化する tooling。
- package policy lint と diagnostics の統合。
