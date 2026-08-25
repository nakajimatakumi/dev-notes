# ReactとVueの違い

ReactとVueは、どちらもコンポーネントを組み合わせてユーザーインターフェースを構築するための技術である。

大きな違いは、ReactがJavaScriptを中心にUIを表現するライブラリであるのに対し、VueはHTMLに近いテンプレートとリアクティビティを組み合わせた段階的に導入できるフレームワークである点にある。ただし、実際の開発では周辺ツールやフレームワークも組み合わせるため、構文や単純なベンチマークだけで優劣を決めることはできない。

## 結論

- Reactは、JavaScriptやTypeScriptの関数とJSXを使い、値とイベント処理を明示的にUIへ渡す。
- Vueは、Single-File Component（SFC）のテンプレート、ロジック、スタイルを組み合わせ、ディレクティブによって状態とDOMを結び付ける。
- Reactは再利用する振る舞いをコンポーネントやCustom Hookへ抽象化する傾向があり、Vueはそれらに加えて既存要素へ低レベルDOM処理を付加するカスタムディレクティブを持つ。
- Reactのstateは各レンダー時点のスナップショットとして扱い、更新関数を呼び出して次のレンダーを要求する。
- Vueはリアクティブな値への依存を追跡し、その値が変わると依存している箇所を更新する。
- 性能はアプリケーションの構成、レンダリング方法、データ量、実装、ビルド設定によって変わるため、「常にどちらが速い」とは判断できない。
- 選定では、チームの経験、既存資産、設計の自由度、公式エコシステム、保守方法まで含めて判断する。

## 全体比較

| 観点 | React | Vue |
| --- | --- | --- |
| 基本的な位置付け | UIを構築するJavaScriptライブラリ | UIを構築するプログレッシブフレームワーク |
| 主な記述形式 | JavaScriptまたはTypeScript内のJSX | `.vue` SFC内の`template`、`script`、`style` |
| UIの表現 | JavaScriptの式、関数、配列処理を使う | HTMLに近いテンプレートとディレクティブを使う |
| 状態 | `useState`などのHook | `ref`、`reactive`などのリアクティビティAPI |
| 状態の考え方 | state更新によって再レンダーを要求する | 依存を追跡して必要な更新を実行する |
| イベント | `onClick={handleClick}` | `@click="handleClick"` |
| フォーム入力 | `value`や`checked`と`onChange`を組み合わせる | `v-model`で入力と状態を結び付ける |
| 条件分岐 | `if`、三項演算子、`&&` | `v-if`、`v-else`、`v-show` |
| 繰り返し | `map()`などのJavaScript | `v-for` |
| ロジックの再利用 | Custom Hook | Composable |
| 既存HTML要素への再利用可能な振る舞いの付加 | propsで処理を渡す。再利用時はコンポーネントやCustom Hookへ抽象化する | 組み込み・カスタムディレクティブを要素へ付けられる。通常のUI部品はコンポーネント化する |
| コンポーネント間の内容差し込み | `children` | Slot |
| TypeScript | JSXを含むファイルでは主に`.tsx`を使う | `<script setup lang="ts">`とテンプレートを組み合わせられる |
| 大規模アプリの構成 | ルーティングやデータ取得を含むReactフレームワークの利用が推奨される | Vue RouterやPiniaなどVue向けの公式ライブラリが用意されている |

この表の項目には似た役割のものがあるが、すべてが一対一で対応するわけではない。たとえば、Reactの`useEffect`とVueの`watch`は似た場面で使うことがあっても、実行条件や設計上の役割まで同じではない。

## 文法とコンポーネントの違い

### React

Reactコンポーネントは、基本的にJSXを返すJavaScriptまたはTypeScriptの関数である。表示条件や一覧生成にはJavaScriptの構文をそのまま使う。

```tsx
type User = {
  id: number;
  name: string;
  active: boolean;
};

function UserList({ users }: { users: User[] }) {
  return (
    <ul>
      {users
        .filter((user) => user.active)
        .map((user) => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}
```

### Vue

