---
name: Imperial Diet Minutes API Reference
description: 帝国議会会議録検索システム API の3エンドポイント仕様（URL・最大件数・recordPacking・ページネーション・返却順・エラーコード・レート制限・利用規約）
---

# 帝国議会会議録検索システム API リファレンス

帝国議会会議録検索システム（NDL 提供）が公開する、第 1〜92 回帝国議会（1890-11〜1947-03-31）の会議録 API の仕様。
日本国憲法施行後の国会会議録 API は対象外（姉妹スキル `jp-diet-minutes` が扱う別 API）。

- Base URL: `https://teikokugikai-i.ndl.go.jp/api/emp/`
- 公式仕様: <https://teikokugikai-i.ndl.go.jp/teikoku_api.html>
- HTTP メソッド: 全エンドポイント `GET`
- 認証: 不要
- 文字エンコード: UTF-8（クエリの値はパーセントエンコードする）
- 検索パラメータ詳細は [parameters.md](./parameters.md)、レスポンス構造詳細は [response-format.md](./response-format.md) を参照

---

## エンドポイント一覧

| エンドポイント | パス（Base URL に続ける） | 最大件数（既定） | 用途 |
| --- | --- | --- | --- |
| 会議単位簡易出力 | `meeting_list` | 100（30） | 会議メタ情報と発言メタ（本文を含まない） |
| 会議単位出力 | `meeting` | 10（3） | 会議メタ情報＋当該会議の全発言本文 |
| 発言単位出力 | `speech` | 100（30） | 検索条件にヒットした発言単位の本文 |

「最大件数」は 1 リクエストあたりの `maximumRecords` の上限。総ヒット件数が上限を超える場合は後述のページネーションで取得する。

### エンドポイント選択指針

応答サイズは `meeting_list` < `speech` < `meeting` の順に大きくなる。軽い方から段階的に絞る。

| ユースケース | 推奨エンドポイント |
| --- | --- |
| 会議の一覧・回次や日付の把握、issueID の取得 | `meeting_list` |
| 特定の議員・キーワードに該当する発言の抽出 | `speech` |
| 特定会議の議事の全文取得 | `meeting`（`issueID` 指定・`maximumRecords=1`） |

---

## 1. `GET /api/emp/meeting_list` — 会議単位簡易出力

会議のメタ情報を返す。各会議に `speechRecord[]` が付くが、発言メタ（`speaker` / `speechID` / `speechOrder` / `speechURL`）のみで本文は含まない。ただし全発言分を返すため、2 会議で 70〜89 KB になる（実測）。`maximumRecords` は小さく（2〜5）する。

- 最大件数 (`maximumRecords`): 1〜100、既定 30
- `startRecord` 既定値: 1

## 2. `GET /api/emp/meeting` — 会議単位出力

会議のメタ情報に加え、当該会議の全発言本文を返す。1 会議で約 120 KB（100 発言、実測）あり、最大件数は他エンドポイントの 10 分の 1 に制限されている。

- 最大件数 (`maximumRecords`): 1〜10、既定 3（既定のままだと重い）
- `startRecord` 既定値: 1
- `meeting_list` / `speech` で対象会議を絞ってから呼ぶ

## 3. `GET /api/emp/speech` — 発言単位出力

検索条件にヒットした発言のみを返す。結果単位は会議ではなく発言で、議員名やキーワードでのピンポイント検索に向く。

- 最大件数 (`maximumRecords`): 1〜100、既定 30
- `startRecord` 既定値: 1

---

## レスポンス形式指定

| パラメータ | 取りうる値 | 既定 |
| --- | --- | --- |
| `recordPacking` | `xml` / `json` | `xml` |

`Accept` ヘッダではなくクエリパラメータで指定する。全リクエストに `recordPacking=json` を付ける。xml は宣言がなく、null が空要素になり、構造も別になる。応答の Content-Type は `application/json;charset=UTF-8`。

