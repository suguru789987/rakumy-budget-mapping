# 予算シミュレーター v5.8 UI仕様書

## 1. 画面構成

### 1.1 全体レイアウト（3タブ構造）

```
┌─────────────────────────────────────────────────────────────┐
│                       ヘッダー                               │
│  予算シミュレーター v5.8 （顧客CSV対応版・変換ロジック強化）     │
├─────────────────────────────────────────────────────────────┤
│  メインタブナビゲーション                                     │
│  [インポート] [予算設定] [エクスポート]                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    タブコンテンツ                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 メインタブ構成

| タブ | 内容 |
|------|------|
| インポート | CSVファイル取込、マッピング設定 |
| 予算設定 | 予算入力・演算・ラクミー入力値・MQ出力・予算一覧・サマリー |
| エクスポート | 出力データ履歴一覧、CSVダウンロード |

---

## 2. ヘッダー

### 2.1 スタイル
```css
.header {
  background: linear-gradient(135deg, #1a237e 0%, #4a148c 100%);
  color: white;
  padding: 15px 20px;
}
```

### 2.2 要素
- タイトル: 「予算シミュレーター v5.8」
- サブタイトル: 「顧客CSV対応版・変換ロジック強化」

---

## 3. メインタブナビゲーション

### 3.1 タブ構成

| タブ | 内容 | アイコン |
|------|------|----------|
| インポート | CSV取込・マッピング設定 | 📥 |
| 予算設定 | 予算データ確認・編集 | 📊 |
| エクスポート | 出力データ履歴・ダウンロード | 📤 |

### 3.2 スタイル
```css
.main-tabs {
  display: flex;
  gap: 0;
  margin-bottom: 20px;
}
.main-tab {
  padding: 12px 30px;
  font-size: 14px;
  font-weight: bold;
  cursor: pointer;
  border: none;
  background: #e0e0e0;
}
.main-tab.active {
  background: linear-gradient(135deg, #1a237e 0%, #4a148c 100%);
  color: white;
}
```

---

## 4. インポートタブ

### 4.1 構成

```
┌─────────────────────────────────────────────────────────────┐
│ CSVインポートセクション                                       │
│  [ファイル選択] [インポート実行] [シミュレーション実行]         │
│  設定: [設定JSON] [設定読込] [全店舗クリア]                   │
├─────────────────────────────────────────────────────────────┤
│                   店舗セレクター                             │
├─────────────────────────────────────────────────────────────┤
│                     情報バー                                │
│  店舗名 │ 会計年度 │ 検出月数 │ マッチ項目 │ 未マッチ項目     │
├─────────────────────────────────────────────────────────────┤
│                 マッピング設定セクション                      │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ マッピング管理ツールバー                              │  │
│  │ [編集モード] [CSV読込] [CSV出力] [未使用検出]         │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ 未使用マッピングセクション                            │  │
│  │ 類似候補セクション                                   │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ 未マッチ項目セクション                                │  │
│  │ マッピング一覧テーブル                                │  │
│  │ 処理ログ                                            │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 CSVインポートセクション

| 要素 | 種類 | イベント |
|------|------|----------|
| CSVファイル選択 | `<input type="file" multiple>` | `handleFileSelect(event)` |
| インポート実行 | `<button>` | `importAllCSVs()` |
| シミュレーション実行 | `<button>` | `runSimulationForCurrentStore()` |
| 設定JSON | `<button>` | `exportAllConfig()` |
| 設定読込 | `<input type="file">` | `importAllConfig(event)` |
| 全店舗クリア | `<button>` | `clearAllStores()` |

### 4.3 マッピング管理ツールバー

```html
<div style="display:flex;gap:10px;flex-wrap:wrap;margin-bottom:15px;padding:10px;background:#f5f5f5;border-radius:8px;">
    <button class="btn btn-primary" onclick="toggleEditMode()">
        <span id="editModeLabel">✏️ 編集モード</span>
    </button>
    <label class="btn btn-info" style="cursor:pointer;margin:0;">
        📥 マッピングCSV読込
        <input type="file" accept=".csv" onchange="importMappingCSV(event)" style="display:none;">
    </label>
    <button class="btn btn-secondary" onclick="exportMappingCSV()">📤 マッピングCSV出力</button>
    <button class="btn btn-warning" onclick="detectUnusedMappings()">🔍 未使用検出</button>
</div>
```

### 4.4 マッピング管理機能

#### 編集モード
- トグルボタンで表示/非表示を切り替え
- 編集モードON時: 全マッピングに編集・削除ボタン表示
- 編集モードOFF時: マッチした項目のみ表示

#### マッピングCSV読込
- CSVファイルからマッピング定義をインポート
- フォーマット: `顧客コード,v15番号,項目名,カテゴリ`

#### マッピングCSV出力
- 現在のマッピング定義をCSVでエクスポート
- BOM付きUTF-8形式

#### 未使用検出
- インポートされたCSVに存在しないマッピングを検出
- 一括削除機能付き

### 4.5 未使用マッピングセクション

```html
<div id="unusedMappingSection" style="display:none;margin-bottom:15px;padding:15px;background:#fff3e0;border-radius:8px;border:1px solid #ff9800;">
    <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px;">
        <h4 style="margin:0;color:#e65100;">⚠️ 未使用マッピング（N件）</h4>
        <button class="btn btn-danger" onclick="removeUnusedMappings()">未使用を一括削除</button>
    </div>
    <div id="unusedMappingList" style="display:flex;flex-wrap:wrap;gap:8px;"></div>
</div>
```

### 4.6 類似候補セクション

```html
<div id="similarSuggestionsSection" style="display:none;margin-bottom:15px;padding:15px;background:#e3f2fd;border-radius:8px;border:1px solid #2196f3;">
    <h4 style="margin:0 0 10px 0;color:#1565c0;">💡 類似項目の提案</h4>
    <div id="similarSuggestionsList"></div>
</div>
```

---

## 5. 予算設定タブ

### 5.1 サブタブ構成

```
┌─────────────────────────────────────────────────────────────┐
│  サブタブナビゲーション                                       │
│  [1.予算入力][2.入力前演算][3.ラクミー入力値]                   │
│  [4.MQ出力][5.予算一覧][6.サマリー]                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                   サブタブコンテンツ                          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 5.2 サブタブ一覧

| タブ | ID | テキスト | ヘッダー色 |
|------|-----|---------|-----------|
| 予算入力 | tab-input | 【1】予算入力 | オレンジグラデーション |
| 入力前演算 | tab-calc | 【2】入力前演算 | シアングラデーション |
| ラクミー入力値 | tab-rakumy | 【3】ラクミー入力値 | グリーングラデーション |
| MQ出力 | tab-output | 【4】MQ出力 | インディゴグラデーション |
| 予算一覧 | tab-budget-list | 【5】予算一覧 | パープルグラデーション |
| サマリー | tab-summary | 【6】サマリー | バイオレットグラデーション |

### 5.3 予算入力タブ（【1】）

#### ヘッダースタイル
```css
.section-header.header-input {
  background: linear-gradient(135deg, #ff6b6b 0%, #ffa726 100%);
}
```

#### テーブル構造
```
┌────────┬────┬────┬────┬...┬────┬────┐
│ 項目   │ 4月│ 5月│ 6月│...│合計│平均│
├────────┼────┼────┼────┼...┼────┼────┤
│[5]売上 │xxxx│xxxx│xxxx│...│xxxx│xxxx│
│[6]F仕入│xxxx│xxxx│xxxx│...│xxxx│xxxx│
└────────┴────┴────┴────┴...┴────┴────┘
```

### 5.4 入力前演算タブ（【2】）

#### ヘッダースタイル
```css
.section-header.header-calc {
  background: linear-gradient(135deg, #26c6da 0%, #00acc1 100%);
}
```

#### 表示項目
- 売上合計
- FD仕入費合計 / FD仕入費率（%）
- PA人件費合計 / PA人件費率（%）
- 固定人件費合計
- ランチ/ディナー/フード/ドリンク/その他売上比率
- 地代家賃合計 / 資材費合計

### 5.5 ラクミー入力値タブ（【3】）

#### ヘッダースタイル
```css
.section-header.header-rakumy {
  background: linear-gradient(135deg, #66bb6a 0%, #43a047 100%);
}
```

#### カテゴリセクション

| カテゴリ | ヘッダー背景 | ボーダー色 |
|----------|-------------|-----------|
| 費用予算設定（固定費用） | #e8f5e9 | #4caf50 |
| 単月費用予算（固定費用） | #fff3e0 | #ff9800 |
| 予算設定（変動費用） | #e3f2fd | #2196f3 |

#### 変換タイプバッジ

| タイプ | クラス | 背景色 |
|--------|--------|--------|
| 直接 | type-direct | #2196f3 |
| 合計 | type-sum | #9c27b0 |
| 率 | type-rate | #e91e63 |
| 入力 | type-input | #4caf50 |
| 出力 | type-output | #607d8b |

### 5.6 MQ出力タブ（【4】）

#### ヘッダースタイル
```css
.section-header.header-output {
  background: linear-gradient(135deg, #5c6bc0 0%, #3949ab 100%);
}
```

#### 出力項目
- 客数、客単価、売上
- 変動費、変動費率
- 粗利、粗利率
- 固定費、営業利益、営業利益率

#### 顧客予算との比較セクション

| 行タイプ | 背景色 | ボーダー色 |
|---------|--------|-----------|
| MQ計算値 | #e3f2fd | #1565c0 |
| 顧客予算 | #fff8e1 | #f57c00 |
| 差異（+） | #e8f5e9 | 緑 |
| 差異（-） | #ffebee | 赤 |

### 5.7 予算一覧タブ（【5】）

#### セクションヘッダー色

| セクション | 背景色 |
|-----------|--------|
| 目標設定 | #1565c0（青） |
| 売上 | #2e7d32（緑） |
| FLコスト | #f57c00（オレンジ） |
| その他経費 | #7b1fa2（紫） |
| 営業利益 | #c62828（赤） |

### 5.8 サマリータブ（【6】）

#### サマリーグリッド
```css
.summary-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 15px;
}
```

#### 表示項目
| カード | 単位 |
|--------|------|
| 年間売上 | 千円 |
| 年間営業利益 | 千円 |
| 営業利益率 | % |
| 年間客数 | 人 |
| 平均客単価 | 円 |
| 変動費率 | % |

---

## 6. エクスポートタブ

### 6.1 構成

```
┌─────────────────────────────────────────────────────────────┐
│                 出力データ履歴一覧                            │
├─────────────────────────────────────────────────────────────┤
│ ヘッダー: 出力データ履歴一覧（N件）    [全履歴クリア]          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📊 ラクミー入力値CSV（N件）                                 │
│  ├─ [履歴カード1] [履歴カード2] ...                         │
│                                                             │
│  📋 予算一覧CSV（N件）                                       │
│  ├─ [履歴カード1] [履歴カード2] ...                         │
│                                                             │
│  📈 MQ出力CSV（N件）                                         │
│  ├─ [履歴カード1] [履歴カード2] ...                         │
│                                                             │
│  ⚙️ 設定JSON（N件）                                          │
│  ├─ [履歴カード1] [履歴カード2] ...                         │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 カテゴリ別グループ表示

#### カテゴリ定義
| カテゴリ | ラベル | メインカラー | 背景色 |
|----------|--------|--------------|--------|
| ラクミー入力値CSV | 📊 ラクミー入力値CSV | #4caf50（緑） | #e8f5e9 |
| 予算一覧CSV | 📋 予算一覧CSV | #2196f3（青） | #e3f2fd |
| MQ出力CSV | 📈 MQ出力CSV | #9c27b0（紫） | #f3e5f5 |
| 設定JSON | ⚙️ 設定JSON | #607d8b（グレー） | #eceff1 |

#### カテゴリヘッダースタイル
```css
.category-header {
  font-weight: bold;
  padding: 8px 12px;
  border-radius: 6px 6px 0 0;
  margin-bottom: 0;
  border-left: 4px solid;
}
```

### 6.3 データ表示モーダル

#### 構成
```html
<div id="dataViewModal" style="display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.7);z-index:9999;">
    <div style="position:absolute;top:50%;left:50%;transform:translate(-50%,-50%);background:#fff;border-radius:8px;width:95%;max-width:1400px;max-height:90vh;overflow:hidden;display:flex;flex-direction:column;">
        <div style="padding:15px 20px;border-bottom:1px solid #ddd;display:flex;justify-content:space-between;align-items:center;background:#f5f5f5;">
            <h3 id="dataViewTitle" style="margin:0;font-size:16px;"></h3>
            <button onclick="hideDataViewModal()" style="background:none;border:none;font-size:24px;cursor:pointer;padding:0 10px;">&times;</button>
        </div>
        <div id="dataViewContent" style="padding:15px;overflow:auto;flex:1;"></div>
    </div>
</div>
```

#### 機能
- CSVデータをテーブル形式で表示
- BOM（Byte Order Mark）除去処理
- セクションタイトル行のフィルタリング
- ヘッダー行は`<th>`タグ、データ行は`<td>`タグで表示

#### タイトル行フィルタリング
【】で始まるセクションタイトル行を非表示：
```javascript
// 【】で始まる行をスキップ
if (trimmedLine.indexOf('【') === 0) return;
// 最初のセルが空でタイトルっぽい行もスキップ
if (cells.length > 1 && !cells[0].trim() && cells[1].trim().indexOf('【') === 0) return;
```

#### テーブルスタイル
```css
.data-view-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 11px;
}
.data-view-table th,
.data-view-table td {
  border: 1px solid #ddd;
  padding: 6px 8px;
  text-align: left;
  white-space: nowrap;
}
.data-view-table th {
  background: #1565c0;
  color: white;
  font-weight: bold;
  position: sticky;
  top: 0;
}
.data-view-table tr:nth-child(even) td {
  background: #f9f9f9;
}
.data-view-table tr:hover td {
  background: #e3f2fd;
}
```

### 6.4 履歴カード

```html
<div class="export-history-item">
    <div style="display:flex;justify-content:space-between;align-items:flex-start;">
        <div>
            <div class="export-history-title">2024-12-27_15:30_店舗A_ラクミー入力値</div>
            <div class="export-history-meta">保存: 2024/12/27 15:30</div>
            <div class="export-history-meta">店舗: 店舗A (1店舗)</div>
        </div>
        <div class="export-history-actions">
            <button class="view-btn" onclick="showExportHistoryItem(id)">👁</button>
            <button class="save-btn" onclick="downloadExportHistoryItem(id)">💾</button>
            <button class="delete-btn" onclick="deleteExportHistoryItem(id)">🗑</button>
        </div>
    </div>
</div>
```

### 6.5 履歴カードスタイル

```css
.export-history-item {
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 6px;
  padding: 10px 12px;
  margin-bottom: 8px;
}
.export-history-item:hover {
  border-color: #1565c0;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}
.export-history-title {
  font-weight: bold;
  font-size: 12px;
  color: #333;
  margin-bottom: 4px;
}
.export-history-meta {
  font-size: 10px;
  color: #666;
}
.export-history-actions {
  display: flex;
  gap: 6px;
}
.export-history-actions button {
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}
.view-btn { background: #2196f3; color: #fff; }
.save-btn { background: #4caf50; color: #fff; }
.delete-btn { background: #f44336; color: #fff; }
```

### 6.6 全履歴クリアボタン

```html
<button class="btn btn-danger" onclick="clearAllExportHistory()">
    🗑 全履歴クリア
</button>
```

### 6.7 データ自動保存

- CSVインポート完了時に自動保存（ラクミー入力値CSV、MQ出力CSV）
- CSVエクスポートボタンクリック時に自動保存
- localStorageに履歴を永続化（最大100件）

---

## 7. 店舗セレクター

### 7.1 表示条件
- 複数店舗がある場合のみ表示

### 7.2 要素
```html
<div class="store-selector">
  <span>店舗選択 (N店舗):</span>
  <div class="store-chips">
    <span class="store-chip active" onclick="switchStore('店舗名')">店舗名</span>
    ...
  </div>
</div>
```

### 7.3 店舗チップスタイル
```css
.store-chip {
  padding: 4px 12px;
  border-radius: 15px;
  background: #e0e0e0;
  cursor: pointer;
  font-size: 11px;
}
.store-chip.active {
  background: #1565c0;
  color: white;
}
```

---

## 8. CSVエクスポート仕様

### 8.1 全店舗エクスポート

「全店舗を選択」時は1つのCSVに全店舗データを縦に連結：

```
店舗名,セクション,項目名,変換タイプ,4月,5月,...,3月,合計,平均
店舗A,費用予算設定,地代家賃,sum,450,450,...,450,5400,450
店舗A,費用予算設定,固定人件費,sum,500,500,...,500,6000,500
...
店舗B,費用予算設定,地代家賃,sum,380,380,...,380,4560,380
店舗B,費用予算設定,固定人件費,sum,450,450,...,450,5400,450
...
```

### 8.2 ファイル名規則

| 条件 | ファイル名 |
|------|-----------|
| 単一店舗 | `{種別}_{店舗名}_{YYYY-MM-DD}.csv` |
| 全店舗 | `{種別}_全店舗_{YYYY-MM-DD}.csv` |

### 8.3 出力形式

- 文字コード: UTF-8 with BOM（Excel対応）
- 金額: 千円単位、整数
- 率: 整数%表示

---

## 9. テーブル共通仕様

### 9.1 基本スタイル
```css
.annual-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 10px;
}
.annual-table th, .annual-table td {
  border: 1px solid #ddd;
  padding: 6px 8px;
  text-align: right;
}
.annual-table th {
  background: #f5f5f5;
  font-weight: bold;
}
```

### 9.2 合計・平均列
```css
.total-col { background: #fff9c4; font-weight: bold; }
.avg-col { background: #e3f2fd; }
```

---

## 10. 数値フォーマット

### 10.1 金額（千円単位）
```javascript
function fmt(val) {
  return Math.round(val).toLocaleString();
}
```

### 10.2 率（%）
```javascript
function fmtRate(val) {
  return val.toFixed(1);
}
```

---

## 11. レスポンシブ対応

### 11.1 テーブルスクロール
- 横スクロール対応 `overflow-x: auto`

### 11.2 グリッドレイアウト
- 画面幅に応じて列数を自動調整
- `grid-template-columns: repeat(auto-fill, minmax(300px, 1fr))`