Vueでは、SFCの`<script>`で状態や処理を定義し、`<template>`で表示方法を記述する。繰り返しや条件分岐には`v-for`や`v-if`などのディレクティブを使う。

```vue
<script setup lang="ts">
type User = {
  id: number;
  name: string;
  active: boolean;
};

defineProps<{ users: User[] }>();
</script>

<template>
  <ul>
    <template v-for="user in users" :key="user.id">
      <li v-if="user.active">{{ user.name }}</li>
    </template>
  </ul>
</template>
```

Reactは「マークアップもJavaScriptの処理の中で組み立てる」という感覚が強い。Vueは「テンプレートへ状態や振る舞いを宣言する」という感覚が強い。この違いが、コードの見た目だけでなく、条件分岐やイベント処理の整理方法にも表れる。

## 既存HTML要素を拡張する方針の違い

比較すべきなのはチェックボックス単体の書き方ではなく、既存の`button`、`input`、`div`などへ独自の振る舞いを追加するとき、その要素を拡張して使い続けるのか、新しいコンポーネントとして抽象化するのかという方針である。

### Reactはpropsで処理を渡し、再利用時はコンポーネントへ抽象化する

Reactでは、既存HTML要素に対して、あらかじめReact DOMが対応しているpropsを使う。イベント処理は`onClick`や`onChange`などへJavaScript関数を渡して追加する。

```tsx
<button onClick={handleSave}>保存</button>
```

独自の`track`属性や`focus`属性をReactの機能として既存要素へ追加するのではなく、再利用したい責務に応じてコンポーネントやCustom Hookへ切り出す。

```tsx
type SaveButtonProps = {
  onSave: () => void;
};

function SaveButton({ onSave }: SaveButtonProps) {
  function handleClick() {
    recordAction("save");
    onSave();
  }

  return <button onClick={handleClick}>保存</button>;
}
```

この方法では、見た目、状態、イベント、アクセシビリティを含むまとまりを`SaveButton`という新しいUI部品として扱う。対象要素を利用箇所ごとに拡張するよりも、コンポーネントのpropsを公開インターフェースとして再利用する考え方が中心になる。

ただし、一度しか使わない短い処理であれば、既存要素のイベントpropsへ直接渡してよい。すべての処理をコンポーネント化するという意味ではない。

### Vueはディレクティブによって既存要素へ振る舞いを付加できる

Vueでは、`v-if`、`v-show`、`v-model`、`v-bind`などの組み込みディレクティブを、既存HTML要素へ宣言的に付けられる。

```vue
<input v-model="keyword" v-focus>
```

さらに、低レベルなDOM操作を再利用する必要がある場合は、カスタムディレクティブを定義できる。

```vue
<script setup lang="ts">
const vFocus = {
  mounted(element: HTMLInputElement) {
    element.focus();
  }
};
</script>

<template>
  <input v-focus>
</template>
```

`v-focus`は、`input`を別のコンポーネントに置き換えず、既存要素へフォーカス処理だけを付加している。この仕組みにより、Vueはプレーンな要素へ横断的な振る舞いを宣言的に追加できる。

ただし、Vue公式ドキュメントでは、コンポーネントを主要な構成要素、Composableを状態を持つロジックの再利用手段と位置付けている。カスタムディレクティブは、直接DOMを操作しなければ実現できない処理に限って使うことが推奨されている。複雑なUIや業務的な意味を持つ部品まで、すべて既存要素の拡張として作る方針ではない。

### 方針の比較

| 判断 | React | Vue |
| --- | --- | --- |
| 利用箇所だけのイベント処理 | 既存要素のイベントpropsへ関数を渡す | `@click`などのイベントディレクティブで関数を呼ぶ |
| 状態とフォーム要素の同期 | `value`または`checked`と`onChange`を明示する | `v-model`で宣言できる |
| UI部品として再利用 | コンポーネント化する | コンポーネント化する |
| 状態を持つロジックの再利用 | Custom Hookへ切り出す | Composableへ切り出す |
| 既存要素へ低レベルDOM処理を再利用可能な形で付加 | `ref`やHookを利用する設計を検討する | カスタムディレクティブを利用できる |

