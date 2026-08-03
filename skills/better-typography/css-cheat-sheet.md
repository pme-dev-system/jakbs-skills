# CSSチートシート

本スキルが扱うすべてのタイポグラフィ関連CSS宣言について、Tailwind 4の等価表現とともに1行で参照できる。ユーティリティが存在しない場合は、任意値（arbitrary-value）形式を示す。プロジェクトに合った列を選ぶ。プレーンなCSS、CSS Modules、styled-components、StyleXのコードベースでは宣言を、Tailwindのコードベースではユーティリティを使う。

## フォント

| Declaration | What it does | Tailwind |
| --- | --- | --- |
| `font-family: sans-serif` | サンセリフファミリー | `font-sans` |
| `font-family: serif` | セリフファミリー | `font-serif` |
| `font-family: monospace` | モノスペースファミリー | `font-mono` |
| `font-size` | タイプスケールからのサイズ | `text-*` |
| `font-weight` | 1から1000までの任意の値 | `font-*` |
| `font-style: italic` | italicスタイルに切り替える | `italic` |
| `-webkit-font-smoothing` + `-moz-osx-font-smoothing` | macOSのフォントレンダリングを滑らかにする。ルートで一度適用する | `antialiased` |
| `font-synthesis: none` | フォールバックと強調表現を検証した後、すべての合成書体を無効化する | `[font-synthesis:none]` |
| `font-feature-settings` | OpenType機能を切り替える | `[font-feature-settings:"ss01"]` |
| `font-variation-settings` | 可変フォントの軸を調整する | `[font-variation-settings:"GRAD"_80]` |
| `font-optical-sizing` | サイズごとに詳細を調整する | `[font-optical-sizing:auto]` |
| `font-variant-caps` | 本物のスモールキャピタル | `[font-variant-caps:small-caps]` |
| `font-variant-position` | 本物の上付き・下付き文字 | `[font-variant-position:super]` |
| `font-variant-numeric: tabular-nums` | 等幅の数字 | `tabular-nums` |
| `font-variant-numeric: slashed-zero` | 0とOを区別する | `slashed-zero` |

## 間隔とレイアウト

| Declaration | What it does | Tailwind |
| --- | --- | --- |
| `letter-spacing` | 文字間のスペース | `tracking-*` |
| `line-height` | 行間のスペース | `leading-*` |
| `font-kerning` | カーニングのオン/オフ | `[font-kerning:none]` |
| `text-box: trim-both` | 上下の余白をトリムする | `[text-box:trim-both_cap_alphabetic]` |
| `max-width`（テキストカラムに対して） | 1行あたり60〜75文字前後に上限を設ける | `max-w-xl` / `max-w-2xl` / `max-w-[65ch]` |
| `text-align` | 行の始まりと終わりの位置 | `text-start` / `text-center` |

## 折り返しとoverflow

| Declaration | What it does | Tailwind |
| --- | --- | --- |
| `text-wrap: balance` | 見出しの行を均等にする | `text-balance` |
| `text-wrap: pretty` | オーファン語を避ける | `text-pretty` |
| `text-overflow: ellipsis` | 切り詰められたテキストに省略記号を付ける | `truncate` |
| `line-clamp` | N行後に打ち切る | `line-clamp-*` |
| `overflow-wrap: break-word` | 長い文字列を改行する | `break-words` |
| `white-space: nowrap` | 折り返しを止める | `whitespace-nowrap` |
| `text-transform` | 大文字小文字を変更する | `uppercase` / `capitalize` |

## 装飾とインタラクション

| Declaration | What it does | Tailwind |
| --- | --- | --- |
| `text-decoration-line: underline` | アンダーラインを描画する | `underline` |
| `text-decoration-color` | アンダーラインの色 | `decoration-*` |
| `text-decoration-thickness` | アンダーラインの太さ | `decoration-1` / `decoration-2` |
| `text-underline-offset` | 線を下に押し下げる | `underline-offset-*` |
| `text-underline-position: from-font` | フォントに基づくアンダーラインの位置 | `[text-underline-position:from-font]` |
| `text-decoration-style` | dotted、dashed、wavy | `decoration-dotted` / `decoration-wavy` |
| `text-decoration-thickness: from-font` | フォントが指定するアンダーライン | `decoration-from-font` |
| `text-decoration-skip-ink` | ディセンダー周りの隙間 | `[text-decoration-skip-ink:auto]` |
| `caret-color` | テキストカーソルの色を変える | `caret-*` |
| `user-select: none` | 検証済みのドラッグ/ジェスチャーの衝突がある場合のみセレクションを抑制する | `select-none` |
| `text-shadow` | 文字の背後の影 | `text-shadow-*` |
| `-webkit-text-stroke` | 文字にアウトラインを付ける | `[-webkit-text-stroke:1px_black]` |
| `background-clip: text` | 背景を文字の形にクリップする | `bg-clip-text` |
| `initial-letter` | ドロップキャップのサイズを指定する | `[initial-letter:3]` |
