---
name: Imperial Diet Minutes API Parameters
description: 帝国議会会議録検索システム API の検索パラメータ仕様（全 22 パラメータの型・既定値・一致方式・2000 バイト上限・組み合わせ例）
---

# 帝国議会会議録検索システム API 検索パラメータ仕様

帝国議会会議録検索システム API の検索パラメータ仕様。エンドポイント・ページネーション・エラーコードは [api-reference.md](./api-reference.md)、レスポンス構造は [response-format.md](./response-format.md) を参照。

公式: <https://teikokugikai-i.ndl.go.jp/teikoku_api.html>（表 1）

「実測」と書いた箇所は本スキルの作成時に実 API で確認した結果、それ以外は公式仕様の記載である。

---

## 共通ルール

- 全パラメータは URL クエリ文字列で渡す（HTTP メソッドは `GET`）
- 文字エンコードは UTF-8、値は 1 つずつパーセントエンコードする（手順は SKILL.md の「URL の組み立て」）
- 複数のパラメータは `&` で連結する。パラメータ間は AND で評価される
- **検索条件は全体で 2000 バイト上限**（後述）
- 次の 16 パラメータのうち **最低 1 つを指定する**。指定が無いと HTTP 400 `(19007)` になる: `nameOfHouse` `nameOfMeeting` `any` `speaker` `from` `until` `speechNumber` `speakerPosition` `speakerGroup` `speakerElection` `speechID` `issueID` `sessionFrom` `sessionTo` `issueFrom` `issueTo`
- `searchRange` / `supplementAndAppendix` / `contentsAndIndex` / `startRecord` / `maximumRecords` / `recordPacking` は必須条件に数えない。これらだけでは `19007` になる
- 全リクエストに `recordPacking=json` を付ける

---

## 2000 バイト上限

検索条件部分（`?` 以降）は全体で 2000 バイトが上限。超えると `(19020)入力可能文字数を超過しています。` が返る（`19020` は公式のエラーコード表に載るが、本スキルでは超過リクエストを実測していない）。

- 日本語 1 文字は UTF-8 エンコード後に 9 バイト（`%E5%B0%BE` の 3 バイト × 3）になる。漢字・ひらがなだけの値は約 220 文字で上限に達する
- 半角スペースは `%20` の 3 バイト、英数字・`-`・`_` は 1 バイト
- 長い `any` や複数語の `speaker` / `nameOfMeeting` を並べる時は、エンコード後の長さを数える。超える場合は語を減らすか、`from` / `until` で分割して複数回に分ける

---

## パラメータ一覧

### 全文検索

| パラメータ | 型 | 既定値 | 検索方式 | 複数指定 | 備考 |
| --- | --- | --- | --- | --- | --- |
| `any` | 文字列 | 条件に含めない | 部分一致・**AND** | 半角スペース区切りで複数指定可 | 発言内容等を対象にする |
| `searchRange` | `冒頭` / `本文` / `冒頭・本文` | `冒頭・本文` | N/A | 不可 | `any` 指定時のみ有効。`any` が無ければ条件に含まれない |

`searchRange` の実測件数（`any=震災`）。

| 指定 | `numberOfRecords` |
| --- | ---: |
| 省略 | 7,865 |
| `冒頭` | 305 |
| `本文` | 7,560 |
| `冒頭・本文` | 7,865 |

冒頭 + 本文 = 省略時で一致する。`冒頭` のヒットは目次や議事冒頭で、`speaker` が null、本文が約 34 KB と大きい。省略時の先頭ヒットも冒頭になるため、議員の発言が欲しければ `searchRange=本文` を指定する。

### 会議属性

| パラメータ | 型 | 既定値 | 検索方式 | 複数指定 | 備考 |
| --- | --- | --- | --- | --- | --- |
| `nameOfHouse` | `貴族院` / `衆議院` / `両院` / `両院協議会` | 条件に含めない | 完全一致（列挙） | 不可 | `両院` と `両院協議会` は結果が同じ。**無効値（例: `参議院`）はエラーにならず条件から外れる**（実測: HTTP 200、全件が対象） |
| `nameOfMeeting` | 文字列 | 条件に含めない | 部分一致・**OR** | 半角スペース区切りで複数指定可 | ひらがな可。例: `文部 外務` で文部 OR 外務を含む会議名 |

`nameOfHouse` の無効値は条件から外れるため、他条件が無いと約 174 万件が対象になる。意図したフィルタが効いているかは `numberOfRecords` で確認する。

### 発言者属性

