# 0025-divergence-totality-and-recursion

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: totality、一般再帰、`Diverge` effect、bottom 排除、partial computation の公開面
- Feature Name: `divergence_totality_and_recursion`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は暗黙の bottom を持たず、total fragment と divergence-capable fragment を区別する。structural recursion と guarded corecursion は total 側に属し、一般再帰や無限ループは `Diverge` effect を持つ computation としてのみ許す。公開面では divergence を hidden failure として扱わず、effect か boundary contract に痕跡を残す。

## Motivation

- `panic` や hidden bottom を排除しても、一般再帰の扱いを決めなければ partial behavior は別名で戻ってくる。
- total kernel と runtime-capable program の両立には、divergence を型理論コアから分離する必要がある。
- use case 1 は、証明や safe fragment では termination 前提を保ちたい spec author である。
- use case 2 は、server loop や retry loop を書きたい runtime programmer である。
- no-op のままだと、どこまでが total reasoning の対象か、どこからが partial computation かを利用者が判別できない。

## Guide-level explanation

Moti では「止まるべきもの」と「止まらないかもしれないもの」を同じ顔で書かない。一般再帰や無限 loop は `Diverge` を持つ計算として扱う。逆に、proof や total helper は `Diverge` を持たない。

```moti
fn spin_forever(): Never / {Diverge} {
  spin_forever()
}
```

この関数は hidden bottom ではなく、明示的に divergence-capable だと読める。読み手は「total か partial か」を型と effect から追える。

## Reference-level explanation

- 分類: 暗黙の bottom を持たないことは恒久不変である。
- 分類: 一般再帰を `Diverge` effect に閉じ込めることは恒久不変である。
- 暗黙の bottom、`undefined`、kernel-level `panic`、unchecked `abort` は持たない。
- total fragment は structural recursion、well-founded recursion、guarded corecursion のみを許す。
- 一般再帰は `Diverge` effect を持つ computation に限って許す。
- `Diverge` は effect row の一員であり、公開面に現れてよい。
- `Exn`、`Result`、`Diverge` は別制度である。divergence を `Result` へ自動正規化してはならない。
- totality check は annotation discipline と連携し、recursive definition で total / partial のどちらを目指すかを判定できなければならない。
- `Diverge` を持たない関数 body に一般再帰が現れた場合は type error とする。
- transparent であっても `Diverge` を持つ定義や partial 定義は kernel conversion に参加しない。conversion の owner は `0027` である。
- FFI や runtime-backed API が divergence を起こしうる場合、それは wrapper effect または dedicated contract に反映しなければならない。
- optimizer は total fragment を前提にした rewrite を `Diverge` computation へ流用してはならない。

## Drawbacks

- 一般再帰に effect 注釈が必要なため、慣れた言語より記述量が増える。
- totality checker の実装コストがある。
- server / runtime コードでは `Diverge` が頻出し、surface がやや重く見える。

## Rationale and alternatives

- 代替案 1 は一般再帰を無条件に許す案だったが、dependent reasoning と safe fragment を弱めるため採らない。
- 代替案 2 は total language に徹し一般再帰を禁止する案だったが、現実的な runtime / systems use case を切り捨てるため採らない。
- 代替案 3 は divergence を exception 扱いする案だったが、typed error surface と partiality を混同するため採らない。
- 何もしない場合、bottom 排除の約束が surface convenience によって静かに破られる。

## Prior art

- Coq、Agda、Lean からは total core を守る discipline の lesson を取る。
- Haskell や ML は implicit bottom の cost を示す contrast として参照する。
- Koka からは partiality を effect として露出する方向を学ぶが、Moti は safe fragment との境界をより強く前面に出す。

## Unresolved questions

- `while` 相当の surface sugar を導入する場合の `Diverge` 付与規則。
- totality checker の accepted proof technique をどこまで norm 化するか。

## Future possibilities

- cost-aware totality と termination metric の補助文法。
- `Diverge` を持つ API 向けの cancellation-friendly library guidance。
