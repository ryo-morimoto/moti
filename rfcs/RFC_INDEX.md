# RFC Index

この索引は、Moti の RFC 群を overview から detailed RFC へたどれる順で並べています。

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
    <tr><td>1</td><td><a href="./0001-canonical-surface-and-declarative-defaults.md"><code>0001</code></a></td><td>正規形と宣言的デフォルト</td><td>以後の syntax と policy を通底する前提だから</td></tr>
    <tr><td>2</td><td><a href="./0021-evaluation-strategy-and-order-of-evaluation.md"><code>0021</code></a></td><td>評価戦略と評価順</td><td>effect / borrow / async の前に sequencing を固定するため</td></tr>
    <tr><td>3</td><td><a href="./0002-module-import-resolution.md"><code>0002</code></a></td><td>module / import / 名前解決</td><td>import-independent resolution の土台だから</td></tr>
    <tr><td>4</td><td><a href="./0003-visibility-and-boundary-modules.md"><code>0003</code></a></td><td>可視性と trust boundary の総論</td><td><code>0028</code>、<code>0029</code>、<code>0044</code> へ分かれる前提を先に掴むため</td></tr>
    <tr><td>5</td><td><a href="./0028-effective-visibility-and-export-closure.md"><code>0028</code></a></td><td>effective visibility と export closure</td><td>公開契約の閉包条件を先に明示するため</td></tr>
    <tr><td>6</td><td><a href="./0029-boundary-modules-and-boundary-only-types.md"><code>0029</code></a></td><td>boundary module と boundary-only type</td><td>FFI / runtime 境界の owner を詳細化するため</td></tr>
    <tr><td>7</td><td><a href="./0016-core-std-and-runtime-profile.md"><code>0016</code></a></td><td><code>core</code>/<code>std</code> と runtime profile の総論</td><td>標準提供物の所有境界を overview で揃えるため</td></tr>
    <tr><td>8</td><td><a href="./0041-core-std-surface-boundary.md"><code>0041</code></a></td><td><code>core</code>/<code>std</code> surface boundary</td><td>pure core と runtime-backed API の分離を明文化するため</td></tr>
    <tr><td>9</td><td><a href="./0042-standard-runtime-profile.md"><code>0042</code></a></td><td>standard runtime profile</td><td>language feature を成立させる最低 runtime 契約を読むため</td></tr>
    <tr><td>10</td><td><a href="./0017-numeric-semantics-and-portability.md"><code>0017</code></a></td><td>数値意味論と portability</td><td>portable contract の具体例として読みやすいため</td></tr>
    <tr><td>11</td><td><a href="./0018-portable-public-api-discipline.md"><code>0018</code></a></td><td>portable public API 規律</td><td>公開 contract の shape を型で固定するため</td></tr>
    <tr><td>12</td><td><a href="./0004-type-theory-foundations.md"><code>0004</code></a></td><td>型理論基盤の総論</td><td>依存型・proof・resource の大枠を先に把握するため</td></tr>
    <tr><td>13</td><td><a href="./0023-type-theory-core-and-universes.md"><code>0023</code></a></td><td>QTT core と universe</td><td>型理論コアの最小形を固定するため</td></tr>
    <tr><td>14</td><td><a href="./0024-proof-erasure-and-relevance.md"><code>0024</code></a></td><td>proof erasure と relevance</td><td>proof と runtime の境界を分けるため</td></tr>
    <tr><td>15</td><td><a href="./0025-divergence-totality-and-recursion.md"><code>0025</code></a></td><td>divergence / totality / recursion</td><td>bottom 排除と partial computation の境界を固定するため</td></tr>
    <tr><td>16</td><td><a href="./0005-kernel-safety.md"><code>0005</code></a></td><td>kernel safety の総論</td><td>trusted kernel の境界を overview で掴むため</td></tr>
    <tr><td>17</td><td><a href="./0026-inductive-and-coinductive-formation.md"><code>0026</code></a></td><td>inductive / coinductive formation</td><td>positivity と codata 規律を明文化するため</td></tr>
    <tr><td>18</td><td><a href="./0027-definitional-equality-and-kernel-conversion.md"><code>0027</code></a></td><td>definitional equality と conversion</td><td>kernel が自動で使う同値を閉じるため</td></tr>
    <tr><td>19</td><td><a href="./0022-annotation-discipline-and-bidirectional-typing.md"><code>0022</code></a></td><td>annotation discipline と bidirectional typing</td><td>surface-to-core elaboration の入口を固定するため</td></tr>
    <tr><td>20</td><td><a href="./0011-pattern-matching-and-dependent-refinement.md"><code>0011</code></a></td><td>pattern matching の総論</td><td>coverage と refinement を読む前に大枠を掴むため</td></tr>
    <tr><td>21</td><td><a href="./0038-pattern-coverage-and-exhaustiveness.md"><code>0038</code></a></td><td>coverage と exhaustive match</td><td>hidden failure path をなくす基礎規則を固定するため</td></tr>
    <tr><td>22</td><td><a href="./0039-dependent-refinement-and-case-tree-elaboration.md"><code>0039</code></a></td><td>dependent refinement と case tree</td><td>pattern から型 refinement を得る規律を読むため</td></tr>
    <tr><td>23</td><td><a href="./0006-memory-model.md"><code>0006</code></a></td><td>二層メモリモデル</td><td>borrow / boundary / runtime の前提となるため</td></tr>
    <tr><td>24</td><td><a href="./0007-borrowing-lifetimes-and-suspend-boundaries.md"><code>0007</code></a></td><td>borrow と suspend の総論</td><td>place model と async interaction の橋渡しだから</td></tr>
    <tr><td>25</td><td><a href="./0030-borrow-core-place-model-and-reborrow.md"><code>0030</code></a></td><td>borrow core / place model / reborrow</td><td>place-sensitive borrow の規則を閉じるため</td></tr>
    <tr><td>26</td><td><a href="./0031-suspend-boundaries-and-await-affine-capabilities.md"><code>0031</code></a></td><td>suspend boundary と await-affine capability</td><td><code>await</code> 越し保持禁止を独立に読むため</td></tr>
    <tr><td>27</td><td><a href="./0008-effects-handlers-and-state.md"><code>0008</code></a></td><td>effect system の総論</td><td>row / handler / state の大枠を先に読むため</td></tr>
    <tr><td>28</td><td><a href="./0032-effect-rows-and-builtin-effects.md"><code>0032</code></a></td><td>effect row と built-in effect</td><td>公開面の effect contract を固定するため</td></tr>
    <tr><td>29</td><td><a href="./0033-handlers-and-handler-state.md"><code>0033</code></a></td><td>handler と handler state</td><td>effect の operational story を閉じるため</td></tr>
    <tr><td>30</td><td><a href="./0034-effect-abstraction-and-limited-polymorphism.md"><code>0034</code></a></td><td>effect abstraction と limited polymorphism</td><td>closed-row surface を守りつつ抽象化するため</td></tr>
    <tr><td>31</td><td><a href="./0010-error-model-and-result-boundaries.md"><code>0010</code></a></td><td>error model と Result 境界</td><td>failure surface を effect から公開 API へ正規化するため</td></tr>
    <tr><td>32</td><td><a href="./0009-structured-concurrency-and-cancellation.md"><code>0009</code></a></td><td>structured concurrency の総論</td><td>async / parallel の overview を先に読むため</td></tr>
    <tr><td>33</td><td><a href="./0035-structured-concurrency-core.md"><code>0035</code></a></td><td>structured concurrency core</td><td>task tree ownership を独立に固定するため</td></tr>
    <tr><td>34</td><td><a href="./0036-cancellation-and-deadline-propagation.md"><code>0036</code></a></td><td>cancellation と deadline 伝播</td><td>typed cancel / timeout story を閉じるため</td></tr>
    <tr><td>35</td><td><a href="./0037-send-share-and-domain-affinity.md"><code>0037</code></a></td><td><code>Send</code>/<code>Share</code> と domain affinity</td><td>cross-domain 安全条件を型で固定するため</td></tr>
    <tr><td>36</td><td><a href="./0012-trait-core-coherence.md"><code>0012</code></a></td><td>trait core coherence</td><td>trait の中心不変条件を先に掴むため</td></tr>
    <tr><td>37</td><td><a href="./0040-associated-items-and-projection-surface.md"><code>0040</code></a></td><td>associated item と projection surface</td><td>trait public contract の shape を決めるため</td></tr>
    <tr><td>38</td><td><a href="./0013-trait-solver-formalization.md"><code>0013</code></a></td><td>trait solver formalization</td><td>trait core の failure mode を閉じるため</td></tr>
    <tr><td>39</td><td><a href="./0014-existential-abstraction.md"><code>0014</code></a></td><td>existential abstraction</td><td>trait と別制度の抽象境界を読むため</td></tr>
    <tr><td>40</td><td><a href="./0015-relational-constraints.md"><code>0015</code></a></td><td>relational constraints</td><td>trait core 外の拡張口を読むため</td></tr>
    <tr><td>41</td><td><a href="./0019-verified-extern-evidence-model.md"><code>0019</code></a></td><td>verified extern の総論</td><td><code>0043</code> と <code>0044</code> の全体像を先に掴むため</td></tr>
    <tr><td>42</td><td><a href="./0043-verified-extern-checker-interface.md"><code>0043</code></a></td><td>verified extern checker interface</td><td>checker verdict の contract を固定するため</td></tr>
    <tr><td>43</td><td><a href="./0044-verified-extern-wrapper-obligations-and-logical-safe.md"><code>0044</code></a></td><td>wrapper obligations と <code>logical-safe</code></td><td>public safe facade の条件を閉じるため</td></tr>
    <tr><td>44</td><td><a href="./0020-diagnostics-contract.md"><code>0020</code></a></td><td>diagnostics contract</td><td>全 RFC の failure surface を最後にまとめるため</td></tr>
  </tbody>
</table>
