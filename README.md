# jp-imperial-diet-minutes

[![skills.sh](https://skills.sh/b/HighBridgeDragon/jp-imperial-diet-minutes-skill)](https://skills.sh/HighBridgeDragon/jp-imperial-diet-minutes-skill)

**Search and retrieve Japanese Imperial Diet (帝国議会) meeting minutes, 1890-1947, from the official NDL Imperial Diet API.** An [Agent Skill](https://agentskills.io) for compatible AI agents (Claude, Codex, Cursor, GitHub Copilot, Goose, Gemini, and more). No authentication required.

第 1〜92 回帝国議会（貴族院・衆議院、1890-11〜1947-03-31）の会議録を、NDL 帝国議会会議録検索システム API 経由で調査するスキル（AI エージェント向け）。議員別の発言抽出、会議全文取得、キーワード・期間・回次での絞り込みを AI エージェントから直接実行できる。

姉妹スキル: 日本国憲法施行後の国会は [jp-diet-minutes-skill](https://github.com/HighBridgeDragon/jp-diet-minutes-skill)、法令本文は [jp-law-skill](https://github.com/HighBridgeDragon/jp-law-skill) を併用する。

## Install

```bash
npx skills add HighBridgeDragon/jp-imperial-diet-minutes-skill
```

クライアント別の動作条件・実行環境は [docs/install.md](docs/install.md) を参照。

## What it does / 機能

インストール後、エージェントに以下のように依頼できる。

- "Show me speeches by Ozaki Yukio" / 「尾崎行雄の発言を見せて」
- "Find statements about the Great Kanto Earthquake in the Diet" / 「帝国議会での震災に関する発言を探して」
- "List House of Peers meetings in 1900" / 「1900 年の貴族院の会議を一覧にして」
- "Show speeches by members elected as highest taxpayers" / 「多額納税者議員の発言を見せて」
- "Get the full transcript of a specific meeting by issue ID" / 「issueID を指定して会議録の全文を取得して」

代表的なユースケース: 近代史・政治史研究、戦前の法制度・政策の審議過程の確認、議員別発言分析、学術調査。

## Capabilities / 提供機能

| Capability | Endpoint | 用途 |
| --- | --- | --- |
| Speech-level search / 発言単位検索 | `GET /api/emp/speech` | 議員名・キーワード・院・期間などで発言を抽出（最大 100 件/req） |
| Meeting list / 会議一覧 | `GET /api/emp/meeting_list` | 会議メタ中心の索引（最大 100 件/req） |
| Meeting full transcript / 会議全文 | `GET /api/emp/meeting` | 会議全発言の取得（最大 10 件/req、サイズ大） |

## 依存

HTTPS GET で生の JSON を取得できる手段（curl、PowerShell の `Invoke-RestMethod`、`mcp-server-fetch` 等）が 1 つあれば動作する。`WebFetch` のように要約モデルを介するツールは、応答に無い内容が混入するため使えない。詳細は [docs/install.md](docs/install.md#実行環境と依存関係) を参照。

## 帝国議会会議録 API

- [帝国議会会議録検索システム](https://teikokugikai-i.ndl.go.jp/)
- [国立国会図書館（NDL）](https://www.ndl.go.jp/)

### 対象範囲

- ✅ **帝国議会会議録**: 第 1〜92 回帝国議会（1890-11〜1947-03-31）の貴族院・衆議院・両院・両院協議会の会議録
- ❌ **国会会議録**: 日本国憲法施行（1947-05-03）後の国会（参議院を含む）は対象外。姉妹の [jp-diet-minutes-skill](https://github.com/HighBridgeDragon/jp-diet-minutes-skill) を使う

境界は年ではなく回次と日付で判断する。1947 年の発言でも、3 月までは本スキル、5 月以降は姉妹スキルの対象になる。

## 関連スキル

- [jp-diet-minutes-skill](https://github.com/HighBridgeDragon/jp-diet-minutes-skill) — 日本国憲法施行後の国会会議録（NDL 国会会議録検索システム API）
- [jp-law-skill](https://github.com/HighBridgeDragon/jp-law-skill) — 日本法令の検索・条文取得（e-Gov 法令 API V2）

## ライセンス

[MIT](LICENSE)
