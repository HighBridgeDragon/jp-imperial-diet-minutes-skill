---
name: Imperial Diet Minutes API Response Format
description: 帝国議会会議録検索システム API のレスポンス構造（共通メタ・エンドポイント別構造・実 JSON サンプル・URL 形式・実測の落とし穴 16 項目）
---

# 帝国議会会議録検索システム API レスポンス構造

帝国議会会議録検索システム API が返す JSON の構造と、実測で分かった落とし穴。エンドポイント概要・ページネーション・エラーコードは [api-reference.md](./api-reference.md)、検索パラメータは [parameters.md](./parameters.md) を参照。

全リクエストに `recordPacking=json` を付ける前提で JSON だけを扱う。`xml`（省略時の既定）は XML 宣言が付かずルートが `<data>`、null が空要素になり、構造も別形（`<records><record><recordData><speechRecord>…`）になる。

「実測」は 2026-10-03 に実 API で確認した結果、それ以外は公式仕様の記載である。

## 目次

- 注意: フェッチツール選定
- 共通メタ
- エンドポイント別の構造
- 実 JSON サンプル
- URL の形式
- 落とし穴（16 項目）
- エラー応答

---

## 注意: フェッチツール選定

`WebFetch` 等の内部要約モデルを介するツールは、レスポンスに存在しないフィールドが混入する事象が姉妹スキルで観測されている。生データ取得には `curl`（PowerShell では `curl.exe`。5.1 の `curl` は `Invoke-WebRequest` の別名）か `Invoke-RestMethod` を使う。

txt ページ（`https://teikokugikai-i.ndl.go.jp/txt/<issueID>/<speechOrder>`）は引用リンクとしてユーザーに提示するためのもの。本文の取得は必ず API 経由で行う。

---

## 共通メタ

全エンドポイントのレスポンス直下に出る。

| 要素 | 型 | 意味 |
| --- | --- | --- |
| `numberOfRecords` | 整数 | 検索条件に該当する総件数 |
| `numberOfReturn` | 整数 | 今回返した件数 |
| `startRecord` | 整数 | 返した範囲の開始位置（1 始まり） |
| `nextRecordPosition` | 整数 / `null` | 次リクエストの `startRecord` に渡す値。次が無いと `null`（キーは省略されない） |

---

## エンドポイント別の構造

構造はフラットではない。エンドポイントごとに入れ子が違う。

| エンドポイント | 結果の配列 | 発言の持ち方 |
| --- | --- | --- |
| `speech` | `speechRecord[]` | 発言 1 件が 1 要素（会議メタを含む全項目） |
| `meeting_list` | `meetingRecord[]` | `meetingRecord[].speechRecord[]`。発言は 4 項目（`speechID` / `speechOrder` / `speaker` / `speechURL`）のみで本文なし |
| `meeting` | `meetingRecord[]` | `meetingRecord[].speechRecord[]`。発言の全項目（`startPage` を含む）と本文あり |

`speech` の 1 要素と `meeting` の構造の違いは、会議メタ（`issueID` / `imageKind` / `searchObject` / `session` / `nameOfHouse` / `nameOfMeeting` / `issue` / `date` / `meetingURL` / `pdfURL`）を持つ位置だけである。`speech` では各発言の要素内、`meeting` / `meeting_list` では `meetingRecord` 側にある。

型は `session`・`speechOrder`・`startPage` が整数、`issue` が `"第17号"` のような文字列、`date` が `YYYY-MM-DD` の文字列。

---

## 実 JSON サンプル

すべて実測値。`speech`（発言本文）は先頭のみの抜粋で、実際は全文が入る。会議・発言の件数も抜粋している。

### speech（`speaker=尾崎行雄`、先頭 1 件）

総 1,680 件。新しい順のため先頭は 1947-03-13 の発言。

