# 細部とアクセシビリティ

アンダーライン、セレクション、フォーム、装飾テキスト、そしてすべてを読みやすく保つための下限値。

## アンダーライン

デフォルトのアンダーラインの位置はブラウザが決定する。近すぎてディセンダーを貫いてしまったり、細すぎたりすることがある。フォント自身のメトリクスから位置と太さを取得する。

```css
a {
  text-underline-position: from-font;
  text-decoration-thickness: from-font;
}
```

線は実線である必要はない。`text-decoration-style`はdotted、dashed、wavyで線を描画できる。dottedのアンダーラインは、略語や定義済み用語のように、単語が追加情報を持っていることを示す一般的なヒントである。

```css
abbr {
  text-decoration: underline dotted;
}
```

あるいは手動で調整する。

```css
a {
  text-decoration-thickness: 1px;
  text-underline-offset: 3px;
  text-decoration-skip-ink: auto;
  text-decoration-color: var(--color-gray-1000);
  transition: text-decoration-color 200ms ease-out;
}

a:hover {
  text-decoration-color: var(--color-gray-1200);
}
```

アニメーションさせるのが色の変化だけでない限り、`text-decoration`を使う代わりにアンダーラインをカスタム要素として構築する。実際のアンダーラインで確実にアニメーションできる部分は色だけである。カスタム要素は、必要な効果に応じて自由にアニメーションさせる。

## セレクション

- `::selection`は選択されたテキストの背景と色を変更する。ブランドをさりげなく組み込む方法である。組み合わせは可読性を保つ。
- アプリケーションのクロームを含め、テキストはデフォルトで選択可能にしておく。ユーザーは、デザイナーが予測しない方法でラベル、識別子、エラー、値をコピーする。
- `user-select: none`は、誤ったセレクションがインタラクションと衝突する特定のドラッグ操作やジェスチャー操作の対象にのみ使う。グローバルに適用したり、ネイティブのクロームを模倣するためだけに適用したりしない。
- `::target-text`は、共有リンクがスクロールする先のフレーズをスタイルする。
- Custom Highlight APIは、検索結果のマッチのように自分で選んだ範囲を、追加のマークアップなしでスタイルする。

## フォームと編集可能なテキスト

- `::placeholder`は空のフィールド内のヒントをスタイルする。
- `caret-color`は点滅する挿入バーの色を変える。キャレットのスタイリングとして可能なのは、ほぼ色の変更までである。完全にカスタムなキャレットの構築は非常に難しく、よほど特定の効果が必要な場合を除き、通常は割に合わない。

### iOSのinputズーム

`16px`より小さいテキストのinputにフォーカスすると、ページ全体がズームされる（これはアクセシビリティ機能である。`16px`はWebのデフォルトであり、Safariはそれより小さいと入力中に読みづらいとみなす）。

```tsx
// Good: モバイルでは16px、smブレークポイント以上ではより小さく
<input className="text-base sm:text-sm" type="email" />
```

対処法として`maximum-scale=1`のviewport metaを使うのは避ける。Safariはピンチズームに対してこの上限を無視するが、他のすべてのブラウザはこれを尊重してピンチズームを制限してしまい、WCAG 1.4.4に違反する。レスポンシブなinputサイズであれば、アクセシビリティを犠牲にすることなくズームの問題を解決できる。

## 装飾的なテキスト

| Property | Effect |
| --- | --- |
| `::first-letter` | ドロップキャップ、幅広くサポートされている |
| `::first-line` | 最初の行のみをスタイルする |
| `initial-letter` | ドロップキャップのサイズを指定する。サポートは限定的で、Firefoxはまだ未対応 |
| `background-clip: text` | 背景やグラデーションを文字の形にクリップする |
| `-webkit-text-stroke` | 文字にアウトラインを付ける。プレフィックスが付いているが、モダンブラウザ全般で動作する |
| `text-shadow` | `box-shadow`に似ているが、文字の形状に沿う |

テキストストロークが文字の内側にも線を描いてしまう場合、それはフォント側の問題である。ストロークはすべての輪郭をなぞり、可変フォントは通常、重なり合う形状を結合せずに保持している。静的フォントにはこの問題はない。

## サイズ

タイポグラフィは、読者がそれを変更しても機能し続けなければならない。ズーム、より大きなブラウザのフォントサイズ、上書きされたline-heightやletter-spacingなど。

| Text | Size |
| --- | --- |
| 長文本文の出発点 | 実際の書体とmeasureで検証した`16px`前後 |
| inputとメニューの出発点 | `14px`前後 |
| キャプション | `13px` |
| 下限 | `12px`を下回ることは滅多にない |

テキストが低コントラストに見える場合は、`better-colors`でレンダリングされた前景/背景のペアを測定し、`better-accessibility`で該当する要件を分類する。ユーザーが是正を求めない限り、プロジェクトの色を変更するかどうかはデザイン上の判断のままである。

## フォントスムージング

macOSでは、テキストは意図より太くレンダリングされる。すべてのテキストに適用されるよう、ルートレイアウトでフォントスムージングを一度適用する。Tailwindの`antialiased`は両方のプロパティを設定する。

```css
html {
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
```

```tsx
<html lang="en">
  <body class="font-sans antialiased">
    <main>{children}</main>
  </body>
</html>
```
