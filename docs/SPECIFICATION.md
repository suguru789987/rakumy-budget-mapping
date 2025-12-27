# 予算シミュレーター v5.8 機能仕様書

## 1. 概要

### 1.1 目的
顧客が作成した年間予算CSVファイルを読み込み、ラクミー経営管理システムへの入力値に自動変換するシミュレーターツール。

### 1.2 主要機能
- 顧客予算CSVの自動解析・インポート
- 項目マッピングによる自動変換
- MQシミュレーション（売上・利益計算）
- ラクミー入力値の自動生成
- 予算一覧の出力
- 複数店舗対応
- マッピング管理機能（編集・CSV入出力・未使用検出・類似マッチング）
- 出力データ履歴管理

---

## 2. システム構成

### 2.1 ファイル構成
```
v15_予算シミュレーター_顧客CSV対応版_v5.8.html
├── HTML構造（UI）
├── CSS（スタイル）
└── JavaScript（ロジック）
```

### 2.2 処理フロー
```
┌─────────────────┐
│ CSVファイル選択 │
└────────┬────────┘
         ↓
┌─────────────────┐
│ CSV解析・店舗検出 │
└────────┬────────┘
         ↓
┌─────────────────┐
│ 月別予算列検出   │
└────────┬────────┘
         ↓
┌─────────────────┐
│ 項目マッピング   │
└────────┬────────┘
         ↓
┌─────────────────┐
│ 入力前演算実行   │
└────────┬────────┘
         ↓
┌─────────────────┐
│ ラクミー入力値生成│
└────────┬────────┘
         ↓
┌─────────────────┐
│ MQシミュレーション│
└────────┬────────┘
         ↓
┌─────────────────┐
│ 予算一覧・サマリー│
└────────┬────────┘
         ↓
┌─────────────────┐
│ 出力データ履歴保存│
└─────────────────┘
```

---

## 3. データ構造

### 3.1 グローバル変数

| 変数名 | 型 | 説明 |
|--------|------|------|
| `activeMonths` | Array | アクティブな月のインデックス配列 |
| `monthLabels` | Array | 月ラベル配列（例：['4月', '5月', ...]） |
| `budgetColumnPositions` | Array | 各月の予算列位置 |
| `annualData` | Object | 年間予算データ（項目番号→月別値） |
| `calcResults` | Object | 演算結果 |
| `rakumyInputs` | Object | ラクミー入力値 |
| `mqOutputs` | Object | MQシミュレーション出力 |
| `judgments` | Object | 費用設定判定結果 |
| `allStoresData` | Object | 全店舗データ |
| `isEditMode` | Boolean | マッピング編集モードフラグ |
| `unusedMappings` | Array | 未使用マッピングリスト |

### 3.2 項目マッピング（CUSTOMER_CODE_MAP）

顧客予算CSVの項目コードとv15項目番号の対応表：

| v15番号 | 項目名 | カテゴリ |
|---------|--------|----------|
| 2 | ランチ売上 | 売上 |
| 3 | ディナー売上 | 売上 |
| 4 | その他売上 | 売上 |
| 5 | 売上高合計 | 売上 |
| 6 | フード仕入 | 原価 |
| 7 | ドリンク仕入 | 原価 |
| 10 | 給料 | 固定人件費 |
| 13 | PA人件費 | PA人件費 |
| 14 | 法定福利 | 固定人件費 |
| 15 | 通勤費 | 固定人件費 |
| 16 | 賞与 | 固定人件費 |
| 17 | 社宅 | 固定人件費 |
| 18 | 休業手当 | 固定人件費 |
| 19 | ヘルプ人件費 | PA人件費 |
| 21-50 | 各種経費 | 経費・固定費 |
| 60 | 店舗営業利益 | 利益 |

### 3.3 演算ルール（CALC_RULES）

| ルール名 | 計算式 | 説明 |
|----------|--------|------|
| 売上合計 | item:5 | 売上高合計 |
| FD仕入費合計 | item:6 + item:7 | フード仕入+ドリンク仕入 |
| FD仕入費率 | FD仕入費合計 / 売上合計 × 100 | FD原価率 |
| PA人件費合計 | item:13 + item:19 | PA人件費+ヘルプ人件費 |
| PA人件費率 | PA人件費合計 / 売上合計 × 100 | PA人件費率 |
| 固定人件費合計 | item:10+14+15+16+17+18 | 固定人件費合計 |
| ランチ売上比率 | item:2 / 売上合計 × 100 | ランチ売上構成比 |
| ディナー売上比率 | item:3 / 売上合計 × 100 | ディナー売上構成比 |
| フード売上比率 | (item:2+3) × 0.7 / 売上合計 × 100 | フード売上構成比 |
| ドリンク売上比率 | (item:2+3) × 0.3 / 売上合計 × 100 | ドリンク売上構成比 |
| 地代家賃合計 | item:47 + item:48 | 家賃+賃借料 |
| 資材費合計 | item:49 + item:50 | 警備料+リース料 |

