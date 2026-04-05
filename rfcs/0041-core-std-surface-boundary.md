# 0041-core-std-surface-boundary

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `core` / `std` ownership、pure surface、runtime-backed API の所在、非目標の切り分け
- Feature Name: `core_std_surface_boundary`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は標準提供物の所有を `core` と `std` に分ける。`core` は pure、runtime-independent、portable な API だけを持ち、`std` は runtime-backed API を担う。surface 上で runtime capability を暗黙 import したり、`core` が hidden runtime 前提を持つことは許さない。

## Motivation

- `core` と `std` が曖昧だと、portable semantics と runtime contract が混ざる。
- implicit prelude を持たない以上、どの API が pure core に属するかを早く固定する必要がある。
- use case 1 は、runtime を変えても同じように読める API を `core` から得たい user である。
- use case 2 は、runtime-backed feature の責務を `std` に閉じ込めたい runtime implementer である。
- no-op のままだと、runtime-dependent feature が pure library の顔をして surface に紛れ込む。

## Guide-level explanation

Moti では `core` は「どの runtime でも同じ意味で読めるもの」、`std` は「runtime の助けが要るもの」である。`core` から timer や scheduler を期待してはならない。

```moti
import core.result as result
import std.fs as fs
```

読み手は、`core` import を見たら pure / portable contract を期待し、`std` import を見たら runtime-backed obligation があると考えればよい。

## Reference-level explanation

- `core` は pure、runtime-independent、portable の三条件を満たす API だけを含む。
- `std` は I/O、scheduler bridge、timer、runtime capability を要する API を含む。
- `core` は hidden runtime token や implicit driver を要求してはならない。
- `std` は runtime-backed API を表面で明示し、必要な effect / result / boundary contract を持たなければならない。
- `core` と `std` の区別は package namespace と documentation owner の両方に反映する。
- `core` 向け test support は scope 内未解決であり、`alloc` 相当の中間層は reserved extension point である。
- optional library は `std` と区別し、標準 runtime contract の一部として扱わない。

## Drawbacks

- `core` に何を入れないかで継続的な圧力がかかる。
- `std` wrapper が厚くなることがある。
- runtime 非依存 abstraction を教えるコストがある。

## Rationale and alternatives

- 代替案 1 は `core` / `std` を分けない案だったが、portable semantics が濁るため採らない。
- 代替案 2 は広い prelude で差を隠す案だったが、Moti の explicitness と衝突するため採らない。
- 代替案 3 はすべて library convention に任せる案だったが、surface boundary として弱すぎるため採らない。
- 何もしない場合、`core` という名前だけが残り、実質的には runtime-facing package になる。

## Prior art

- Rust の `core` / `std` 分離は主要な参考になる。
- Go や Deno は runtime 前提が強い言語の contrast を与える。
- Moti は runtime contract を別 RFC に分け、surface boundary 自体をここで固定する。

## Unresolved questions

- `core` 向け test support をどこまで許すか。

## Future possibilities

- `alloc` 相当の中間層を導入する後続 RFC。
- `core` / `std` / profile を跨ぐ API ownership lint。
- no-std profile の追加文書。
