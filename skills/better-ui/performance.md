# パフォーマンス

トランジション対象の指定とGPU合成のヒント。

## 変化するプロパティだけをトランジションする

`transition: all`やTailwindの`transition-all`は絶対に使わない。変化する正確なプロパティを必ず指定する。(Tailwindの単体の`transition`は、`all`ではなく色・opacity・shadow・transformを厳選したデフォルトリストにマッピングされる。それでも、変化するものを正確に指定することを優先する。)

### 理由

- `transition: all`はブラウザにすべてのプロパティの変化を監視させることになる
- 意図していないプロパティ(色、パディング、シャドウ)で予期しないトランジションが起きる
- ブラウザの最適化を妨げる

### CSSの例

```css
/* 良い例: 変化するものだけをトランジションする */
.button {
  transition-property: scale, background-color;
  transition-duration: 150ms;
  transition-timing-function: ease-out;
}

/* 悪い例: すべてをトランジションする */
.button {
  transition: all 150ms ease-out;
}
```

### Tailwind

```tsx
// 良い例: プロパティを明示する
<button className="transition-[scale,background-color] duration-150 ease-out">

// 悪い例: すべてをトランジションする
<button className="transition-all duration-150 ease-out">
```

### Tailwindの`transition-transform`に関する注意

Tailwindの`transition-transform`は`transition-property: transform, translate, scale, rotate`にマッピングされる。つまり`transform`だけでなく、transform関連のプロパティすべてをカバーする。transformのみをアニメーションさせる場合はこれを使う。transform以外の複数のプロパティには、ブラケット構文を使う: `transition-[scale,opacity,filter]`。

## `will-change`は控えめに使う

`will-change`は、要素を専用のGPU合成レイヤーへあらかじめ昇格させるようブラウザにヒントを与える。これを指定しない場合、ブラウザはアニメーションが開始した時点で初めて要素を昇格させる。この一度限りのレイヤー昇格が、最初のフレームでの微妙なカクつきを引き起こすことがある。

これは特に、要素が`scale`や`rotation`を変化させたり、`transform`で動き回ったりする場合に役立つ。それ以外のプロパティに対してはあまり効果がない。そもそもブラウザがそれらをGPUで合成できないためである。

### ルール

```css
/* 良い例: GPU合成の恩恵を受ける具体的なプロパティ */
.animated-card {
  will-change: transform;
}

/* 良い例: 合成に適した複数のプロパティ */
.animated-card {
  will-change: transform, opacity;
}

/* 悪い例: will-change: all は絶対に使わない */
.animated-card {
  will-change: all;
}

/* 悪い例: そもそもGPUで合成できないプロパティ */
.animated-card {
  will-change: background-color, padding;
}
```

### 有用なプロパティ

| プロパティ | GPU合成可能 | `will-change`を使う価値 |
| --- | --- | --- |
| `transform` | 可能 | あり |
| `opacity` | 可能 | あり |
| `filter`(blur、brightness) | 可能 | あり |
| `clip-path` | 新しめのChromiumのみ | ほとんどなし。クロスブラウザで信頼できない |
| `top`、`left`、`width`、`height` | 不可能 | なし |
| `background`、`border`、`color` | 不可能 | なし |

### 省略すべき場面

最近のブラウザは、すでに自力での最適化が得意である。最初のフレームでのカクつきに気づいたときだけ`will-change`を追加する。特にSafariはこれの恩恵を受けやすい。すべてのアニメーション要素に予防的に追加しない。合成レイヤーが増えるたびにメモリコストがかかる。
