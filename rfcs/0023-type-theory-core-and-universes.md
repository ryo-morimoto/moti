# 0023-type-theory-core-and-universes

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: QTT コア、multiplicity、cumulative universes、`Type : Type` 不採用
- Feature Name: `type_theory_core_and_universes`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の型理論コアは Quantitative Type Theory に基づき、binder usage を `0`、`1`、`ω` の multiplicity で追跡する。universe は cumulative とし、`Type : Type` は認めない。これにより、proof、linear resource、通常値を同じ基盤で扱いながら、小さい trusted kernel の前提を守る。

## Motivation

- Moti は依存型、effect 追跡、borrow、safe fragment を同じ言語内で整合させたい。
- multiplicity を外付け制度にすると、proof erasure と resource discipline の境界が局所ルール化しやすい。
- use case 1 は、proof を runtime から分離しつつ型で追跡したい specification author である。
- use case 2 は、linear resource を依存型と別制度にせず一つの型基盤で扱いたい language designer である。
- no-op のままだと、borrow、proof、effect が別々の基盤に分裂し、Moti の一貫性目標に反する。

## Guide-level explanation

Moti では binder は「何回使ってよいか」を型理論の一部として持つ。`0` は proof のように runtime に残らない用途、`1` は一回使用の資源、`ω` は通常の値である。universe は段階を持ち、`Type` が自分自身に含まれることはない。

```moti
fn keep_once(x: OwnedBuf): OwnedBuf {
  x
}
```

この関数の mental model は、`OwnedBuf` が `1` 側の資源であり、複製ではなく移送されるというものである。読み手は multiplicity を borrow や proof のための後付け属性ではなく、binder の基本性質として捉える。

## Reference-level explanation

- 分類: QTT ベース core、`0/1/ω` multiplicity、cumulative universe、`Type : Type` 不採用は恒久不変である。
- core language は dependent function、dependent pair、inductive family、universe hierarchy を持つ。
- kernel typing judgment、formation judgment、universe judgment、multiplicity context judgment の owner はこの RFC とする。
- 各 binder は multiplicity を持ち、少なくとも `0`、`1`、`ω` を区別する。
- `0` binder は type checking と proof construction では利用できるが、runtime observation の主体にはならない。
- `1` binder は linear use を要求し、elaboration 後の core で usage check の対象になる。
- `ω` binder は unrestricted use を許す。
- universe は `Type0`, `Type1`, ... の cumulative hierarchy として扱う。
- `Type i : Type (i + 1)` は許すが、`Type : Type` は許さない。
- core judgment は multiplicity context を含む typing judgment を前提とする。
- algorithmic bidirectional typing と annotation discipline は `0022` の owner である。
- conversion relation の閉包範囲は `0027` の owner である。
- effect、borrow、trait は kernel formation rule そのものを増やさず、必要な部分で multiplicity と core type formation に接続する。
- kernel は implementation convenience のために別 universe shortcut を導入してはならない。

## Drawbacks

- 単純型や HM に比べて教えるコストが高い。
- multiplicity を型理論へ入れるため、surface と core の距離が開く。
- universe hierarchy を持つことで、初心者向けの単純さは失われる。

## Rationale and alternatives

- 代替案 1 は依存型なしの ML 系コアだったが、Moti の型レベル安全性目標に届かないため採らない。
- 代替案 2 は依存型は持つが multiplicity を borrow 専用制度へ分離する案だったが、proof / resource 一貫性が弱くなるため採らない。
- 代替案 3 は `Type : Type` を許す単純 kernel を採る案だったが、小さい trusted kernel を壊すため採らない。
- 何もしない場合、型理論コアが各 RFC の都合で拡張され、safe fragment の説明責任が崩れる。

## Prior art

- QTT からは multiplicity と依存型を一つの基盤で扱う lesson を取る。
- Idris2、Agda、Lean からは universe discipline と small kernel の必要を学ぶ。
- Rust は resource discipline の lesson を与えるが、Moti は ownership-only には留まらず dependent kernel へ統合する。

## Unresolved questions

- multiplicity subtyping あるいは coercion をどこまで許すか。
- universe level 表記の surface ergonomics をどこまで sugar に任せるか。

## Future possibilities

- `ω` より細かい quantitative class の追加。
- cost-aware typing を universe / multiplicity と接続する拡張。
