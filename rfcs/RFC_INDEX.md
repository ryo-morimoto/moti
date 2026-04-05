# RFC Index

この索引は、メンタルモデルを構築しやすい順序で RFC を並べています。

<table class="rfc-index-table">
  <thead>
    <tr>
      <th>順</th>
      <th>RFC</th>
      <th>焦点</th>
      <th>先に読む理由</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>1</td><td><a href="./0001-canonical-surface-and-declarative-defaults.md"><code>0001</code></a></td><td>正規形と宣言的デフォルト</td><td>言語全体の「どう書くか」を先に固定するため</td></tr>
    <tr><td>2</td><td><a href="./0002-module-import-resolution.md"><code>0002</code></a></td><td>module / import / 名前解決</td><td>以後の trait と標準提供物の読み方の土台になるため</td></tr>
    <tr><td>3</td><td><a href="./0003-visibility-and-boundary-modules.md"><code>0003</code></a></td><td>公開境界と boundary module</td><td>何を公開面に出せるかを最初に掴むため</td></tr>
    <tr><td>4</td><td><a href="./0016-core-std-and-runtime-profile.md"><code>0016</code></a></td><td><code>core</code>/<code>std</code> と runtime profile</td><td>標準提供物と runtime 契約の境界を先に読むため</td></tr>
    <tr><td>5</td><td><a href="./0017-numeric-semantics-and-portability.md"><code>0017</code></a></td><td>数値意味論と portability</td><td>portable semantics の具体例として読みやすいため</td></tr>
    <tr><td>6</td><td><a href="./0018-portable-public-api-discipline.md"><code>0018</code></a></td><td>portable public API 規律</td><td>公開 contract の shape を固定するため</td></tr>
    <tr><td>7</td><td><a href="./0004-type-theory-foundations.md"><code>0004</code></a></td><td>型理論の基盤</td><td>依存型・multiplicity・proof の土台を先に読むため</td></tr>
    <tr><td>8</td><td><a href="./0005-kernel-safety.md"><code>0005</code></a></td><td>kernel safety</td><td>trusted kernel の境界を固めるため</td></tr>
    <tr><td>9</td><td><a href="./0011-pattern-matching-and-dependent-refinement.md"><code>0011</code></a></td><td>pattern matching と refinement</td><td>型理論コアの応用として読むと負荷が低いため</td></tr>
    <tr><td>10</td><td><a href="./0006-memory-model.md"><code>0006</code></a></td><td>memory model</td><td>高レベル層と低レベル層の分離を先に掴むため</td></tr>
    <tr><td>11</td><td><a href="./0007-borrowing-lifetimes-and-suspend-boundaries.md"><code>0007</code></a></td><td>borrow / lifetime / suspend</td><td>async や FFI 制約の前提になるため</td></tr>
    <tr><td>12</td><td><a href="./0008-effects-handlers-and-state.md"><code>0008</code></a></td><td>effects / handlers / state</td><td>状態変化と制御効果の見方を揃えるため</td></tr>
    <tr><td>13</td><td><a href="./0010-error-model-and-result-boundaries.md"><code>0010</code></a></td><td>error model と Result 境界</td><td>失敗の公開面を先に固定するため</td></tr>
    <tr><td>14</td><td><a href="./0009-structured-concurrency-and-cancellation.md"><code>0009</code></a></td><td>structured concurrency</td><td>borrow / effect / error を踏まえて async を読むため</td></tr>
    <tr><td>15</td><td><a href="./0012-trait-core-coherence.md"><code>0012</code></a></td><td>trait core coherence</td><td>trait を便利機能ではなく解決規律として捉えるため</td></tr>
    <tr><td>16</td><td><a href="./0013-trait-solver-formalization.md"><code>0013</code></a></td><td>trait solver formalization</td><td>trait core の operational meaning を閉じるため</td></tr>
    <tr><td>17</td><td><a href="./0014-existential-abstraction.md"><code>0014</code></a></td><td>existential abstraction</td><td>trait と別制度の抽象境界として読むため</td></tr>
    <tr><td>18</td><td><a href="./0015-relational-constraints.md"><code>0015</code></a></td><td>relational constraints</td><td>trait core に入れない拡張口を理解するため</td></tr>
    <tr><td>19</td><td><a href="./0019-verified-extern-evidence-model.md"><code>0019</code></a></td><td>verified extern</td><td>FFI trust boundary の総仕上げだから</td></tr>
    <tr><td>20</td><td><a href="./0020-diagnostics-contract.md"><code>0020</code></a></td><td>diagnostics contract</td><td>全 RFC の failure surface を最後にまとめるため</td></tr>
  </tbody>
</table>
