# 取消功能设计文档

## 概述

取消功能为用户提供取消充电位预留的操作，包含清晰的视觉反馈和操作确认流程。

## 页面设计

### 视觉设计
- **背景**：全屏白色背景
- **中心内容**：取消确认信息

### 主要元素

#### 1. 标题文字
- **内容**："已取消预留"
- **颜色**：青绿色（#00BCD4）
- **字号**：20sp
- **位置**：页面上方

#### 2. 状态插画
- **图标元素**：
  - 对话气泡："CANCEL!"
  - 充电桩图标
  - 取消标记（X）
  - 虚线圆圈表示已取消状态
- **配色**：
  - 青绿色对话气泡
  - 灰色虚线
  - 黑色充电桩

#### 3. 提示信息
- **内容**："期待您的再次使用。"
- **字号**：16sp
- **颜色**：深灰色（#333333）
- **位置**：插画下方

#### 4. 操作按钮
- **文字**："返回首页"
- **样式**：
  - 橙色背景（#FF6B35）
  - 白色文字
  - 圆角按钮
  - 宽度：80%屏幕宽度
- **位置**：页面底部

## 交互流程

### 1. 触发场景
- 用户主动取消预留
- 预留时间超时自动取消
- 系统异常导致取消

### 2. 取消流程
1. 用户在预留详情页点击取消
2. 显示取消确认弹窗
3. 用户确认取消操作
4. 显示取消成功页面
5. 点击返回首页按钮回到主界面

### API接口

#### 获取预约详情
```
GET /api/v1/reservations/{reservation_id}
```
**响应:**
```json
{
  "reservation_id": "res_123456",
  "type": "hold",
  "status": "active",
  "station": {
    "id": 1,
    "name": "北山关电设施",
    "address": "东京都品川区北品川5-3-1"
  },
  "start_time": "2024-02-15T19:15:00Z",
  "end_time": "2024-02-15T20:15:00Z",
  "cancellation_policy": {
    "fee": 200,
    "free_cancellation_before": "2024-02-15T19:00:00Z"
  }
}
```

#### 取消预约
```
POST /api/v1/reservations/{reservation_id}/cancel
```
**请求:**
```json
{
  "reservation_id": "res_123456",
  "cancellation_reason": "change_of_plans",
  "confirm": true
}
```
**响应:**
```json
{
  "success": true,
  "reservation_id": "res_123456",
  "status": "cancelled",
  "cancellation_fee": 200,
  "refund_amount": 0,
  "cancelled_at": "2024-02-15T19:30:00Z"
}
```

### 3. 动画效果
- 页面进入：淡入效果
- 插画显示：缩放动画
- 按钮点击：按压效果

## 设计规范

### 布局规范
- **内容垂直居中**对齐
- **元素间距**：
  - 标题与插画：40dp
  - 插画与提示文字：32dp
  - 提示文字与按钮：48dp
- **边距**：左右各16dp

### 插画规范
- **尺寸**：200dp x 200dp
- **风格**：扁平化设计
- **配色**：与品牌色保持一致

### 文案规范
- **语气**：友好、积极
- **措辞**：简洁明了
- **情感**：减少负面情绪

## 用户体验考虑

### 1. 情感化设计
- 友好的插画减轻取消带来的负面感受
- 积极的文案鼓励再次使用
- 明快的色彩避免沉重感

### 2. 操作引导
- 清晰的返回路径
- 单一的操作选项避免困惑
- 醒目的按钮设计

### 3. 信息传达
- 明确告知取消成功
- 友好的再见信息
- 快速返回主流程

## 扩展功能建议

### 1. 取消原因收集
- 可选的取消原因选择
- 帮助改进服务

### 2. 推荐其他时段
- 显示附近可用充电站
- 推荐其他预约时段

### 3. 优惠补偿
- 提供下次使用优惠券
- 积分补偿机制

## 技术实现

### 状态管理
- 取消等待中
- 取消成功
- 取消失败
- 错误处理

### 数据分析
- 追踪取消原因
- 监控取消率
- 用户行为模式

### 性能优化
- 快速页面加载
- 流畅动画效果
- 即时响应

### 其他API接口

#### 获取取消原因选项
```
GET /api/v1/cancellation/reasons
```
**响应:**
```json
{
  "reasons": [
    {
      "id": "change_of_plans",
      "label": "计划有变",
      "category": "user"
    },
    {
      "id": "found_better_option",
      "label": "找到更好的选择",
      "category": "user"
    },
    {
      "id": "emergency",
      "label": "紧急情况",
      "category": "emergency"
    },
    {
      "id": "other",
      "label": "其他原因",
      "category": "other",
      "requires_comment": true
    }
  ]
}
```

#### 提交取消反馈
```
POST /api/v1/cancellation/feedback
```
**请求:**
```json
{
  "reservation_id": "res_123456",
  "reason_id": "change_of_plans",
  "comment": "会议时间改变了",
  "rating": 4
}
```
**响应:**
```json
{
  "success": true,
  "feedback_id": "fb_789",
  "coupon_awarded": {
    "coupon_id": "coup_abc",
    "discount_percentage": 10,
    "valid_until": "2024-03-15"
  }
}
```

#### 获取推荐替代方案
```
GET /api/v1/reservations/alternatives
```
**查询参数:**
- `original_reservation_id`: "res_123456"
- `latitude`: 35.6762
- `longitude`: 139.6503

**响应:**
```json
{
  "alternatives": [
    {
      "station_id": 2,
      "station_name": "南山充电站",
      "distance": 850,
      "available_slots": [
        {
          "start_time": "2024-02-15T20:00:00Z",
          "end_time": "2024-02-15T21:00:00Z",
          "type": "hold"
        }
      ]
    }
  ]
}
```