```json
{
  "numberOfRecords": 1680,
  "numberOfReturn": 2,
  "startRecord": 1,
  "nextRecordPosition": 3,
  "speechRecord": [
    {
      "speechID": "009213242X01719470313_006",
      "issueID": "009213242X01719470313",
      "imageKind": "会議録",
      "searchObject": 6,
      "session": 92,
      "nameOfHouse": "衆議院",
      "nameOfMeeting": "本会議",
      "issue": "第17号",
      "date": "1947-03-13",
      "speechOrder": 6,
      "speaker": "尾崎行雄",
      "speakerYomi": "おざきゆきお",
      "speakerGroup": "無所属",
      "speakerPosition": null,
      "speakerElection": null,
      "officeTerm": "昭和21年10月11日～昭和22年3月31日",
      "speech": "○尾崎行雄君　總選擧はいずれの時においてもきわめて大切なものでありまするが、今日",
      "startPage": 2,
      "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/009213242X01719470313/6",
      "meetingURL": "https://teikokugikai-i.ndl.go.jp/txt/009213242X01719470313",
      "pdfURL": "https://teikokugikai-i.ndl.go.jp/img/009213242X01719470313/2"
    }
  ]
}
```

### speech（`speakerElection=多額納税者議員`、選出区分あり）

総 17,752 件。`speakerElection` が値入りで返る例（外側のメタは上と同じ形のため、`speechRecord` の要素だけ示す）。

```json
{
  "speechID": "009201502X00119470331_007",
  "issueID": "009201502X00119470331",
  "imageKind": "会議録",
  "searchObject": 7,
  "session": 92,
  "nameOfHouse": "貴族院",
  "nameOfMeeting": "衆議院議員選挙法の一部を改正する法律案特別委員会",
  "issue": "第1号",
  "date": "1947-03-31",
  "speechOrder": 7,
  "speaker": "長島銀藏",
  "speakerYomi": "ながしまぎんぞう",
  "speakerGroup": "研究会",
  "speakerPosition": null,
  "speakerElection": "多額納税者議員",
  "officeTerm": "昭和21年5月18日～昭和22年5月2日",
  "speech": "○長島銀藏君　中選擧區單記制に致しますると、此の前の選擧の時には連記制でありまし",
  "startPage": 2,
  "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/009201502X00119470331/7",
  "meetingURL": "https://teikokugikai-i.ndl.go.jp/txt/009201502X00119470331",
  "pdfURL": "https://teikokugikai-i.ndl.go.jp/img/009201502X00119470331/2"
}
```

### meeting_list（第 1 回・本会議、1 会議・発言 3 件のみ抜粋）

総 109 件。実際は 1 会議に全発言（276〜333 件）が入る。

```json
{
  "numberOfRecords": 109,
  "numberOfReturn": 2,
  "startRecord": 1,
  "nextRecordPosition": 3,
  "meetingRecord": [
    {
      "issueID": "000103242X04518910307",
      "imageKind": "会議録",
      "searchObject": 0,
      "session": 1,
      "nameOfHouse": "貴族院",
      "nameOfMeeting": "本会議",
      "issue": "第45号",
      "date": "1891-03-07",
      "speechRecord": [
        {
          "speechID": "000103242X04518910307_000",
          "speechOrder": 0,
          "speaker": "会議録情報",
          "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307/0"
        },
        {
          "speechID": "000103242X04518910307_001",
          "speechOrder": 1,
          "speaker": "東久世通禧",
          "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307/1"
        },
        {
          "speechID": "000103242X04518910307_002",
          "speechOrder": 2,
          "speaker": "藤村紫朗",
          "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307/2"
        }
      ],
      "meetingURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307",
      "pdfURL": "https://teikokugikai-i.ndl.go.jp/img/000103242X04518910307"
    }
  ]
}
```

### meeting（`issueID` 指定・1 件、発言 2 件のみ抜粋）

実際は 100 発言・約 120 KB。先頭の `speechOrder=0` は `speaker="会議録情報"` の行。

