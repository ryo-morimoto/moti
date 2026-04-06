# 0045-alloc-surface-and-explicit-allocation

- 状態: `Draft`
- 日付: 2026-04-06
- 対象範囲: `alloc` surface、explicit allocator passing、allocation-visible API、`core` / `std` 境界
- Feature Name: `alloc_surface_and_explicit_allocation`
- Start Date: 2026-04-06
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、二層メモリモデルの low-level 側を公開面で安定して読めるようにするため、`core` と `std` のあいだに `alloc` surface を導入する。allocation policy を caller が選ぶ必要のある API は、raw allocator capability ではなく `alloc` owner の safe facade である `Allocator` を明示引数として受け取り、その事実を型シグネチャに残さなければならない。`core` は allocator-free に保ち、`std` は runtime-backed wrapper を提供できるが、allocator を hidden global default として導入してはならない。

## Motivation

- `0006` は二層メモリモデルを固定したが、allocator-visible な surface の owner はまだ独立していない。
- `0041` は `core` / `std` 境界を固定したが、明示 allocation をどの標準層へ置くかは reserved extension point のままである。
- use case 1 は、parser、serializer、builder の scratch memory policy を caller が選びたい library author である。
- use case 2 は、allocation が必要な API と pure / portable API を import だけで見分けたい user である。
- no-op のままだと、allocator-aware API が `std` の runtime-backed wrapper と混ざり、hidden allocation と explicit allocation の境界が曖昧になる。さらに `Allocator` が raw boundary capability なのか safe facade なのかも読めなくなる。

## Guide-level explanation

Moti では、allocation policy が意味を持つ API は `alloc` に置く。読む側は、`Allocator` 引数を見た時点で「この関数は caller-controlled allocation を行う」と理解できる。ここで見えている `Allocator` は raw allocator handle そのものではなく、boundary module が供給する safe facade である。

```moti
import alloc/vec as Vec

fn parse(alloc: Allocator, input: Bytes): Result[Ast, ParseError] {
  let scratch = Vec.with_capacity(alloc, 256)
  ...
}
```

この関数は `core` には置かれない。`core` は allocator を知らず、`alloc` は allocation-visible な低レベル surface を担う。高レベルの pure API や runtime-backed convenience API は、この境界をまたぐ場合に wrapper で責任を明示する。

```moti
fn render(xs: List[Text]): Text {
  ...
}
```

こちらは `core` 側の関数であり、allocation policy を caller が選ばない。読み手は、`Allocator` が見えるかどうかで「policy を選ぶ low-level API か」「高レベルの値 API か」を切り分けられる。

## Reference-level explanation

- 分類: allocation-visible API が allocation requirement をシグネチャへ明示することは恒久不変である。
- 分類: `core` を allocator-free に保つことは恒久不変である。
- 分類: `alloc` を `core` と `std` のあいだの標準層として置くことは現行プロファイルである。
- 分類: `Allocator` を `alloc` owner の safe facade として公開し、raw allocator capability 自体は boundary-only に留めることは現行プロファイルである。
- 分類: current profile では `Allocator` facade 自体は `Send` / `Share` を得ず、same-domain owner に留めることは現行プロファイルである。
- 分類: allocator ABI、vtable shape、concrete allocator family は reserved extension point である。
- `core` は `Allocator`、mutable buffer builder、capacity-reserving collection constructor を公開しない。
- raw allocator capability、allocator ABI token、platform-specific allocator handle は `0003` が定める boundary-only 表現に留まり、`alloc` 以外の public surface へ直接出してはならない。
- `alloc` は `Allocator`、mutable buffer、capacity-aware collection builder、allocation policy を caller が選ぶ helper を所有する。
- `Allocator` は raw allocator capability を隠した safe facade であり、allocation / free / resize policy の public contract だけを担う。backend-specific proof obligation と raw handle lifetime は boundary module 側の owner とする。
- `Allocator` 明示 API は `alloc` namespace か boundary-local / package-local API にのみ置ける。portable public API として広く読ませる high-level surface は、`Allocator` 明示をそのまま外へ漏らしてはならない。
- caller-controlled allocation を行う API は `Allocator` を明示引数として受け取らなければならない。
- `Allocator` は hidden global default、implicit import、thread-local ambient context により供給してはならない。
- `std` が allocator を hidden に使ってよいのは、runtime-owned scratch memory または wrapper-local implementation detail に留まり、caller が allocation policy、capacity growth、reuse timingを選ばない場合に限る。caller tuning が意味を持つ API は `alloc` owner とする。
- current profile では `Allocator` facade 自体は `Send` / `Share` を得ない。cross-domain allocation を許す場合は、`0037` の規律に従う別 wrapper を reserved extension として導入する。
- `alloc` surface を portable public API へどう露出できるかの最終 admission rule は `0018` の owner だが、少なくとも `Allocator` 明示 API は high-level portable public API の正規形にはならない。本 RFC はその lower-bound を定める。
- `std` は runtime-backed API の内部で allocation を使ってよいが、その場合でも `core` が allocator knowledge を持つことはない。
- page allocator / arena allocator / debug allocator の具体 family はこの RFC では固定しない。

## Drawbacks

- 標準層が `core` / `alloc` / `std` の三層になり、初学者への説明が一段増える。
- library author は allocation-visible API と high-level API を分けて設計する必要がある。
- `std` convenience API でどこまで allocator を隠してよいかに継続的な議論が生じる。

## Rationale and alternatives

- 代替案 1 は `core` / `std` の二層を維持し、allocator-aware API を `std` に置く案だったが、runtime-backed API と caller-controlled allocation を同じ層へ混ぜるため採らない。
- 代替案 2 は Zig のように allocation が必要な API すべてへ allocator を常に要求する案だったが、Moti の高レベル永続データ層まで low-level policy を持ち込むため採らない。
- 代替案 3 は hidden global allocator を標準化する案だったが、予測可能なメモリ挙動と explicitness を壊すため採らない。
- 代替案 4 は `Send` / `Share` や portable public API の規律までこの RFC で持つ案だったが、owner 境界が `0037` と `0018` にまたがるため採らない。
- 何もしない場合、二層メモリモデルの low-level 側が surface owner を持たず、`std` の convenience と low-level control が同じ語で語られてしまう。

## Prior art

- Zig の allocator discipline は「allocation requirement を型シグネチャへ出す」lesson を与える。
- Rust の `alloc` crate は `core` と `std` のあいだに別 owner を置く実務上の precedent である。
- Moti は low-level allocation control を `alloc` に閉じ込め、high-level persistent data の mental model と分離する点を重視する。

## Unresolved questions

- `alloc` に含める collection / builder の最小集合をどこまで v1 で固定するか。
- `alloc` と `std` のあいだに置く adapter helper をどこまで標準化するか。

## Future possibilities

- region / arena discipline を `Allocator` facade の上で標準化する後続 RFC。
- `alloc` / `std` / portable public API の越境 lint。
- allocation diagnostics を `0020` の structured field に接続する補助 RFC。