| パラメータ | 型 | 既定値 | 検索方式 | 複数指定 | 備考 |
| --- | --- | --- | --- | --- | --- |
| `speaker` | 文字列 | 条件に含めない | 部分一致・**OR** | 半角スペース区切りで複数指定可 | 議員名はひらがな可（件数が減る。後述） |
| `speakerPosition` | 文字列 | 条件に含めない | 部分一致 | 不可 | 肩書き。実測: 返る値は `国務大臣兼経済安定本部総務長官兼物価庁長官` のように兼職が連結される。`国務大臣` で 4,172 件 |
| `speakerGroup` | 文字列 | 条件に含めない | 部分一致 | 不可 | 所属会派。データ上は正式名称のみ。実測: `研究会` で 157,226 件 |
| `speakerElection` | 文字列 | 条件に含めない | 部分一致 | 不可 | 選出類型。値の例は下記 |
| `speechNumber` | 整数（0 以上） | 条件に含めない | 完全一致 | 不可 | 会議録内の発言番号（`speechOrder` と同じ数） |

`speakerElection` の値の例。実測で確認したのは `多額納税者議員`（17,752 件）と `子爵議員`。`公爵議員` `勅選議員` は公式仕様に記載された例で、本スキルでは実測していない。

`speakerGroup` / `speakerPosition` / `speakerElection` は単独指定で必須条件を満たす（400 にならない）が、件数が多いため他条件との併用を前提にする。

### 期間・回次・号数

| パラメータ | 型 | 既定値 | 検索方式 | 複数指定 | 備考 |
| --- | --- | --- | --- | --- | --- |
| `from` | 日付 `YYYY-MM-DD` | `0000-01-01` | 範囲（始点） | 不可 | 会議の開催日の始点。`from` ≦ `until` が必要（違反は `19014`） |
| `until` | 日付 `YYYY-MM-DD` | `9999-12-31` | 範囲（終点） | 不可 | 会議の開催日の終点。特定の日だけを得るには `from` と `until` に同じ日付を指定する |
| `sessionFrom` | 整数（3 桁まで・自然数） | 条件に含めない | 範囲または完全一致 | 不可 | 帝国議会の回次（第 1〜92 回）の始まり |
| `sessionTo` | 整数（3 桁まで・自然数） | 条件に含めない | 範囲または完全一致 | 不可 | 回次の終わり |
| `issueFrom` | 整数（3 桁まで） | 条件に含めない | 範囲または完全一致 | 不可 | 号数の始まり。目次・索引・附録・追録は 0 号扱い |
| `issueTo` | 整数（3 桁まで） | 条件に含めない | 範囲または完全一致 | 不可 | 号数の終わり |

- `sessionFrom` / `sessionTo`、`issueFrom` / `issueTo` は、**片方だけを指定するとその回次・号数だけ**（完全一致）が対象になり、**両方を指定すると範囲**になる。第 20 回だけを得るなら `sessionFrom=20` 単独でよい
- 回次の上限は第 92 回。日本国憲法施行後の国会の回次は本 API の対象外
- 日付は会議の開催日で比較する。回次と日付の境界は SKILL.md の「対象範囲」を参照

### 同定 ID

| パラメータ | 型 | 既定値 | 検索方式 | 複数指定 | 備考 |
| --- | --- | --- | --- | --- | --- |
| `issueID` | 21 桁の英数字 | 条件に含めない | 完全一致 | 不可 | 会議録（冊子）を識別する ID。例: `009213242X03119470330`。書式不正はエラー |
| `speechID` | `<issueID 21 文字>_<発言番号>` | 条件に含めない | 完全一致 | 不可 | 発言番号は 0 埋め 3 桁（4 桁の場合は 4 桁）。例: `009213242X03119470330_002`。書式不正は `19011`（`details` に `19026`）で、実測済み |

### 補助フラグ

| パラメータ | 型 | 既定値 | 備考 |
| --- | --- | --- | --- |
| `supplementAndAppendix` | `true` / `false` | `false` | `true` で検索対象を追録・附録に限定する |
| `contentsAndIndex` | `true` / `false` | `false` | `true` で検索対象を目次・索引に限定する |

どちらも必須条件に数えない。通常の発言検索では指定しない。

### ページネーション・出力

| パラメータ | 型 | 既定値 | 備考 |
| --- | --- | --- | --- |
| `startRecord` | 整数 | `1` | 取得開始位置。範囲は 1〜総ヒット件数。範囲外は `19004` |
| `maximumRecords` | 整数 | `speech` / `meeting_list`: `30`、`meeting`: `3` | 上限は `speech` / `meeting_list` が `100`、`meeting` が `10`。超過は HTTP 400（`19011`、`details` に `19005` / `19006`） |
| `recordPacking` | `xml` / `json` | `xml` | 常に `json` を指定する |

ページネーションの手順は [api-reference.md](./api-reference.md) を参照。

---

## 検索方式の使い分けまとめ

