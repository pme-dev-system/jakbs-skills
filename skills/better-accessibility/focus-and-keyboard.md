# フォーカスとキーボード

フォーカスリング、スキップリンク、tabindex、フォーカストラップ、APGキーボードパターン。

## フォーカスリング

素の`:focus`ではなく`:focus-visible`にスタイルを適用する。ブラウザはキーボードや支援技術によるフォーカスでは`:focus-visible`を表示するが、フォーカスがすでに明らかなマウスクリックでは抑制する。可視の代替なしに`outline: none`や`focus:outline-none`を書くことは決してしない。それは、目が見えるキーボードユーザーからキーボード操作を奪う。

ブラウザ標準の未改変のフォーカスインジケーターを優先する。これはプラットフォームやforced-color設定に応じて適応し、作者があらゆる背景を予測する必要がない。`outline-offset`だけを追加すれば、通常そのインジケーターは維持される。色を指定しないカスタムの`outline: 2px solid`は`currentColor`として描画される。アウトラインはテキスト自身の背景とは異なる色を横切ることがあるため、これは自動的にアクセシブルになるわけではない。したがって優先順位は次の通りである:

```css
/* 最善: ブラウザのリングを保ち、余白を与えるだけにする */
:focus-visible {
  outline-offset: 2px;
}

/* デザイン上カスタムリングが必要な場合: プロジェクトの検証済みトークンを使う */
:focus-visible {
  outline: 2px solid var(--focus-ring);
  outline-offset: 2px;
}
```

```tsx
// Tailwind: プロジェクトのフォーカストークンか、確立されたfocus-ringユーティリティを使う
<button className="focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-[var(--focus-ring)]">
  Save
</button>
```

カスタムのフォーカスインジケーターは、可視面積とコントラストの変化について、該当するプロジェクト/WCAGのターゲットを満たさなければならない。コンポーネントの塗り、ページのサーフェス、画像、グラデーション、ホバー/選択状態を含め、横切るすべての隣接色に対して外周全体を検査する。トークン、ブランドカラー、`currentColor`は、その描画チェックに合格した場合にのみ使用できる。

`forced-colors: active`(Windows高コントラストモード)では、デフォルトの色調整を維持するか、`Highlight`のようなシステムカラーを明示的に使う。コントロールが引き続き知覚可能である場合を除き、`forced-color-adjust: none`で作者指定の色を固定することは決してしない。

内部の入力がフォーカスされている間にラッパーを光らせたい場合は、`:focus-within`でフォーカススタイルをグループ化する(例: 枠内にアイコンがある検索ボックス)。

## スキップリンク

繰り返されるナビゲーションやその他の繰り返しのchromeが主要コンテンツより前にある場合、最初のフォーカス可能な要素は`<main id="main">`をターゲットにした「Skip to content」リンクにする。フォーカスされるまで視覚的に隠す:

```css
.skip-link {
  position: absolute;
  inset-inline-start: -999px;
}
.skip-link:focus {
  inset-inline-start: 16px;
  top: 16px;
}
```

```html
<body>
  <a class="skip-link" href="#main">Skip to content</a>
  <header>…</header>
  <main id="main">…</main>
</body>
```

ページ内アンカーのターゲットには`scroll-margin-top`を指定し(例: 固定ヘッダー下では`scroll-margin-top: 80px`)、ジャンプ先が隠れないようにする。

## tabindexのルール

- `tabindex="0"`: 要素を自然なタブ順序に加える。ネイティブにフォーカス不可なカスタムのインタラクティブ要素にのみ使う。
- `tabindex="-1"`: JavaScriptからのみフォーカス可能にする(`el.focus()`)。フォーカスを移動する見出し、モーダルコンテナ、ローミングタブインデックスのメンバーに使う。
- 正の`tabindex`: 決して使わない。ページ全体のタブ順序を乗っ取ってしまう。代わりにDOMの順序を修正する。

### ローミングタブインデックス

複合ウィジェット(タブ、メニュー、ツールバー、ラジオグループ)は1つのTabストップを占有する。アクティブな項目は`tabindex="0"`を持ち、それ以外はすべて`tabindex="-1"`を持つ。矢印キーはフォーカスと`0`の両方を移動させる:

