# 预约功能设计文档

## 概述

预约功能分为三种类型：充电位预留、时间段预约、全天预约。每种预约方式针对不同的用户需求场景。

## 1. 充电位预留功能

### 1.1 功能说明页面

#### 页面内容
- **标题**："什么是预留？"
- **功能图解**：
  - "WAITING..."等待状态动画
  - 虚线框表示的预留车位
  - 充电桩图标
- **文字说明**：
  - "最多可预留空闲车位60分钟。"
  - "当您想'现在就去那里！'时，可以确保车位。"
- **费用提醒**：
  - "※取消预留会产生取消费用。"
- **操作选项**：
  - 复选框："下次不再显示"
  - "OK"确认按钮

### API接口

#### 创建预留
```
POST /api/v1/reservations/hold
```
**请求:**
```json
{
  "station_id": 1,
  "user_id": "user123",
  "duration_minutes": 60
}
```
**响应:**
```json
{
  "success": true,
  "hold_id": "hold_abc123",
  "station": {
    "id": 1,
    "name": "北山关电设施",
    "address": "东京都品川区北品川5-3-1"
  },
  "hold_start": "2024-02-15T19:15:00Z",
  "hold_end": "2024-02-15T20:15:00Z",
  "cancellation_fee": 200
}
```

### 1.2 预留状态页面

#### 页面布局
- **倒计时显示**：
  - 大字体橙色时间："预留至20:15"
  - 橙色进度条显示剩余时间
- **预留状态插画**：
  - "WAITING..."文字
  - 预留车位示意图
- **操作按钮**：
  - "已到达"白色按钮
  - "入场指南"帮助链接
- **预留信息**：
  - 预留状况标题
  - 设施图片占位
  - 设施名称：北山关电设施
  - 地址：东京都品川区北品川5-3-1
  - 营业时间：8:00～23:00
  - 预留开始时间：19:15
  - 预留结束时间：20:15

### API接口

#### 获取预留状态
```
GET /api/v1/reservations/hold/{hold_id}
```
**响应:**
```json
{
  "hold_id": "hold_abc123",
  "status": "active",
  "remaining_minutes": 27,
  "station": {
    "id": 1,
    "name": "北山关电设施",
    "address": "东京都品川区北品川5-3-1",
    "business_hours": "8:00-23:00"
  },
  "hold_start": "2024-02-15T19:15:00Z",
  "hold_end": "2024-02-15T20:15:00Z"
}
```

#### 确认到达
```
POST /api/v1/reservations/hold/{hold_id}/arrive
```
**请求:**
```json
{
  "hold_id": "hold_abc123",
  "arrival_time": "2024-02-15T19:45:00Z"
}
```
**响应:**
```json
{
  "success": true,
  "hold_id": "hold_abc123",
  "charging_session_id": "chrg_xyz789",
  "next_step": "start_charging"
}
```

### 1.3 入场指引弹窗

#### 弹窗内容
- **标题**："入场指南"
- **安全提醒图解**：
  - 显示智能栏杆位置
  - 车辆停放示意
- **文字说明**：
  - "请在现场黄色智能栏杆前停车。"
  - "操作应用时，请停好车并充分确认周围安全。"
- **操作按钮**："OK"

### 1.4 取消确认页面

#### 页面内容
- **取消政策说明**：
  - 标题："取消政策"
  - 详细的取消费用说明：
    - (1)快速充电器：500日元
    - (2)普通充电器：200日元
- **取消确认弹窗**：
  - "要取消预留吗？"
  - "取消将产生取消费用。"
  - "取消预留"橙色按钮

### API接口

#### 取消预留
```
POST /api/v1/reservations/hold/{hold_id}/cancel
```
**请求:**
```json
{
  "hold_id": "hold_abc123",
  "reason": "user_cancelled"
}
```
**响应:**
```json
{
  "success": true,
  "hold_id": "hold_abc123",
  "cancellation_fee": 200,
  "fee_charged": true,
  "refund_amount": 0
}
```

## 2. 时间段预约功能

### 页面设计
- **设施信息**：
  - 标题：SBPS测试
  - 充电规格选择（已选择6kW）
  - 充电费用显示
- **日期选择**：
  - 月历视图（2月）
  - 周日历快速选择
  - 已选日期高亮显示（17日，周六）
