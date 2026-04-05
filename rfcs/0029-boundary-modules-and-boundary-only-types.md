# 0029-boundary-modules-and-boundary-only-types

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: boundary module、boundary-only type、raw contract 隔離、safe wrapper only
- Feature Name: `boundary_modules_and_boundary_only_types`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は FFI や runtime-backed trust boundary を `boundary module` に閉じ込める。boundary module 内では raw `extern`、foreign handle、runtime token、verification metadata などの boundary-only type を扱えるが、それらは public / package surface に出してはならない。公開できるのは safe wrapper と、その wrapper が約束する型付き契約だけである。

## Motivation

- Moti の根本目標は safe fragment と trust boundary の明示である。
- raw contract と wrapper contract を同じ層に置くと、利用者に proof obligation を押し付ける。
- use case 1 は、POSIX や C ABI を包みたい boundary author である。
- use case 2 は、runtime token や raw handle を public API に漏らさず facade だけを見せたい package maintainer である。
- no-op のままだと、unsafe responsibility の所在が module 階層で読めなくなる。

## Guide-level explanation

boundary module は「ここから先は信頼境界だ」と明示する owner である。raw FFI 宣言や runtime capability はこの中に閉じ込め、外には `Result` と safe type だけを出す。

```moti
boundary module posix_file {
  extern fn raw_open(...): RawFd

  pub fn open(path: Path): Result[File, OpenError] {
    ...
  }
}
```

読み手は `open` を見ればよく、`RawFd` や checker metadata を追いかける必要はない。boundary module は unsafe implementation detail の owner であって、raw API を広く再輸出する場所ではない。

## Reference-level explanation

- boundary module は FFI、runtime primitive、verified boundary を所有する専用 module owner である。
- raw `extern fn`、foreign handle、raw pointer、runtime token、local root、verification metadata は boundary-only type とする。
- boundary-only type は `private` owner に閉じ、public / package signature に現れてはならない。
- boundary module の public surface は safe wrapper のみを出す。
- safe wrapper は error normalization、ownership precondition、domain affinity、effect surface を型付き契約で示さなければならない。
- boundary-only type の alias や transparent newtype による偽装公開を禁止する。
- re-export によって boundary-only item を package facade へ持ち出してはならない。
- verified extern と checker result の詳細は後続 RFC に委ねるが、ここでは ownership と visibility の原則を固定する。

## Drawbacks

- wrapper 実装量が増える。
- raw escape hatch が欲しい implementer には窮屈に見える。
- boundary module という新しい owner 概念を教える必要がある。

## Rationale and alternatives

- 代替案 1 は `unsafe` public API を許す案だったが、safe fragment の説明責任を利用者へ押し付けるため採らない。
- 代替案 2 は boundary module を置かず module 慣習に任せる案だったが、ownership が曖昧になるため採らない。
- 代替案 3 は package surface までは boundary-only type を許す案だったが、accidental misuse を増やすため採らない。
- 何もしない場合、Moti の差別化点である trust boundary discipline が名前だけになる。

## Prior art

- Rust の safe wrapper over unsafe FFI は重要な先行実践である。
- capability-based design は権限を surface へどう出さないかの lesson を与える。
- Moti は `boundary module` を制度化して owner を明示する点で、慣習より強い規律を採る。

## Unresolved questions

- `boundary module` の宣言構文を keyword にするか attribute にするか。
- generated docs で boundary owner をどう表示するか。

## Future possibilities

- boundary audit tooling。
- wrapper skeleton 自動生成。
