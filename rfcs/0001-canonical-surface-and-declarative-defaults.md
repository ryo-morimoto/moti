# 0001-canonical-surface-and-declarative-defaults

- 状態: `Draft`
- 日付: 2026-04-05
- 対象範囲: 表面構文、正規形、宣言的デフォルト、一般局所可変の不採用
- Feature Name: `canonical_surface_and_declarative_defaults`
- Start Date: 2026-04-05
- RFC PR: 未作成
- Moti Issue: 未作成

## Summary

Moti は、同じ意味を複数の表層で書ける自由度よりも、低エントロピーで一意な正規形を優先する。これにより、表面構文は `fn`、`let`、式ベースブロック、セミコロンなし、operator overloading 不可を基本とし、一般局所可変変数 `var` は導入しない。状態変化は effect と handler state への明示 opt-in としてのみ許す。

## Motivation

- Moti の根本目標には「正規形が一つに定まる低エントロピーなコードスタイル」が含まれている。
- エージェントが大量のコードを生成しても、構文の揺れや命令的 escape hatch でコード形状が発散しないことが重要である。
- 暗黙の可変性や複数の等価構文は、読み手のメンタルモデルを崩し、review と mechanization を難しくする。
- 宣言的デフォルトは、effect system、error model、async model の理解を一貫したものにする。

## Guide-level explanation

Moti では「どう書けるか」より先に「どう書くべきか」が固定される。関数は `fn` で定義し、局所束縛は `let` だけを使い、ブロックは最後の式を返す。一般の `var` はなく、状態を持ちたいときは handler の state や明示的な effect を使う。

```moti
fn incr(x: Int): Int {
  let y = x + 1
  y
}
```

この RFC の mental model は単純である。Moti では「便利な別記法」を増やして書き心地を良くするのではなく、同じ意味に同じ形を対応させる。状態や命令的制御は、表面のどこかに痕跡が残るようにしか導入しない。

## Reference-level explanation

- 表面構文の既定は `fn`、`let`、`=>`、`/ Effect`、`/ {A, B}`、`{}` ブロック、セミコロンなしである。
- 文としてのブロックではなく式ベースを採る。最後の式が値を返す。
- 一般局所可変変数 `var` は導入しない。
- operator overloading は導入しない。
- 命令的状態は effect と handler state の制度へ押し出す。
- 追加 sugar は、正規形への一方向 elaboration が明確で、読み手のメンタルモデルを増やさない場合に限って検討する。

## Drawbacks

- 表面の自由度が低いため、他言語の慣れた記法をそのまま持ち込みにくい。
- 小さな convenience syntax でも厳しく却下されやすい。
- 一般の可変ローカル変数がないことで、命令的な記述を好む利用者には窮屈に見える。

## Rationale and alternatives

- 代替案 1 は「複数の等価構文を許し、formatter で寄せる」だったが、agentic code generation 時代には構文自由度そのものが drift の原因になるため採らない。
- 代替案 2 は「`var` を許し、後から effect inference で整理する」だったが、状態変化が表面から消えるので採らない。
- 代替案 3 は「まずは実装しやすい syntax を許し、後で削る」だったが、Moti は単調増加を優先し、v1 の実装容易性で本体を汚さない。
- 何もしない場合、構文判断が局所最適に流れ、正規形と宣言的デフォルトが実質目標のまま固定されない。

## Prior art

- Go は記法の自由度を制限し、読みやすさを揃える点で参考になる。
- Rust は formatter と lint による規律を持つが、Moti はさらに一段進めて、言語設計そのもので表面の分岐を減らす。
- ML / Haskell 系は式ベースと immutability default を持つが、Moti は effect と trust boundary の明示性をより強く要求する。

## Unresolved questions

- formatter が language law としてどこまで normative になるか。
- sugar を将来許す場合の admission criteria をどこまで厳密に書くか。

## Future possibilities

- 一方向 elaboration が明確で、読み手のメンタルモデルを増やさない sugar の追加。
- formatter / linter / canonical printer を unified pipeline として扱うための後続 RFC。
