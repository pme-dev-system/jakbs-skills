# 色域の意識とTailwind v4

## sRGBとDisplay P3

すべてのsRGBカラーはDisplay P3内に存在するが、すべてのP3カラーがsRGB内に存在するわけではない。Display P3は約50%多くの色をカバーする。

## 最大彩度は明度と色相によって変わる

色域の境界は不規則である。sRGBでL=0.5の場合:

- 最高彩度: 紫(H ≈ 285)でC ≈ 0.29
- 赤橙(H ≈ 0-30): C ≈ 0.20
- 最低彩度: シアン(H ≈ 195)でC ≈ 0.09

ピークとなる色相は明度によって変化する。L=0.7ではマゼンタがピークになり、L=0.9では緑がピークになる。シアンは一貫して最大彩度が最も低い。

## 色域チェック

ある色の彩度が、そのL/H/色空間における最大値を超えると、クリップする。修正方法: LとHを一定に保ちながら彩度を下げる。

```css
/* sRGBの色域外 */
oklch(0.7 0.35 150)

/* 最大彩度にクランプ */
oklch(0.7 0.22 150)
```

## CSSフォールバックのパターン

```css
/* すべてのブラウザ向けのsRGBフォールバック */
.accent {
  color: oklch(0.7 0.2 150);
}

/* より広い色域のディスプレイ向けのP3拡張 */
@media (color-gamut: p3) {
  .accent {
    color: oklch(0.7 0.3 150);
  }
}
```

oklchをサポートしないブラウザ向け:

```css
.accent {
  color: #4ade80;
}

@supports (color: oklch(0 0 0)) {
  .accent {
    color: oklch(0.7 0.2 150);
  }

  @media (color-gamut: p3) {
    .accent {
      color: oklch(0.7 0.3 150);
    }
  }
}
```

## Tailwind v4

Tailwind CSS v4は、デフォルトパレットをoklchで定義している。カスタムテーマも同じ規約に従うべき。

### @themeによるカスタムカラースケール

```css
@theme {
  --color-brand-50: oklch(0.971 0.012 250);
  --color-brand-100: oklch(0.932 0.028 250);
  --color-brand-200: oklch(0.882 0.048 250);
  --color-brand-300: oklch(0.812 0.078 250);
  --color-brand-400: oklch(0.722 0.148 250);
  --color-brand-500: oklch(0.623 0.188 250);
  --color-brand-600: oklch(0.535 0.168 250);
  --color-brand-700: oklch(0.445 0.138 250);
  --color-brand-800: oklch(0.362 0.108 250);
  --color-brand-900: oklch(0.289 0.078 250);
  --color-brand-950: oklch(0.215 0.048 250);
}
```

これにより、`bg-brand-500`や`text-brand-200`などが自動的に使えるようになる。

### 不透明度修飾子

Tailwindの不透明度修飾子構文はoklchでも機能する:

```html
<div class="bg-brand-500/50"></div>
<!-- コンパイル結果: oklch(0.623 0.188 250 / 0.5) -->
```

### 既存テーマの移行

1. `@theme`内のすべてのhex値をoklchに変換する
2. hexを使っていた`theme()`参照をすべて置き換える
3. ダークモードをテストする: 知覚的な精度により、oklch値はわずかに異なって見えることがある
4. コンポーネントコード内にハードコードされたhexがないか確認し、それらも変換する
