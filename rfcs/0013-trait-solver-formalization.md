# 0013-trait-solver-formalization

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: solver judgment、ambiguity、projection、failure mode、termination
- Feature Name: `trait_solver_formalization`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の trait solver は、有限・決定的・import-independent でなければならない。solver は明示的な judgment を持ち、ambiguity を declaration-site で拒否し、projection normalization は十分な evidence がある場合にだけ行う。backtracking に依存する open-ended search は採らない。

## Motivation

- trait core の不変条件を守るには、solver 自体の failure mode を formal に書かなければならない。
- ambiguity を call-site まで遅らせると、public API と diagnostics の両方が不透明になる。
- projection や recursive obligation の扱いを曖昧にしたままでは、existential や relational constraints の接続面を設計できない。
- import-independent resolution は solver judgment が固定されて初めて守られる。

## Guide-level explanation

読み手は、trait constraint を見たときに「解決される」「曖昧なので拒否される」「規則上解決不能なので拒否される」のどれかを予測できる必要がある。Moti の solver は賢く guess してくれる箱ではなく、失敗理由が制度上説明可能な checker である。

## Reference-level explanation

- solver は明示的な judgment を持つ。
- 候補集合は package graph と宣言済み impl から有限に決まる。
- ambiguity は declaration-site で reject する。
- projection normalization は十分な evidence がある場合にだけ行う。
- recursive obligation は termination 条件に反すると reject する。
- import による candidate 増減や local defaulting は導入しない。
- failure mode は success / no-solution / ambiguity / invalid-program を区別する。

## Drawbacks

- solver は保守的になり、一部の書けそうなプログラムを reject する。
- declaration-site reject は早すぎる制約に見えることがある。
- formalization の記述量が多い。

## Rationale and alternatives

- 代替案 1 は backtracking を許す柔軟 solver だったが、predictability を損なうため採らない。
- 代替案 2 は ambiguity を call-site まで遅らせる案だったが、public signature の contract を曖昧にするため採らない。
- 代替案 3 は projection を広く normalize する案だったが、unsoundness と hidden assumptions を生みやすいため採らない。
- 何もしない場合、trait core は原理だけあって operational meaning がないまま残る。

## Prior art

- Rust の trait solver と coherence discipline は主要な比較対象である。
- GHC の typeclass / type family solver は表現力と複雑性の両方の教訓を与える。
- logic programming 的な open search は Moti が避けたい方向を明確にしてくれる。

## Unresolved questions

- judgment の presentation をどこまで inference-rule で書くか。
- declaration-site check と diagnostics contract の接続。

## Future possibilities

- relational constraints 層との橋渡し。
- solver 実装の多様性を残しつつ behavior contract を固定する追加文書。
