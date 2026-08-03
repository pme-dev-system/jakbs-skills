# アニメーション

中断可能なアニメーション、登場・退場トランジション、コンテキストに応じたアイコンアニメーション、モーションの抑制。

## 中断可能なアニメーション

ユーザーはインタラクションの途中で意図を変える。アニメーションが中断可能でないと、インターフェースは壊れているように感じられる。

### CSSトランジション vs. CSSキーフレームアニメーション

| | CSSトランジション | CSSキーフレームアニメーション |
| --- | --- | --- |
| **挙動** | 最新の状態に向けて補間する | 固定のタイムラインで実行される |
| **中断可能か** | 可能。アニメーションの途中で目標値を再設定できる | 不可能。最初からやり直す |
| **使いどころ** | インタラクティブな状態変化(ホバー、トグル、開閉) | 一度だけ実行される段階的なシーケンス(登場アニメーション、ローディング) |
| **持続時間** | 固定。タイムラインではなく値を途中で再設定する | 固定のタイムライン。最初からやり直す |

```css
/* 良い例: トグルの中断可能なトランジション */
.drawer {
  transform: translateX(-100%);
  transition: transform 200ms ease-out;
}
.drawer.open {
  transform: translateX(0);
}

/* アニメーションの途中で再度クリックしても、カクつかずスムーズに逆再生される */
```

```css
/* 悪い例: インタラクティブな要素へのキーフレームアニメーション */
.drawer.open {
  animation: slideIn 200ms ease-out forwards;
}

/* アニメーションの途中で閉じると、スナップしたり最初からやり直したりして、壊れているように感じる */
```

**ルール:** インタラクティブな要素には常にCSSトランジションを優先する。キーフレームは一度きりのシーケンス用に取っておく。

## 登場アニメーション: 分割してスタガーさせる

このパターンは、ページヒーローの初回表示、成功状態、エンプティステートなど、シーケンスが階層構造を伝えるのに役立つ、頻度の低い段階的な登場に使う。大きなコンテナを意味のある塊に分割し、それぞれを個別にアニメーションさせる。行のホバー、キー入力、繰り返されるタブ切り替えのような日常的なインタラクションはスタガーさせない。

### 手順

1. **分割する**: 論理的なグループに分ける(タイトル、説明、ボタン)
2. **スタガーさせる**: グループ間に約100msの遅延を入れる
3. **タイトルの場合**は、個々の単語に分割して約80msでスタガーさせることを検討する
4. **組み合わせる**: `opacity`、`blur`、`translateY`を組み合わせて登場エフェクトを作る

### コード例

```tsx
// Motion(Framer Motion): スタガーさせた登場
function PageHeader() {
  return (
    <motion.div
      initial="hidden"
      animate="visible"
      variants={{
        visible: { transition: { staggerChildren: 0.1 } },
      }}
    >
      <motion.h1
        variants={{
          hidden: { opacity: 0, y: 12, filter: "blur(4px)" },
          visible: { opacity: 1, y: 0, filter: "blur(0px)" },
        }}
      >
        Welcome
      </motion.h1>

      <motion.p
        variants={{
          hidden: { opacity: 0, y: 12, filter: "blur(4px)" },
          visible: { opacity: 1, y: 0, filter: "blur(0px)" },
        }}
      >
        A description of the page.
      </motion.p>

      <motion.div
        variants={{
          hidden: { opacity: 0, y: 12, filter: "blur(4px)" },
          visible: { opacity: 1, y: 0, filter: "blur(0px)" },
        }}
      >
        <Button>Get started</Button>
      </motion.div>
    </motion.div>
  );
}
```

### CSSのみによるスタガー

```css
.stagger-item {
  opacity: 0;
  transform: translateY(12px);
  filter: blur(4px);
  animation: fadeInUp 400ms ease-out forwards;
}

.stagger-item:nth-child(1) { animation-delay: 0ms; }
.stagger-item:nth-child(2) { animation-delay: 100ms; }
.stagger-item:nth-child(3) { animation-delay: 200ms; }

@keyframes fadeInUp {
  to {
    opacity: 1;
    transform: translateY(0);
    filter: blur(0);
  }
}
```