| 検索方式 | 該当パラメータ |
| --- | --- |
| 部分一致・AND（スペース区切り） | `any` |
| 部分一致・OR（スペース区切り） | `nameOfMeeting`, `speaker` |
| 部分一致（単一値） | `speakerPosition`, `speakerGroup`, `speakerElection` |
| 完全一致（列挙） | `nameOfHouse`（無効値は条件から外れる） |
| 完全一致（数値・ID） | `speechNumber`, `issueID`, `speechID` |
| 範囲または完全一致 | `sessionFrom` / `sessionTo`, `issueFrom` / `issueTo` |
| 範囲 | `from` / `until` |
| 限定フラグ | `supplementAndAppendix`, `contentsAndIndex` |

---

## 表記揺れとひらがな

- `speaker` は新字体・旧字体・異体字を同一視する。実測: `尾崎行雄` と `尾﨑行雄`（異体字）はどちらも 1,680 件、`長島銀蔵` と `長島銀藏` はどちらも 81 件。返る `speaker` はデータ上の表記（例: `長島銀藏`）。字体を変えて試す必要は無い
- ひらがなは `speakerYomi` との一致になり、漢字より件数が減る。実測: `長島銀蔵` 81 件に対し `ながしまぎんぞう` 53 件。網羅したい時は漢字で検索し、ひらがなは補助にとどめる
- `nameOfMeeting` もひらがなを指定できる（公式仕様）

---

## 複数パラメータの併用

複数パラメータは AND で評価されるため、条件を足すほど件数は減る。`any` と `speaker` のように検索対象軸が異なる条件を併用すると積集合になり、一方だけの場合より大きく減る。

- 議員の全発言が欲しい: `speaker` 単独（`any` に同じ名前を重ねない）
- 議員への言及が欲しい: `any` 単独
- `searchRange` は `any` と一緒にだけ使う

`speech` では条件に合った発言だけが返る。`meeting_list` では条件に合う発言を含む会議単位で返り、同一会議の複数ヒットは 1 会議にまとまる。`maximumRecords` の上限と既定値以外、パラメータは 3 エンドポイント共通。

---

## よく使う組み合わせ例

値はすべてエンコード済みの実値で、実 API で 200 を確認した。

### 1. 特定議員の期間内の発言

```bash
curl -s 'https://teikokugikai-i.ndl.go.jp/api/emp/speech?speaker=%E5%B0%BE%E5%B4%8E%E8%A1%8C%E9%9B%84&from=1946-01-01&until=1947-03-31&maximumRecords=2&recordPacking=json'
```

- `speaker` は部分一致のためフルネームを推奨する。結果は新しい順で、`maximumRecords=2` なら最も新しい 2 件が返る

### 2. 複数語を含む貴族院の発言（AND）

```bash
curl -s 'https://teikokugikai-i.ndl.go.jp/api/emp/speech?any=%E5%8C%97%E6%B5%B7%E9%81%93%20%E9%9D%92%E6%A3%AE&searchRange=%E6%9C%AC%E6%96%87&nameOfHouse=%E8%B2%B4%E6%97%8F%E9%99%A2&maximumRecords=2&recordPacking=json'
```

- `any` に半角スペース（`%20`）で区切った複数語を渡すと、全部を含む発言だけがヒットする。`searchRange=本文` で目次・冒頭を除く

### 3. 回次の範囲と院で会議一覧

```bash
curl -s 'https://teikokugikai-i.ndl.go.jp/api/emp/meeting_list?sessionFrom=20&sessionTo=30&nameOfHouse=%E4%B8%A1%E9%99%A2%E5%8D%94%E8%AD%B0%E4%BC%9A&issueFrom=1&maximumRecords=2&recordPacking=json'
```

- 第 20〜30 回の両院協議会のうち、第 1 号（`issueFrom=1` 単独は完全一致）だけに絞る。`meeting_list` は全発言メタを返すため `maximumRecords` は小さくする

### 4. 会派と選出区分の組み合わせ

```bash
curl -s 'https://teikokugikai-i.ndl.go.jp/api/emp/speech?speakerGroup=%E7%A0%94%E7%A9%B6%E4%BC%9A&speakerElection=%E5%AD%90%E7%88%B5%E8%AD%B0%E5%93%A1&from=1947-03-01&maximumRecords=2&recordPacking=json'
```

- `speakerGroup` は正式名称で指定する。単独では 15 万件超になるため、`speakerElection` と期間で絞る

### 5. 発言 ID で 1 件を特定

```bash
curl -s 'https://teikokugikai-i.ndl.go.jp/api/emp/speech?speechID=009213242X01719470313_006&maximumRecords=1&recordPacking=json'
```

- `speechID` は完全一致。`issueID` が分かっていれば `meeting?issueID=<issueID>&maximumRecords=1&recordPacking=json` で会議全文を 1 件取れる（1 会議で約 120 KB）
