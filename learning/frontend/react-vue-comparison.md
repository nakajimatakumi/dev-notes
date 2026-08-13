# ReactとVueの違い

ReactとVueは、どちらもコンポーネントを組み合わせてユーザーインターフェースを構築するための技術である。

大きな違いは、ReactがJavaScriptを中心にUIを表現するライブラリであるのに対し、VueはHTMLに近いテンプレートとリアクティビティを組み合わせた段階的に導入できるフレームワークである点にある。ただし、実際の開発では周辺ツールやフレームワークも組み合わせるため、構文や単純なベンチマークだけで優劣を決めることはできない。

## 結論

- Reactは、JavaScriptやTypeScriptの関数とJSXを使い、値とイベント処理を明示的にUIへ渡す。
- Vueは、Single-File Component（SFC）のテンプレート、ロジック、スタイルを組み合わせ、ディレクティブによって状態とDOMを結び付ける。
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

## HTML要素への振る舞いの付け方

チェックボックスのような既存のHTML要素へ独自処理を追加する場合、ReactとVueでは状態との結び付け方が異なる。

### Reactのチェックボックス

```tsx
import { useState, type ChangeEvent } from "react";

export default function NotificationSetting() {
  const [enabled, setEnabled] = useState(false);

  function handleChange(event: ChangeEvent<HTMLInputElement>) {
    const nextEnabled = event.currentTarget.checked;
    setEnabled(nextEnabled);
    saveNotificationSetting(nextEnabled);
  }

  return (
    <label>
      <input
        type="checkbox"
        checked={enabled}
        onChange={handleChange}
      />
      通知を有効にする
    </label>
  );
}
```

Reactでは、`checked`で現在の状態を渡し、`onChange`で変更を受け取る。このようにReactのstateで値を管理する入力をcontrolled inputと呼ぶ。`checked`を指定した場合は、通常、値を同期的に更新する`onChange`も必要になる。

短い処理ならJSXへ直接書くこともできる。

```tsx
<input
  type="checkbox"
  checked={enabled}
  onChange={(event) => setEnabled(event.currentTarget.checked)}
/>
```

ただし、複数の更新、通信、検証などを行う場合は、名前を付けた関数へ切り出した方が目的を読み取りやすい。

### Vueのチェックボックス

```vue
<script setup lang="ts">
import { ref } from "vue";

const enabled = ref(false);

function handleChange(event: Event) {
  const target = event.currentTarget as HTMLInputElement;
  saveNotificationSetting(target.checked);
}
</script>

<template>
  <label>
    <input
      v-model="enabled"
      type="checkbox"
      @change="handleChange"
    >
    通知を有効にする
  </label>
</template>
```

Vueでは、`v-model`が入力要素と状態の同期をまとめて表現する。独自処理は`@change`などのイベントディレクティブから呼び出せる。

したがって、本質的な違いは「Reactは要素内に処理を書き、Vueは書かない」ことではない。

- Reactは、JSXのpropsとして値やJavaScript関数を要素へ渡す。
- Vueは、テンプレートディレクティブによって値や処理を要素へ結び付ける。
- どちらも短いインライン処理を書けるが、複雑な処理は関数へ切り出せる。

ソースコード上では要素の属性部分に処理が見えるが、ブラウザへ出力されるHTML属性へ任意の関数コードがそのまま保存される、という意味ではない。

## 状態管理と更新の考え方

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
- チェックボックスでは、Reactの`checked`と`onChange`、Vueの`v-model`と`@change`に違いが表れやすい。
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
- [Vue.js: Rendering Mechanism](https://vuejs.org/guide/extras/rendering-mechanism.html)
- [Vue.js: State Management](https://vuejs.org/guide/scaling-up/state-management.html)
- [Vue.js: Using Vue with TypeScript](https://vuejs.org/guide/typescript/overview.html)
- 取得日: 2026-08-13