---

## ページネーション

全エンドポイント共通。

| パラメータ | 範囲 | 既定値 |
| --- | --- | --- |
| `startRecord` | 1 〜 総ヒット件数 | 1 |
| `maximumRecords`（`meeting_list` / `speech`） | 1 〜 100 | 30 |
| `maximumRecords`（`meeting`） | 1 〜 10 | 3 |

1. 初回リクエストは `startRecord=1`、必要に応じて `maximumRecords` を指定する
2. 応答の `nextRecordPosition` を次回リクエストの `startRecord` に渡す
3. `nextRecordPosition` が `null` なら最終ページ

`numberOfRecords`（総ヒット件数）、`numberOfReturn`（今回返却件数）と合わせて進捗を判断する。`startRecord` が範囲外だと `19004` になる。

---

## 結果の返却順

公式仕様は返却順を明記していないが、実測で全エンドポイントとも開催日の新しい順に固定されている。ソート指定パラメータは無い。

- `maximumRecords` で部分取得すると、新しい側の N 件だけが返る
- 最古の発言・会議を得るには、まず `startRecord=1&maximumRecords=1` 相当の軽い呼び出しで同一条件の `numberOfRecords` を得る
- 次に同一条件で `startRecord=<numberOfRecords>&maximumRecords=1` を指定すると、末尾（最古側）の 1 件が取れる
- 別の条件で得た `numberOfRecords` を流用しない（範囲外になり `19004` を踏む）
- 同日内の並び順は公式に記載が無い。最古の 1 件を断定したい場合は、特定した日付に `from` / `until` を絞って件数を確認する

---

## エラーレスポンス

リクエスト不正時は HTTP 400 で、`recordPacking=json` なら次の JSON を返す（xml の場合は `<diagnostics>` 配下の `<message>` / `<details>`）。

- `message`: エラーコードを `(19007)` の形で先頭に含むメッセージ
- `details`: 検索条件の入力誤りの場合のみ付く配列（複数の誤りがあれば複数要素）。`19007` などでは `details` キー自体が無いため、無条件に `details[0]` を読まず、`message` だけで判定できる作りにする

実測した応答（いずれも HTTP 400）。

`19007` 検索条件なし（`https://teikokugikai-i.ndl.go.jp/api/emp/speech?maximumRecords=1&recordPacking=json`）。`details` は無い。

```json
{"message":"(19007)検索条件を指定してください。"}
```

`19011` + `19026` `speechID` の書式不正（`https://teikokugikai-i.ndl.go.jp/api/emp/speech?speechID=INVALID&maximumRecords=1&recordPacking=json`）。`19026` は公式のエラーコード表に無く、実測でのみ確認した。

```json
{"message":"(19011)検索条件の入力に誤りがあります。","details":["(0) (19026)speechID:発言IDを『冊子ID(21文字)_発言番号(3～4桁)』で入力してください。"]}
```

`19011` + `19006` `maximumRecords` 超過（`https://teikokugikai-i.ndl.go.jp/api/emp/meeting?issueID=000103242X04518910307&maximumRecords=11&recordPacking=json`）。

```json
{"message":"(19011)検索条件の入力に誤りがあります。","details":["(0) (19006)maximumRecordsには1～10の値を指定してください。"]}
```

### エラーコード表

公式仕様（<https://teikokugikai-i.ndl.go.jp/teikoku_api.html>「表 2：エラーメッセージ」）の転記。「実測」列は、本スキルの作成時に実際の応答で確認したかどうかを示す。