---

## 4. マッピング管理機能

### 4.1 概要
顧客予算項目の変更に柔軟に対応するためのマッピング管理機能。

### 4.2 機能一覧

| 機能 | 関数 | 説明 |
|------|------|------|
| 編集モード切替 | `toggleEditMode()` | マッピング編集UIの表示/非表示 |
| マッピング編集 | `editMapping(code)` | 個別マッピングの編集 |
| マッピング保存 | `saveEditMapping(code)` | 編集内容の保存 |
| マッピング削除 | `deleteMapping(code)` | 個別マッピングの削除 |
| マッピング追加 | `addMapping()` | 新規マッピングの追加 |
| CSV読込 | `importMappingCSV(event)` | CSVからマッピング定義を読込 |
| CSV出力 | `exportMappingCSV()` | マッピング定義をCSV出力 |
| 未使用検出 | `detectUnusedMappings()` | 未使用マッピングの検出 |
| 未使用削除 | `removeUnusedMappings()` | 未使用マッピングの一括削除 |
| 類似検索 | `findSimilarItems()` | 未マッチ項目の類似候補検索 |
| 類似適用 | `applyMappingSuggestion()` | 類似候補からマッピング作成 |

### 4.3 編集モード

```javascript
var isEditMode = false;

function toggleEditMode() {
    isEditMode = !isEditMode;
    document.getElementById('editModeLabel').textContent =
        isEditMode ? '✅ 編集モード ON' : '✏️ 編集モード';
    renderMappingTable();
}
```

### 4.4 マッピングCSVフォーマット

#### エクスポート形式
```
顧客コード,v15番号,項目名,カテゴリ
A0001,5,売上高合計,売上
B0001,6,フード仕入,原価
...
```

#### インポート形式
- ヘッダー行: `顧客コード,v15番号,項目名,カテゴリ`
- 文字コード: UTF-8（BOMあり/なし両対応）

### 4.5 未使用マッピング検出

```javascript
function detectUnusedMappings() {
    var usedCodes = new Set();
    // CSVから検出されたコードを収集
    csvData.forEach(row => usedCodes.add(row[0]));

    // マッピング定義から未使用を検出
    unusedMappings = [];
    Object.keys(CUSTOMER_CODE_MAP).forEach(code => {
        if (!usedCodes.has(code)) {
            unusedMappings.push(code);
        }
    });

    renderUnusedMappingSection();
}
```

### 4.6 類似項目マッチング

未マッチ項目に対して類似候補を提案：

```javascript
function findSimilarItems(unmatchedName) {
    var suggestions = [];
    Object.entries(CUSTOMER_CODE_MAP).forEach(([code, info]) => {
        var similarity = calculateSimilarity(unmatchedName, info.name);
        if (similarity > 0.5) {
            suggestions.push({ code, info, similarity });
        }
    });
    return suggestions.sort((a, b) => b.similarity - a.similarity);
}
```

---

## 5. ラクミー入力値構造

### 5.1 費用予算設定（固定費用）- 29項目

年間を通して同一値を設定する項目：

| キー | 項目名 | データソース |
|------|--------|--------------|
| rent | 地代家賃 | 地代家賃合計（家賃+賃借料） |
| fixedLabor | 固定人件費 | 固定人件費合計 |
| depreciation | 減価償却費 | item:46 |
| material | 資材費 | 資材費合計（警備料+リース料） |
| promotion | 販売促進費 | item:21 |
| menuCopy | メニューコピー | item:22 |
| survey | 調査費 | item:23 |
| sanitary | 衛生費 | item:24 |
| service | サービス費 | item:25 |
| electricity | 電気代 | item:26 |
| gas | ガス代 | item:27 |
| water | 水道代 | item:28 |
| consumables | 消耗品費 | item:29 |
| maintenance | 保守修繕費 | item:30 |
| taxes | 租税公課 | item:31 |
| entertainment | 接待交際費 | item:32 |
| travel | 旅費交通費 | item:33 |
| communication | 通信費 | item:34 |
| creditFee | クレジット手数料 | item:35 |
| paymentFee | 支払手数料 | item:36 |
| meeting | 会議費 | item:37 |
| membership | 諸会費 | item:38 |
| education | 図書教育費 | item:39 |
| training | 研修費 | item:40 |
| research | 業態研究費 | item:41 |
| recruitment | 採用費 | item:42 |
| it | IT費 | item:43 |
| outsource | 外注費 | item:44 |
| misc | 雑費 | item:45 |