## 退場アニメーション

退場アニメーションは、登場アニメーションよりも穏やかで、注意を引きすぎないものにする。ユーザーの意識は次のものへ移っている最中であり、そこで注意を奪い合うべきではない。

### 控えめな退場(推奨)

```tsx
// 小さな固定値のtranslateY: 大げさにせず方向だけを示す
<motion.div
  exit={{
    opacity: 0,
    y: -12,
    filter: "blur(4px)",
    transition: { duration: 0.15, ease: "easeOut" },
  }}
>
  {content}
</motion.div>
```

### 完全な退場(コンテキストが重要な場合)

```tsx
// 完全にスライドアウトさせる: 空間的なコンテキストが重要な場合に使う
// (例: リストに戻っていくカード、閉じるドロワーなど)
<motion.div
  exit={{
    opacity: 0,
    x: "-100%",
    transition: { duration: 0.2, ease: "easeOut" },
  }}
>
  {content}
</motion.div>
```

### 良い例 vs. 悪い例

```css
/* 良い例: 控えめな退場 */
.item-exit {
  opacity: 0;
  transform: translateY(-12px);
  transition: opacity 150ms ease-out, transform 150ms ease-out;
}

/* 悪い例: 注意を奪う大げさな退場 */
.item-exit {
  opacity: 0;
  transform: translateY(-100%) scale(0.5);
  transition: all 400ms ease-out;
}

/* 状況によっては正しい: モーションがコンテキストを何も伝えない場合は即座に削除する */
.item-exit {
  display: none;
}
```

**要点:**
- コンテナ全体の高さではなく、小さな固定値の`translateY`(例: `-12px`)を使う
- 要素がどこへ移動したかを示す、方向性のある動きを少し残す
- 退場の持続時間は登場より短くする(150ms対300ms)
- 空間的なコンテキストを保持する場合は控えめな退場を使う。モーションが何の情報も加えない場合、そのインタラクションが頻繁に繰り返される場合、またはモーション低減が指定されている場合は即座に削除する

## コンテキストに応じたアイコンアニメーション

アイコンがコンテキストに応じて(ホバー時や状態変化時に)表示・非表示になる場合は、表示・非表示を単純に切り替えるのではなく、`opacity`、`scale`、`blur`でアニメーションさせる。

### Motionの例

この例では`motion`パッケージを使用している。プロジェクトが代わりに`framer-motion`を使っている場合は、同じAPIを`"framer-motion"`からインポートする。インストール済みのパッケージと別パッケージのインポートパスを混在させてはならない。

```tsx
import { AnimatePresence, motion } from "motion/react";

function IconButton({ isActive, icon: Icon }) {
  return (
    <button>
      <AnimatePresence mode="popLayout">
        <motion.span
          key={isActive ? "active" : "inactive"}
          initial={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
          animate={{ opacity: 1, scale: 1, filter: "blur(0px)" }}
          exit={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
          transition={{ type: "spring", duration: 0.3, bounce: 0 }}
        >
          <Icon />
        </motion.span>
      </AnimatePresence>
    </button>
  );
}
```

### CSSトランジションによるアプローチ(Motion不使用)

プロジェクトがMotion(Framer Motion)を使用していない場合は、両方のアイコンをDOMに残し、CSSトランジションでクロスフェードさせる。どちらのアイコンもアンマウントされないため、登場・退場の両方が滑らかにアニメーションする。

仕組み: 一方のアイコンをもう一方の上にabsolute配置する。状態を切り替えると両者はクロスフェードする: 登場するアイコンは`0.25`から拡大し、退場するアイコンは`0.25`まで縮小する。どちらもopacityとblurを伴う。

