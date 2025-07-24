# 充電状態管理設計書

## 概要

充電状態管理モジュールは、ユーザーにリアルタイムの充電状態監視と制御機能を提供し、残り時間表示、充電終了オプション、充電完了リマインダーなどを含みます。

## 充電中画面1：充電進行中

### ページタイトル
- 「充電可能です...」

### 主要機能
1. **残り時間表示**
   - 大きなフォントで残り時間を表示：「残り時間 0:27」
   - 時計アイコンによる視覚的サポート
   - オレンジ色の円形プログレスバーで充電進捗を表示

2. **充電終了ボタン**
   - 白いボタン：「利用を終了する」
   - ヘルプリンク：「利用を終了するには」

3. **利用状況情報**
   - 利用開始時刻：19:12
   - 利用終了時刻：19:40
   - 利用時間：1分未満切り上げ
   - うち無料時間：準備時間として -1分

### デザイン特徴
- 残り時間に焦点を当てたクリーンなインターフェースデザイン
- 直感的な充電進捗表示のための円形プログレスバー
- 視覚的疲労を軽減するグレー背景

### APIインターフェース

#### 充電状態取得
```
GET /api/v1/charging/status/{session_id}
```
**レスポンス:**
```json
{
  "session_id": "sess_abc123",
  "status": "charging",
  "start_time": "2024-02-15T19:12:00Z",
  "estimated_end_time": "2024-02-15T19:40:00Z",
  "remaining_minutes": 27,
  "progress_percentage": 3.7,
  "current_power_kw": 6,
  "energy_delivered_kwh": 0.5,
  "free_minutes_used": 1,
  "current_cost": 0
}
```

#### リアルタイム充電データ（WebSocket）
```
WS /api/v1/charging/realtime/{session_id}
```
**メッセージ:**
```json
{
  "type": "status_update",
  "timestamp": "2024-02-15T19:13:00Z",
  "remaining_seconds": 1620,
  "progress_percentage": 3.7,
  "current_power_kw": 6.2,
  "total_energy_kwh": 0.52,
  "connection_status": "connected"
}
```

## 充電中画面2：終了確認ダイアログ

### ダイアログタイトル
- 「利用を終了するには？」

### ダイアログ内容
1. **操作説明イラスト**
   - 充電器取り外し手順を表示
   - ユーザー操作図

2. **テキスト説明**
   - 「充電器を抜いてそのまま車室から退出する、または、「利用を終了する」ボタンを押すと利用を終了することができます。」

3. **アクションボタン**
   - 「OK」確認ボタン

### インタラクションデザイン
- 半透明オーバーレイ背景
- 明確な操作ガイダンス
- シンプルな確認プロセス

## 充電中画面3：退出リマインダーダイアログ

### ダイアログタイトル
- 「利用終了後は、EV車室からご退出ください。」

### ダイアログ内容
1. **警告アイコン**
   - 「CAUTION!」警告サイン
   - オレンジ色の強調色

2. **重要なリマインダー**
   - 「準備が整ってから利用を終了してください。」

3. **料金通知**
   - 「※終了後もEV車室に停車されている場合、ペナルティ料金の発生や、アカウント停止をする場合があります。」

4. **アクションボタン**
   - 「利用を終了する」オレンジボタン

### デザイン要素
- 目立つ警告通知
- 明確な料金説明
- ユーザーエラーの防止

### APIインターフェース

