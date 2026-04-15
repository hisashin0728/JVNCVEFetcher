# JVN CVE Fetcher

Microsoft Security Copilot 用カスタムプラグインです。\
[JPCERT/CC](https://www.jpcert.or.jp/m/#home) および [JVN iPedia（JVNDB）](https://jvndb.jvn.jp/) から脆弱性・セキュリティ情報を取得します。

[MyJVN API](https://jvndb.jvn.jp/apis/index.html)（IPA/JPCERT/CC が無償提供）を通じて、Security Copilot のプロンプトから直接 JVN 脆弱性情報・セキュリティ警戒情報・統計データを検索できます。

---

## 機能一覧

| スキル | API / ソース | 説明 |
|---|---|---|
| **GetMyJVNAlertList** | MyJVN `getAlertList` | IPA/JPCERT 発行の注意警戒情報一覧を取得（ページネーション対応） |
| **GetJPCERTSecurityAlerts** | JPCERT/CC RSS | JPCERT/CC が直接発行するセキュリティ注意喚起フィードを取得（最新6件） |
| **GetMyJVNVendorList** | MyJVN `getVendorList` | JVN iPedia 登録ベンダの一覧・検索（ベンダ ID 取得） |
| **GetJVNDBVulnerabilities** | MyJVN `getVulnOverviewList` | CVE 脆弱性概要情報の一覧取得（キーワード・CVSS・期間フィルタ対応） |
| **GetMyJVNVulnDetailInfo** | MyJVN `getVulnDetailInfo` | JVNDB ID 指定による脆弱性対策詳細情報の取得 |
| **GetMyJVNStatisticsCvss3** | MyJVN `getStatistics` | CVSSv3 深刻度別統計データの取得（年別/四半期別/月別） |

---

## スキル詳細

### GetMyJVNAlertList — 注意警戒情報一覧

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `startItem` | ページネーション開始位置（1〜） | `1` |
| `maxCountItem` | 取得件数（最大50件） | `50` |
| `lang` | レスポンス言語（`ja` / `en`） | `ja` |

---

### GetJPCERTSecurityAlerts — JPCERT/CC 注意喚起 RSS

パラメータなし。JPCERT/CC の注意喚起フィード（`jpcert.rdf`）から最新情報を取得します。  
情報元: https://www.jpcert.or.jp/m/#home

---

### GetMyJVNVendorList — ベンダ一覧

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `keyword` | キーワード絞り込み（CPE 識別子に対してマッチング） | — |
| `cpeVendor` | CPE ベンダ識別子で正確に絞り込み | — |
| `startItem` | ページネーション開始位置 | `1` |
| `maxCountItem` | 取得件数（最大50件） | `50` |
| `lang` | レスポンス言語（`ja` / `en`） | `ja` |

> 取得したベンダ ID（`vid`）は `GetJVNDBVulnerabilities` の `vendorId` パラメータに使用できます。

---

### GetJVNDBVulnerabilities — 脆弱性対策概要情報一覧

| パラメータ | 説明 | 値 | デフォルト |
|---|---|---|---|
| `rangeDatePublic` | 発見日（公表日）の期間フィルタ **【必須】** | `w`=過去1週間 / `m`=過去1ヶ月 / `n`=無制限 | `m` |
| `rangeDatePublished` | 更新日の期間フィルタ **【必須】** | `w` / `m` / `n` | `n` |
| `rangeDateFirstPublished` | JVN iPedia 登録日の期間フィルタ **【必須】** | `w` / `m` / `n` | `n` |
| `keyword` | キーワード検索（製品名、ベンダ名、CVE番号など） | 任意の文字列 | — |
| `severity` | CVSS 深刻度フィルタ（CVSSv3 基準）**【必須】** | `c`=緊急 / `h`=重要 / `m`=警告 / `l`=注意 | `h` |
| `startItem` | ページネーション開始位置 | 整数（1〜） | `1` |
| `maxCountItem` | 取得件数（最大10件） | 1〜10 | `10` |
| `lang` | レスポンス言語 | `ja`=日本語 / `en`=英語 | `ja` |

> **⚠️ 重要 — 日付フィルタの AND 条件に注意**  
> MyJVN API は 3 つの日付フィルタを **AND 条件**で評価します。パラメータを省略するとサーバー側が自動的に `w`（過去1週間）を適用します。  
> **省略した場合、実質「3つ全て過去1週間の AND」となり 0件になる場合があります。**  
> 使用するフィルタ以外は必ず `n`（無制限）を明示してください。
>
> | 目的 | rangeDatePublic | rangeDatePublished | rangeDateFirstPublished |
> |---|---|---|---|
> | 過去1ヶ月の発見日で絞り込む | `m` | **`n`** | **`n`** |
> | 過去1週間の発見日で絞り込む | `w` | **`n`** | **`n`** |
> | 更新日で絞り込む | **`n`** | `m` | **`n`** |
> | 全期間取得 | `n` | `n` | `n` |

> **⚠️ 重要 — severity（深刻度）は省略不可**  
> `severity=n`（全深刻度）は使用できません。必ず `c` / `h` / `m` / `l` のいずれかを指定してください。  
> 「全深刻度」「フィルタなし」と指示された場合は `h`（High以上）を使用してください。

> **日付の絶対指定について：** 「2025年1月1日以降」のような絶対日付での絞り込みはこの API ではサポートしていません。期間コード（`w`/`m`/`n`）のみ使用できます。

---

### GetMyJVNVulnDetailInfo — 脆弱性対策詳細情報

| パラメータ | 説明 | デフォルト |
|---|---|---|
| `vulnId` | JVNDB 脆弱性 ID **【必須】**（例: `JVNDB-2024-001234`） | — |
| `lang` | レスポンス言語（`ja` / `en`） | `ja` |

> `GetJVNDBVulnerabilities` のレスポンスに含まれる `sec:identifier` の値を `vulnId` に指定してください。

---

### GetMyJVNStatisticsCvss3 — CVSSv3 統計データ

| パラメータ | 説明 | 値 | デフォルト |
|---|---|---|---|
| `theme` | 集計テーマ **【必須】** | `sumCvss`=CVSSv3深刻度 / `sumJvnDb`=登録件数 / `sumCwe`=CWE分類 / `sumAll`=全統計 | `sumCvss` |
| `type` | 集計粒度 **【必須】** | `y`=年別 / `q`=四半期別 / `m`=月別 | `y` |
| `datePublicStartY` | 集計開始年 **【必須】**（例: `2024`） | 整数（西暦4桁） | — |
| `datePublicEndY` | 集計終了年 **【必須】**（例: `2024`） | 整数（西暦4桁） | — |
| `datePublicStartM` | 集計開始月（`type=m` 時に使用） | 1〜12 | — |
| `datePublicEndM` | 集計終了月（`type=m` 時に使用） | 1〜12 | — |
| `lang` | レスポンス言語（`ja` / `en`） | `ja` |

レスポンスには深刻度別件数（`cntC`=Critical / `cntH`=High / `cntM`=Medium / `cntL`=Low）が含まれます。

---

## プロンプト例

```
最新の IPA セキュリティ注意警戒情報を取得してください
```
```
JPCERT/CC が発行した最新のセキュリティ注意喚起を確認してください
```
```
Microsoft のベンダ ID を MyJVN API で調べてください
```
```
直近1ヶ月の CVE 脆弱性情報を取得してください
```
```
過去1週間に公開された Windows に関する脆弱性を取得してください
```
```
深刻度が重要（High）な脆弱性を過去1ヶ月分取得してください
```
```
Apache の重大な CVE 脆弱性を検索してください
```
```
JVNDB-2024-001234 の脆弱性詳細情報を取得してください
```
```
2024 年の CVSSv3 深刻度別の脆弱性統計を取得してください
```
```
2022 年から 2024 年の年別 Critical 脆弱性件数の推移を教えてください
```

---

## ファイル一覧

```
JVNCVEFetcher/
├── JVNCVEFetcher.yaml                       # プラグインマニフェスト
├── JVNCVEFetcher_myjvn_alert_openapi.yaml   # GetMyJVNAlertList 用 OpenAPI 仕様
├── JVNCVEFetcher_alert_openapi.yaml         # GetJPCERTSecurityAlerts 用 OpenAPI 仕様（JPCERT/CC RSS）
├── JVNCVEFetcher_vendor_openapi.yaml        # GetMyJVNVendorList 用 OpenAPI 仕様
├── JVNCVEFetcher_vuln_openapi.yaml          # GetJVNDBVulnerabilities 用 OpenAPI 仕様
├── JVNCVEFetcher_detail_openapi.yaml        # GetMyJVNVulnDetailInfo 用 OpenAPI 仕様
├── JVNCVEFetcher_stats_openapi.yaml         # GetMyJVNStatisticsCvss3 用 OpenAPI 仕様
├── JVNCVEFetcher_card.html                  # Plugin Card（ブラウザで視覚確認用）
└── README.md                                # このファイル
```

---

## デプロイ手順

### 1. OpenAPI ファイルをホスティング

全 6 つの OpenAPI ファイルを、Security Copilot からアクセスできる公開 URL にホスティングします。

**GitHub を使う場合（推奨）：**

このリポジトリを公開リポジトリとして GitHub に Push すると、以下の形式の Raw URL が使えます。

```
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_myjvn_alert_openapi.yaml
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_alert_openapi.yaml
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_vendor_openapi.yaml
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_vuln_openapi.yaml
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_detail_openapi.yaml
https://raw.githubusercontent.com/<YOUR-ORG>/<YOUR-REPO>/main/Builder/output/JVNCVEFetcher/JVNCVEFetcher_stats_openapi.yaml
```

### 2. マニフェストの URL を確認

`JVNCVEFetcher.yaml` の各 `OpenApiSpecUrl` が実際の公開 URL を指していることを確認します。

### 3. Security Copilot にアップロード

1. [Microsoft Security Copilot](https://securitycopilot.microsoft.com/) を開く
2. 右上の「Sources」→「Custom plugins」→「Add plugin」を選択
3. Upload から `JVNCVEFetcher.yaml` をアップロード
4. プラグインが有効化されたことを確認

---

## データソース

| ソース | URL | 提供元 |
|---|---|---|
| JVN iPedia（JVNDB） | https://jvndb.jvn.jp/ | IPA（独立行政法人 情報処理推進機構） |
| JPCERT/CC | https://www.jpcert.or.jp/m/#home | JPCERT/CC |
| MyJVN API | https://jvndb.jvn.jp/apis/index.html | IPA / JPCERT/CC（無償） |
| JPCERT/CC RSS | https://www.jpcert.or.jp/rss/jpcert.rdf | JPCERT/CC |

本プラグインが使用する MyJVN API は IPA と JPCERT/CC が共同運営する無償の公開 API です。\
ご利用の際は [MyJVN 利用上の規約](https://jvndb.jvn.jp/apis/termsofuse.html) をご確認ください。

---

## 制限事項

- MyJVN API の 1 回のリクエストあたりの取得上限は **50件** です。51件以降は `startItem` でページネーションしてください。
- 期間指定は相対期間（`w`=過去1週間, `m`=過去1ヶ月, `n`=無制限）のみサポートされています。絶対日付指定は `n` で全期間取得後にレスポンスの `dc:date` で絞り込んでください。
- `GetJVNDBVulnerabilities` は 3 つの日付フィルタ（`rangeDatePublic` / `rangeDatePublished` / `rangeDateFirstPublished`）を AND 条件で評価します。使用しないフィルタは必ず `n` を明示してください（省略すると 0件になる場合があります）。
- `GetJPCERTSecurityAlerts` は JPCERT/CC RSS の配信仕様上、注意喚起の最新 6 件と Weekly Report のみ返します。
- 本プラグインは読み取り専用です（書き込みアクションはありません）。

---

## 関連リンク

- [MyJVN API ドキュメント](https://jvndb.jvn.jp/apis/index.html)
- [MyJVN API よくある質問](https://jvndb.jvn.jp/apis/myjvnapi_faq.html)
- [JVN iPedia](https://jvndb.jvn.jp/)
- [JPCERT/CC](https://www.jpcert.or.jp/m/#home)
- [Security Copilot カスタムプラグイン ドキュメント](https://learn.microsoft.com/ja-jp/copilot/security/custom-plugins)
