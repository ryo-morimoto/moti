# 0022-annotation-discipline-and-bidirectional-typing

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: bidirectional typing、annotation 必須位置、local inference、surface-to-core elaboration
- Feature Name: `annotation_discipline_and_bidirectional_typing`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の surface typing は bidirectional typing を採る。公開シグネチャ、recursive binding、lambda binder、effectful boundary では明示 annotation を要求し、局所式では synthesize 可能な範囲だけを推論する。これにより依存型・trait・effect を含む surface でも、algorithmic typing と elaboration の責務境界を固定する。

## Motivation

- QTT ベース依存型では、完全自動推論を広く許すとエラー局所化と elaboration の説明が壊れる。
- trait solver や dependent refinement は declaration-site contract を早く固定する必要がある。
- use case 1 は、公開 API で型・effect 契約を明示したい library author である。
- use case 2 は、局所 `let` や単純式では冗長な注釈を書きたくない application programmer である。
- no-op のままだと、どこまで inference に頼ってよいかが不透明になり、実装差や diagnostics 差が大きくなる。

## Guide-level explanation

Moti では「境界では書く、局所では推論させる」が基本である。`pub fn`、recursive `fn`、lambda 引数、公開 trait requirement などは注釈を書く。一方、単純な `let` や constructor application は期待型や項の形から推論してよい。

```moti
pub fn id[T](x: T): T {
  let y = x
  y
}
```

この例では `id` の型パラメータと引数型は明示するが、`y` の型は書かない。読み手は「public boundary と recursion point は annotation を要求される」と覚えればよい。

## Reference-level explanation

- 分類: 公開境界と recursion point で annotation を要求することは恒久不変である。
- typing は checking judgment と synthesis judgment の 2 系統を持つ。
- kernel typing judgment と universe / multiplicity / formation rule の owner は `0023` であり、本 RFC はその上に乗る algorithmic checking / synthesis と annotation discipline を扱う。
- 公開関数、package 公開関数、trait method、impl method の公開面に現れる binder は annotation を要する。
- recursive binding は parameter type と result type を明示しなければならない。
- lambda binder は annotation を要する。anonymous lambda の result type は期待型または body checking から得る。
- `let` 束縛は右辺が synthesize 可能な場合に限って annotation を省略できる。
- pattern binding は branch の期待型または scrutinee 型から checking される。
- trait obligation の候補 discovery は inference で増やさない。solver は既に得られた obligations だけを処理する。
- effect row は公開 signature では明示する。局所式では期待型から閉じた row を recover できる場合に限って省略できる。
- elaboration は surface program を annotation 補完済み core へ一方向に写す。algorithmic typing で得られない情報を後段の kernel conversion に押し付けてはならない。
- conversion check は checking judgment の一部として発生するが、その relation の owner は `0027` である。annotation 不足を conversion で埋めてはならない。
- diagnostics は `checking` 失敗と `synthesis` 失敗を区別できなければならない。

## Drawbacks

- Rust や ML よりも注釈要求が強く見える場面がある。
- anonymous lambda や local generic helper の記述量が増える。
- 実装者は elaboration と constraint solving の境界を明確に維持する必要がある。

## Rationale and alternatives

- 代替案 1 は HM 風に広い推論を許す案だったが、依存型と effect row の組み合わせで failure mode が読みにくくなるため採らない。
- 代替案 2 は surface でもほぼ全文注釈を要求する案だったが、日常利用の記述負荷が高すぎるため採らない。
- 代替案 3 は trait / effect の解決まで含めて global inference に任せる案だったが、declaration-site contract を弱めるため採らない。
- 何もしない場合、algorithmic typing は実装の都合に流れ、canonical surface の一方向 elaboration 原則が曖昧になる。

## Prior art

- Dunfield and Krishnaswami の bidirectional typing からは、説明可能な algorithmic surface を作る lesson を得る。
- Idris2、Agda、Lean からは、dependent typing で annotation discipline を境界に置く必要を学ぶ。
- OCaml / HM からは局所推論の ergonomic lesson を取るが、Moti はその推論自由度を依存型・trait・effect の全面には輸入しない。

## Unresolved questions

- `impl` 内 private helper で result type 省略をどこまで許すか。
- effect alias と annotation discipline の接続。

## Future possibilities

- IDE が補完できる annotation template の標準化。
- higher-rank polymorphism を導入する場合の additional checking mode。