#### 充電終了
```
POST /api/v1/charging/terminate
```
**リクエスト:**
```json
{
  "session_id": "sess_abc123",
  "termination_type": "user_initiated",
  "confirm": true
}
```
**レスポンス:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "termination_time": "2024-02-15T19:13:30Z",
  "final_duration_minutes": 1,
  "status": "terminating",
  "estimated_completion": "2024-02-15T19:13:45Z"
}
```

#### 終了状態取得
```
GET /api/v1/charging/termination-status/{session_id}
```
**レスポンス:**
```json
{
  "session_id": "sess_abc123",
  "status": "terminated",
  "safe_to_unplug": true,
  "barrier_status": "open",
  "exit_deadline": "2024-02-15T19:18:45Z",
  "penalty_warning": {
    "grace_period_minutes": 5,
    "penalty_per_minute": 100
  }
}
```

## 充電完了画面

### ページタイトル
- 「ご利用ありがとうございました。」

### メインコンテンツ
1. **完了イラスト**
   - 充電ステーションから離れる車両のアニメーション
   - 「THANK YOU」メッセージ
   - フレンドリーな終了体験

2. **料金サマリー**
   - 料金合計：¥500
   - 金額のオレンジ色ハイライト

3. **利用詳細**
   - 充電ステーション：SBPSテスト
   - 利用時間：2024/2/15(木) 19:12～19:13
   - 利用料金：¥0
   - 予約料金：¥500
   - 料金単価：¥100/15分
   - 充電サービス利用時間：0分（無料時間5分を除く）

4. **アクションオプション**
   - 「OK」確認ボタン
   - 「領収書を発行する」リンク

### ユーザーエクスペリエンス最適化
- フレンドリーな完了画面
- 明確な料金内訳
- 便利な領収書発行

### APIインターフェース

#### 充電完了詳細取得
```
GET /api/v1/charging/completion/{session_id}
```
**レスポンス:**
```json
{
  "session_id": "sess_abc123",
  "station": {
    "name": "SBPSテスト",
    "id": 1
  },
  "usage_period": {
    "start": "2024-02-15T19:12:00Z",
    "end": "2024-02-15T19:13:00Z",
    "duration_minutes": 1,
    "charged_minutes": 0,
    "free_minutes": 1
  },
  "charges": {
    "usage_fee": 0,
    "reservation_fee": 500,
    "total": 500,
    "unit_price": "¥100/15分"
  },
  "energy_delivered": {
    "kwh": 0.1,
    "average_power_kw": 6
  },
  "receipt_available": true
}
```

## デザイン仕様

### カラー使用
- プライマリー：ティール（#00BCD4）タイトルと重要情報用
- アクセント：オレンジ（#FF6B35）ボタンと進捗表示用
- 背景：ライトグレー（#F5F5F5）
- テキスト：ダークグレー（#333333）

### インタラクション原則
1. リアルタイム充電状態更新
2. 重要操作の二次確認
3. 透明な料金情報
4. シンプルで直感的な操作フロー

### 安全性への配慮
- 充電終了の確認ステップ
- 明確な料金とペナルティ通知
- 誤操作防止のデザイン

## 技術要件

### 更新頻度
- 残り時間：1秒ごと
- プログレスバー：10秒ごと
- 状態変更：リアルタイム

### 状態管理
- アクティブ充電状態
- 一時停止状態
- 完了状態
- エラー状態

### 通知システム
- 完了5分前警告
- 完了通知
- 超過時間警告

### その他のAPIインターフェース

#### 充電通知送信
```
POST /api/v1/charging/notifications
```
**リクエスト:**
```json
{
  "session_id": "sess_abc123",
  "notification_type": "5_minute_warning",
  "delivery_method": ["push", "email"]
}
```
**レスポンス:**
```json
{
  "success": true,
  "notifications_sent": [
    {
      "type": "push",
      "status": "delivered",
      "timestamp": "2024-02-15T19:35:00Z"
    },
    {
      "type": "email",
      "status": "queued",
      "timestamp": "2024-02-15T19:35:01Z"
    }
  ]
}
```

#### 充電履歴取得
```
GET /api/v1/charging/history
```
**クエリパラメータ:**
- `user_id`: "user123"
- `limit`: 10
- `offset`: 0

**レスポンス:**
```json
{
  "total_count": 25,
  "sessions": [
    {
      "session_id": "sess_abc123",
      "date": "2024-02-15",
      "station_name": "SBPSテスト",
      "duration_minutes": 1,
      "energy_kwh": 0.1,
      "cost": 500,
      "status": "completed"
    }
  ]
}
```

#### 充電問題報告
```
POST /api/v1/charging/report-issue
```
**リクエスト:**
```json
{
  "session_id": "sess_abc123",
  "issue_type": "unable_to_terminate",
  "description": "終了ボタンが反応しません",
  "urgency": "high"
}
```
**レスポンス:**
```json
{
  "success": true,
  "ticket_id": "ticket_789",
  "support_contact": "+81-3-1234-5678",
  "estimated_response_time": "5 minutes"
}
```