| コード | メッセージ内容 | 実測 |
| --- | --- | --- |
| 19001 | 現在、混み合っております。もうしばらくしてから再度アクセスしてください。 | 未 |
| 19004 | startRecordには 1から検索件数までの値を指定してください。 | 未 |
| 19005 | maximumRecordsには1～100の値を指定してください。 | 未（`details` 内。`speech` / `meeting_list` 向け） |
| 19006 | maximumRecordsには1～10の値を指定してください。 | 済（`meeting` で 11 を指定、`details` 内） |
| 19007 | 検索条件を指定してください。 | 済（トップレベルの `message`） |
| 19011 | 検索条件の入力に誤りがあります。 | 済（トップレベルの `message`、詳細は `details`） |
| 19012 | from:開会日付をYYYY-MM-DD形式で入力してください。 | 未 |
| 19013 | until:開会日付をYYYY-MM-DD形式で入力してください。 | 未 |
| 19014 | fromはuntil以下、もしくはuntilはfrom以上の日付で入力してください。 | 未 |
| 19018 | from:存在しない日付です。検索条件を見直し、再度検索してください。 | 未 |
| 19019 | until:存在しない日付です。検索条件を見直し、再度検索してください。 | 未 |
| 19020 | 入力可能文字数を超過しています。検索条件を見直してください。 | 未 |

公式は、これ以外のエラーメッセージが出た場合はメッセージを添えて問い合わせ先に連絡するよう案内している。

### 対処

- `19001`（混雑）は時間を空けて再試行する。並列化や即時の連打で回避しない
- `19011` は `details` の内容（例: `19026`、`19005` / `19006`）で原因を特定する
- `19007` は検索条件を 1 つ以上足す。数えるのは 16 パラメータ（`nameOfHouse` `nameOfMeeting` `any` `speaker` `from` `until` `speechNumber` `speakerPosition` `speakerGroup` `speakerElection` `speechID` `issueID` `sessionFrom` `sessionTo` `issueFrom` `issueTo`）で、`searchRange` / `supplementAndAppendix` / `contentsAndIndex` / `startRecord` / `maximumRecords` / `recordPacking` は数えない
- 日本語を含む呼び出しで本文なしの 403 が返る場合は文字コードが原因である。UTF-8 でパーセントエンコードした URL を使う（手順は SKILL.md の「URL の組み立て」）

### エラーにならない不正入力

`nameOfHouse` は `貴族院` / `衆議院` / `両院` / `両院協議会` の 4 値のみ有効。無効値（例: `参議院`）は HTTP 200 で、条件から外れる。他に条件が無いと全件（約 174 万件）が対象になる（実測: `numberOfRecords` は 1748061）。エラーとして検知できないため、意図したフィルタが効いているかを `numberOfRecords` の妥当性で必ず確認する。

---

## レート制限・利用規約

公式の文言（<https://teikokugikai-i.ndl.go.jp/teikoku_api.html>「4. 利用条件・免責事項」）は以下のとおり。

> 機械的なアクセスを行う場合、多重リクエストは避けてください。
> また、データを取得し終えてから数秒程度空けて次のリクエストを行うようにしてください。
>
> このシステムの運用に影響のあるような負荷のかかる利用（短時間での大量アクセス等）はご遠慮ください。安定運用に支障をきたすと当館が判断した場合には、予告なくアクセスを遮断する等の措置をとることがあります。

具体的な閾値（毎秒上限値など）は公開されていない。運用は次のとおり。

- 並列リクエストを発行しない（1 リクエスト完了 → 3 秒以上待機 → 次リクエスト）
- 大量取得が必要な場合は、`maximumRecords` を上限近くまで上げてリクエスト総数を減らす
- `19001` や 5xx を受けたら、間隔を空けて再試行する

著作権（同「4. 利用条件・免責事項」より）。

- データベース自体の著作権は国立国会図書館に帰属する
- 発言の著作権は個々の発言者に帰属し、利用には原則として著作権者の許諾が必要になる。保護期間が満了している場合や、権利制限規定（私的使用のための複製、引用、報道、電子計算機による情報解析等）が適用される場合は、許諾なしで利用できる
- 許諾の要否や条件は利用者自身が確認する。本ファイルは法的助言ではない
