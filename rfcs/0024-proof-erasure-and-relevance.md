# 0024-proof-erasure-and-relevance

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: proof relevance、erasure 境界、runtime 非観測、diagnostics との接続
- Feature Name: `proof_erasure_and_relevance`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は proof relevance を保持しつつ、`0` multiplicity の binder と proof payload を runtime では erase する。erasure は elaboration 後の core で定義され、surface convenience や optimizer 都合で変わらない。proof は型検査と仕様記述には参加するが、runtime 観測、FFI、public ABI へは持ち込まれない。

## Motivation

- Moti は依存型を持つが、proof object をそのまま runtime へ流すと成果物サイズと cost model が読めなくなる。
- proof relevance を完全に捨てると、diagnostics や specification author の reasoning が弱くなる。
- use case 1 は、証明付き API を書きたいが runtime 表現は増やしたくない library author である。
- use case 2 は、proof term を型エラー説明には使いたいが ABI 契約には載せたくない implementer である。
- no-op のままだと、proof がいつ runtime に残るかが曖昧になり、safe fragment と performance の両方で誤解を生む。

## Guide-level explanation

Moti では proof は「型で読むが、実行では運ばない」ものとして扱う。関数が proof を受け取っても、その値は runtime object ではなく elaboration 後に消える。

```moti
fn head(xs: Vec[T], p: IsNonEmpty(xs)): T {
  ...
}
```

この `p` は `head` が安全に呼べることを示すために使うが、実行時に引数として保持されることは期待しない。読み手は proof を runtime data と混同しなくてよい。

## Reference-level explanation

- 分類: proof relevance を保持しつつ runtime では erase することは恒久不変である。
- proof relevance は type checker 上では保持する。
- erasure 対象は少なくとも `0` multiplicity binder、proof-only constructor field、compile-time equality witness を含む。
- erasure は elaborated core に対する構文変換として定義する。
- erased term は runtime semantics に参加しない。
- pattern matching は erased proof field を runtime branch condition に使ってはならない。
- FFI 境界、public ABI、boundary-only metadata へ erased proof を持ち込んではならない。
- diagnostics は erased proof の source-level 存在を参照してよいが、runtime layout の説明に混ぜてはならない。
- effect row、borrow capability、`Send` / `Share` 判定は erased proof の有無ではなく surviving term に基づく。
- 実装は dead argument elimination を用いてもよいが、仕様上の erasure 境界を拡張してはならない。

## Drawbacks

- proof relevance と runtime erasure の二層説明が必要になる。
- ABI と source term の見え方が異なるため、FFI 周辺で教育コストがある。
- 実装者は diagnostics 用情報と runtime codegen 用情報を分けて持つ必要がある。

## Rationale and alternatives

- 代替案 1 は proof irrelevance を既定にする案だったが、型エラーや specification 側の説明力が落ちるため採らない。
- 代替案 2 は proof を runtime object として保持する案だったが、低コスト・小成果物目標に反するため採らない。
- 代替案 3 は optimizer 任せにして仕様で erasure を定義しない案だったが、conformance と ABI reasoning が曖昧になるため採らない。
- 何もしない場合、proof が safe fragment のための制度なのか runtime feature なのかが読めなくなる。

## Prior art

- Coq、Agda、Lean からは proof term を reasoning に使いながら runtime へ持ち込まない lesson を取る。
- Idris2 からは relevance と erasure を言語レベルで分けて説明する実践を学ぶ。
- Rust の zero-sized witness は局所例として近いが、Moti は erasure 境界を optimizer 慣習ではなく language law で固定する。

## Unresolved questions

- erased proof を source-level reflection API からどこまで隠すか。
- debugger / profiler が erased source binder をどう見せるか。

## Future possibilities

- proof-guided optimization の説明を追加する補助文書。
- extracted core viewer の標準 tooling。