したがって、VueはReactよりも「既存要素へ振る舞いを付加する」ための明示的な仕組みを持つ。一方で、両者ともアプリケーションの中心的なUI設計ではコンポーネント化を重視する。違いを「既存要素を拡張するVue、必ず新規要素を作るReact」と二分するのではなく、Vueには限定用途のディレクティブという追加の選択肢がある、と捉えるのが適切である。

## 状態管理と更新の考え方

状態更新時に実行し直す範囲、派生値、子コンポーネント、状態更新方法の違いは、[ReactとVueの状態更新時の差分](./react-vue-state-update-differences.md)にまとめる。

### Reactのstateはレンダー時点のスナップショット

Reactでは、stateを更新しても、現在実行中の処理が参照しているstate変数そのものが即座に書き換わるわけではない。更新関数は次のレンダーを要求し、コンポーネントは更新後のstateを使って再度呼び出される。

複数のコンポーネントで状態を共有する場合は、共通の親へ状態を移す、Contextを使う、reducerと組み合わせる、外部の状態管理ライブラリを使う、といった選択肢がある。

### Vueはリアクティブな依存を追跡する

Vueでは、`ref`や`reactive`で作った値をテンプレートや算出処理が参照すると、その依存関係が追跡される。値が変化すると、依存している箇所が更新される。

小規模な共有状態はリアクティビティAPIだけでも構成できる。大規模なアプリケーションでは、開発ツール、SSR、チーム内の規約なども考慮し、Piniaのような状態管理ライブラリを検討する。

## 設計思想の違い

### React

- UIをコンポーネントというJavaScript関数の組み合わせとして捉える。
- 表示条件、一覧変換、イベントハンドラーをJavaScriptで表現する。
- React自体がルーティングやデータ取得方法を一つに固定せず、アプリ全体ではフレームワークやライブラリを組み合わせる。
- 選択の自由度が高い一方、プロジェクトで採用する構成や規約を決める必要がある。

### Vue

- 標準のHTML、CSS、JavaScriptを土台に、テンプレート構文とリアクティビティを加える。
- SFCによって、コンポーネント単位でテンプレート、処理、スタイルをまとめる。
- 小さな機能の追加からSPA、SSRまで段階的に利用できる。
- 公式のルーターや状態管理ライブラリがあり、代表的な構成を選びやすい。

両者とも宣言的UI、コンポーネント化、段階的な導入を重視している。思想を完全な対立として捉えるのではなく、「JavaScriptとテンプレートのどちらを中心に見せるか」「周辺構成をどこまで公式エコシステムに寄せるか」という傾向として比較する。

## 性能の違い

ReactとVueはどちらも、状態からUIを宣言し、必要に応じてDOMを更新する。VueもVirtual DOMを使うが、テンプレートをコンパイルするときに静的な部分や更新対象の情報を生成し、実行時の更新を効率化するCompiler-Informed Virtual DOMを採用している。

一方、アプリケーション全体の性能はレンダリング機構だけでは決まらない。

| 確認する性能 | 主な確認内容 |
| --- | --- |
| 初期表示 | JavaScriptの転送量、解析時間、SSR・SSGの有無 |
| 更新性能 | 更新範囲、コンポーネント構成、一覧の件数、計算量 |
| 入力応答 | 入力のたびに更新される範囲、重い処理の有無 |
| メモリ | 保持するデータ、コンポーネント数、キャッシュ方法 |
| 通信 | APIの回数、データ量、キャッシュ、読み込み順序 |
| 実運用 | 利用端末、ネットワーク、画面の操作内容 |

比較するときは、同じ画面、同じデータ、同じレンダリング方式、同程度に最適化した実装を用意し、ブラウザのPerformanceパネルや実環境に近い計測で確認する。特定の小さなベンチマーク結果だけで、すべてのアプリケーションにおける優劣を決めない。

## エコシステムと開発体験

