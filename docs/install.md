# インストールガイド / Installation Guide

本スキル（`jp-imperial-diet-minutes`）の各種 AI エージェントおよびクライアントへの導入手順と動作条件です。

## CLI / パッケージマネージャ

対応クライアント: Claude Code, Cursor, GitHub Copilot CLI, Gemini CLI ほか

```bash
npx skills add HighBridgeDragon/jp-imperial-diet-minutes-skill
```

上記コマンドでお使いのエージェント環境（プロジェクトの `.agents/skills` やグローバル設定など）に自動インストールされます。

## 各種クライアントへの導入手順（デスクトップ / Web アプリ）

Claude Desktop, claude.ai, OpenAI Codex, Goose, Gemini CLI 等への共通導入手順は、以下の共通ガイドをご確認ください。

👉 [**クライアント別共通インストールガイド (jp-skills-shared)**](https://github.com/HighBridgeDragon/jp-skills-shared/blob/main/docs/install-guide.md)

- Releases 添付の `jp-imperial-diet-minutes.zip` を用いた導入手順
- 各クライアントでの配置先および動作要件

## 本スキル固有の動作要件・必須設定

### claude.ai / Claude Desktop の許可ドメイン

claude.ai のコード実行環境から本スキルを利用する場合、以下のドメインへの外部アクセス許可が必要です：

- **許可ドメイン**: `teikokugikai-i.ndl.go.jp`

組織オーナー（Organization Owner）による許可ドメインへの追加設定を行ってください（手順詳細は上記共通ガイドの「動作条件」節を参照。個人プランには追加設定がありません）。許可ドメインを追加できない環境では、Claude Code 経由をご利用ください。

### 実行環境とフェッチ手段の注意

HTTPS GET で生の JSON を取得できる手段が 1 つあれば動作します。**本スキルに同梱スクリプトは無く、素の HTTP 呼び出し**で使います。

| 手段 | 備考 |
| --- | --- |
| curl（POSIX シェル） | 最も単純です。値は UTF-8 のパーセントエンコード済みで渡します |
| PowerShell | `Invoke-RestMethod` または `curl.exe` を使います。5.1 の `curl` は `Invoke-WebRequest` の別名なので使いません |
| `mcp-server-fetch` 等の MCP フェッチ | 生の応答を返すものに限ります |
| `WebFetch` 等の要約モデル経由のツール | **使いません（禁止）**。応答に無いフィールド（`summary` 等）が混入する事象が、同じ NDL の国会会議録 API で観測されています |

### Windows の文字コード注意

- Git Bash から `curl --data-urlencode` で日本語を渡すと引数が CP932 で送られ、CloudFront が本文なしの HTTP 403 を返すことを実測で確認しています。UTF-8 でパーセントエンコードした URL を組み立てて渡してください（手順は SKILL.md の「URL の組み立て」）。

### 利用上の注意（並列禁止・3秒間隔）

帝国議会会議録検索システム API への**並列呼び出しは避け、連続呼び出しは 3 秒以上空けてください**。本スキルはこの制約を SKILL.md の記述で守る設計です。手動で繰り返し呼び出す場合も、呼び出し間隔にご注意ください。

## 出典

- [帝国議会会議録検索システム](https://teikokugikai-i.ndl.go.jp/)
- クライアント仕様・規格の出典一覧は [共通インストールガイドの出典節](https://github.com/HighBridgeDragon/jp-skills-shared/blob/main/docs/install-guide.md#出典) を参照
