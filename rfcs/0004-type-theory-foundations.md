# 0004-type-theory-foundations

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: QTT ベース依存型、universe、proof erasure、bottom 排除
- Feature Name: `type_theory_foundations`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の型基盤は QTT ベースの依存型であり、multiplicity を `0/1/ω` で扱う。universe は cumulative で、`Type : Type` は認めない。proof は relevance を持ちながら erasure 可能であり、未定義値や暗黙の bottom を導入しない。

## Motivation

- Moti の目標には依存型と型レベル安全性の証明が含まれている。
- しかし、依存型を入れるだけでは safe fragment は守れない。どこまでを trusted kernel に入れるかを早く固定する必要がある。
- multiplicity を型基盤へ入れておくと、borrow、effects、async の後付け拡張ではなく、最初から一貫した意味論になる。
- bottom 排除は「失敗や未定義を別経路へ隠さない」という全体方針と接続する。

## Guide-level explanation

Moti の依存型は「強い型を何でも許す」ためのものではない。むしろ、どの値が proof で、どの値が runtime に残り、どの資源が 1 回だけ使えるかを、同じ型基盤で扱うための制度である。

読み手の mental model は次の通りである。

- `0` は proof など erase 可能なもの
- `1` は線形・一回使用の資源
- `ω` は通常の複数回使用可能な値

この基盤の上に borrow、effect、trait、existential を積んでいく。

## Reference-level explanation

- 型理論の基盤は Quantitative Type Theory を採る。
- multiplicity は少なくとも `0`、`1`、`ω` を持つ。
- universe は cumulative とし、`Type : Type` は認めない。
- proof relevance を保持しつつ、proof erasure を許す。
- 暗黙の bottom、`undefined`、kernel-level `panic`、`abort` を持たない。
- divergence は effect / result / explicit control によって扱い、型基盤へ無制限の bottom を入れない。

## Drawbacks

- 型基盤の理解コストが高い。
- 実装者には elaboration と erasure の境界設計が要求される。
- 単純な Hindley-Milner 系よりも surface と core の距離が広がる。

## Rationale and alternatives

- 代替案 1 は依存型なしの ML 系コアだったが、Moti の根本目標に届かない。
- 代替案 2 は依存型を持つが multiplicity を別制度にする案だったが、borrow と effect の一貫性が弱くなる。
- 代替案 3 は `Type : Type` や広い kernel escape hatch を許す案だったが、小さい trusted kernel 目標に反する。
- 何もしない場合、後段の borrow / async / proof safety が局所制度の継ぎはぎになる。

## Prior art

- QTT は multiplicity と依存型を同じ土台で扱う点で直接の参考になる。
- Lean / Idris2 は依存型言語として参照価値があるが、Moti は runtime safety と boundary discipline をより強く前面に出す。
- Rust は所有権と lifetime を強く持つが、依存型 kernel は持たない。この差が Moti の特徴になる。

## Unresolved questions

- surface syntax と elaborated core の対応をどこまで RFC 本文で規定するか。
- proof erasure と diagnostics の接続をどこまで明文化するか。

## Future possibilities

- multiplicity の追加区分。
- proof search や tactic 的補助を入れる場合の境界整理。
