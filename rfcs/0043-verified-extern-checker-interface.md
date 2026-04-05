# 0043-verified-extern-checker-interface

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `verified extern` checker interface、判定語彙、attach 単位、invalidation protocol
- Feature Name: `verified_extern_checker_interface`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は `verified extern` の concrete proof format を規定せず、abstract checker interface だけを規定する。証跡 attach 単位は verified extern 宣言ごととし、checker result は少なくとも `verified`、`rejected`、`inconclusive` と、capability safety、memory safety、panic / cancellation safety、scheduler / domain affinity の各判定欄を返さなければならない。checker invalidation は declaration 単位で追跡する。

## Motivation

- verified boundary の価値は proof format そのものではなく、何を検証したかを language contract として読めることにある。
- attach 単位が粗いと diagnostics と invalidation の粒度が悪くなる。
- use case 1 は、C ABI wrapper ごとに証跡を管理したい boundary author である。
- use case 2 は、different backend verifier を使っても同じ language-level disposition を得たい implementer である。
- no-op のままだと、`verified extern` は marketing label であり、checker と language law の接続がない。

## Guide-level explanation

`verified extern` は proof format の標準化ではない。言語が知るのは「この宣言に対して checker が何を verified / rejected / inconclusive と判定したか」である。読み手は raw proof artifact を追わず、checker result の語彙だけを信じればよい。

```moti
verified extern fn raw_hash(...): Digest
```

この宣言には 1 つの checker result が結び付く。別の宣言の検証結果とまとめて一括で安全だとするのではなく、宣言単位で責任を持つ。

```moti
verified extern fn raw_probe(...): ProbeResult
```

この宣言に `inconclusive` が返った場合、boundary-private のまま検討や再検証はできるが、package / public surface へ安全な facade として昇格はできない。

## Reference-level explanation

- checker interface は abstract contract として定義し、proof format や prover 実装は規定しない。
- attach 単位は verified extern 宣言ごとである。
- checker result は少なくとも `verified`、`rejected`、`inconclusive` の disposition を持つ。
- checker result は capability safety、memory safety、panic / cancellation safety、scheduler / domain affinity の判定欄を持つ。
- disposition ごとの admission は次に従う。
  - `verified`: wrapper obligations を別 RFC に従って満たす候補になれる
  - `rejected`: 当該 verified extern 宣言は compile error とし、呼び出し可能宣言として成立しない
  - `inconclusive`: boundary-private declaration としては残せるが、package / public surface へ昇格してはならない
- invalidation は declaration 依存グラフに基づき、対象宣言ごとに起きる。
- checker result は boundary-only metadata であり、public surface に露出してはならない。
- diagnostics contract は checker result から `verification_phase`、`disposition`、`failed_checks` を導出できなければならない。
- checker result は kernel conversion を拡張してはならない。

## Drawbacks

- checker interface 設計に抽象度の難しさがある。
- verifier 実装者に declaration-level invalidation を要求する。
- `inconclusive` の扱いが利用者に曖昧に見える可能性がある。

## Rationale and alternatives

- 代替案 1 は concrete proof format まで仕様に含める案だったが、backend 依存を固定しすぎるため採らない。
- 代替案 2 は module pack 単位で attach する案だったが、invalidation と diagnostics が粗くなるため採らない。
- 代替案 3 は checker result を単なる bool にする案だったが、どの safety axis が欠けたか読めないため採らない。
- 何もしない場合、verified extern は runtime trust boundary を説明する制度になれない。

## Prior art

- proof-carrying code と verified boundary の発想は checker/language 分離の lesson を与える。
- Rust の unsafe FFI は proof ownership を言語外に置く contrast を提供する。
- Moti は抽象 interface に留めつつ、判定語彙だけは language law に固定する。

## Unresolved questions

- checker result schema versioning をどこで導入するか。
- incremental build と invalidation protocol をどこまで normative に書くか。

## Future possibilities

- backend-specific verifier profile。
- checker result cache の標準 metadata。
