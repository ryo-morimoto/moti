# 0037-send-share-and-domain-affinity

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `Send` / `Share`、domain affinity、構造導出、cross-domain transfer 制約
- Feature Name: `send_share_and_domain_affinity`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は cross-domain 並行実行の安全条件を `Send`、`Share`、domain affinity で表す。`Send` / `Share` は構造導出を基本とし、user-defined manual impl は許さない。domain-local resource、await-affine capability、boundary-only capability は cross-domain transfer できず、runtime-backed wrapper はこの制約を surface contract に反映しなければならない。

## Motivation

- structured concurrency を採っても、domain 越境の値移送規律がなければ data-race freedom を説明できない。
- manual impl を許すと boundary proof obligation が trait impl の陰に隠れる。
- use case 1 は、parallel task へ安全に値を渡せるかを型だけで読みたい user である。
- use case 2 は、runtime token や local root の domain-local 性を wrapper で保ちたい implementer である。
- no-op のままだと、`Parallel` は syntactic sugar に留まり、安全性主張が弱くなる。

## Guide-level explanation

Moti では same-domain `await` と cross-domain `Parallel` を分ける。`Parallel` に渡す値は `Send` または `Share` でなければならず、domain-local な capability は渡せない。

```moti
parallel map(xs, fn (x) => work(x))
```

この `map` は、`xs` と closure capture が cross-domain に安全であることを要求する。読み手は `Parallel` を見たら「ここから先は domain affinity rule が掛かる」と考えればよい。

## Reference-level explanation

- `Send` は ownership transfer による cross-domain 移送可能性を表す。
- `Share` は同時観測可能な shared transfer 可能性を表す。
- `Send` / `Share` は構造導出を既定とし、user-defined manual impl は許さない。
- raw pointer、`Borrow*`、`LocalRoot`、`RuntimeToken`、domain-local foreign handle、await-affine capability は `Send` / `Share` を得ない。
- immutable persistent data は `Share` を得られる。
- current profile では interior mutability facade は、別 RFC で明示的に許可されるまで `Share` を得ない。
- linear owned resource は移送時に domain affinity 条件を満たす場合のみ `Send` を得られる。
- `Parallel` spawn、cross-domain channel send、verified extern callback registration はこの規則に従う。
- diagnostics は value か capture のどこが domain-local であるかを示さなければならない。

## Drawbacks

- manual escape hatch がないため、一部 wrapper 実装が冗長になる。
- `Send` / `Share` 導出条件を教えるコストがある。
- FFI 連携では domain-local handle を包む facade 設計が強く要求される。

## Rationale and alternatives

- 代替案 1 は Rust 風の manual unsafe impl を許す案だったが、proof obligation の所在が surface から消えるため採らない。
- 代替案 2 は `Send` / `Share` を導入せず runtime に委ねる案だったが、parallel safety を型で説明できないため採らない。
- 代替案 3 は shareable / sendable を単一 trait に統合する案だったが、ownership transfer と shared aliasing を混同するため採らない。
- 何もしない場合、`Parallel` は typed boundary を持たない危険な sugar になる。

## Prior art

- Rust の `Send` / `Sync` は主要な比較対象である。
- Pony の reference capabilities は race freedom の別解を示す。
- Moti は manual impl を禁じ、proof obligation を wrapper / derivation side に残す点で意図的に保守的である。

## Unresolved questions

- foreign callback handle の標準 wrapper 形。

## Future possibilities

- capability-aware derive diagnostics。
- distributed domain profile への拡張。
