# 施設検索とフィルター設計書

## 概要

施設検索とフィルター機能は、ユーザーがニーズに合った充電ステーションを素早く見つけるのに役立ち、地図表示、検索機能、多次元フィルタリングを含みます。

## 1. 地図メインインターフェース

### ページレイアウト
- **地図表示エリア**：
  - フルスクリーン地図背景
  - ユーザーの現在位置マーカー（青い点）
  - 充電ステーション位置マーカー（緑のアイコン）
  - 通り名ラベル

- **施設情報カード**：
  - 下部に配置、上下にスワイプ可能
  - 選択した充電ステーション情報を表示

- **アクションボタン**：
  - 右下：メニューボタン（3本の横線）
  - 右下：位置ボタン（矢印アイコン）

### 施設情報表示
- **タイトル**：きたやま関電施設
- **利用可能状態**：空（急速:1/1）- 緑の点インジケーター
- **距離**：現在地から 57m
- **営業状態**：営業中｜8:00～23:00
- **予約オプション**：
  - 今すぐ（即時）- プラグアイコン
  - 取り置き - 1h時計アイコン
  - 1日予約 - カレンダーアイコン

### APIインターフェース

#### 近くのステーションを取得
```
GET /api/v1/stations/nearby
```
**クエリパラメータ:**
- `latitude`: 35.6762
- `longitude`: 139.6503
- `radius`: 5000 (メートル)

**レスポンス:**
```json
{
  "stations": [
    {
      "id": 1,
      "name": "きたやま関電施設",
      "location": {
        "latitude": 35.6765,
        "longitude": 139.6510,
        "address": "東京都品川区北品川5-3-1"
      },
      "distance": 57,
      "status": "available",
      "chargers": {
        "quick": {
          "available": 1,
          "total": 1
        },
        "standard": {
          "available": 0,
          "total": 0
        }
      },
      "business_hours": {
        "open": "08:00",
        "close": "23:00",
        "is_open": true
      },
      "reservation_types": ["immediate", "hold", "all_day"]
    }
  ]
}
```

#### ステーション詳細を取得
```
GET /api/v1/stations/{station_id}
```
**レスポンス:**
```json
{
  "id": 1,
  "name": "きたやま関電施設",
  "location": {
    "latitude": 35.6765,
    "longitude": 139.6510,
    "address": "東京都品川区北品川5-3-1"
  },
  "chargers": [
    {
      "id": "chrg_001",
      "type": "quick",
      "power": "50kW",
      "status": "available",
      "price_per_15min": 500
    }
  ],
  "amenities": ["toilet", "convenience_store"],
  "payment_methods": ["credit_card", "ic_card", "app"],
  "photos": ["station_photo1.jpg", "station_photo2.jpg"]
}
```

## 2. 検索インターフェース

### 検索ボックスデザイン
- **入力プロンプト**：「行き先はどちらですか？」
- **検索候補リスト**：
  - きたやま関電施設
    - 急速:1/1（緑の状態）
    - 57m｜東京都品川区北品川5-3-1
    - 営業中｜8:00～23:00
  - SBPSテスト
    - 普通:0/1（赤の状態）
    - 510m｜東京都品川区大崎２丁目１－１
    - 営業中｜6:00～22:00

### キーボードインタラクション
- 日本語入力サポート
- リアルタイム検索候補
- クイック入力オプション

### APIインターフェース

#### ステーション検索
```
POST /api/v1/stations/search
```
**リクエスト:**
```json
{
  "query": "きたやま",
  "location": {
    "latitude": 35.6762,
    "longitude": 139.6503
  },
  "max_results": 10
}
```
**レスポンス:**
```json
{
  "results": [
    {
      "id": 1,
      "name": "きたやま関電施設",
      "match_score": 0.95,
      "location": {
        "address": "東京都品川区北品川5-3-1",
        "distance": 57
      },
      "availability": {
        "quick": "1/1",
        "standard": "0/0"
      },
      "business_hours": {
        "display": "8:00～23:00",
        "is_open": true
      }
    },
    {
      "id": 2,
      "name": "SBPSテスト",
      "match_score": 0.75,
      "location": {
        "address": "東京都品川区大崎２丁目１－１",
        "distance": 510
      },
      "availability": {
        "quick": "0/0",
        "standard": "0/1"
      },
      "business_hours": {
        "display": "6:00～22:00",
        "is_open": true
      }
    }
  ]
}
```

