# 充电签到流程设计文档

## 概述

充电签到是用户使用充电服务的第一步。整个流程分为4个步骤，通过顶部的进度指示器展示当前所处阶段。

## 步骤1：车位选择

### 页面标题
- "充电前准备"

### 进度指示
- 4个圆点，第1个高亮显示
- 步骤标签：车室选择

### 主要内容
- **提示文字**："请选择编号。"
- **充电桩信息展示**：
  - 编号：1
  - 充电规格：普通 6/3kW ¥~100/15分
  - 每日最大：¥100,000
  - 线缆可用性：有线缆
- **视觉元素**：充电桩图标展示

### 交互细节
- 用户需要选择充电桩编号
- 显示充电桩的详细规格信息
- 底部帮助链接："不知道停车位置"

### 设计特点
- 清晰的编号选择
- 关键信息突出显示
- 充电设备的视觉呈现

### API接口

#### 获取可用充电站
```
GET /api/v1/stations/available
```
**响应:**
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

#### 选择充电站
```
POST /api/v1/checkin/select-station
```
**请求:**
```json
{
  "station_id": 1,
  "user_id": "user123"
}
```
**响应:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "next_step": "power_selection"
}
```

## 步骤2：功率选择

### 页面标题
- "充电前准备"

### 进度指示
- 4个圆点，第2个高亮显示
- 步骤标签：输出选择

### 主要内容
- **提示文字**："请选择输出功率。"
- **功率选项**：
  1. **6kW**
     - 价格：¥100/15分
     - 每日最大：¥100,000
  2. **3kW**
     - 价格：¥50/15分
     - 每日最大：¥50,000

### 交互细节
- 用户必须选择充电功率
- 不同功率对应不同的收费标准
- 选中的选项显示视觉反馈

### 设计特点
- 清晰的价格对比
- 单选按钮选择
- 高亮的选中状态

### API接口

#### 选择输出功率
```
POST /api/v1/checkin/select-power
```
**请求:**
```json
{
  "session_id": "sess_abc123",
  "power_option": "6kW",
  "price_per_15min": 100
}
```
**响应:**
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

## 步骤3：内容确认

### 页面标题
- "充电前准备"

### 进度指示
- 4个圆点，第3个高亮显示
- 步骤标签：内容确认

### 主要内容
- **提示文字**："即将开始使用SBPS测试。"
- **确认信息**：
  - 测试名称：SBPS测试
  - 地址：东京都品川区大崎2丁目1-1
  - Payment标识
- **充电器信息**：
  - 普通 6kW（有线缆）
  - 价格：¥100/15分（每日最大 ¥100,000）
- **费用详情**：
  - 使用费：¥100/15分
  - 预计总费用：¥334
- **重要注意事项**：
  - 实际使用时间计费说明
  - 准备时间免费说明
  - 停车费另计通知
  - 超时费用说明

### 设计特点
- 全面的信息展示
- 清晰的费用明细
- 重要事项突出显示

### API接口

#### 获取充电摘要
```
GET /api/v1/checkin/summary/{session_id}
```
**响应:**
```json
{
  "session_id": "sess_abc123",
  "station": {
    "name": "SBPS测试",
    "address": "东京都品川区大崎2丁目1-1",
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

#### 确认充电
```
POST /api/v1/checkin/confirm
```
**请求:**
```json
{
  "session_id": "sess_abc123",
  "user_confirmation": true,
  "payment_method": "default"
}
```
**响应:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "charging_id": "chrg_xyz789",
  "next_step": "parking_guidance"
}
```

## 步骤4：入库指引

### 页面标题
- "充电前准备"

### 进度指示
- 4个圆点，第4个高亮显示
- 步骤标签：入库

### 主要内容
- **停车指引插画**：
  - 充电站布局展示
  - "PARKING"位置标记
  - 车辆停放示意图
- **安全提示**：
  - "智能栏杆完全降下后，请确认周围安全后再停车。"
- **确认按钮**："OK"

### 交互细节
- 最后的安全提醒
- 用户确认后开始充电

### 设计特点
- 可视化停车引导
- 安全第一的方法
- 清晰的行动召唤

### API接口

#### 开始充电会话
```
POST /api/v1/charging/start
```
**请求:**
```json
{
  "charging_id": "chrg_xyz789",
  "station_id": 1,
  "user_confirmation": "parked"
}
```
**响应:**
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

#### 更新栏杆状态（WebSocket）
```
WS /api/v1/charging/barrier-status
```
**消息:**
```json
{
  "charging_id": "chrg_xyz789",
  "barrier_status": "lowered",
  "safe_to_proceed": true
}
```

## 设计规范

### 视觉设计
- 清晰的步骤进度指示
- 重要信息突出显示
- 插画辅助理解
- 统一的色彩系统

### 用户体验
- 步骤清晰，循序渐进
- 重要信息多次确认
- 费用透明，避免误解
- 全面的安全提示

### 交互设计
- 每个步骤都需要用户主动确认
- 支持返回功能修改选择
- 帮助信息随时可访问

## 色彩方案
- 主色：青绿色（#00BCD4）
- 强调色：橙色（#FF6B35）
- 背景色：浅灰色（#F5F5F5）
- 文字色：深灰色（#333333）

## 字体规范
- 标题：20sp，粗体
- 正文：16sp，常规
- 说明文字：14sp，常规
- 按钮文字：16sp，中粗体