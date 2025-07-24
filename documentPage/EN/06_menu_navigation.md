# Menu Navigation Design Document

## Overview

Menu navigation provides users with quick access to all functions within the application, using a list-style layout with a combination of icons and text design.

## Menu Structure

### Page Title
- "Menu"
- Back button in top left corner

### User Information Area
- Username display: Kitayama Shiori
- Points display: 0pt

### API Interfaces

#### Get User Profile
```
GET /api/v1/user/profile
```
**Response:**
```json
{
  "user_id": "user123",
  "name": "Kitayama Shiori",
  "points": 0,
  "member_since": "2024-01-15",
  "avatar_url": "https://api.example.com/avatars/user123.jpg"
}
```

### Function Menu List

#### 1. My Page
- **Icon**: Person silhouette icon
- **Function**: Personal information management, account settings
- **Position**: First item

#### 2. Account Information Settings
- **Icon**: Gear icon
- **Function**: Modify account-related settings
- **Position**: Second item

#### 3. Usage History
- **Icon**: Document icon
- **Function**: View historical charging records
- **Position**: Third item

### API Interfaces

#### Get Usage History
```
GET /api/v1/user/usage-history
```
**Query Parameters:**
- `page`: 1
- `limit`: 20
- `start_date`: "2024-01-01"
- `end_date`: "2024-02-15"

**Response:**
```json
{
  "total_records": 45,
  "page": 1,
  "records": [
    {
      "id": "usage_001",
      "date": "2024-02-15",
      "station_name": "Kitayama Kanden Facility",
      "duration_minutes": 45,
      "energy_kwh": 25.5,
      "cost": 1530,
      "payment_method": "credit_card"
    }
  ]
}
```

#### 4. Coupons
- **Icon**: Tag icon
- **Function**: View and use coupons
- **Position**: Fourth item

#### Get Coupons List
```
GET /api/v1/user/coupons
```
**Response:**
```json
{
  "coupons": [
    {
      "id": "coupon_001",
      "title": "New User Discount",
      "discount_type": "percentage",
      "discount_value": 20,
      "min_amount": 1000,
      "valid_until": "2024-03-31",
      "status": "active"
    }
  ]
}
```

#### 5. Payment Methods
- **Icon**: Credit card icon
- **Function**: Manage payment methods
- **Position**: Fifth item

#### Get Payment Methods
```
GET /api/v1/user/payment-methods
```
**Response:**
```json
{
  "payment_methods": [
    {
      "id": "pm_001",
      "type": "credit_card",
      "brand": "VISA",
      "last4": "1234",
      "exp_month": 12,
      "exp_year": 2025,
      "is_default": true
    }
  ]
}
```

#### 6. Register Facility Code
- **Icon**: Gear icon
- **Function**: Add specific facility codes
- **Position**: Sixth item

#### 7. First-time Guide
- **Icon**: Shield icon
- **Function**: Application usage tutorial
- **Position**: Seventh item

#### 8. FAQ
- **Icon**: Tag icon
- **Function**: Frequently asked questions
- **Position**: Eighth item

#### 9. Contact Us
- **Icon**: Gear icon
- **Function**: Customer support
- **Position**: Ninth item

## Design Specifications

### Visual Design
- **Background Color**: White (#FFFFFF)
- **Divider Lines**: Light Gray (#E0E0E0)
- **Text Colors**:
  - Title: Dark Gray (#333333)
  - Menu Items: Dark Gray (#333333)
  - Username: Black (#000000)

### Icon Specifications
- **Style**: Linear icons
- **Color**: Dark Gray (#666666)
- **Size**: 24dp x 24dp
- **Alignment**: Left-aligned with consistent spacing from text

### Layout Specifications
- **Menu Item Height**: 56dp
- **Left/Right Margins**: 16dp
- **Icon-Text Spacing**: 16dp
- **Divider Lines**: Between each menu item

### Interaction Design
- **Click Feedback**: Ripple effect on tap
- **Navigation Animation**: Slide in from right, slide back to left
- **Arrow Indicator**: ">" arrow on right side of each menu item

## User Experience Optimization

### Information Architecture
1. Frequently used functions at top (My Page, Account Settings)
2. Related functions grouped (Payment-related, Help-related)
3. Clear function naming

### Accessibility
- Sufficient tap area (minimum 44dp)
- Clear visual hierarchy
- Icons to aid understanding

### Navigation Efficiency
- Direct access from main menu
- Prominent back button
- Clear function categorization

## Technical Implementation

### Menu Item Structure
```
- Icon (24dp) | Text Label | Arrow (>)
- Divider Line
```

### Touch Target
- Minimum height: 48dp
- Full width clickable area
- Visual feedback on touch

### Performance
- Lazy loading for menu content
- Smooth scrolling for long lists
- Instant navigation response

### Additional API Interfaces

#### Get Menu Configuration
```
GET /api/v1/menu/config
```
**Response:**
```json
{
  "menu_items": [
    {
      "id": "my_page",
      "title": "My Page",
      "icon": "person",
      "route": "/my-page",
      "enabled": true
    },
    {
      "id": "account_settings",
      "title": "Account Information Settings",
      "icon": "settings",
      "route": "/account-settings",
      "enabled": true
    }
  ],
  "version": "1.0.0"
}
```

#### Register Facility Code
```
POST /api/v1/facility/register-code
```
**Request:**
```json
{
  "facility_code": "FAC123456",
  "user_id": "user123"
}
```
**Response:**
```json
{
  "success": true,
  "facility": {
    "code": "FAC123456",
    "name": "Special Charging Facility",
    "benefits": ["Priority booking", "Discount offers"]
  }
}
```