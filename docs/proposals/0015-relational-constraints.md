# 0015-relational-constraints

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: trait core 外の関係制約層、solver 境界、export surface との接続
- Feature Name: `relational_constraints`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、型間関係の表現力を上げるための relational constraints を trait core とは別の declaration-site constraint 層として定義する。constraint は明示 clause としてのみ現れ、trait impl 候補の discovery には直接参加しない。trait solver の coherence-first 性、有限性、決定性、import-independent resolution を壊さない接続面だけを許す。

## Motivation

- 型間関係を表したい圧力は確実に存在する。
- しかし、その圧力を trait core にそのまま流し込むと solver regime が壊れる。
- trait core を小さく保ったまま拡張口を予約しておくことで、将来の表現力向上と現在の安定性を両立したい。
- mental model 上も「今の trait が何をしないか」を明示する必要がある。
- use case 1 は、型 A と B の関係が成り立つときだけ API を開きたい library designer である。
- use case 2 は、trait core の一意解性を壊さずに richer relation を後から導入したい language designer である。

## Guide-level explanation

この RFC は「いま関係制約を全面導入する」ためのものではない。むしろ、Moti が trait core を守るために、どこまでを後続の別制度へ送るかを決める RFC である。

読み手は次のように理解すればよい。

- trait core は一意解の静的制約解決
- relational constraints は declaration-site で明示する別層
- trait solver は relation から impl 候補を発見しない

## Reference-level explanation

- relational constraints は trait core と別制度である。
- relational constraints は declaration-site の明示 clause としてのみ現れ、trait impl 候補の discovery には直接参加しない。
- 最小 constraint 形式は `rel(R, A, B)` とし、`R` は relation symbol、`A` と `B` は型引数である。
- 最小 judgment 形は `Γ ⊢ rel(R, A, B) ✓` または structured rejection とする。
- relational checker は yes/no evidence か structured rejection を返し、trait solver へは candidate ではなく前提済み fact としてのみ渡る。
- trait solver へ渡してよい fact は、trait impl 候補を増やさず、associated type equation を新設せず、declaration-site で ambiguity / no-solution / invalid-program を判定可能なものに限る。
- import によって constraint solution が変わる設計は採らない。
- export surface に現れる relational constraints は declaration-site で ambiguity / no-solution / invalid-program を判定可能でなければならない。
- 本 RFC は最小 formal model と admissibility 条件を固定し、豊富な構文 sugar は後続 RFC に委ねる。

## Drawbacks

- 今すぐの表現力向上にはつながらない。
- 「できないこと」の RFC に見えやすい。
- 将来拡張のために別の mental model を予約する必要がある。

## Rationale and alternatives

- 代替案 1 は trait core へ直接統合する案だったが、solver complexity と ambiguity を悪化させるため採らない。
- 代替案 2 は関係制約を全面的に禁じる案だったが、将来の表現力拡張口を閉じすぎる。
- 代替案 3 は library-level encoding へ委ねる案だったが、境界条件の設計判断を先送りしすぎる。
- 何もしない場合、将来の拡張議論が毎回 trait core を壊してよいかからやり直しになる。

## Prior art

- Haskell の型族や constraint system は powerful だが、Moti が避けたい entanglement も示す。
- GADT / associated type / logical relation 系の仕組みは比較対象になる。

## Unresolved questions

- relation symbol の宣言構文を独立構文にするか attribute にするか。
- export / visibility / diagnostics の wording をどこまで厳密にするか。

## Future possibilities

- trait core を壊さない relational layer の導入。
- theorem-proving friendly な constraint language への橋渡し。
