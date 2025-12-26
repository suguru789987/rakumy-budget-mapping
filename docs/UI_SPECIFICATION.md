# 予算シミュレーター v5.8 UI仕様書

## 1. 画面構成

### 1.1 全体レイアウト

```
┌─────────────────────────────────────────────────────────────┐
│                       ヘッダー                               │
│  予算シミュレーター v5.8 （顧客CSV対応版・変換ロジック強化）     │
├─────────────────────────────────────────────────────────────┤
│ CSVファイル選択 │ 設定エクスポート │ 設定インポート            │
├─────────────────────────────────────────────────────────────┤
│                   店舗セレクター                             │
├─────────────────────────────────────────────────────────────┤
│                     情報バー                                │
│  店舗名 │ 会計年度 │ 検出月数 │ マッチ項目 │ 未マッチ項目     │
├─────────────────────────────────────────────────────────────┤
│  タブナビゲーション                                          │
│  [マッピング設定][1.予算入力][2.入力前演算][3.ラクミー入力値]   │
│  [4.MQ出力][5.予算一覧][6.サマリー]                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│                    タブコンテンツ                            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

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

## 3. ツールバー

### 3.1 インポートセクション（1行目）

| 要素 | 種類 | ID/クラス | イベント |
|------|------|-----------|----------|
| CSVファイル選択 | `<input type="file">` | csvFile | `handleFileSelect(event)` |
| インポート実行 | `<button>` | btn-success | `importAllCSVs()` |
| シミュレーション実行 | `<button>` | btn-warning | `runSimulationForCurrentStore()` |

### 3.2 エクスポートセクション（2行目）

| ボタン | 背景色 | 関数 | 出力形式 |
|--------|--------|------|----------|
| ラクミー入力値CSV | #4caf50（緑） | `exportRakumyInputsCSV()` | CSV |
| 予算一覧CSV | #2196f3（青） | `exportBudgetListCSV()` | CSV |
| MQ出力CSV | #9c27b0（紫） | `exportMQOutputCSV()` | CSV |
| 設定JSON | btn-info | `exportAllConfig()` | JSON |
| 設定読込 | btn-secondary | `importAllConfig(event)` | JSON |
| 全店舗クリア | btn-danger | `clearAllStores()` | - |

### 3.3 ツールバーHTML構造
```html
<div class="import-controls">
    <input type="file" id="csvFile" accept=".csv" multiple>
    <button class="btn btn-primary">CSVファイル選択</button>
    <span id="fileName">ファイル未選択</span>
    <button class="btn btn-success">インポート実行</button>
    <button class="btn btn-warning">シミュレーション実行</button>
</div>
<div class="import-controls" style="margin-top:10px;">
    <span style="font-weight:bold;">エクスポート:</span>
    <button style="background:#4caf50;color:#fff;">ラクミー入力値CSV</button>
    <button style="background:#2196f3;color:#fff;">予算一覧CSV</button>
    <button style="background:#9c27b0;color:#fff;">MQ出力CSV</button>
    <button class="btn btn-info">設定JSON</button>
    <button class="btn btn-secondary">設定読込</button>
    <button class="btn btn-danger">全店舗クリア</button>
</div>
```

---

## 4. 店舗セレクター

### 4.1 表示条件
- 複数店舗がある場合のみ表示
- `display: flex` / `display: none`

### 4.2 要素
```html
<div class="store-selector">
  <span>店舗選択 (N店舗):</span>
  <div class="store-chips">
    <span class="store-chip active" onclick="switchStore('店舗名')">店舗名</span>
    ...
  </div>