```tsx
function IconButton({ isActive, ActiveIcon, InactiveIcon }) {
  return (
    <button>
      <div className="relative">
        <div
          className={cn(
            "absolute inset-0 flex items-center justify-center",
            "transition-[opacity,filter,scale] duration-300",
            "ease-[cubic-bezier(0.2,0,0,1)]",
            isActive
              ? "scale-100 opacity-100 blur-0"
              : "scale-[0.25] opacity-0 blur-[4px]"
          )}
        >
          <ActiveIcon />
        </div>
        <div
          className={cn(
            "transition-[opacity,filter,scale] duration-300",
            "ease-[cubic-bezier(0.2,0,0,1)]",
            isActive
              ? "scale-[0.25] opacity-0 blur-[4px]"
              : "scale-100 opacity-100 blur-0"
          )}
        >
          <InactiveIcon />
        </div>
      </div>
    </button>
  );
}
```

absolute配置ではないアイコン(InactiveIcon)がレイアウトサイズを決める。absolute配置のアイコン(ActiveIcon)はフローに影響を与えずにその上に重なる。

### MotionとCSSの使い分け

| | Motion(Framer Motion) | CSSトランジション(両方のアイコンをDOMに残す) |
| --- | --- | --- |
| **登場アニメーション** | 可能 | 可能 |
| **退場アニメーション** | 可能(`AnimatePresence`経由) | 可能(クロスフェード、アイコンは決してアンマウントされない) |
| **スプリング物理演算** | 可能 | 不可。近似として`cubic-bezier(0.2, 0, 0, 1)`を使う |
| **使いどころ** | プロジェクトが既に`motion`または`framer-motion`を使用している場合 | モーション系の依存関係がない場合、またはバンドルを小さく保ちたい場合 |

**ルール:** プロジェクトの`package.json`を確認する。`motion`がインストールされていれば`"motion/react"`から、`framer-motion`がインストールされていれば`"framer-motion"`からインポートする。両方存在する場合は、そのコンポーネントや近くの既存コードで使われているインポートに従う。どちらも存在しない場合はCSSクロスフェードパターンを使う。アイコンのトランジションのためだけに依存関係を追加しない。

### アイコンをアニメーションさせるべき場面

| アニメーションさせる | アニメーションさせない |
| --- | --- |
| ホバー時に表示されるアイコン(アクションボタン) | 常に表示されている静的なナビゲーションアイコン |
| 状態変化を示すアイコン(再生→一時停止、いいね前→いいね後) | 装飾用のアイコン |
| コンテキストに応じたツールバー内のアイコン | 常に表示されているアイコン |
| ローディング/成功状態のインジケーター | アイコンのラベル(アイコンの隣のテキスト) |

**重要:** コンテキストに応じたアイコンアニメーションでは、必ず次の値をそのまま使い、逸脱しない:
- `scale`: `0.25` → `1`(`0.5`や`0.6`は使わない)
- `opacity`: `0` → `1`
- `filter`: `"blur(4px)"` → `"blur(0px)"`
- `transition`: `{ type: "spring", duration: 0.3, bounce: 0 }`。**bounceは必ず`0`にする**。`0.1`など他の値は使わない

## プレス時のスケール

クリック時に控えめに縮小させると、ボタンに触覚的なフィードバックが生まれる。値は必ず`scale(0.96)`にする。`0.95`より小さい値は絶対に使わない。それ以下だと大げさに見える。中断可能にするためCSSトランジションを使う。これにより、ユーザーがプレスの途中で指を離しても、滑らかに元へ戻る。

すべてのボタンにこれが必要なわけではない。モーションが邪魔になる場合にスケール効果を無効化できるよう、ボタンコンポーネントに`static`propを追加する。

### CSSの例

```css
.button {
  transition-property: scale;
  transition-duration: 150ms;
  transition-timing-function: ease-out;
}

.button:active {
  scale: 0.96;
}
```

### Tailwindの例

```tsx
<button className="transition-transform duration-150 ease-out active:scale-[0.96]">
  Click me
</button>
```

### Motionの例

```tsx
<motion.button whileTap={{ scale: 0.96 }}>
  Click me
</motion.button>
```

