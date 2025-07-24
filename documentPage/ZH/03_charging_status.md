# 充电状态管理设计文档

## 概述

充电状态管理模块为用户提供实时的充电状态监控和控制功能，包括剩余时间显示、终止充电选项、充电完成提醒等。

## 充电中画面1：充电进行中

### 页面标题
- "充电可能中..."

### 主要功能
1. **剩余时间显示**
   - 大字体显示剩余时间："剩余时间 0:27"
   - 时钟图标辅助显示
   - 橙色环形进度条表示充电进度

2. **终止充电按钮**
   - 白色按钮："终止使用"
   - 帮助链接："如何终止使用"

3. **使用状况信息**
   - 使用开始时间：19:12
   - 使用结束时间：19:40
   - 使用时长：不足1分钟按1分钟计
   - 其中免费时间：作为准备时间 -1分

### 设计特点
- 简洁的界面设计，重点突出剩余时间
- 环形进度条直观展示充电进度
- 灰色背景降低视觉疲劳

### API接口

#### 获取充电状态
```
GET /api/v1/charging/status/{session_id}
```
**响应:**
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

#### 实时充电数据（WebSocket）
```
WS /api/v1/charging/realtime/{session_id}
```
**消息:**
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

## 充电中画面2：终止确认弹窗

### 弹窗标题
- "如何终止使用？"

### 弹窗内容
1. **操作说明插画**
   - 展示充电器移除步骤
   - 用户操作示意图

2. **文字说明**
   - "拔出充电器后直接离开车位，或者按'终止使用'按钮即可结束使用。"

3. **操作按钮**
   - "OK"确认按钮

### 交互设计
- 半透明遮罩背景
- 清晰的操作指引
- 简单的确认流程

## 充电中画面3：退出提醒弹窗

### 弹窗标题
- "使用结束后，请从EV车位退出。"

### 弹窗内容
1. **注意图标**
   - "CAUTION!"警告标识
   - 橙色强调色

2. **重要提醒**
   - "请在准备完毕后终止使用。"

3. **费用说明**
   - "※结束后如果仍停在EV车位，可能会产生罚金或账号被停用。"

4. **操作按钮**
   - "终止使用"橙色按钮

### 设计要点
- 醒目的警告提示
- 明确的费用说明
- 防止用户误操作

### API接口

#### 终止充电
```
POST /api/v1/charging/terminate
```
**请求:**
```json
{
  "session_id": "sess_abc123",
  "termination_type": "user_initiated",
  "confirm": true
}
```
**响应:**
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

#### 获取终止状态
```
GET /api/v1/charging/termination-status/{session_id}
```
**响应:**
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

## 充电结束画面

### 页面标题
- "感谢您的使用。"

### 主要内容
1. **完成插画**
   - 车辆离开充电站的动画
   - "THANK YOU"感谢语
   - 友好的结束体验

2. **费用汇总**
   - 费用合计：¥500
   - 橙色突出显示金额

3. **使用详情**
   - 充电站：SBPS测试
   - 使用时间：2024/2/15(周四) 19:12～19:13
   - 使用费：¥0
   - 预约费：¥500
   - 单价：¥100/15分
   - 充电服务使用时间：0分（扣除5分钟免费时间）

4. **操作选项**
   - "OK"确认按钮
   - "发行收据"链接

### 用户体验优化
- 友好的结束画面
- 清晰的费用明细
- 便捷的收据发行功能

### API接口

#### 获取充电完成详情
```
GET /api/v1/charging/completion/{session_id}
```
**响应:**
```json
{
  "session_id": "sess_abc123",
  "station": {
    "name": "SBPS测试",
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

## 设计规范

### 颜色使用
- 主色：青绿色（#00BCD4）用于标题和重要信息
- 强调色：橙色（#FF6B35）用于按钮和进度显示
- 背景色：浅灰色（#F5F5F5）
- 文字色：深灰色（#333333）

### 交互原则
1. 实时更新充电状态
2. 重要操作需要二次确认
3. 费用信息透明公开
4. 操作流程简单直观

### 安全考虑
- 终止充电需要确认步骤
- 明确的费用和罚金提示
- 防止误操作的设计

## 技术要求

### 更新频率
- 剩余时间：每1秒
- 进度条：每10秒
- 状态变化：实时

### 状态管理
- 活跃充电状态
- 暂停状态
- 完成状态
- 错误状态

### 通知系统
- 完成前5分钟警告
- 完成通知
- 超时警告

### 其他API接口

#### 发送充电提醒
```
POST /api/v1/charging/notifications
```
**请求:**
```json
{
  "session_id": "sess_abc123",
  "notification_type": "5_minute_warning",
  "delivery_method": ["push", "email"]
}
```
**响应:**
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

#### 获取充电历史
```
GET /api/v1/charging/history
```
**查询参数:**
- `user_id`: "user123"
- `limit`: 10
- `offset`: 0

**响应:**
```json
{
  "total_count": 25,
  "sessions": [
    {
      "session_id": "sess_abc123",
      "date": "2024-02-15",
      "station_name": "SBPS测试",
      "duration_minutes": 1,
      "energy_kwh": 0.1,
      "cost": 500,
      "status": "completed"
    }
  ]
}
```

#### 报告充电问题
```
POST /api/v1/charging/report-issue
```
**请求:**
```json
{
  "session_id": "sess_abc123",
  "issue_type": "unable_to_terminate",
  "description": "终止按钮无响应",
  "urgency": "high"
}
```
**响应:**
```json
{
  "success": true,
  "ticket_id": "ticket_789",
  "support_contact": "+81-3-1234-5678",
  "estimated_response_time": "5 minutes"
}
```