### 5.2 単月費用予算（固定費用）- 28項目

月別に値が異なる場合のみ表示（地代家賃を除く）：
- 費用予算設定と同じ項目（rentを除く）
- 月別差異がある項目のみ実際の値を表示
- 差異がない項目は非表示

### 5.3 予算設定（変動費用）

| キー | 項目名 | 説明 |
|------|--------|------|
| fdRate | FD仕入費率 | FD仕入費÷売上×100 |
| paRate | PA人件費率 | PA人件費÷売上×100 |
| targetUnitPrice | 目標客単価（税抜） | 売上÷客数（逆算） |
| targetCustomers | 目標客数 | 自動計算 |
| lunchRatio | ランチ売上比率 | ランチ売上÷売上×100 |
| dinnerRatio | ディナー売上比率 | ディナー売上÷売上×100 |
| foodRatio | フード売上比率 | フード売上÷売上×100 |
| drinkRatio | ドリンク売上比率 | ドリンク売上÷売上×100 |
| otherRatio | その他売上比率 | その他売上÷売上×100 |

---

## 6. MQシミュレーション

### 6.1 計算ロジック

```javascript
// 売上 = 客数 × 客単価
sales = customers × unitPrice

// 変動費 = 売上 × (FD仕入費率 + PA人件費率) / 100
variableCost = sales × (fdRate + paRate) / 100

// 変動費率
variableRate = (fdRate + paRate)

// 粗利 = 売上 - 変動費
grossProfit = sales - variableCost

// 粗利率
grossProfitRate = grossProfit / sales × 100

// 固定費 = 費用予算設定の全項目合計
fixedCost = Σ(全費用項目)

// 営業利益 = 粗利 - 固定費
operatingProfit = grossProfit - fixedCost

// 営業利益率
operatingProfitRate = operatingProfit / sales × 100
```

### 6.2 出力項目

| 項目 | 説明 |
|------|------|
| 客数 | 目標客数 |
| 客単価 | 目標客単価 |
| 売上 | 客数×客単価 |
| 変動費 | 売上×変動費率 |
| 変動費率 | FD仕入費率+PA人件費率 |
| 粗利 | 売上-変動費 |
| 粗利率 | 粗利÷売上×100 |
| 固定費 | 全費用項目合計 |
| 営業利益 | 粗利-固定費 |
| 営業利益率 | 営業利益÷売上×100 |

---

## 7. CSVエクスポート機能

### 7.1 全店舗エクスポート

「全店舗を選択」チェック時は全店舗データを1つのCSVに縦連結：

```javascript
function exportRakumyInputsCSV() {
    var targetStores = getExportTargetStores();
    var csv = [];
    var header = ['店舗名', 'セクション', '項目名', '変換タイプ'];
    // 月ヘッダー追加
    monthLabels.forEach(m => header.push(m));
    header.push('合計', '平均');
    csv.push(header.join(','));

    // 各店舗データを順に追加
    targetStores.forEach(storeName => {
        var storeData = allStoresData[storeName];
        // ... データ行追加
    });

    downloadCSV(csv, filename);
}
```

### 7.2 出力ファイル名規則

| 条件 | ファイル名 |
|------|-----------|
| 単一店舗 | `{種別}_{店舗名}_{YYYY-MM-DD}.csv` |
| 全店舗 | `{種別}_全店舗_{YYYY-MM-DD}.csv` |

### 7.3 ラクミー入力値CSV構造

```
店舗名,セクション,項目名,変換タイプ,4月,5月,...,3月,合計,平均
店舗A,費用予算設定,地代家賃,sum,450,450,...,450,5400,450
店舗A,費用予算設定,固定人件費,sum,500,500,...,500,6000,500
...
```

### 7.4 予算一覧CSV構造

```
店舗名,セクション,項目名,4月,5月,...,3月,合計,平均
店舗A,目標設定,月次目標利益,xxx,xxx,...,xxx,xxxx,xxx
店舗A,売上,売上 計,xxxx,xxxx,...,xxxx,xxxxx,xxxx
...
```

### 7.5 MQ出力CSV構造

```
店舗名,項目,4月,5月,...,3月,合計,平均
店舗A,客数,xxxx,xxxx,...,xxxx,xxxxx,xxxx
店舗A,客単価,xxxx,xxxx,...,xxxx,xxxxx,xxxx
店舗A,売上,xxxx,xxxx,...,xxxx,xxxxx,xxxx
...
店舗A,顧客予算売上,xxxx,xxxx,...,xxxx,xxxxx,xxxx
店舗A,売上差異（MQ-予算）,+xxx,+xxx,...,+xxx,+xxxx,-
...
```

