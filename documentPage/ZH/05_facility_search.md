# 设施查找与筛选设计文档

## 概述

设施查找与筛选功能帮助用户快速找到符合需求的充电站，包括地图展示、搜索功能、多维度筛选等。

## 1. 地图主界面

### 页面布局
- **地图展示区**：
  - 全屏地图背景
  - 用户当前位置标记（蓝点）
  - 充电站位置标记（绿色图标）
  - 街道名称标注

- **设施信息卡片**：
  - 位于底部，可上下滑动
  - 显示选中的充电站信息

- **操作按钮**：
  - 右下角：菜单按钮（三条横线）
  - 右下角：定位按钮（箭头图标）

### 设施信息展示
- **标题**：北山关电设施
- **可用状态**：空（快速:1/1）- 绿色圆点表示
- **距离**：距当前位置 57m
- **营业状态**：营业中｜8:00～23:00
- **预约选项**：
  - 立即（即时）- 插头图标
  - 预留 - 1h时钟图标
  - 全天预约 - 日历图标

### API接口

#### 获取附近充电站
```
GET /api/v1/stations/nearby
```
**查询参数:**
- `latitude`: 35.6762
- `longitude`: 139.6503
- `radius`: 5000 (米)

**响应:**
```json
{
  "stations": [
    {
      "id": 1,
      "name": "北山关电设施",
      "location": {
        "latitude": 35.6765,
        "longitude": 139.6510,
        "address": "东京都品川区北品川5-3-1"
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

#### 获取充电站详情
```
GET /api/v1/stations/{station_id}
```
**响应:**
```json
{
  "id": 1,
  "name": "北山关电设施",
  "location": {
    "latitude": 35.6765,
    "longitude": 139.6510,
    "address": "东京都品川区北品川5-3-1"
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

## 2. 搜索界面

### 搜索框设计
- **输入提示**："您要去哪里？"
- **搜索建议列表**：
  - 北山关电设施
    - 快速:1/1（绿色状态）
    - 57m｜东京都品川区北品川5-3-1
    - 营业中｜8:00～23:00
  - SBPS测试
    - 普通:0/1（红色状态）
    - 510m｜东京都品川区大崎2丁目1-1
    - 营业中｜6:00～22:00

### 键盘交互
- 支持日文输入
- 实时搜索建议
- 快捷输入选项

### API接口

#### 搜索充电站
```
POST /api/v1/stations/search
```
**请求:**
```json
{
  "query": "北山",
  "location": {
    "latitude": 35.6762,
    "longitude": 139.6503
  },
  "max_results": 10
}
```
**响应:**
```json
{
  "results": [
    {
      "id": 1,
      "name": "北山关电设施",
      "match_score": 0.95,
      "location": {
        "address": "东京都品川区北品川5-3-1",
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
      "name": "SBPS测试",
      "match_score": 0.75,
      "location": {
        "address": "东京都品川区大崎2丁目1-1",
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

## 3. 筛选功能

### 筛选维度

#### 3.1 充电类型
- **选项**（多选）：
  - ✓ 快速充电
  - ✓ 普通充电

#### 3.2 使用类型
- **选项**（多选）：
  - ✓ 预留
  - ✓ 全天预约
  - ✓ 无需预约使用
- **帮助链接**："什么是预留？"

#### 3.3 设施种类
- **选项**（多选）：
  - ✓ 显示Plugo充电器
  - ✓ 显示其他充电器

#### 3.4 其他条件
- **选项**（多选）：
  - ✓ 显示预定安装的充电器

### 筛选操作
- **重置功能**："显示全部" - 橙色文字
- **应用筛选**："13746个设施" - 蓝色按钮
- **关闭按钮**：右上角 X

### 交互设计
- 复选框支持多选
- 实时更新筛选结果数量
- 筛选条件可保存
- 一键重置所有筛选

### API接口

#### 应用筛选条件
```
POST /api/v1/stations/filter
```
**请求:**
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
**响应:**
```json
{
  "total_count": 13746,
  "stations": [
    {
      "id": 1,
      "name": "北山关电设施",
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

#### 获取筛选选项
```
GET /api/v1/stations/filter-options
```
**响应:**
```json
{
  "charging_types": [
    {
      "id": "quick",
      "name": "快速充电",
      "count": 8523
    },
    {
      "id": "standard", 
      "name": "普通充电",
      "count": 12456
    }
  ],
  "usage_types": [
    {
      "id": "hold",
      "name": "预留",
      "count": 9876,
      "help_text": "什么是预留？"
    },
    {
      "id": "all_day",
      "name": "全天预约",
      "count": 7654
    },
    {
      "id": "no_reservation",
      "name": "无需预约使用",
      "count": 13746
    }
  ],
  "facility_types": [
    {
      "id": "plugo",
      "name": "Plugo充电器",
      "count": 5432
    },
    {
      "id": "others",
      "name": "其他充电器",
      "count": 8314
    }
  ]
}
```

## 4. 设计规范

### 颜色规范
- 可用状态：绿色（#4CAF50）
- 不可用状态：红色（#F44336）
- 选中状态：橙色（#FF6B35）
- 主色调：青绿色（#00BCD4）

### 图标设计
- 充电站图标：绿色背景+白色充电符号
- 状态指示：圆点（绿色=可用，红色=占用）
- 功能图标：线性图标风格

### 信息层级
1. 设施名称和状态（最重要）
2. 距离和地址（次要）
3. 营业时间（辅助信息）

### 用户体验考虑
- 地图和列表两种查看方式
- 快速筛选常用条件
- 实时显示充电站状态
- 一键导航功能
- 支持多种预约方式

## 技术特性

### 地图集成
- 实时位置跟踪
- 平滑缩放和平移
- 多站点聚合显示
- 自定义标记设计

### 搜索算法
- 模糊匹配支持
- 基于位置的优先级
- 最近搜索历史
- 预测建议

### 筛选性能
- 即时筛选应用
- 持久化筛选选择
- 筛选组合逻辑
- 结果数量优化