### staticプロパティのパターン

スケール用のクラスを変数として切り出し、`static`propに応じて適用するかどうかを条件分岐させる:

```tsx
const tapScale = "active:not-disabled:scale-[0.96]";

function Button({ static: isStatic, className, children, ...props }) {
  return (
    <button
      className={cn(
        "transition-transform duration-150 ease-out",
        !isStatic && tapScale,
        className,
      )}
      {...props}
    >
      {children}
    </button>
  );
}

// 使用例
<Button>Click me</Button>           {/* プレス時にスケールする */}
<Button static>Submit</Button>       {/* スケールしない */}
```

## ページ読み込み時はアニメーションをスキップ

`AnimatePresence`に`initial={false}`を指定し、初回レンダリング時に登場アニメーションが発火しないようにする。すでにデフォルト状態にある要素は、ページ読み込み時にはアニメーションさせず、その後の状態変化のときだけアニメーションさせるべきである。

### うまくいく場合

```tsx
// 良い例: アイコンはマウント時にはアニメーションせず、状態変化のときだけアニメーションする
<AnimatePresence initial={false} mode="popLayout">
  <motion.span
    key={isActive ? "active" : "inactive"}
    initial={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
    animate={{ opacity: 1, scale: 1, filter: "blur(0px)" }}
    exit={{ opacity: 0, scale: 0.25, filter: "blur(4px)" }}
  >
    <Icon />
  </motion.span>
</AnimatePresence>
```

これがうまく機能するのは、アイコンの切り替え、トグル、タブ、セグメントコントロールなど、ページ読み込み時にデフォルト状態を持つあらゆるものである。

### うまくいかない場合

コンポーネントが、スタガーされたページヒーローやローディング状態のように、初回登場アニメーションのセットアップを`initial`propに依存している場合は、`initial={false}`を使わない。そのような場合、初回アニメーションを取り除くと登場演出全体がスキップされてしまう。

```tsx
// 悪い例: initial={false}にすると、スタガーされたページの登場が完全にスキップされてしまう
<AnimatePresence initial={false}>
  <motion.div initial="hidden" animate="visible" variants={...}>
    ...
  </motion.div>
</AnimatePresence>
```

これを適用する前に、ページを完全にリフレッシュしてもコンポーネントが正しく見えるか確認する。

## モーションの抑制

モーションは予算であり、飾りではない。あるアニメーションを実装すべきかどうかは、次の3つのルールで判断する:

- **高頻度のインタラクションにカスタムアニメーションを使わない。** ユーザーが絶えずトリガーするもの(すべてのキー入力、すべてのリスト行のホバー、業務ツールでのすべてのタブ切り替え)へのアニメーションは、トリガーのたびに注意のコストを課す。表現力のあるモーションは、頻度の低い場面(ビューの初回表示、成功状態、エンプティステート)のために取っておく。高頻度のインタラクションには、即座のフィードバックか、可能な限り控えめなトランジション(`opacity`/`background-color`を≤150msで)を使う。
- **モーションを唯一のフィードバック手段にしない。** アニメーションが伝える状態変化は、アニメーションが実行されないときにも見えなければならない: 色の変化、アイコンの切り替え、ラベルなど。モーション低減を有効にしているユーザーや、まばたきをした人にも、何が起きたかが見える必要がある。
- **目立つことより、短く的確であることを優先する。** より短く小さなアニメーションで同じことを伝えられるなら、そちらを使う。迷ったら、明確さではなく持続時間を削る。

```css
/* 良い例: 高頻度のホバーには最小限のトランジションを使う */
.row:hover {
  background-color: var(--surface-hover);
  transition: background-color 100ms ease-out;
}

/* 悪い例: ホバーのたびにフルの登場演出が再生される */
.row:hover .row-icon {
  animation: bounceIn 500ms;
}
```

`prefers-reduced-motion`への対応は`better-accessibility`スキルが扱う。このファイル内のすべてのアニメーションに適用すること。
