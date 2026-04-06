# 0006-memory-model

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: 二層メモリモデル、低レベル資源管理、高レベル永続データ
- Feature Name: `memory_model`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、通常のプログラミングを支える高レベル永続データ層と、allocator・バッファ・FFI・低レベル I/O を支える明示的資源層を分ける二層メモリモデルを採用する。大半のコードは高レベル層で書き、低レベル層は trust boundary と capability discipline の下に局所化する。

## Motivation

- Moti の要件には予測可能なメモリ挙動と小さな成果物がある。
- すべてを GC 的高レベル層で扱うと cost model が見えにくい。
- 逆に、すべてを低レベル所有権へ落とすと日常プログラミングの認知負荷が高すぎる。
- FFI や runtime-backed API では低レベル資源管理が避けられないため、層を分けて責務を明確にする必要がある。

## Guide-level explanation

普通の Moti プログラムでは、高レベルの永続データを使う。内部で RC や reuse analysis が使われても、公開される mental model は「予測可能で安全な値」である。一方、バッファや FFI root のような資源は、低レベル層で明示的に扱う。

```moti
pub fn parse(input: Bytes): Result[Ast, ParseError] {
  ...
}
```

読み手は、この `Bytes` が public surface で安全に使える abstraction か、それとも boundary-only resource かを区別できなければならない。その区別を言語側が曖昧にしない。

## Reference-level explanation

- メモリモデルは高レベル層と低レベル層の二層を持つ。
- 高レベル層は永続データと予測可能なメモリ挙動を提供する。
- 低レベル層は allocator、mutable buffer、FFI root、runtime 資源を扱う。
- 低レベル層の値は boundary discipline と capability discipline の下で制御する。
- high-level / low-level 間の変換は明示的に行う。
- portable public API は低レベル表現をそのまま露出しない。

## Drawbacks

- 層の境界設計に慎重さが要る。
- 高レベル層の抽象と低レベル層の実コストの対応を文書化しないと誤解が生じる。
- 実装者にとっては optimizer と runtime の責務分割が難しい。

## Rationale and alternatives

- 代替案 1 は単一の低レベル ownership model に寄せる案だったが、日常利用の認知負荷が高い。
- 代替案 2 は高レベル層だけで押し通す案だったが、FFI と runtime の明示境界が弱い。
- 代替案 3 は low-level details を public API へ漏らす案だったが、portable public surface 目標に反する。
- 何もしない場合、memory safety の説明と performance の説明が別々の局所ルールへ分解される。

## Prior art

- Rust は低レベル ownership を前面に出す。
- Go や GC 言語は高レベル体験を優先するが、cost model の透明性は弱くなりやすい。
- Moti は両者の中間ではなく、「層を分けて explicit にする」方向を採る。

## Unresolved questions

- 高レベル層の reuse analysis を仕様へどこまで書くか。
- `alloc` surface と low-level buffer abstraction の最小集合をどこで固定するか。これは後続 RFC `0045` の主対象である。

## Future possibilities

- allocator policy と `alloc` surface を後続 RFC `0045` で切る。
- 高レベル / 低レベル変換の audit tooling。