```json
{
  "numberOfRecords": 1,
  "numberOfReturn": 1,
  "startRecord": 1,
  "nextRecordPosition": null,
  "meetingRecord": [
    {
      "issueID": "000103242X04518910307",
      "imageKind": "会議録",
      "searchObject": 0,
      "session": 1,
      "nameOfHouse": "貴族院",
      "nameOfMeeting": "本会議",
      "issue": "第45号",
      "date": "1891-03-07",
      "speechRecord": [
        {
          "speechID": "000103242X04518910307_000",
          "speechOrder": 0,
          "speaker": "会議録情報",
          "speakerYomi": null,
          "speakerGroup": null,
          "speakerPosition": null,
          "speakerElection": null,
          "officeTerm": null,
          "speech": "　　明治二十四年三月七日（土曜日）\r\n　　　　午前十時四分開",
          "startPage": 1,
          "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307/0"
        },
        {
          "speechID": "000103242X04518910307_001",
          "speechOrder": 1,
          "speaker": "東久世通禧",
          "speakerYomi": "ひがしくぜみちとみ",
          "speakerGroup": null,
          "speakerPosition": "貴族院副議長",
          "speakerElection": null,
          "officeTerm": "明治23年10月1日～明治24年8月31日",
          "speech": "○副議長(伯爵東久世通禧君)　本日ノ議事ヲ開キマス、",
          "startPage": 1,
          "speechURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307/1"
        }
      ],
      "meetingURL": "https://teikokugikai-i.ndl.go.jp/txt/000103242X04518910307",
      "pdfURL": "https://teikokugikai-i.ndl.go.jp/img/000103242X04518910307"
    }
  ]
}
```

---

## URL の形式

| 項目 | 形式 | 備考 |
| --- | --- | --- |
| `speechURL` | `https://teikokugikai-i.ndl.go.jp/txt/<issueID>/<speechOrder>` | 発言の出典 URL |
| `meetingURL` | `https://teikokugikai-i.ndl.go.jp/txt/<issueID>` | 会議の出典 URL |
| `pdfURL` | `https://teikokugikai-i.ndl.go.jp/img/<issueID>` | 存在する場合のみ付く。`speech` の出力では `/img/<issueID>/<startPage>`（該当ページの画像） |

`pdfURL` は付かないことがあるため、存在を確認してから使う（`meeting_list` で実測した 2 会議には付いていた）。

---

## 落とし穴（16 項目）

### 1. 日本語パラメータは UTF-8 のパーセントエンコード必須

Windows の Git Bash で `curl -G --data-urlencode "speaker=尾崎行雄"` とすると、curl.exe に ANSI（CP932）で渡って `%94%f6…` になり、CloudFront が本文なしの 403 を返す（4 件連続で再現）。UTF-8 の URL（`%E5%B0%BE…`）なら 200。
対処: エンコード済みの URL を組み立てて渡す（手順は SKILL.md の「URL の組み立て」）。日本語を含む呼び出しで 403 が出たら原因は文字コードと判断する。

### 2. 無効な院名はエラーにならず全件ヒットする

`nameOfHouse=参議院` は条件から外れるだけで 200 を返す。他に条件が無いと全 1,748,061 件が対象になり、既定 30 件でも 659 KB になる。姉妹スキルの「参議院」を流用すると起きる。
対処: 院名は `貴族院` / `衆議院` / `両院` / `両院協議会` の 4 値に限る。参議院は本 API の対象外。

### 3. 必須条件の不足は HTTP 400

`{"message":"(19007)検索条件を指定してください。"}` が返る。構造化エラーは `message` と、書式エラー時の `details` 配列だけ。
対処: [parameters.md](./parameters.md) の 16 パラメータのうち 1 つ以上を指定する。

### 4. `speechID` の書式

`<issueID 21 桁>_<発言番号 3〜4 桁>`。書式不正は HTTP 400（19011 / 19026）。`speechOrder` は 0 始まり。
対処: `issueID` に `_` と発言番号を連結して作る。

### 5. `meeting_list` は本文なしだが発言メタを全件返す

1 会議で 276〜333 発言を返し、2 会議で 70〜89 KB になる。発言メタは `speaker` / `speechID` / `speechOrder` / `speechURL` の 4 項目だけ。
対処: `maximumRecords` は会議数の基準で小さく（2〜5）する。

### 6. `meeting` は 1 会議で約 120 KB

1 会議 100 発言で約 120 KB。既定 3 件・最大 10 件のままだと 1 MB を超えうる。
対処: `issueID` 指定・`maximumRecords=1` で 1 件ずつ取る。

### 7. 本文外の行（`speechOrder=0`・目次・附録など）

先頭の発言（`speechOrder=0`）は `speaker="会議録情報"` で、日付・開場時刻・議事日程などが入る。`speakerYomi` ほかは全て null。`imageKind` が `附録` / `目次` / `索引` / `追録` の行もあり、`附録` は `date: null`・号数「第0号」で返ることがある（`nameOfHouse=参議院` の全件応答で確認）。
対処: 議員発言の集計は `speechOrder>0` かつ `imageKind=会議録` に絞る。