- **时间选择**：
  - 时间范围：19:10 ~ 19:40（30分钟）
  - 24小时时间轴
  - 可拖动选择时间段
  - 绿色表示已选时间段
- **操作按钮**：
  - "前往确认页面"

### 交互特点
- 直观的时间轴拖动选择
- 实时显示选择的时间长度
- 日期和时间联动选择

### API接口

#### 获取可用时间段
```
GET /api/v1/reservations/timeslots/available
```
**查询参数:**
- `station_id`: 1
- `date`: "2024-02-17"
- `power_option`: "6kW"

**响应:**
```json
{
  "station_id": 1,
  "date": "2024-02-17",
  "available_slots": [
    {
      "start_time": "19:00",
      "end_time": "19:30",
      "available": true
    },
    {
      "start_time": "19:30",
      "end_time": "20:00",
      "available": true
    }
  ]
}
```

#### 创建时间段预约
```
POST /api/v1/reservations/timeslot
```
**请求:**
```json
{
  "station_id": 1,
  "date": "2024-02-17",
  "start_time": "19:10",
  "end_time": "19:40",
  "power_option": "6kW",
  "user_id": "user123"
}
```
**响应:**
```json
{
  "success": true,
  "reservation_id": "res_time123",
  "station_id": 1,
  "reservation_details": {
    "date": "2024-02-17",
    "start_time": "19:10",
    "end_time": "19:40",
    "duration_minutes": 30,
    "power": "6kW",
    "estimated_cost": 200
  }
}
```

## 3. 全天预约功能

### 3.1 预约选择页面

#### 页面布局
- **设施信息**（同上）
- **预约类型切换**：
  - "立即"
  - "预留"
  - "全天预约" - 已选中
- **预约日期**：
  - 预约日：2024/2/15
- **充电规格**：
  - 快速 100kW ¥50/分
  - 最大60分｜有线缆
- **操作选项**：
  - "什么是全天预约？"帮助链接
  - "前往确认页面"

### API接口

#### 创建全天预约
```
POST /api/v1/reservations/all-day
```
**请求:**
```json
{
  "station_id": 1,
  "date": "2024-02-15",
  "power_option": "100kW",
  "user_id": "user123"
}
```
**响应:**
```json
{
  "success": true,
  "reservation_id": "res_day456",
  "station_id": 1,
  "reservation_details": {
    "date": "2024-02-15",
    "type": "all_day",
    "power": "100kW",
    "max_duration_minutes": 60,
    "daily_fee": 5000,
    "cancellation_policy": {
      "same_day_fee": 5000,
      "advance_fee": 1000
    }
  }
}
```

#### 检查全天预约可用性
```
GET /api/v1/reservations/all-day/availability
```
**查询参数:**
- `station_id`: 1
- `start_date`: "2024-02-15"
- `end_date`: "2024-05-15"

**响应:**
```json
{
  "station_id": 1,
  "available_dates": [
    {
      "date": "2024-02-15",
      "available": true,
      "quick_chargers": 1,
      "standard_chargers": 3
    },
    {
      "date": "2024-02-16",
      "available": false,
      "reason": "fully_booked"
    }
  ]
}
```

### 3.2 全天预约说明弹窗

#### 弹窗内容
- **标题**："什么是全天预约？"
- **注意图标**："CAUTION!"
- **功能说明**：
  - "可以指定3个月内的日期确保车位。"
  - "适合长时间停留的出行。"
- **费用提醒**：
  - "※使用全天预约或当天取消需要付费。"
- **操作选项**：
  - 复选框："下次不再显示"
  - "OK"按钮

## 设计规范

### 视觉层级
1. 时间信息最优先（大字体、醒目颜色）
2. 操作按钮次优先（对比色、适中尺寸）
3. 详细信息最后（常规字体、灰色）

### 交互原则
1. 预约前充分说明规则
2. 取消操作需要确认
3. 费用信息透明展示
4. 操作流程简单直观

### 用户体验优化
- 多种预约方式满足不同需求
- 视觉化的时间选择
- 清晰的费用和规则说明
- 便捷的取消和修改功能

## 色彩方案
- 主色：青绿色（#00BCD4）
- 强调色：橙色（#FF6B35）
- 成功色：绿色（#4CAF50）
- 背景色：浅灰色（#F5F5F5）

## 字体规范
- 标题：20sp 粗体
- 正文：16sp 常规
- 说明文字：14sp 常规
- 时间显示：24sp 粗体