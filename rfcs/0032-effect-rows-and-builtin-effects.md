# 0032-effect-rows-and-builtin-effects

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: closed effect row、built-in effect 集合、row 正規形、公開 surface 規律
- Feature Name: `effect_rows_and_builtin_effects`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の effect surface は closed effect row のみを露出する。built-in effect の最小集合は `IO`、`Diverge`、`Async`、`Parallel`、`State[S]`、`Exn[E]` とし、row は順序非依存・重複なしの正規形を持つ。さらに built-in effect を handleable effect と latent tracked effect に分類し、user-level handler が消去できる範囲をここで固定する。

## Motivation

- Moti は effect を explicit に追跡したいが、row variable をそのまま surface へ出すと mental model が重くなる。
- async、error、state、divergence を別制度にしすぎると hidden control flow が増える。
- use case 1 は、関数シグネチャだけで副作用面を把握したい library user である。
- use case 2 は、effect solver の内部変数を public API に漏らしたくない language designer である。
- no-op のままだと、effect は存在するが surface contract として読めない制度になる。

## Guide-level explanation

Moti では関数が何をし得るかは `/ { ... }` で見る。順番に意味はなく、同じ effect を二度書かない。読む側は「この関数は `IO` と `Exn[ParseError]` を起こしうる」と素直に理解すればよい。

```moti
fn parse(path: Path): Ast / {IO, Exn[ParseError]}
```

このシグネチャは、`parse` が pure ではなく、I/O と parse 失敗の内部制御を持ちうることを示す。internal inference detail を知る必要はない。

## Reference-level explanation

- source で露出する effect row は closed row のみとする。
- built-in effect の基礎集合は `IO`、`Diverge`、`Async`、`Parallel`、`State[S]`、`Exn[E]` である。
- current profile では `State[S]`、`Exn[E]`、`Async` を handleable effect とする。
- current profile では `IO`、`Diverge`、`Parallel` を latent tracked effect とし、通常の user-level handler はこれらを除去しない。
- row equality は順序非依存・重複なしの正規形で判定する。
- surface syntax は `/ E` と `/ {E1, E2, ...}` を許すが、正規形では singleton を含め row 集合として扱う。
- pure computation は空 row を持つ。
- effect row は公開 signature、trait method、handler clause、boundary wrapper obligation の一部として現れうる。
- internal solver variable や open row variable は public surface に露出しない。effect abstraction の詳細は `0034` を正本とする。
- effect row は typed error surface、borrow、async、boundary contract と相互作用するが、それぞれの追加制約をこの RFC 単体で吸収しない。

## Drawbacks

- row polymorphism を期待する利用者には保守的に見える。
- built-in effect 集合を増やす議論が将来起きやすい。
- `Exn` と `Result` の違いを教える必要がある。

## Rationale and alternatives

- 代替案 1 は open row variable を surface に出す案だったが、読み手の mental model を重くするため採らない。
- 代替案 2 は state / async / divergence を別制度へ分ける案だったが、effect surface の統一性が崩れるため採らない。
- 代替案 3 は exception だけ special-case する案だったが、typed control flow の一貫性を弱めるため採らない。
- 何もしない場合、effect system はあるが API contract としては読めないままになる。

## Prior art

- Koka は effect-first surface の重要な先行例である。
- algebraic effect 系研究は row reasoning の基盤を与える。
- Moti は表現力より public readability を優先し、open row surface を抑制する。

## Unresolved questions

- `Parallel` と `Async` の row ergonomics を surface sugar でどう補うか。
- built-in effect 名称を将来変更せずに保てるか。

## Future possibilities

- handleable / latent 分類を壊さない新 built-in effect family。
- row normalization を示す formatter support。