</div>
```

### 4.3 店舗チップスタイル
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

## 5. 情報バー

### 5.1 レイアウト
```css
.info-bar {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
  padding: 10px 15px;
  background: #f8f9fa;
  border-radius: 6px;
}
```

### 5.2 表示項目
| ID | ラベル | 初期値 |
|----|--------|--------|
| detectedStore | 店舗名 | - |
| detectedFiscalYear | 会計年度 | - |
| detectedMonthCount | 検出月数 | - |
| detectedItemCount | マッチ項目 | - |
| unmatchedCount | 未マッチ項目 | - |

---

## 6. タブナビゲーション

### 6.1 タブ一覧
| タブ | ID | テキスト |
|------|-----|---------|
| マッピング設定 | tab-mapping | マッピング設定 |
| 予算入力 | tab-input | 【1】予算入力 |
| 入力前演算 | tab-calc | 【2】入力前演算 |
| ラクミー入力値 | tab-rakumy | 【3】ラクミー入力値 |
| MQ出力 | tab-output | 【4】MQ出力 |
| 予算一覧 | tab-budget-list | 【5】予算一覧 |
| サマリー | tab-summary | 【6】サマリー |

### 6.2 タブスタイル
```css
.tabs {
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
  margin-bottom: 15px;
}
.tab {
  padding: 8px 16px;
  border-radius: 4px 4px 0 0;
  cursor: pointer;
  background: #e0e0e0;
  font-size: 11px;
}
.tab.active {
  background: #1565c0;
  color: white;
}
```

### 6.3 タブ切り替え処理
```javascript
function showTab(tabId) {
  document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
  document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
  event.target.classList.add('active');
  document.getElementById(tabId).classList.add('active');
}
```

---

## 7. タブコンテンツ

### 7.1 マッピング設定タブ

#### 未マッチセクション
```html
<div class="unmatched-section">
  <h3>未マッチ項目</h3>
  <div id="unmatchedList"></div>
  <!-- 新規マッピングフォーム -->
</div>
```

#### マッピング一覧テーブル
| カラム | 説明 |
|--------|------|
| コード | 顧客CSVの項目コード |
| 項目名 | 顧客CSVの項目名 |
| → | 矢印 |
| v15 No | マッピング先番号 |
| v15項目名 | マッピング先名称 |
| カテゴリ | 項目カテゴリ |
| 操作 | 削除ボタン |

#### 処理ログ
```html
<div id="preprocessLog" class="preprocess-log">
  CSVをインポートしてください
