# 0039-dependent-refinement-and-case-tree-elaboration

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: dependent refinement、inaccessible pattern、absurd pattern、case tree elaboration
- Feature Name: `dependent_refinement_and_case_tree_elaboration`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は pattern match による dependent refinement を `inaccessible pattern` と `absurd pattern` で表し、surface match を refinement-aware な case tree へ一方向 elaboration する。refinement は scrutinee と constructor equality から得られる情報だけを使い、codata matching や hidden proof search を導入しない。

## Motivation

- dependent typing を持つ以上、pattern match は case split だけでなく型精密化の中心になる。
- refinement の取得方法を曖昧にすると、coverage と typing が別々の箱になる。
- use case 1 は、length-indexed data の分岐で index 情報を自然に取り出したい user である。
- use case 2 は、surface pattern を core case tree へ安定して落としたい implementer である。
- no-op のままだと、dependent pattern は「見た目では書けるが規則が曖昧」な制度になる。

## Guide-level explanation

Moti では分岐に入ると型情報が狭まる。到達不可能な場合分けは inaccessible や absurd で明示する。読み手は、`match` が branch ごとに refinement context を持ち、surface pattern が case tree node へ落ちると考えればよい。

```moti
pub fn head[T, n](xs: Vec[T, Succ[n]]): T {
  match xs {
    Nil => absurd impossible_non_empty
    Cons(head, tail) => head
  }
}
```

この例では `xs` の型が `Succ[n]` 添字を持つため、`Nil` 分岐は不可能である。Moti はその不可能性を hidden proof search ではなく branch-local refinement と case tree に残す。

## Reference-level explanation

- refinement は scrutinee の constructor choice、equality witness、index relation から得られる。
- inaccessible pattern は branch 内で決まる index / term を source に反映する記法である。
- absurd pattern は branch が空集合であることを示す elimination である。
- surface match は refinement-aware case tree へ elaboration される。
- elaboration は branch 順序、coverage、refinement hypothesis を明示した core representation を生成する。
- pipeline は少なくとも `scrutinee の constructor choice` -> `branch-local equality / index facts` -> `inaccessible / absurd admission check` -> `case tree node 生成` の順で進む。
- codata は projection-only であり、refinement のための match 対象にしてはならない。
- trait solver、effect solver、general proof search に refinement obligation を投げて hidden discharge してはならない。
- diagnostics は inaccessible / absurd の根拠不足を branch-local に示さなければならない。

## Drawbacks

- inaccessible / absurd pattern は学習コストが高い。
- case tree elaboration の説明が必要になる。
- hidden proof search を抑えるため、ユーザが明示的に書く量が増えることがある。

## Rationale and alternatives

- 代替案 1 は dependent refinement を hidden unification に寄せる案だったが、mental model が見えなくなるため採らない。
- 代替案 2 は absurd pattern を導入せず auxiliary proof term に委ねる案だったが、match と証明の結び付きが弱くなるため採らない。
- 代替案 3 は codata も refinement 対象にする案だったが、kernel safety の projection-only ルールと衝突するため採らない。
- 何もしない場合、dependent pattern matching は例だけ存在して仕様がない状態になる。

## Prior art

- Agda、Idris、Lean の inaccessible / absurd pattern は直接の先行例である。
- dependent pattern compilation 研究は case tree elaboration の機械的基盤を与える。
- Moti は refinement を hidden proof automation より surface readability で固定する。

## Unresolved questions

- equation-style syntax をどこまで許すか。
- case tree を canonical debug output へ露出するか。

## Future possibilities

- refined pattern synonym。
- teaching-oriented refinement trace。
