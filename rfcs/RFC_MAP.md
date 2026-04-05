# RFC Map

矢印は、**先に読むと理解しやすい依存関係** と **参照してから読むべき関係** を表します。

## 読順の骨格

```mermaid
flowchart TD
  R0001[0001 正規形]
  R0002[0002 module/import]
  R0003[0003 公開境界]
  R0016[0016 core/std]
  R0017[0017 数値 portability]
  R0018[0018 public API]
  R0004[0004 型理論基盤]
  R0005[0005 kernel safety]
  R0011[0011 pattern/refinement]
  R0006[0006 memory model]
  R0007[0007 borrow/suspend]
  R0008[0008 effects/handlers]
  R0010[0010 error model]
  R0009[0009 structured concurrency]
  R0012[0012 trait core]
  R0013[0013 trait solver]
  R0014[0014 existential]
  R0015[0015 relational constraints]
  R0019[0019 verified extern]
  R0020[0020 diagnostics]

  R0001 --> R0002 --> R0003 --> R0016 --> R0017 --> R0018 --> R0019 --> R0020
  R0004 --> R0005 --> R0011
  R0006 --> R0007 --> R0009
  R0008 --> R0010 --> R0009
  R0012 --> R0013 --> R0014
  R0013 --> R0015
```

## 相互参照の強いリンク

```mermaid
flowchart LR
  R0001[0001] --> R0008[0008]
  R0002[0002] --> R0012[0012]
  R0002 --> R0016[0016]
  R0003[0003] --> R0010[0010]
  R0003 --> R0018[0018]
  R0003 --> R0019[0019]
  R0003 --> R0020[0020]
  R0004[0004] --> R0006[0006]
  R0004 --> R0008[0008]
  R0004 --> R0012[0012]
  R0005[0005] --> R0019[0019]
  R0007[0007] --> R0019[0019]
  R0007 --> R0020[0020]
  R0008[0008] --> R0020[0020]
  R0010[0010] --> R0018[0018]
  R0010 --> R0020[0020]
  R0009[0009] --> R0016[0016]
  R0013[0013] --> R0020[0020]
  R0018[0018] --> R0019[0019]
  R0019[0019] --> R0020[0020]
```

## すぐ飛びたいときのリンク

- [RFC Index](./RFC_INDEX.md)
- [0001](./0001-canonical-surface-and-declarative-defaults.md)
- [0003](./0003-visibility-and-boundary-modules.md)
- [0016](./0016-core-std-and-runtime-profile.md)
- [0019](./0019-verified-extern-evidence-model.md)
- [0020](./0020-diagnostics-contract.md)
