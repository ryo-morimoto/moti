# 0014-existential-abstraction

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: `exists`、`pack`、`open`、hidden type scope、existential-safe
- Feature Name: `existential_abstraction`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は trait resolution と別制度として存在抽象を持つ。`exists`、`pack`、`open` によって hidden type を導入し、その hidden type は `open` の scope を越えて escape してはならない。公開面で `exists Trait` を許すのは existential-safe 条件を満たす trait に限定する。

## Motivation

- trait だけでは abstraction boundary を十分に表現できない。
- しかし、existential を trait solver に混ぜると解決規律が崩れる。
- hidden type scope を曖昧にすると abstraction leak が起きる。
- public API で existential を使うなら、どの trait が existential-safe かを固定する必要がある。

## Guide-level explanation

Moti では「ある trait を満たすが具体型は隠したい」ときに `exists` を使う。重要なのは、hidden type は便利な dynamic dispatch の別名ではなく、scope で閉じた抽象境界だという点である。

```moti
pack[T = Hidden] value as exists Displayable
```

読み手は、`open` した場所でだけ hidden type を知れるが、その知識を外へ持ち出せないと理解すればよい。

## Reference-level explanation

- 存在抽象は trait resolution と別制度である。
- `exists`、`pack`、`open` の正規構文を持つ。
- hidden type は `open` scope を越えて escape してはならない。
- implicit opening、typecase、downcast は導入しない。
- public surface で `exists Trait` を許すには existential-safe 条件を満たす必要がある。
- existential-safe は `Self` 露出、associated type、receiver、generic method などの条件で判定する。

## Drawbacks

- trait object 的な ergonomics を一部失う。
- hidden type scope の規則が初見では難しい。
- existential-safe の条件が保守的に見える。

## Rationale and alternatives

- 代替案 1 は trait object / dynamic dispatch 中心だったが、Moti の abstraction story とずれる。
- 代替案 2 は existential を trait core に吸収する案だったが、solver complexity を増やすため採らない。
- 代替案 3 は implicit open を許す案だったが、abstraction boundary が読めなくなるため採らない。
- 何もしない場合、trait と package boundary のあいだを埋める抽象制度が欠ける。

## Prior art

- ML 系の existential package は直接の参考になる。
- Rust の trait object と `impl Trait` は近い論点を持つが、Moti は hidden type scope をより明示的に扱う。
- dependent type 言語の package / sigma / exists の経験が参考になる。

## Unresolved questions

- existential-safe 条件をどこまで本文で閉じるか。
- associated type を含む existential の surface ergonomics。

## Future possibilities

- existential-safe trait の library-side marker。
- package boundary に特化した sugar。
