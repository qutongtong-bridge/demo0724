# 充電チェックインフロー設計書

## 概要

充電チェックインは、ユーザーが充電サービスを利用する最初のステップです。全体のプロセスは4つのステップに分かれており、上部の進捗インジケーターで現在の段階を表示します。

## ステップ1：車室選択

### ページタイトル
- 「充電前準備」

### 進捗表示
- 4つの点、1つ目がハイライト表示
- ステップラベル：車室選択

### メインコンテンツ
- **プロンプトテキスト**：「番号を選択してください。」
- **充電器情報表示**：
  - 番号：1
  - 充電仕様：普通 6/3kW ¥~100/15分
  - 1日最大：¥100,000
  - ケーブル有無：ケーブル有
- **視覚要素**：充電器アイコン表示

### インタラクション詳細
- ユーザーは充電器番号を選択する必要がある
- 充電器の詳細仕様を表示
- 下部にヘルプリンク：「駐車する車室が分からない」

### デザイン特徴
- 明確な番号選択
- 重要情報の目立つ表示
- 充電設備の視覚的表現

### APIインターフェース

#### 利用可能な充電ステーションの取得
```
GET /api/v1/stations/available
```
**レスポンス:**
```json
{
  "stations": [
    {
      "id": 1,
      "number": "1",
      "type": "standard",
      "power_options": ["6kW", "3kW"],
      "price_per_15min": 100,
      "daily_max": 100000,
      "cable_available": true,
      "status": "available"
    }
  ]
}
```

#### 充電ステーションの選択
```
POST /api/v1/checkin/select-station
```
**リクエスト:**
```json
{
  "station_id": 1,
  "user_id": "user123"
}
```
**レスポンス:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "next_step": "power_selection"
}
```

## ステップ2：出力選択

### ページタイトル
- 「充電前準備」

### 進捗表示
- 4つの点、2つ目がハイライト表示
- ステップラベル：出力選択

### メインコンテンツ
- **プロンプトテキスト**：「出力電力を選択してください。」
- **出力オプション**：
  1. **6kW**
     - 価格：¥100/15分
     - 1日最大：¥100,000
  2. **3kW**
     - 価格：¥50/15分
     - 1日最大：¥50,000

### インタラクション詳細
- ユーザーは充電出力を選択する必要がある
- 異なる出力レベルには異なる料金設定
- 選択されたオプションは視覚的フィードバックを表示

### デザイン特徴
- 明確な価格比較
- ラジオボタン選択
- ハイライトされた選択状態

### APIインターフェース

#### 出力電力の選択
```
POST /api/v1/checkin/select-power
```
**リクエスト:**
```json
{
  "session_id": "sess_abc123",
  "power_option": "6kW",
  "price_per_15min": 100
}
```
**レスポンス:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "selected_power": "6kW",
  "estimated_cost": {
    "per_15min": 100,
    "daily_max": 100000
  },
  "next_step": "confirmation"
}
```

## ステップ3：内容確認

### ページタイトル
- 「充電前準備」

### 進捗表示
- 4つの点、3つ目がハイライト表示
- ステップラベル：内容確認

### メインコンテンツ
- **プロンプトテキスト**：「SBPSテストの利用を開始します。」
- **確認情報**：
  - テスト名：SBPSテスト
  - 住所：東京都品川区大崎２丁目１－１
  - Paymentバッジ
- **充電器情報**：
  - 普通 6kW（ケーブル有）
  - 価格：¥100/15分（1日最大 ¥100,000）
- **料金詳細**：
  - 利用料金：¥100/15分
  - ご請求予定合計料金：¥334
- **重要な注意事項**：
  - 実際の利用時間に応じた課金説明
  - 準備時間の無料説明
  - 駐車料金は別途発生
  - 超過料金の説明

### デザイン特徴
- 包括的な情報表示
- 明確な料金内訳
- 重要な注意事項の強調

### APIインターフェース

#### 充電サマリーの取得
```
GET /api/v1/checkin/summary/{session_id}
```
**レスポンス:**
```json
{
  "session_id": "sess_abc123",
  "station": {
    "name": "SBPSテスト",
    "address": "東京都品川区大崎２丁目１－１",
    "number": "1"
  },
  "charging_details": {
    "power": "6kW",
    "cable_available": true,
    "price_per_15min": 100,
    "daily_max": 100000
  },
  "estimated_total": 334,
  "free_prep_time": 1,
  "terms": {
    "actual_time_billing": true,
    "parking_fee_separate": true,
    "overtime_charges": true
  }
}
```

#### 充電の確認
```
POST /api/v1/checkin/confirm
```
**リクエスト:**
```json
{
  "session_id": "sess_abc123",
  "user_confirmation": true,
  "payment_method": "default"
}
```
**レスポンス:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "charging_id": "chrg_xyz789",
  "next_step": "parking_guidance"
}
```

## ステップ4：入庫案内

### ページタイトル
- 「充電前準備」

### 進捗表示
- 4つの点、4つ目がハイライト表示
- ステップラベル：入庫

### メインコンテンツ
- **駐車ガイドイラスト**：
  - 充電ステーションレイアウト表示
  - 「PARKING」位置マーカー
  - 車両駐車図
- **安全指示**：
  - 「スマートバリアが完全に下りた後、周囲の安全を確認してから、駐車してください。」
- **確認ボタン**：「OK」

### インタラクション詳細
- 最終的な安全注意喚起
- ユーザーが確認して充電を開始

### デザイン特徴
- 視覚的な駐車ガイダンス
- 安全第一のアプローチ
- 明確なコールトゥアクション

### APIインターフェース

#### 充電セッションの開始
```
POST /api/v1/charging/start
```
**リクエスト:**
```json
{
  "charging_id": "chrg_xyz789",
  "station_id": 1,
  "user_confirmation": "parked"
}
```
**レスポンス:**
```json
{
  "success": true,
  "charging_session": {
    "id": "chrg_xyz789",
    "status": "active",
    "start_time": "2024-02-15T19:12:00Z",
    "station_id": 1,
    "power": "6kW"
  },
  "smart_barrier_status": "lowering"
}
```

#### バリアステータスの更新（WebSocket）
```
WS /api/v1/charging/barrier-status
```
**メッセージ:**
```json
{
  "charging_id": "chrg_xyz789",
  "barrier_status": "lowered",
  "safe_to_proceed": true
}
```

## デザイン仕様

### ビジュアルデザイン
- 明確なステップ進捗表示
- 重要情報のハイライト
- 理解を助けるイラスト
- 一貫したカラーシステム

### ユーザーエクスペリエンス
- 明確で順次的なステップ
- 重要情報の複数回確認
- 誤解を避ける透明な料金
- 包括的な安全のヒント

### インタラクションデザイン
- 各ステップでユーザーの能動的な確認が必要
- 選択を変更するための戻る機能をサポート
- ヘルプ情報は常にアクセス可能

## カラースキーム
- プライマリー：ティール（#00BCD4）
- アクセント：オレンジ（#FF6B35）
- 背景：ライトグレー（#F5F5F5）
- テキスト：ダークグレー（#333333）

## タイポグラフィ
- ヘッダー：20sp、太字
- 本文：16sp、標準
- キャプション：14sp、標準
- ボタンテキスト：16sp、中太字