| 観点 | 確認すること |
| --- | --- |
| ルーティング | 公式または採用フレームワークの標準構成が要件を満たすか |
| 状態管理 | ローカル状態と共有状態をどの仕組みで管理するか |
| SSR・SSG | SEO、初期表示、サーバー運用の要件に対応できるか |
| テスト | コンポーネント、ユーザー操作、E2Eをどう検証するか |
| TypeScript | Props、イベント、テンプレート、共通コンポーネントの型を扱いやすいか |
| DevTools | 状態、コンポーネント、性能を調査しやすいか |
| UIライブラリ | 必要な部品、アクセシビリティ、デザイン要件を満たすか |
| 更新と移行 | メジャーバージョンや周辺ライブラリの更新を継続できるか |

ReactとVueの両方がTypeScriptに対応している。ReactではPropsやHook、DOMイベントをTypeScriptで型付けできる。Vueは公式パッケージに型定義が含まれ、SFCの型チェックには`vue-tsc`を利用できる。単に「TypeScript対応」と比較するだけでなく、実際にProps、イベント、フォーム、汎用コンポーネントを書いて、IDE補完やエラーの分かりやすさを確認する。

## 選定時の判断基準

実際の新規プロダクトやアプリケーションで挙げられた理由は、[React・Vueの新規開発における技術選定事例](./react-vue-new-development-selection-cases.md)にまとめる。

### Reactが候補になりやすい場合

- JavaScriptやTypeScriptの式と関数を中心にUIを組み立てたい。
- 採用済みのReactフレームワーク、ライブラリ、コンポーネント資産がある。
- Web以外も含め、Reactの知識やエコシステムを活用したい。
- チームで周辺ライブラリと設計規約を選び、維持できる。

### Vueが候補になりやすい場合

- HTMLに近いテンプレートを中心にコンポーネントを読みたい。
- SFCでテンプレート、ロジック、スタイルの置き場所を揃えたい。
- `v-model`やディレクティブを使って、フォームやDOMとの関係を簡潔に表したい。
- Vue RouterやPiniaなど、Vue向けの公式エコシステムを基準に構成したい。

これは絶対的な基準ではない。最終的には、同じ小規模な画面を両方で実装し、次の項目をチームで確認する方が判断しやすい。

1. 初見で状態の場所と更新経路を追えるか。
2. フォーム、API取得、エラー表示を自然に実装できるか。
3. Propsやイベントの型エラーを発見しやすいか。
4. テストを書きやすいか。
5. チームで記述方法を統一できるか。
6. 既存システムへ導入・移行しやすいか。
7. 性能要件を実測で満たせるか。

## まとめ

- ReactはJSXとJavaScript関数を中心に、値と処理を明示的にUIへ渡す。
- VueはHTMLに近いテンプレートとディレクティブを中心に、状態とDOMを結び付ける。
- Reactはprops、コンポーネント、Custom Hookを中心に振る舞いを構成し、Vueはそれらに相当する仕組みに加えて、既存要素へ低レベルDOM処理を付加するカスタムディレクティブを持つ。
- Reactはstate更新と再レンダー、Vueはリアクティブな依存追跡という考え方を理解すると、構文の理由が分かりやすい。
- 性能はフレームワーク名だけで決めず、同じ要件と条件で計測する。
- 選定では、文法、性能、思想に加えて、エコシステム、TypeScript、テスト、移行、チームの経験、保守性を比較する。

## 出典

- [React公式ドキュメント](https://react.dev/)
- [React: Describing the UI](https://react.dev/learn/describing-the-ui)
- [React: State as a Snapshot](https://react.dev/learn/state-as-a-snapshot)
- [React: input](https://react.dev/reference/react-dom/components/input)
- [React: Using TypeScript](https://react.dev/learn/typescript)
- [Vue.js: Introduction](https://vuejs.org/guide/introduction.html)
- [Vue.js: Form Input Bindings](https://vuejs.org/guide/essentials/forms.html)
- [Vue.js: Custom Directives](https://vuejs.org/guide/reusability/custom-directives.html)
- [Vue.js: Rendering Mechanism](https://vuejs.org/guide/extras/rendering-mechanism.html)
- [Vue.js: State Management](https://vuejs.org/guide/scaling-up/state-management.html)
- [Vue.js: Using Vue with TypeScript](https://vuejs.org/guide/typescript/overview.html)
- 取得日: 2026-08-13
