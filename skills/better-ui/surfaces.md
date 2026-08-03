# サーフェス

角丸(border radius)、オプティカルアライメント、シャドウ、画像のアウトライン。

## 同心円状の角丸(border radius)

丸みを帯びた要素を入れ子にするとき、外側の半径は内側の半径とその間のパディングの合計に等しくなければならない:

```
outerRadius = innerRadius + padding
```

このルールは、入れ子になったサーフェス同士が近接している場合に最も役立つ。パディングが`24px`より大きい場合は、それらのレイヤーを別々のサーフェスとして扱い、厳密な同心円状の計算を強制するのではなく、それぞれの半径を個別に選ぶ。

### 例

```css
/* 良い例: 同心円状の半径 */
.card {
  border-radius: 20px; /* 12 + 8 */
  padding: 8px;
}
.card-inner {
  border-radius: 12px;
}

/* 悪い例: 両方とも同じ半径 */
.card {
  border-radius: 12px;
  padding: 8px;
}
.card-inner {
  border-radius: 12px;
}
```

### Tailwindの例

```tsx
// 良い例: 外側の半径がパディングを考慮している
<div className="rounded-2xl p-2">       {/* 16pxの半径、8pxのパディング */}
  <div className="rounded-lg">          {/* 8pxの半径 = 16 - 8 ✓ */}
    ...
  </div>
</div>

// 悪い例: 両方とも同じ半径
<div className="rounded-xl p-2">
  <div className="rounded-xl">          {/* 同じ半径で不自然に見える */}
    ...
  </div>
</div>
```

近接して入れ子になったサーフェス間で角丸(border radius)が揃っていないことは、視覚的な違和感のよくある原因である。レイヤーが目に見える均等な内側の余白を共有している場合は同心円状に計算し、レイヤーが独立しているか、パディングが意図的に非対称である場合は、確立されたコンポーネントトークンを維持する。

## オプティカルアライメント

幾何学的な中央揃えが不自然に見える場合は、代わりにオプティカルアライメント(視覚的な整列)を行う。

### テキスト+アイコンのボタン

アイコンによって、本来対称なパディングが不均衡に見える場合は、アイコン側のパディングをわずかに減らす。目安となる出発点は次の通り:
`icon-side padding = text-side padding - 2px`。

```css
/* 良い例: アイコン側のパディングを減らす */
.button-with-icon {
  padding-inline-start: 16px;
  padding-inline-end: 14px; /* 末尾のアイコン側 = テキスト側 - 2px */
}

/* 悪い例: 均等なパディングだと、アイコンが右に寄りすぎて見える */
.button-with-icon {
  padding-inline: 16px;
}
```

```tsx
// Tailwind
<button className="ps-4 pe-3.5 flex items-center gap-2">
  <span>Continue</span>
  <ArrowRightIcon />
</button>
```

### 再生ボタンの三角形

再生アイコンは三角形であり、その幾何学的な中心は視覚的な中心と一致しない。わずかに右へずらす:

```css
/* 良い例: オプティカルに中央揃え */
.play-button svg {
  transform: translateX(2px); /* グリフ自体への物理的な補正 */
}

/* 悪い例: 幾何学的には中央だが不自然に見える */
.play-button svg {
  /* 調整なし */
}
```

### 非対称なアイコン(星、矢印、キャレット)

一部のアイコンは視覚的な重心が偏っている。最善の修正方法は、コンポーネントのコード側で余分なマージン/パディングが不要になるよう、SVG自体を直接調整することである。

```tsx
// 最善: SVG自体で修正する
// viewBoxまたはpathを調整して、アイコンを視覚的に中央揃えにする

// フォールバック: マージンで調整する
<span className="translate-x-px">
  <StarIcon />
</span>
```

## ボーダーの代わりにシャドウを使う

奥行きや高さ表現(エレベーション)のためにボーダーを使っている**ボタン、カード、コンテナ**では、それを控えめな`box-shadow`に置き換えることを優先する。シャドウは透明度を利用するため、どんな背景にも適応する。単色のボーダーはそうはいかない。これは画像や複数の色を背景に使う場合にも役立つ。単色のボーダーは、それが設計された背景以外ではうまく機能しない。

