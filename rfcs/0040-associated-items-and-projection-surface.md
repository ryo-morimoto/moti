# 0040-associated-items-and-projection-surface

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: associated item の最小集合、projection surface、公開契約、solver 境界
- Feature Name: `associated_items_and_projection_surface`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の trait core は associated item を最小集合に保ち、v1 では associated type だけを projection surface の対象として扱う。projection は公開シグネチャに現れてよいが、declaration-site で一意に意味が定まる場合に限る。solver が projection を guess して public contract を補完することは許さない。

## Motivation

- trait core の小ささを守るには、associated item を何でも許さない必要がある。
- projection が surface に現れるなら、その意味が declaration-site で読めなければならない。
- use case 1 は、trait を通じて result type を抽象化したい library author である。
- use case 2 は、projection ambiguity を call-site ではなく declaration-site で拒否したい implementer である。
- no-op のままだと、associated type はあるがどこまで surface contract に使えるかが曖昧になる。

## Guide-level explanation

Moti では trait は ad-hoc polymorphism のための小さい制度である。associated type は使えるが、「solver が賢く補う前提」で public API を書いてはならない。associated const はこの RFC の scope 外であり、future extension として扱う。

```moti
trait Iterator[T] {
  type Item
}
```

読み手は、`Iterator[T]::Item` が public signature に現れるなら、その projection が declaration-site で閉じているはずだと期待できる。

## Reference-level explanation

- v1 の projection surface が扱う associated item は associated type に限る。
- associated method は通常 method として扱い、本 RFC の scope では projection source にしない。
- associated const は将来拡張であり、本 RFC の current decision には含めない。
- projection type は public / package signature に現れてよいが、必要 obligation が declaration-site で完結しなければならない。
- ambiguity、no-solution、invalid-program は declaration-site で reject する。
- solver は projection normalization を十分な evidence がある場合にのみ行う。
- existential-safe 判定、export closure、diagnostics contract は projection を明示 dependency として扱う。
- default associated item、specialization、implementation-local defaulting は採らない。

## Drawbacks

- Rust / Haskell より associated item の使い方が保守的になる。
- public signature で projection を使う際の annotation がやや重くなる。
- trait library 設計に discipline を要求する。

## Rationale and alternatives

- 代替案 1 は associated type を広く許し call-site resolution に委ねる案だったが、public contract を曖昧にするため採らない。
- 代替案 2 は associated item を一切禁止する案だったが、trait abstraction の表現力を落としすぎるため採らない。
- 代替案 3 は type family 的機構を直接導入する案だったが、trait core を大きくしすぎるため採らない。
- 何もしない場合、projection surface の可否が RFC ごとに場当たりで決まる。

## Prior art

- Rust の associated types は主要な比較対象である。
- Haskell の type family は power と entanglement の両方の lesson を与える。
- Moti は solver predictability を優先し、projection を declaration-site で閉じる。

## Unresolved questions

- trait alias を導入した場合の projection export rule。

## Future possibilities

- associated const を projection surface に加える後続 RFC。
- closed structural trait と projection の統合。
- projection-aware diagnostics expansion。
