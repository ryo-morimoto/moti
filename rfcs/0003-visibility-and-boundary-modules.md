# 0003-visibility-and-boundary-modules

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: 可視性、effective visibility、package 境界、boundary module
- Feature Name: `visibility_and_boundary_modules`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は `private`、`package`、`pub` の三段階可視性を持ち、公開境界と trust boundary の総論をここで固定する。effective visibility が参照する型・effect・capability より広くならないこと、boundary module では raw contract を private に閉じ込めて safe wrapper だけを公開することを大方針とする。詳細な export closure は `0028`、boundary-only type と boundary module discipline の詳細は `0029` と `0044` で精密化する。

## Motivation

- 安全境界を型で隔離するには、公開面へ出せるものを明示的に制御する必要がある。
- raw contract や boundary-only capability が public surface へ漏れると、safe fragment の主張が崩れる。
- import と名前解決の明示性だけでは足りず、何が export 可能かも同じ規律で制御しなければならない。
- portable public API discipline と verified extern を支える土台が必要である。

## Guide-level explanation

Moti では「見せてよいものだけが見える」。`pub` は単なる公開修飾子ではなく、その宣言が参照する型や effect まで含めて公開可能でなければ成立しない。

```moti
boundary module posix_file {
  extern fn raw_open(...): RawFd

  pub fn open(path: Path): Result[File, OpenError] {
    ...
  }
}
```

読み手は、boundary module の外に見えているものだけを信じればよい。raw FFI 宣言や boundary-only capability を追いかけなくても、公開 API の責務を理解できることが目標である。

## Reference-level explanation

- 可視性レベルは `private`、`package`、`pub` の三段階とする。
- effective visibility は、シグネチャに現れる型、effect、trait requirement、associated type、boundary capability の可視性より広くなれない。
- 再公開は明示的に行う。
- boundary module は FFI や runtime 権限の trust boundary を閉じ込める専用の owner である。
- boundary module 内の `extern fn` は private primitive とする。
- raw pointer、foreign handle、`Borrow*`、`LocalRoot`、`RuntimeToken` など boundary-only 表現は public/package surface に出してはならない。
- checker result、evidence handle、verification metadata も boundary-only とし、public/package surface に出してはならない。
- 公開できるのは safe wrapper と、その wrapper が約束する型付き契約のみである。
- effective visibility の閉包規則は `0028` を正本とする。
- boundary-only type の分類と wrapper-only discipline の詳細は `0029` と `0044` を正本とする。

## Drawbacks

- API surface の設計が厳格になるため、短期的には wrapper 記述量が増える。
- 実装者には effective visibility の理解が要求される。
- 境界に置きたい便利な raw escape hatch を公開しにくい。

## Rationale and alternatives

- 代替案 1 は Rust 風の可視性中心で、boundary discipline は慣習に任せる案だったが、Moti の trust boundary 目標には弱い。
- 代替案 2 は boundary module を導入せず、`unsafe` API を public に出す案だったが、safe wrapper only の原則に反するため採らない。
- 代替案 3 は raw capability を package surface までは出せる案だったが、package 内の accidental misuse を増やすため採らない。
- 何もしない場合、FFI 境界と公開 API の責務が混ざり、mental model が不安定になる。

## Prior art

- Rust の `pub(crate)` や module privacy は参考になるが、Moti はさらに boundary module を制度として強く位置付ける。
- capability-based design は「権限の露出を surface で管理する」点で参考になる。
- Deno や安全な FFI wrapper 設計の実践は、raw binding と safe facade を分ける点で近い。

## Unresolved questions

- boundary module の宣言構文を専用 keyword にするか、annotation にするか。
- effective visibility の diagnostics をどこまで規範化するか。

## Future possibilities

- package-private boundary audit tooling。
- boundary module を対象にした自動 wrapper 生成や checker 連携。