### 7.6 共通仕様

| 項目 | 仕様 |
|------|------|
| 文字コード | UTF-8 with BOM（Excel対応） |
| 金額 | 千円単位、整数、カンマなし |
| 率 | 整数%表示（小数点なし） |

---

## 8. 出力データ履歴管理

### 8.1 概要
エクスポートしたデータをlocalStorageに保存し、後から表示・ダウンロード・削除できる機能。

### 8.2 保存データ構造

```json
{
  "id": "1703593200000",
  "title": "ラクミー入力値_店舗A_2024-12-27",
  "type": "rakumy_inputs",
  "storeName": "店舗A",
  "csvContent": "...",
  "savedAt": "2024-12-27T12:30:00.000Z"
}
```

### 8.3 機能一覧

| 関数 | 説明 |
|------|------|
| `saveExportData(type, storeName, csvContent)` | 出力データを履歴に保存 |
| `viewExportData(id)` | 履歴データをプレビュー表示 |
| `downloadExportData(id)` | 履歴データをCSVダウンロード |
| `deleteExportData(id)` | 履歴データを削除 |
| `clearAllExportHistory()` | 全履歴をクリア |
| `renderExportHistoryList()` | 履歴リストを描画 |
| `loadExportHistory()` | localStorageから履歴を読み込み |

### 8.4 自動保存タイミング

- CSVインポート完了時
- CSVエクスポートボタンクリック時

### 8.5 制限事項

- 最大保存件数: 50件
- 保存容量: localStorageの制限に依存（通常5MB）
- 容量超過時はエラー通知

---

## 9. インポート履歴管理機能

### 9.1 概要
インポートしたデータをlocalStorageに保存し、後から復元できる機能。

### 9.2 保存データ構造
```json
{
  "id": "1703593200000",
  "title": "2024-12-26_21:30_店舗A, 店舗B",
  "dateTime": "2024-12-26_21:30",
  "storeNames": ["店舗A", "店舗B"],
  "storeCount": 2,
  "data": { /* allStoresData */ },
  "savedAt": "2024-12-26T12:30:00.000Z"
}
```

### 9.3 機能一覧

| 関数 | 説明 |
|------|------|
| `saveToHistory()` | 現在のデータを履歴に保存 |
| `loadFromHistory(id)` | 履歴からデータを復元 |
| `deleteFromHistory(id)` | 履歴を削除 |
| `clearAllHistory()` | 全履歴をクリア |
| `renderHistoryList()` | 履歴リストを描画 |

---

## 10. 費用設定判定ロジック

### 10.1 判定基準
- **全月同一**: 12ヶ月すべて同じ値 → 費用予算設定（年間デフォルト）
- **月別差異**: 月によって値が異なる → 単月費用予算

### 10.2 表示ロジック
```
費用予算設定:
  月別差異あり → 「-」表示 + 「単月設定」バッジ
  全月同一 → 年間デフォルト値表示 + 「年間固定」バッジ

単月費用予算:
  月別差異あり → 実際の月別値を表示
  全月同一 → 非表示（項目自体を表示しない）
```

---

## 11. 設定のエクスポート/インポート

### 11.1 エクスポート形式
```json
{
  "version": "5.8",
  "mapping": { ... },
  "calcRules": { ... },
  "rakumyItems": { ... },
  "items": { ... },
  "allStoresData": { ... }
}
```

### 11.2 ファイル名
`budget_config_v5.8_YYYY-MM-DD.json`

---

## 12. 複数ファイルインポート

### 12.1 概要
複数のCSVファイルを一度に選択し、まとめてインポート・シミュレーションを実行。

### 12.2 処理フロー
1. 複数ファイル選択（input[type="file"] multiple属性）
2. `selectedFiles`配列にファイルを格納
3. 「インポート実行」ボタンクリック
4. 各ファイルを順次処理:
   - CSV解析
   - シミュレーション実行
   - 店舗データを`allStoresData`に保存
5. 全店舗サマリー更新

### 12.3 関連関数

| 関数 | 説明 |
|------|------|
| `handleFileSelect(event)` | ファイル選択時の処理 |
| `importAllCSVs()` | 全ファイルのインポート実行 |
| `saveCurrentStoreData()` | 現在の店舗データを保存 |
| `updateStoreSelector()` | 店舗セレクターを更新 |
| `updateAllStoresSummary()` | 全店舗サマリーを更新 |

---

## 13. エラーハンドリング

### 13.1 CSVパースエラー
- 行数不足
- 月列検出失敗
- 文字コード問題

### 13.2 データ検証
- 必須項目の欠落
- 数値変換エラー
- 範囲外の値

### 13.3 ストレージエラー
- localStorage容量超過
- 読み込み/書き込み失敗