**区切り線**(`border-b`、`border-t`、サイドボーダー)や、要素の奥行きではなくレイアウトの分離を目的とするボーダーには、これを適用しない。それらはボーダーのままにする。

### ボーダーとしてのシャドウ(ライトモード)

このシャドウは3つのレイヤーで構成される。1つ目は1pxのボーダーリングとして機能し、2つ目はわずかな浮き上がりを加え、3つ目は環境光による奥行きを与える:

```css
:root {
  --shadow-border:
    0px 0px 0px 1px oklch(0 0 0 / 0.06),
    0px 1px 2px -1px oklch(0 0 0 / 0.06),
    0px 2px 4px 0px oklch(0 0 0 / 0.04);
  --shadow-border-hover:
    0px 0px 0px 1px oklch(0 0 0 / 0.08),
    0px 1px 2px -1px oklch(0 0 0 / 0.08),
    0px 2px 4px 0px oklch(0 0 0 / 0.06);
}
```

### ボーダーとしてのシャドウ(ダークモード)

ダークモードでは、単一の白いリングに簡略化する。レイヤー化された奥行きシャドウは、暗い背景では見えないためである:

```css
/* ダークモード: プロジェクトが使用しているどの仕組みにも合わせる
   (prefers-color-scheme、class、data属性など) */
--shadow-border: 0 0 0 1px oklch(1 0 0 / 0.08);
--shadow-border-hover: 0 0 0 1px oklch(1 0 0 / 0.13);
```

### ホバートランジションでの使用

変数を適用し、スムーズなホバーのために`transition-[box-shadow]`を追加する:

```css
.card {
  box-shadow: var(--shadow-border);
  transition-property: box-shadow;
  transition-duration: 150ms;
  transition-timing-function: ease-out;
}

.card:hover {
  box-shadow: var(--shadow-border-hover);
}
```

### シャドウとボーダーの使い分け

| シャドウを使う | ボーダーを使う |
| --- | --- |
| 奥行きのあるカード、コンテナ | リスト項目間の区切り線 |
| ボーダースタイルのボタン | テーブルセルの境界 |
| 高さ表現のある要素(ドロップダウン、モーダル) | フォーム入力欄のアウトライン(アクセシビリティのため) |
| さまざまな背景の上にある要素 | 密なUIにおけるヘアラインの区切り |
| 浮き上がり効果を出すホバー/フォーカス状態 | |

## 画像のアウトライン

画像には、不透明度の低い`1px`のアウトラインを追加する。これにより、特に他の要素がボーダーやシャドウを使っているデザインシステムにおいて、一貫した奥行きが生まれる。

### 色のルール(絶対厳守)

- **ライトモード**: 純粋な黒、`oklch(0 0 0 / 0.1)`。
- **ダークモード**: 純粋な白、`oklch(1 0 0 / 0.1)`。
- プロジェクトのパレットから黒や白に近い色(例: slate-900、zinc-900、`#0a0a0a`、`#111827`、`#f5f5f7`)を使わない。色味のついたアウトラインは周囲のサーフェスの色を拾ってしまい、画像の端が汚れて見える。
- アウトラインをプロジェクトのアクセントカラーやインクカラーに合わせない。アウトラインはテーマに沿った要素ではなく、ニュートラルな区切りである。

### ライトモード

```css
img {
  outline: 1px solid oklch(0 0 0 / 0.1);
  outline-offset: -1px; /* リングを画像の端のすぐ内側に描く */
}
```

### ダークモード

```css
img {
  outline: 1px solid oklch(1 0 0 / 0.1);
  outline-offset: -1px;
}
```

### ダークモード対応のTailwind

```tsx
<img
  className="outline outline-1 -outline-offset-1 outline-black/10 dark:outline-white/10"
  src={src}
  alt={alt}
/>
```

`outline-black/10`と`outline-white/10`を具体的に使い、`outline-slate-*`、`outline-zinc-*`、`outline-neutral-*`など色味のついたスケールは使わない。

**なぜborderではなくoutlineなのか?** `outline`はどのオフセットでも幅や高さを追加せず、レイアウトに影響を与えない。また`outline-offset: -1px`によってリングが画像の端のすぐ内側に描かれるため、外側にはみ出さずに角丸(border radius)にぴったり沿う。