</div>
```

---

### 7.2 予算入力タブ（【1】）

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

---

### 7.3 入力前演算タブ（【2】）

#### ヘッダースタイル
```css
.section-header.header-calc {
  background: linear-gradient(135deg, #26c6da 0%, #00acc1 100%);
}
```

#### 表示項目
- 売上合計
- FD仕入費合計
- FD仕入費率（%表示）
- PA人件費合計
- PA人件費率（%表示）
- 固定人件費合計
- ランチ売上比率
- ディナー売上比率
- フード売上比率
- ドリンク売上比率
- その他売上比率
- 地代家賃合計
- 資材費合計

---

### 7.4 ラクミー入力値タブ（【3】）

#### ヘッダースタイル
```css
.section-header.header-rakumy {
  background: linear-gradient(135deg, #66bb6a 0%, #43a047 100%);
}
```

#### カテゴリセクション

##### 費用予算設定（固定費用）
```css
.rakumy-category.cost h4 {
  background: #e8f5e9;
  color: #2e7d32;
  border-left: 4px solid #4caf50;
}
```

##### 単月費用予算（固定費用）
```css
.rakumy-category.monthly h4 {
  background: #fff3e0;
  color: #e65100;
  border-left: 4px solid #ff9800;
}
```

##### 予算設定（変動費用）
```css
.rakumy-category.budget h4 {
  background: #e3f2fd;
  color: #1565c0;
  border-left: 4px solid #2196f3;
}
```

#### 変換タイプバッジ
| タイプ | クラス | 背景色 | 説明 |
|--------|--------|--------|------|
| 直接 | type-direct | #2196f3 | 顧客予算をそのまま転記 |
| 合計 | type-sum | #9c27b0 | 複数項目を合算 |
| 率 | type-rate | #e91e63 | 売上に対する率を計算 |
| 入力 | type-input | #4caf50 | ユーザー入力値 |
| 出力 | type-output | #607d8b | MQ計算で算出 |

#### 設定バッジ
| タイプ | クラス | 背景色 | 説明 |
|--------|--------|--------|------|
| 年間固定 | annual-badge | #4caf50 | 全月同一値 |
| 単月設定 | monthly-badge | #ff9800 | 月別に値が異なる |

---

### 7.5 MQ出力タブ（【4】）

#### ヘッダースタイル
```css
.section-header.header-output {
  background: linear-gradient(135deg, #5c6bc0 0%, #3949ab 100%);
}
```

#### 出力テーブル構造
```
┌──────────┬────┬────┬...┬────┬────┐
│ 項目     │ 4月│ 5月│...│合計│平均│
├──────────┼────┼────┼...┼────┼────┤
│ 客数     │xxxx│xxxx│...│xxxx│xxxx│
│ 客単価   │xxxx│xxxx│...│xxxx│xxxx│
│ 売上     │xxxx│xxxx│...│xxxx│xxxx│
│ 変動費   │xxxx│xxxx│...│xxxx│xxxx│
│ 変動費率 │xx.x│xx.x│...│  - │xx.x│
│ 粗利     │xxxx│xxxx│...│xxxx│xxxx│
│ 粗利率   │xx.x│xx.x│...│  - │xx.x│
│ 固定費   │xxxx│xxxx│...│xxxx│xxxx│
│ 営業利益 │xxxx│xxxx│...│xxxx│xxxx│
│ 営業利益率│xx.x│xx.x│...│  - │xx.x│
└──────────┴────┴────┴...┴────┴────┘
```

#### 顧客予算との比較セクション

##### セクションヘッダー
```css
background: #1565c0;
color: white;
font-size: 14px;
text-align: center;
```

##### グループヘッダー（【売上】対比、【営業利益】対比）
```css
background: #263238;
color: white;
```

##### MQ計算値行
```css
background: #e3f2fd;
border-left: 4px solid #1565c0;
color: #1565c0;
font-weight: bold;
```

##### 顧客予算行
```css
background: #fff8e1;
border-left: 4px solid #f57c00;
color: #e65100;
font-weight: bold;
```

##### 差異行
```css
/* プラスの場合 */
background: #e8f5e9;
color: #2e7d32;

/* マイナスの場合 */
background: #ffebee;
color: #c62828;
```

---

### 7.6 予算一覧タブ（【5】）

#### ヘッダースタイル
```css
background: linear-gradient(135deg, #1a237e 0%, #4a148c 100%);
color: white;
```

#### セクションヘッダー色
| セクション | 背景色 |
|-----------|--------|
| 目標設定 | #1565c0（青） |
| 売上 | #2e7d32（緑） |
| FLコスト | #f57c00（オレンジ） |
| その他経費 | #7b1fa2（紫） |
| 営業利益 | #c62828（赤） |

#### 行スタイル
- 親項目: 通常背景
- 子項目（インデント）: `background: #f5f5f5;`

---

### 7.7 サマリータブ（【6】）

#### ヘッダースタイル
```css
.section-header.header-summary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}
```

#### サマリーグリッド
```css
.summary-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 15px;
}
```

#### サマリーカード
```html
<div class="summary-card">
  <div class="summary-label">年間売上</div>
  <div class="summary-value">xxx,xxx</div>
  <div class="summary-sub">千円</div>
</div>
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

## 8. テーブル共通仕様

### 8.1 基本スタイル
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

### 8.2 月ヘッダー
```css
.month-header {
  min-width: 55px;
  text-align: center;
}
```

### 8.3 合計・平均列
```css
.total-col {
  background: #fff9c4;
  font-weight: bold;
}
.avg-col {
  background: #e3f2fd;
}
```

### 8.4 セル種別
| クラス | 用途 |
|--------|------|
| input-cell | 予算入力値 |
| calc-cell | 演算結果 |
| rakumy-cell | ラクミー入力値 |
| output-cell | MQ出力値 |

---

## 9. ステータスメッセージ

### 9.1 種別
| 種別 | 背景色 | アイコン |
|------|--------|----------|
| info | #e3f2fd | 青 |
| success | #e8f5e9 | 緑 |
| error | #ffebee | 赤 |

### 9.2 表示位置
- ヘッダー直下
- 全幅表示

---

## 10. レスポンシブ対応

### 10.1 テーブルスクロール
- 横スクロール対応
- `overflow-x: auto`

### 10.2 タブ折り返し
```css
.tabs {
  flex-wrap: wrap;
}
```

### 10.3 サマリーグリッド
- 画面幅に応じて列数を調整
- 最小幅でカードが縦並びに

---

## 11. 数値フォーマット

### 11.1 金額（千円単位）
```javascript
function fmt(val) {
  return Math.round(val).toLocaleString();
}
// 例: 12345678 → "12,345,678"
```

### 11.2 率（%）
```javascript
function fmtRate(val) {
  return val.toFixed(1);
}
// 例: 32.567 → "32.6"
```
