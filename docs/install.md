# インストールガイド / Installation Guide

本スキル（`jp-imperial-diet-minutes`）の導入手順と動作条件。

## CLI / パッケージマネージャ

対応クライアント: Claude Code, Cursor, GitHub Copilot CLI, Gemini CLI ほか

```bash
npx skills add HighBridgeDragon/jp-imperial-diet-minutes-skill
```

上記コマンドで、使用中のエージェント環境（プロジェクトの `.agents/skills` やグローバル設定など）に自動でインストールされる。

## デスクトップ / Web アプリ

claude.ai / Claude Desktop への導入用 zip の配布は未整備（今後対応予定）。以下は動作条件の見込みである。

### 動作条件（claude.ai / Claude Desktop）

- **プラン**: Pro / Max / Team / Enterprise のいずれか。
- **コード実行**: 有効になっていること。本スキルは curl 等の HTTP 呼び出しをコード実行で行う。
- **ネットワークアクセス**: サンドボックスから `teikokugikai-i.ndl.go.jp` へ到達できること。ブロックされる場合は、許可ドメインに `teikokugikai-i.ndl.go.jp` の追加が必要になる見込み。**到達性は未検証**であり、動作するとは断言できない。満たせない環境では Claude Code 経由を使う。
- **Claude API 経由**: API の Skills サンドボックスはネットワークアクセスを持たないため、原理的に帝国議会会議録 API を呼び出せない。

### Google Gemini についての注意

- **Gemini CLI / Google Antigravity**: Agent Skills（`SKILL.md`）仕様に準拠しており、ローカル端末上で動作する。
- **Web 版 Gemini（gemini.google.com）**: シェル実行機構を持たないため、本スキルは動作しない。

## 実行環境と依存関係

HTTPS GET で生の JSON を取得できる手段が 1 つあれば動作する。本スキルに wrapper script は無く、素の HTTP 呼び出しで使う。

| 手段 | 備考 |
| --- | --- |
| curl（POSIX シェル） | 最も単純。値は UTF-8 のパーセントエンコード済みで渡す |
| PowerShell | `Invoke-RestMethod` または `curl.exe`。5.1 の `curl` は `Invoke-WebRequest` の別名なので使わない |
| `mcp-server-fetch` 等の MCP フェッチ | 生の応答を返すものに限る |
| `WebFetch` 等の要約モデル経由のツール | 使わない。応答に無いフィールドが混入する事象が姉妹スキルで観測された |

### Windows の文字コード

- Git Bash の `curl` は日本語の引数を CP932 で送り、CloudFront が本文なしの HTTP 403 を返す。日本語を含む呼び出しで 403 になる原因は文字コードである。UTF-8 でパーセントエンコードした URL を組み立てて渡す（手順は SKILL.md の「URL の組み立て」）。
- `mcp-server-fetch` を Windows で使う場合は、文字化け対策に `PYTHONIOENCODING=utf-8` を設定する。

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

帝国議会会議録検索システム API への並列呼び出しは禁止し、連続呼び出しは 3 秒以上空ける。本スキルはこの制約を SKILL.md の記述で守る設計である。手動で繰り返し呼び出す場合も間隔に注意する。

## 出典

- [Agent Skills (agentskills.io)](https://agentskills.io)
- [Agent Skills Overview (Anthropic)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [How to create custom Skills (Claude Help)](https://support.claude.com/en/articles/12512198-creating-custom-skills)
- [帝国議会会議録検索システム](https://teikokugikai-i.ndl.go.jp/)
