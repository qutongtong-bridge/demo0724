# Receipt Issuance Function Design Document

## Overview

The receipt issuance function allows users to enter receipt header information and generate electronic receipts after charging service ends, facilitating reimbursement and record management.

## Page Design

### Page Title
- "Receipt Issuance"
- Back button in top left corner

### Main Content Area

#### 1. Function Description
- **Instruction Text**:
  - "Please enter the recipient name for the receipt and press the 'Issue' button."

#### 2. Input Area
- **Input Field Label**: "Recipient"
- **Input Field**:
  - Placeholder text display
  - Supports text input
  - White background
  - Rounded border design

#### 3. Action Button
- **Issue Button**:
  - Text: "Issue without recipient name"
  - Orange background (#FF6B35)
  - White text
  - Rounded button design
  - Located below input field

### Keyboard Interaction
- Keyboard automatically appears when tapping input field
- Supports Japanese input method
- Keyboard type: Text keyboard
- Completion button: "Done"

## Interaction Flow

### 1. Basic Flow
1. User clicks "Issue Receipt" link from charging completion page
2. Enters receipt issuance page
3. User can choose:
   - Enter receipt recipient name
   - Directly click "Issue without recipient name"
4. System generates electronic receipt
5. Provides download or share options

### API Interfaces

#### Get Charging Session Details
```
GET /api/v1/charging/session/{session_id}
```
**Response:**
```json
{
  "session_id": "sess_abc123",
  "charging_id": "chrg_xyz789",
  "station": {
    "name": "Kitayama Kanden Facility",
    "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1"
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

### 2. Input States
- **Empty State**: Shows placeholder prompt
- **Inputting**: Real-time display of input content
- **Input Complete**: Button text changes to "Issue"

### 3. Error Handling
- Special character restriction notice
- Character limit notice (if applicable)
- Network error retry mechanism

#### Generate Receipt
```
POST /api/v1/receipts/generate
```
**Request:**
```json
{
  "session_id": "sess_abc123",
  "recipient": "ABC Corporation",
  "language": "en",
  "format": "pdf"
}
```
**Response:**
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

#### Get Receipt History
```
GET /api/v1/user/receipts
```
**Query Parameters:**
- `page`: 1
- `limit`: 20

**Response:**
```json
{
  "total_count": 15,
  "receipts": [
    {
      "receipt_id": "rcpt_123456",
      "issue_date": "2024-02-15",
      "recipient": "ABC Corporation",
      "amount": 1540,
      "station_name": "Kitayama Kanden Facility",
      "download_url": "https://api.example.com/receipts/rcpt_123456/download"
    }
  ]
}
```

## Design Specifications

### Visual Design
- **Background Color**: Light Gray (#F5F5F5)
- **Input Field**:
  - Background: White (#FFFFFF)
  - Border: Light Gray (#E0E0E0)
  - Border Radius: 8px
- **Button**:
  - Background: Orange (#FF6B35)
  - Text: White (#FFFFFF)
  - Border Radius: 24px

### Layout Specifications
- **Content Margin**: 16dp
- **Element Spacing**:
  - Instruction text to input field: 24dp
  - Input field to button: 32dp
- **Input Field Height**: 48dp
- **Button Height**: 48dp

### Typography
- **Instruction Text**:
  - Font Size: 14sp
  - Color: Dark Gray (#666666)
- **Input Field Label**:
  - Font Size: 12sp
  - Color: Medium Gray (#999999)
- **Button Text**:
  - Font Size: 16sp
  - Font Weight: Medium

## Feature Extension Considerations

### 1. Receipt Templates
- Preset common recipient names
- History of recipient names
- Quick selection function

### 2. Receipt Format
- PDF format export
- Email sending function
- Print-friendly layout

### 3. Information Completeness
- Charging details
- Time information
- Amount breakdown
- Tax amount display

## User Experience Optimization

### Convenience
- One-click issuance option
- Quick fill from history
- Simplified operation flow

### Reliability
- Issuance success confirmation
- Re-issuance function
- Receipt viewing and management

### Compliance
- Meets financial reimbursement requirements
- Includes necessary legal information
- Supports corporate reimbursement process

## Technical Implementation

### Data Structure
```json
{
  "recipient": "string",
  "amount": "number",
  "date": "datetime",
  "chargingDetails": {
    "duration": "minutes",
    "power": "kW",
    "location": "string"
  }
}
```

### Receipt Generation
- Server-side PDF generation
- Secure storage
- Unique receipt ID
- QR code for verification

### Additional API Interfaces

#### Get Frequent Recipients
```
GET /api/v1/user/receipt-recipients
```
**Response:**
```json
{
  "recipients": [
    {
      "id": "rec_001",
      "name": "ABC Corporation",
      "usage_count": 5,
      "last_used": "2024-02-10"
    },
    {
      "id": "rec_002",
      "name": "XYZ Limited",
      "usage_count": 3,
      "last_used": "2024-01-20"
    }
  ]
}
```

#### Save Frequent Recipient
```
POST /api/v1/user/receipt-recipients
```
**Request:**
```json
{
  "recipient_name": "New Company Name"
}
```
**Response:**
```json
{
  "id": "rec_003",
  "name": "New Company Name",
  "created_at": "2024-02-15T19:57:00Z"
}
```

#### Reissue Receipt
```
POST /api/v1/receipts/{receipt_id}/reissue
```
**Request:**
```json
{
  "recipient": "Updated Company Name"
}
```
**Response:**
```json
{
  "receipt_id": "rcpt_123457",
  "original_receipt_id": "rcpt_123456",
  "status": "generated",
  "download_url": "https://api.example.com/receipts/rcpt_123457/download"
}
```