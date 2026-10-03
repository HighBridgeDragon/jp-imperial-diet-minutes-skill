# インストールガイド / Installation Guide

本スキル（`jp-imperial-diet-minutes`）の各種 AI エージェントおよびクライアントへの導入手順と動作条件です。

## CLI / パッケージマネージャ

対応クライアント: Claude Code, Cursor, GitHub Copilot CLI, Gemini CLI ほか

```bash
npx skills add HighBridgeDragon/jp-imperial-diet-minutes-skill
```

上記コマンドでお使いのエージェント環境（プロジェクトの `.agents/skills` やグローバル設定など）に自動インストールされます。

## デスクトップ / Web アプリ

[Releases](https://github.com/HighBridgeDragon/jp-imperial-diet-minutes-skill/releases) に添付されている `jp-imperial-diet-minutes.zip` をダウンロードして利用します。

### claude.ai / Claude Desktop

1. [Releases](https://github.com/HighBridgeDragon/jp-imperial-diet-minutes-skill/releases) から `jp-imperial-diet-minutes.zip` をダウンロードします。
2. Customize > Skills を開き、「+」→「+ Create skill」→「Upload a skill」の順に選んで `jp-imperial-diet-minutes.zip` をアップロードします。

> [!IMPORTANT]
> アップロードできるのは **Releases に添付された `jp-imperial-diet-minutes.zip`** だけです。GitHub リポジトリ画面の **Code > Download ZIP** や Releases の **Source code (zip)** で取得した zip は、展開時のルートが `jp-imperial-diet-minutes-skill-<ref>/`（直下に `jp-imperial-diet-minutes/`）になり `SKILL.md` が直下に来ないため、skill として認識されません。

Custom Skill は claude.ai・Claude API・Claude Code の間で同期しません。Claude Code に導入済みでも、claude.ai では別途アップロードが必要です。

#### 動作条件（claude.ai / Claude Desktop）

zip を導入しても、以下を満たさない環境では動作しません。

- **プラン**: Free / Pro / Max / Team / Enterprise のいずれかであること。
- **コード実行**: 有効になっていること。本スキルは `curl` 等の HTTP 呼び出しをコード実行で行います。
- **ネットワークアクセス**: サンドボックスから `teikokugikai-i.ndl.go.jp` へ到達できること。claude.ai の既定の許可ドメインはパッケージマネージャー等に限られ、`teikokugikai-i.ndl.go.jp` は含まれません（[Approved network domains](https://support.claude.com/en/articles/12111783-create-and-edit-files-with-claude)）。Team / Enterprise では、組織オーナーが Organization settings > Capabilities で許可ドメインに `teikokugikai-i.ndl.go.jp` を追加する必要があります。個人プラン（Free / Pro / Max）には許可ドメインを追加する設定が無いため、Claude Code 経由をご利用ください。
- **Claude API 経由**: API の Skills サンドボックスはネットワークアクセスを持たないため、原理的に帝国議会会議録 API を呼び出せません。

### OpenAI Codex

[OpenAI Codex のスキル仕様](https://developers.openai.com/codex/skills/) に準拠した配置手順です。

1. [Releases](https://github.com/HighBridgeDragon/jp-imperial-diet-minutes-skill/releases) から `jp-imperial-diet-minutes.zip` をダウンロードして展開します。
2. 展開された `jp-imperial-diet-minutes` フォルダ（直下に `SKILL.md` があるフォルダ）を、ユーザー共通スキルディレクトリ（`~/.agents/skills/` 直下）またはプロジェクトの `.agents/skills/` 直下に配置します（配置後のパス: `~/.agents/skills/jp-imperial-diet-minutes/SKILL.md`。二重フォルダ `jp-imperial-diet-minutes/jp-imperial-diet-minutes/` にならないようご注意ください）。

> [!NOTE]
> 上記の配置パスは OpenAI Codex 公式ドキュメントに基づく仕様です。ChatGPT Desktop 等におけるローカルスキルの読み込み仕様や対応状況については、OpenAI の公式アナウンスをご確認ください。

### Goose

Block 主導のオープンソースエージェント Goose は [Agent Skills オープン標準](https://agentskills.io/clients) に対応しています。

1. [Releases](https://github.com/HighBridgeDragon/jp-imperial-diet-minutes-skill/releases) から `jp-imperial-diet-minutes.zip` をダウンロードして展開します。
2. スキルの配置先や読み込み方法については、[Goose 公式ドキュメント](https://block.github.io/goose/) の指示に従ってください。なお、HTTP 呼び出しを実行できるシェル環境が必要です。

### Google Gemini についての注意

- **Gemini CLI / Google Antigravity**: Agent Skills（`SKILL.md`）仕様に準拠しており、ローカル端末上で動作します。
- **Web 版 Gemini（gemini.google.com）**: シェル実行機構を持たないため、本スキルは動作しません。Gemini CLI または Google Antigravity をご利用ください。

## 実行環境と依存関係

HTTPS GET で生の JSON を取得できる手段が 1 つあれば動作します。本スキルに同梱スクリプトは無く、素の HTTP 呼び出しで使います。

| 手段 | 備考 |
| --- | --- |
| curl（POSIX シェル） | 最も単純です。値は UTF-8 のパーセントエンコード済みで渡します |
| PowerShell | `Invoke-RestMethod` または `curl.exe` を使います。5.1 の `curl` は `Invoke-WebRequest` の別名なので使いません |
| `mcp-server-fetch` 等の MCP フェッチ | 生の応答を返すものに限ります |
| `WebFetch` 等の要約モデル経由のツール | 使いません。応答に無いフィールド（`summary` 等）が混入する事象が、同じ NDL の国会会議録 API で観測されています |

### Windows の文字コード

- Git Bash から `curl --data-urlencode` で日本語を渡すと引数が CP932 で送られ、CloudFront が本文なしの HTTP 403 を返すことを実測で確認しています。UTF-8 でパーセントエンコードした URL を組み立てて渡してください（手順は SKILL.md の「URL の組み立て」）。
- `mcp-server-fetch` を Windows で使う場合は、文字化け対策に `PYTHONIOENCODING=utf-8` を設定します。

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"],
      "env": {
        "PYTHONIOENCODING": "utf-8"
      }
    }
  }
}
```

## 利用上の注意

帝国議会会議録検索システム API への並列呼び出しは避け、連続呼び出しは 3 秒以上空けてください。本スキルはこの制約を SKILL.md の記述で守る設計です。手動で繰り返し呼び出す場合も、呼び出し間隔にご注意ください。

## 出典

- [Agent Skills (agentskills.io)](https://agentskills.io)
- [Agent Skills Overview (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [How to create custom Skills (Claude Help)](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [Using Skills in Claude (Claude Help)](https://support.claude.com/en/articles/12512180-using-skills-in-claude)
- [Build skills (OpenAI Codex)](https://developers.openai.com/codex/skills/)
- [Goose (Block)](https://block.github.io/goose/)
- [帝国議会会議録検索システム](https://teikokugikai-i.ndl.go.jp/)
