<a href="https://interfaces.dev/">
  <img width="320" height="168" alt="interfaces.dev" src="https://ho1jr3x2dcwdu3t5.public.blob.vercel-storage.com/interfaces-og-image.png" />
</a>

[![skills.sh](https://skills.sh/b/jakubkrehel/skills)](https://skills.sh/jakubkrehel/skills)

優れたインターフェースを構築するさまざまな側面を支援する、エージェントスキルのコレクション。アニメーションやUIの磨き込みから、アクセシビリティやプロダクトライティングまでを扱う。

## スキル

- [**better-interface**](skills/better-interface/SKILL.md): ユーザーが呼び出す、下記すべてのスキルを統括する分野横断的なインターフェースレビュー。
- [**better-ui**](skills/better-ui/SKILL.md): インターフェースに磨き込まれた印象を与えるデザインエンジニアリングの細部: 角丸、シャドウ、アニメーション、マイクロインタラクション。
- [**better-typography**](skills/better-typography/SKILL.md): フォント選定から、スペーシング、折り返し、アクセシビリティまでのWebタイポグラフィ。
- [**better-colors**](skills/better-colors/SKILL.md): OKLCH色空間: パレット生成、コントラスト、色域の扱い、テーマ設定。
- [**better-accessibility**](skills/better-accessibility/SKILL.md): フォーカス状態、キーボード操作、ARIA、フォーム、スクリーンリーダー、ヒットエリア、モーション。
- [**better-layout**](skills/better-layout/SKILL.md): レイアウト構造、グルーピング、整列、読み順、プログレッシブディスクロージャー、アダプティブなブレークポイント。
- [**better-writing**](skills/better-writing/SKILL.md): ボタンラベルからエラー、設定、空状態まで、UXライティングとインターフェースの文言。

## インストール

### Claude Codeプラグインとして

7つのスキルすべてを一括インストールし、その場で更新する。Claude Code内で以下を実行する:

```text
/plugin marketplace add jakubkrehel/skills
/plugin install interfaces@interfaces
```

### skills CLIを使う

Claude Code、Codex、その他のエージェントで動作する。インストールするスキルを個別に選ぶことも、すべてインストールすることもできる。`better-interface` は他の6つのスキルを統括するので、包括的なレビューをしたい場合はコレクション全体をインストールする。

```bash
npx skills add jakubkrehel/skills
```

```bash
npx skills add jakubkrehel/skills --skill '*'
```

## 使い方

デフォルトのレビューモードは `full`。より短いレビューにしたい場合は `quick` を渡し、モードの後に画面・フロー・機能名を追加する。

Claude Codeでプラグインとして使う場合。プラグインのスキルは名前空間化されるため、すべてのスキルに `interfaces:` という接頭辞が付く。

```text
/interfaces:better-interface
/interfaces:better-interface quick
/interfaces:better-interface full checkout flow
```

Claude Codeでskills CLIを使ってインストールした場合:

```text
/better-interface
/better-interface quick
/better-interface full checkout flow
```

Codexの場合:

```text
$better-interface
$better-interface quick
$better-interface full checkout flow
```

この接頭辞は、名前を指定して呼び出すスキルにのみ影響する。他の6つのスキルは、いずれの場合も文脈から自動的に適用される。