## 3. フィルター機能

### フィルター次元

#### 3.1 充電種別
- **オプション**（複数選択）：
  - ✓ 急速充電
  - ✓ 普通充電

#### 3.2 利用タイプ
- **オプション**（複数選択）：
  - ✓ 取り置き
  - ✓ 一日予約
  - ✓ 予約なしでの利用
- **ヘルプリンク**：「取り置きとは？」

#### 3.3 施設種別
- **オプション**（複数選択）：
  - ✓ プラゴの充電器を表示
  - ✓ その他の充電器を表示

#### 3.4 その他
- **オプション**（複数選択）：
  - ✓ 設置予定の充電器を表示

### フィルター操作
- **リセット機能**：「すべて表示」- オレンジ色のテキスト
- **フィルター適用**：「13746件の施設」- 青いボタン
- **閉じるボタン**：右上のX

### インタラクションデザイン
- チェックボックスは複数選択をサポート
- フィルター結果数のリアルタイム更新
- フィルター条件は保存可能
- ワンクリックですべてのフィルターをリセット

### APIインターフェース

#### フィルター適用
```
POST /api/v1/stations/filter
```
**リクエスト:**
```json
{
  "location": {
    "latitude": 35.6762,
    "longitude": 139.6503,
    "radius": 10000
  },
  "filters": {
    "charging_types": ["quick", "standard"],
    "usage_types": ["hold", "all_day", "no_reservation"],
    "facility_types": ["plugo", "others"],
    "other_conditions": ["show_planned"]
  }
}
```
**レスポンス:**
```json
{
  "total_count": 13746,
  "stations": [
    {
      "id": 1,
      "name": "きたやま関電施設",
      "location": {
        "latitude": 35.6765,
        "longitude": 139.6510,
        "distance": 57
      },
      "charger_types": ["quick"],
      "usage_types": ["hold", "all_day", "no_reservation"],
      "facility_type": "plugo",
      "status": "operational"
    }
  ],
  "applied_filters": {
    "charging_types": 2,
    "usage_types": 3,
    "facility_types": 2,
    "other_conditions": 1
  }
}
```

#### フィルターオプション取得
```
GET /api/v1/stations/filter-options
```
**レスポンス:**
```json
{
  "charging_types": [
    {
      "id": "quick",
      "name": "急速充電",
      "count": 8523
    },
    {
      "id": "standard", 
      "name": "普通充電",
      "count": 12456
    }
  ],
  "usage_types": [
    {
      "id": "hold",
      "name": "取り置き",
      "count": 9876,
      "help_text": "取り置きとは？"
    },
    {
      "id": "all_day",
      "name": "一日予約",
      "count": 7654
    },
    {
      "id": "no_reservation",
      "name": "予約なしでの利用",
      "count": 13746
    }
  ],
  "facility_types": [
    {
      "id": "plugo",
      "name": "プラゴの充電器",
      "count": 5432
    },
    {
      "id": "others",
      "name": "その他の充電器",
      "count": 8314
    }
  ]
}
```

## 4. デザイン仕様

### カラー仕様
- 利用可能状態：グリーン（#4CAF50）
- 利用不可状態：レッド（#F44336）
- 選択状態：オレンジ（#FF6B35）
- プライマリーカラー：ティール（#00BCD4）

### アイコンデザイン
- 充電ステーションアイコン：緑の背景+白い充電シンボル
- 状態インジケーター：点（緑=利用可能、赤=使用中）
- 機能アイコン：線形アイコンスタイル

### 情報階層
1. 施設名と状態（最重要）
2. 距離と住所（二次的）
3. 営業時間（補助情報）

### ユーザーエクスペリエンスの考慮事項
- 地図とリスト表示オプション
- 一般的な条件のクイックフィルター
- リアルタイム充電ステーション状態表示
- ワンクリックナビゲーション機能
- 複数の予約方法をサポート

## 技術的特徴

### 地図統合
- リアルタイム位置追跡
- スムーズなズームとパン
- 複数ステーションのクラスタリング
- カスタムマーカーデザイン

### 検索アルゴリズム
- ファジーマッチングサポート
- 位置ベースの優先順位付け
- 最近の検索履歴
- 予測候補

### フィルターパフォーマンス
- 即時フィルター適用
- 永続的なフィルター選択
- フィルター組み合わせロジック
- 結果数の最適化