```tsx
<div role="tablist">
  {tabs.map((tab, i) => (
    <button
      role="tab"
      tabIndex={i === activeIndex ? 0 : -1}
      aria-selected={i === activeIndex}
      onKeyDown={handleArrowKeys} // ArrowLeft/ArrowRightでactiveIndexを移動、末端でラップ
    >
      {tab.label}
    </button>
  ))}
</div>
```

## フォーカストラップと復元

モーダルはフォーカスをトラップしなければならない。現代的な手法は、ダイアログの背後にあるすべてに`inert`属性を付けることである。これにより背景コンテンツをタブ順序と支援技術の両方から一度に除外できる:

```tsx
// 開いたとき
document.getElementById("app-content").inert = true;
const dialog = dialogRef.current;
(dialog.querySelector("[autofocus]") ??
  dialog.querySelector("button, [href], input, select, textarea"))?.focus();

// 閉じたとき
document.getElementById("app-content").inert = false;
triggerRef.current?.focus(); // 常に、開いた要素にフォーカスを戻す
```

`showModal()`を使ったネイティブの`<dialog>`は、トラップ、`inert`な背景、Escapeの処理を無料で提供してくれるので、これを優先する。`<dialog>`を使えないカスタムオーバーレイには、`role="dialog"`、`aria-modal="true"`、アクセシブルネーム(見出しを指す`aria-labelledby`)が必要である。いずれの場合も:

- 開いたときは最初のフォーカス可能な要素にフォーカスする。破壊的な確認の場合は、代わりに最も破壊性の低いアクションにフォーカスする。
- 閉じたときはトリガーにフォーカスを戻す。トリガーがもう存在しない場合は、最も近い論理的なコンテナにフォーカスを移動する。
- ダイアログに`overscroll-behavior: contain`を追加し、内部のスクロールが背後のページを決してスクロールしないようにする。

## キーボードパターン(ARIA APG)

ネイティブ要素にはこれらの挙動が備わっている。カスタムウィジェットはそれを実装しなければならない。ロールは約束である: 何かに`role="tab"`を与えれば、ユーザーは完全なタブのキーボードモデルを期待する。

| ウィジェット | キー |
| --- | --- |
| Dialog | Tab/Shift+Tabで内部を循環(端でラップ)。Escapeで閉じる |
| Tabs | 矢印キーでタブ間を移動(ラップする)。Tabでパネルへ抜ける。Home/Endで最初/最後へジャンプ |
| Menu button | Enter/Space/ArrowDownで開いて最初の項目にフォーカス。ArrowUpで開いて最後の項目にフォーカス。矢印キーでナビゲート。Escapeで閉じてボタンに再フォーカス |
| Disclosure / accordion | ヘッダーは`<button aria-expanded>`。EnterとSpaceでトグル |
| Combobox | ArrowDownでリストを開く/リスト内を移動。Enterで確定。Escapeで閉じて入力欄に戻る。入力するとフィルタされる |
| Listbox / radio group | 矢印キーで選択を移動。グループ全体で1つのTabストップ |

共通のルール:

- Escapeは最後に開いたものを閉じる: ツールチップ、次にメニュー、その次にダイアログ。
- 複合ウィジェット内の移動はTabではなく矢印キーで行う。Tabはウィジェット間を移動する。
- Tabsはアクティベーションモードを選択する: パネルが即座に描画される場合は自動(矢印キーのフォーカスでパネルが切り替わる)、切り替えにコストがかかる場合は手動(Enter/Spaceでアクティブにする)。
- Enterはフォーカスされた入力欄のフォームを送信する。`<textarea>`ではEnterで改行が挿入され、⌘/Ctrl+Enterで送信される。

## SPAのルート変更

クライアントサイドのナビゲーションは、フォーカスをリセットしたり何かを読み上げたりしない。ルート変更時には: `document.title`を新しいコンテキストに合わせて更新し、次に新しいビューの`<h1>`(`tabindex="-1"`を付与)または`<main>`にフォーカスを移動する。戻る/進むのナビゲーションではスクロール位置を復元し、進むナビゲーションでは先頭にスクロールする。
