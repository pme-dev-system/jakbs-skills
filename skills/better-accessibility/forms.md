# フォーム

ラベル、autocomplete、エラーメッセージ、入力タイプ、送信の挙動。

## ラベル

すべてのコントロールにはプログラム的なラベルが必要である: 入力欄の`id`を指す`<label for>`、またはラップする`<label>`。プレースホルダーは決してラベルにはならない。ユーザーが入力した瞬間に消え、たいていコントラストも基準を満たさない。

```html
<!-- 良い例: 明示的な関連付け -->
<label for="email">Email</label>
<input id="email" type="email" autocomplete="email" />

<!-- 良い例: ラップするラベルにより、ラベルとコントロールが1つのヒットターゲットを共有する -->
<label>
  <input type="checkbox" /> Send me updates
</label>
```

ラベルとコントロールは1つのヒットターゲットを共有しなければならない: 「Send me updates」というテキストをクリックするとチェックボックスがトグルし、両者の間にデッドゾーンはない。必須フィールドにはネイティブの`required`に加え、フォームごとに一度説明する可視のインジケーター(「* required」)を付ける。

プレースホルダーをラベルに*加えて*使う場合は、期待される形式の例を示す: `placeholder="name@company.com"`。

## エラーメッセージ

完全なパターン:

```html
<label for="email">Email</label>
<input
  id="email"
  type="email"
  autocomplete="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>
<p id="email-error">Enter a valid email address.</p>
```

- 失敗したフィールドには`aria-invalid="true"`を付け、修正されたら削除する。
- `aria-describedby`はフィールドとインラインのエラーを結びつけ、スクリーンリーダーがフィールドと一緒に読み上げるようにする。
- エラーはフィールドの隣にインラインで表示し、アイコンかテキストを伴わせる。赤いボーダーだけは決して使わない(色のみの手がかりは基準を満たさない)。
- 送信時には最初の無効なフィールドにフォーカスする。
- バリデーションが表面化するよう、不完全な送信を許可する。有効になるまで送信を無効化しない(後述)。
- 自由なテキストを受け付け、後から検証する。ユーザーが入力している最中に入力をブロックしたり文字をフィルタしたりしない。検証前に値をトリムする。autocompleteやテキスト展開は末尾にスペースを追加する。

## autocompleteと入力タイプ

意味のある`name`を伴う`autocomplete`はワンタップでフォームを埋めることができ、ユーザーに関するフィールドについてはWCAGの要件(1.3.5)でもある。よく使うトークンは次の通り:

| フィールド | `autocomplete` |
| --- | --- |
| 名前 | `name`(または`given-name` / `family-name`) |
| メール | `email` |
| 電話番号 | `tel` |
| 住所 | `street-address`、`address-line1`、`postal-code`、`country` |
| カード | `cc-number`、`cc-exp`、`cc-csc`、`cc-name` |
| ログイン | `username`、`current-password` |
| 新規登録 / リセット | `new-password` |
| 2FAコード | `one-time-code` |

該当する場合はセクションを接頭辞として付ける: `autocomplete="shipping street-address"`。

正しい`type`と`inputmode`で適切なモバイルキーボードを選ばせる:

| 入力 | 使用する値 |
| --- | --- |
| メール、URL、電話番号 | `type="email"`、`type="url"`、`type="tel"` |
| OTP / PIN / カード番号 | `type="text" inputmode="numeric"`(テキストのセマンティクスを維持し、スピナーを出さない) |
| 金額、小数 | `type="text" inputmode="decimal"` |
| 純粋な数量 | `type="number"` |

メール、コード、ユーザー名ではスペルチェックを無効にする: `spellcheck="false"`。

## ユーザーのツールを妨げない

- `<input>`や`<textarea>`でのペーストを決してブロックしない。ユーザーはパスワードやワンタイムコードをペーストする。
- パスワードマネージャーと2FAの自動入力に対応し続ける: 本物の`<form>`、正しい`autocomplete`、偽の入力欄を作らない。
- iOSの入力ズームを止めるために`user-scalable=no`や`maximum-scale=1`を決して使わない。代わりにモバイルの入力テキストを`16px`に保つ(`better-typography`が扱う)。

## 送信の挙動

- 送信ボタンはリクエストが開始するまで有効のままにし、開始したら無効化してスピナーを表示するが、*元のラベルは保つ*: スピナー付きの「Save」であって、スピナー単体ではない。ラベルこそが、どのボタンが処理中かを支援技術に伝える。
- 結果を読み上げる: 成功はpoliteなライブリージョンを通す。送信失敗の場合は最初の無効なフィールドにフォーカスする。フォーカスの移動そのものが通知であり、`role="alert"`はフィールドに紐づかないフォームレベルのエラー用に確保する([screen-readers.md](screen-readers.md)参照)。
- ナビゲーション前に未保存の変更を警告し、再レンダリングによって入力済みのテキストを決して失わないようにする。ハイドレーションはフォーカスと値を保持しなければならない。
- Enterはフォーカスされたどの入力欄からでも送信する。`<textarea>`では⌘/Ctrl+Enterで送信する。
