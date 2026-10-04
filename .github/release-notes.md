# 導入方法

## CLI / パッケージマネージャ

対応クライアント: Claude Code, Cursor, GitHub Copilot CLI, Gemini CLI ほか

```bash
npx skills add HighBridgeDragon/jp-imperial-diet-minutes-skill
```

## デスクトップ / Web アプリ（zip 導入）

本 Release に添付の `jp-imperial-diet-minutes.zip` をダウンロードして導入します。

- **claude.ai / Claude Desktop**: Skills 設定から `jp-imperial-diet-minutes.zip` をアップロードします（アップロードできるのは本 Release 添付の zip のみです。Source code (zip) や Code > Download ZIP の zip はリポジトリ全体を含み `SKILL.md` が直下に来ないため使えません。また、Custom Skill は面をまたいで同期しないため個別にアップロードが必要です。操作手順詳細は [インストールガイド](https://github.com/HighBridgeDragon/jp-imperial-diet-minutes-skill/blob/main/docs/install.md) を参照）。
- **OpenAI Codex**: `~/.agents/skills/`（またはプロジェクトの `.agents/skills/`）直下に展開後の `jp-imperial-diet-minutes` フォルダを配置します（配置後のパス: `~/.agents/skills/jp-imperial-diet-minutes/SKILL.md`）。二重フォルダ（`jp-imperial-diet-minutes/jp-imperial-diet-minutes/`）にならないようご注意ください。
- **Goose ほか対応クライアント**: 各ツールの設定手順に従って配置します（詳細は下記インストールガイドを参照）。

各クライアント別の詳細な導入手順や動作条件（ネットワーク設定・フェッチ環境等）は [インストールガイド (docs/install.md)](https://github.com/HighBridgeDragon/jp-imperial-diet-minutes-skill/blob/main/docs/install.md) を参照してください。

## 主な動作条件

- **生の HTTP 呼び出し**: 本スキルに同梱スクリプトは無く、`curl` や PowerShell の `Invoke-RestMethod` などで API を直接呼び出します。`WebFetch` のように要約モデルを介するツールは、応答に無い内容が混入するため使えません。
- **ネットワークアクセス**: サンドボックスや実行環境から `teikokugikai-i.ndl.go.jp` へ到達できる必要があります。claude.ai では許可ドメインへの追加が必要となるため、追加できない環境では Claude Code 経由をご利用ください（設定条件の詳細はインストールガイドを参照）。
- **claude.ai / Claude Desktop 利用時の要件**: コード実行の有効化が必要です（詳細はインストールガイドを参照）。
- **Web 版 Gemini / Claude API**: シェル実行サンドボックスやネットワークアクセスを持たないため、原理的に動作しません（CLI やデスクトップ版をご利用ください）。

## 利用上の注意

帝国議会会議録検索システム API への並列呼び出しは避け、連続呼び出しは 3 秒以上空けてください。本スキルはこの制約を SKILL.md の記述で守る設計です。手動で繰り返し呼び出す場合も、呼び出し間隔にご注意ください。

## 出典

- [Agent Skills (agentskills.io)](https://agentskills.io)
- [Agent Skills (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [How to create custom Skills (Claude)](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Using Skills in Claude (Claude Help)](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [Build skills (OpenAI Codex)](https://developers.openai.com/codex/skills/)
- [Goose (Block)](https://block.github.io/goose/)
- [帝国議会会議録検索システム](https://teikokugikai-i.ndl.go.jp/)
