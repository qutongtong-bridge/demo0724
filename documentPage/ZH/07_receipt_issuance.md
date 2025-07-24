# 收据发行功能设计文档

## 概述

收据发行功能允许用户在充电服务结束后，输入收据抬头信息并生成电子收据，方便报销和记录管理。

## 页面设计

### 页面标题
- "收据发行"
- 左上角返回按钮

### 主要内容区

#### 1. 功能说明
- **说明文字**：
  - "请输入收据的抬头名称，然后点击'发行'按钮。"

#### 2. 输入区域
- **输入框标签**："抬头"
- **输入框**：
  - 占位符文字显示
  - 支持文本输入
  - 白色背景
  - 圆角边框设计

#### 3. 操作按钮
- **发行按钮**：
  - 文字："不填写抬头发行"
  - 橙色背景（#FF6B35）
  - 白色文字
  - 圆角按钮设计
  - 位于输入框下方

### 键盘交互
- 点击输入框自动弹出键盘
- 支持日文输入法
- 键盘类型：文本键盘
- 完成按钮："完成"

## 交互流程

### 1. 基本流程
1. 用户从充电完成页面点击"发行收据"链接
2. 进入收据发行页面
3. 用户可选择：
   - 输入收据抬头
   - 直接点击"不填写抬头发行"
4. 系统生成电子收据
5. 提供下载或分享选项

### API接口

#### 获取充电会话详情
```
GET /api/v1/charging/session/{session_id}
```
**响应:**
```json
{
  "session_id": "sess_abc123",
  "charging_id": "chrg_xyz789",
  "station": {
    "name": "北山关电设施",
    "address": "东京都品川区北品川5-3-1"
  },
  "start_time": "2024-02-15T19:12:00Z",
  "end_time": "2024-02-15T19:57:00Z",
  "duration_minutes": 45,
  "energy_kwh": 25.5,
  "amount": {
    "subtotal": 1400,
    "tax": 140,
    "total": 1540
  },
  "payment_method": "credit_card"
}
```

### 2. 输入状态
- **空状态**：显示占位符提示
- **输入中**：实时显示输入内容
- **输入完成**：按钮文字变为"发行"

### 3. 错误处理
- 特殊字符限制提示
- 字数限制提示（如有）
- 网络错误重试机制

#### 生成收据
```
POST /api/v1/receipts/generate
```
**请求:**
```json
{
  "session_id": "sess_abc123",
  "recipient": "ABC株式会社",
  "language": "zh",
  "format": "pdf"
}
```
**响应:**
```json
{
  "receipt_id": "rcpt_123456",
  "status": "generated",
  "download_url": "https://api.example.com/receipts/rcpt_123456/download",
  "expires_at": "2024-02-16T19:57:00Z",
  "metadata": {
    "receipt_number": "R20240215001",
    "issue_date": "2024-02-15",
    "verification_code": "ABCD1234"
  }
}
```

#### 获取收据历史
```
GET /api/v1/user/receipts
```
**查询参数:**
- `page`: 1
- `limit`: 20

**响应:**
```json
{
  "total_count": 15,
  "receipts": [
    {
      "receipt_id": "rcpt_123456",
      "issue_date": "2024-02-15",
      "recipient": "ABC株式会社",
      "amount": 1540,
      "station_name": "北山关电设施",
      "download_url": "https://api.example.com/receipts/rcpt_123456/download"
    }
  ]
}
```

## 设计规范

### 视觉设计
- **背景色**：浅灰色（#F5F5F5）
- **输入框**：
  - 背景：白色（#FFFFFF）
  - 边框：浅灰色（#E0E0E0）
  - 圆角：8px
- **按钮**：
  - 背景：橙色（#FF6B35）
  - 文字：白色（#FFFFFF）
  - 圆角：24px

### 布局规范
- **内容边距**：16dp
- **元素间距**：
  - 说明文字与输入框：24dp
  - 输入框与按钮：32dp
- **输入框高度**：48dp
- **按钮高度**：48dp

### 文字规范
- **说明文字**：
  - 字号：14sp
  - 颜色：深灰色（#666666）
- **输入框标签**：
  - 字号：12sp
  - 颜色：中灰色（#999999）
- **按钮文字**：
  - 字号：16sp
  - 字重：中等

## 功能扩展考虑

### 1. 收据模板
- 预设常用抬头
- 历史抬头记录
- 快速选择功能

### 2. 收据格式
- PDF格式导出
- 邮件发送功能
- 打印友好版式

### 3. 信息完整性
- 充电详情
- 时间信息
- 金额明细
- 税额显示

## 用户体验优化

### 便捷性
- 一键发行选项
- 历史记录快速填充
- 简化的操作流程

### 可靠性
- 发行成功确认
- 重新发行功能
- 收据查看和管理

### 合规性
- 符合财务报销要求
- 包含必要的法定信息
- 支持企业报销流程

## 技术实现

### 数据结构
```json
{
  "recipient": "字符串",
  "amount": "数字",
  "date": "日期时间",
  "chargingDetails": {
    "duration": "分钟",
    "power": "千瓦",
    "location": "字符串"
  }
}
```

### 收据生成
- 服务端PDF生成
- 安全存储
- 唯一收据ID
- 验证二维码

### 其他API接口

#### 获取常用抬头
```
GET /api/v1/user/receipt-recipients
```
**响应:**
```json
{
  "recipients": [
    {
      "id": "rec_001",
      "name": "ABC株式会社",
      "usage_count": 5,
      "last_used": "2024-02-10"
    },
    {
      "id": "rec_002",
      "name": "XYZ有限公司",
      "usage_count": 3,
      "last_used": "2024-01-20"
    }
  ]
}
```

#### 保存常用抬头
```
POST /api/v1/user/receipt-recipients
```
**请求:**
```json
{
  "recipient_name": "新公司名称"
}
```
**响应:**
```json
{
  "id": "rec_003",
  "name": "新公司名称",
  "created_at": "2024-02-15T19:57:00Z"
}
```

#### 重新发行收据
```
POST /api/v1/receipts/{receipt_id}/reissue
```
**请求:**
```json
{
  "recipient": "更新后的公司名称"
}
```
**响应:**
```json
{
  "receipt_id": "rcpt_123457",
  "original_receipt_id": "rcpt_123456",
  "status": "generated",
  "download_url": "https://api.example.com/receipts/rcpt_123457/download"
}
```