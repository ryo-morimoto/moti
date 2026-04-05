# 0030-borrow-core-place-model-and-reborrow

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: canonical place、borrow conflict、reborrow、field sensitivity、lifetime core
- Feature Name: `borrow_core_place_model_and_reborrow`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti の borrow core は place-sensitive かつ flow-sensitive である。競合判定は canonical place に基づき、field projection は区別するが index / key projection の自動非衝突推論には頼らない。reborrow は明示規則で扱い、two-phase borrow と temporary lifetime extension は導入しない。

## Motivation

- borrow を「賢い alias analysis」に寄せると、読み手が rejection 理由を予測できなくなる。
- canonical place を固定しないと diagnostics と async 相互作用を安定して語れない。
- use case 1 は、同一構造体の別 field を安全に借用したい programmer である。
- use case 2 は、container index 由来の alias を保守的に拒否しても理解可能性を優先したい language designer である。
- no-op のままだと、borrow checker の強さが implementation strategy に引きずられる。

## Guide-level explanation

Moti では borrow は変数ではなく place に対して起きる。`p.x` と `p.y` は別 field なので分離できるが、`xs[i]` と `xs[j]` は index から自動では分離しない。読み手は alias analysis の巧妙さではなく、surface place の形だけを見ればよい。

```moti
let a = &mut pair.left
let b = &mut pair.right
```

これは許せる。一方、`&mut xs[i]` と `&mut xs[j]` は、`i != j` を checker が自動推論しない限りではなく、そもそも自動推論しないと決める。

## Reference-level explanation

- borrow 判定は canonical place を単位に行う。
- canonical place は local root と projection chain からなる。
- field projection は互いに区別する。
- index projection、map key projection、opaque accessor は保守的に同一 container place とみなす。
- mutable borrow と shared borrow の競合は canonical place の prefix / overlap 関係で判定する。
- reborrow は既存 borrow から短い lifetime を切り出す操作として定義する。
- reborrow 中は元 borrow の利用可能範囲を規則で制限する。
- method receiver sugar は elaboration 後の receiver place に展開して判定する。
- function / method call のための短い auto-reborrow は、explicit reborrow と同じ conflict rule を満たす場合にのみ導入してよい。
- two-phase borrow は導入しない。
- temporary lifetime extension を前提にした implicit rescue rule は導入しない。
- diagnostics は `conflicting_place` と `previous_use_span` を示さなければならない。

## Drawbacks

- 安全そうに見える index-split pattern も reject される。
- 一時束縛や explicit split API が必要になる。
- Rust より保守的に見える場面がある。

## Rationale and alternatives

- 代替案 1 は index non-overlap を積極推論する案だったが、predictability を損なうため採らない。
- 代替案 2 は two-phase borrow を入れる案だったが、mental model が複雑になるため採らない。
- 代替案 3 は linear type だけで borrow を表現し place model を持たない案だったが、日常 API との接続が弱いため採らない。
- 何もしない場合、borrow は「時々通る賢い checker」になり、Moti の低エントロピー目標に反する。

## Prior art

- Rust の place-based borrow checker は直接の先行例である。
- region / affine systems は lifetime と reborrow の考え方を与える。
- Moti は diagnostics predictability を優先し、two-phase や aggressive split inference を採らない。

## Unresolved questions

- collection-specific split API を標準ライブラリでどこまで提供するか。

## Future possibilities

- proof 付き safe split API。
- borrow graph 可視化ツール。