### 8. null 値

`speakerPosition` / `speakerElection` / `speakerGroup` / `speakerYomi` / `officeTerm` / `date` が null になりうる。キーは省略されず null で入る。`nextRecordPosition` も次が無いと null。
対処: 値の参照前に null を判定する。キーの有無ではなく値で判断する。

### 9. 改行は `\r\n`

`speech` 内の改行は CRLF。
対処: 本文を処理・表示する時に `\r\n` を前提にする（行分割は `\r?\n`）。

### 10. 旧字体・旧仮名の本文と発言者名

本文・発言者名とも旧字体・旧仮名で返る（`總選擧` `舊憲法` `長島銀藏`）。ただし `speaker` の検索では新字体・旧字体・異体字が同一視される（落とし穴 16）。
対処: 返る `speaker` はデータ上の表記と心得て、提示・集計ではそのまま使うか表記ゆれを正規化する。

### 11. ソートは開催日の新しい順で固定

並び順を指定する方法は無い。`maximumRecords` で部分取得すると新しい側だけが返る。
対処: 最古の発言が欲しければ `numberOfRecords` から末尾の `startRecord` を計算して取る。

### 12. `searchObject` は区分として使えない

仕様上は「議事冒頭・本文」の区分だが、実測では発言番号と一致する数値（`speech` で 6・7、`speechOrder=0` では 0）。`冒頭` でヒットした場合も 0。`session`・`speechOrder` も数値（整数）、`issue` は `"第17号"` の文字列。
対処: `searchObject` で冒頭と本文を判別しない。区別には `speechOrder` と `imageKind` を使う。

### 13. JSON の構造がフラットでない

`speech` は `speechRecord` 配列、`meeting` / `meeting_list` は `meetingRecord[].speechRecord[]` の入れ子。`meeting_list` の `speechRecord` は 4 項目、`meeting` は全項目に `startPage` が付く（表は「エンドポイント別の構造」）。
対処: エンドポイントごとに配列のパスを切り替える。`meeting` / `meeting_list` の会議メタは `meetingRecord` 側にある。

### 14. `pdfURL` は存在する場合のみ

付かないことがある。`speechURL` は `/txt/<issueID>/<speechOrder>`、`meetingURL` は `/txt/<issueID>`、`pdfURL` は `/img/<issueID>`（`speech` の出力は `/img/<issueID>/<startPage>`）。
対処: `pdfURL` の存在を確認してから使う。出典には応答の `speechURL` / `meetingURL` をそのまま使ってよい。

### 15. `searchRange` 省略時の先頭ヒットは目次・冒頭

`any=震災` で省略時の総数は 7,865 件で、先頭は `imageKind=目次`、`speaker=null`、`speechOrder=0`、本文は約 34 KB。`冒頭` のヒット（305 件）は通常の発言ではない。`冒頭`（305）と `本文`（7,560）の合計は省略時・`冒頭・本文`（7,865）と一致する。
対処: 議員の発言だけが欲しければ `searchRange=本文` を指定する（既定の推奨）。

### 16. 旧字体を試す必要は無い

`speaker` 検索で新字体・旧字体・異体字は同一視される（`尾崎行雄` と `尾﨑行雄` は共に 1,680 件、`長島銀蔵` と `長島銀藏` は共に 81 件）。返る `speaker` はデータ上の表記（`長島銀藏`）。ひらがなは `speakerYomi` との一致になり、漢字より件数が減る（`ながしまぎんぞう` は 53 件）。
対処: 漢字の正式表記で検索し、ひらがなは補助にとどめる。0 件の時に字体を変えて再検索する必要は無い。

---

## エラー応答

HTTP 400 で次の形の JSON が返る（実測）。

```json
{"message":"(19007)検索条件を指定してください。"}
{"message":"(19011)検索条件の入力に誤りがあります。","details":["(0) (19026)speechID:発言IDを『冊子ID(21文字)_発言番号(3～4桁)』で入力してください。"]}
```

エラーコードの一覧と意味は [api-reference.md](./api-reference.md) を参照。
