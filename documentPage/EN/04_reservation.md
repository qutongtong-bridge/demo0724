# Reservation Features Design Document

## Overview

The reservation feature offers three types: parking space hold, time slot reservation, and all-day reservation. Each reservation method caters to different user scenarios.

## 1. Parking Space Hold Feature

### 1.1 Feature Explanation Page

#### Page Content
- **Title**: "What is Hold?"
- **Feature Illustration**:
  - "WAITING..." status animation
  - Dotted line frame indicating reserved space
  - Charging station icon
- **Text Explanation**:
  - "You can hold an available parking space for up to 60 minutes."
  - "When you want to 'go there right now!', you can securely reserve a parking space."
- **Fee Reminder**:
  - "※Cancellation of hold will incur cancellation fees."
- **Action Options**:
  - Checkbox: "Don't show next time"
  - "OK" confirmation button

### API Interfaces

#### Create Hold Reservation
```
POST /api/v1/reservations/hold
```
**Request:**
```json
{
  "station_id": 1,
  "user_id": "user123",
  "duration_minutes": 60
}
```
**Response:**
```json
{
  "success": true,
  "hold_id": "hold_abc123",
  "station": {
    "id": 1,
    "name": "Kitayama Kanden Facility",
    "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1"
  },
  "hold_start": "2024-02-15T19:15:00Z",
  "hold_end": "2024-02-15T20:15:00Z",
  "cancellation_fee": 200
}
```

### 1.2 Hold Status Page

#### Page Layout
- **Countdown Display**:
  - Large orange text: "Hold until 20:15"
  - Orange progress bar showing remaining time
- **Hold Status Illustration**:
  - "WAITING..." text
  - Reserved parking space diagram
- **Action Buttons**:
  - "Arrived" white button
  - "Parking Guide" help link
- **Hold Information**:
  - Hold Status title
  - Facility image placeholder
  - Facility Name: Kitayama Kanden Facility
  - Address: Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1
  - Business Hours: 8:00～23:00
  - Hold Start Time: 19:15
  - Hold End Time: 20:15

### API Interfaces

#### Get Hold Status
```
GET /api/v1/reservations/hold/{hold_id}
```
**Response:**
```json
{
  "hold_id": "hold_abc123",
  "status": "active",
  "remaining_minutes": 27,
  "station": {
    "id": 1,
    "name": "Kitayama Kanden Facility",
    "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1",
    "business_hours": "8:00-23:00"
  },
  "hold_start": "2024-02-15T19:15:00Z",
  "hold_end": "2024-02-15T20:15:00Z"
}
```

#### Confirm Arrival
```
POST /api/v1/reservations/hold/{hold_id}/arrive
```
**Request:**
```json
{
  "hold_id": "hold_abc123",
  "arrival_time": "2024-02-15T19:45:00Z"
}
```
**Response:**
```json
{
  "success": true,
  "hold_id": "hold_abc123",
  "charging_session_id": "chrg_xyz789",
  "next_step": "start_charging"
}
```

### 1.3 Parking Guide Dialog

#### Dialog Content
- **Title**: "Parking Guide"
- **Safety Reminder Illustration**:
  - Shows smart barrier location
  - Vehicle parking diagram
- **Text Instructions**:
  - "Please park in front of the parking space with yellow smart barrier on site."
  - "Also, when operating the app, please park your car and thoroughly confirm safety around you."
- **Action Button**: "OK"

### 1.4 Cancellation Confirmation Page

#### Page Content
- **Cancellation Policy**:
  - Title: "Cancellation Policy"
  - Detailed cancellation fee explanation:
    - (1) Quick charger: ¥500
    - (2) Standard charger: ¥200
- **Cancellation Confirmation Dialog**:
  - "Cancel the hold?"
  - "Cancellation will incur cancellation fees."
  - "Cancel Hold" orange button

### API Interfaces

#### Cancel Hold Reservation
```
POST /api/v1/reservations/hold/{hold_id}/cancel
```
**Request:**
```json
{
  "hold_id": "hold_abc123",
  "reason": "user_cancelled"
}
```
**Response:**
```json
{
  "success": true,
  "hold_id": "hold_abc123",
  "cancellation_fee": 200,
  "fee_charged": true,
  "refund_amount": 0
}
```

## 2. Time Slot Reservation Feature

### Page Design
- **Facility Information**:
  - Title: SBPS Test
  - Charging specification selection (6kW selected)
  - Charging fee display
- **Date Selection**:
  - Monthly calendar view (February)
  - Weekly calendar quick selection
  - Selected date highlighted (17th, Saturday)
- **Time Selection**:
  - Time range: 19:10 ~ 19:40 (30 minutes)
  - 24-hour timeline
  - Draggable time slot selection
  - Green indicates selected time slot
- **Action Button**:
  - "Go to Confirmation Screen"

### Interaction Features
- Intuitive timeline drag selection
- Real-time display of selected duration
- Date and time coordinated selection

### API Interfaces

#### Get Available Time Slots
```
GET /api/v1/reservations/timeslots/available
```
**Query Parameters:**
- `station_id`: 1
- `date`: "2024-02-17"
- `power_option`: "6kW"

**Response:**
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

#### Create Time Slot Reservation
```
POST /api/v1/reservations/timeslot
```
**Request:**
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
**Response:**
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

## 3. All-Day Reservation Feature

### 3.1 Reservation Selection Page

#### Page Layout
- **Facility Information** (same as above)
- **Reservation Type Toggle**:
  - "Now" (immediate)
  - "Hold" (parking hold)
  - "1-Day Reservation" (all-day) - selected
- **Reservation Date**:
  - Reservation Date: 2024/2/15
- **Charging Specifications**:
  - Quick 100kW ¥50/min
  - Max 60min | Cable Available
- **Action Options**:
  - "What is 1-Day Reservation?" help link
  - "Go to Confirmation Screen"

### API Interfaces

#### Create All-Day Reservation
```
POST /api/v1/reservations/all-day
```
**Request:**
```json
{
  "station_id": 1,
  "date": "2024-02-15",
  "power_option": "100kW",
  "user_id": "user123"
}
```
**Response:**
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

#### Check All-Day Availability
```
GET /api/v1/reservations/all-day/availability
```
**Query Parameters:**
- `station_id`: 1
- `start_date`: "2024-02-15"
- `end_date`: "2024-05-15"

**Response:**
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

### 3.2 All-Day Reservation Explanation Dialog

#### Dialog Content
- **Title**: "What is 1-Day Reservation?"
- **Caution Icon**: "CAUTION!"
- **Feature Explanation**:
  - "You can reserve a parking space by specifying a date up to 3 months in advance."
  - "Convenient for long-stay destinations."
- **Fee Reminder**:
  - "※Using 1-day reservation or same-day cancellation will incur fees."
- **Action Options**:
  - Checkbox: "Don't show next time"
  - "OK" button

## Design Specifications

### Visual Hierarchy
1. Time information priority (large font, prominent color)
2. Action buttons secondary (contrast color, medium size)
3. Detailed information last (regular font, gray)

### Interaction Principles
1. Full explanation of rules before reservation
2. Cancellation requires confirmation
3. Transparent fee information display
4. Simple and intuitive operation flow

### User Experience Optimization
- Multiple reservation methods for different needs
- Visual time selection
- Clear fee and rule explanations
- Convenient cancellation and modification

## Color Scheme
- Primary: Teal (#00BCD4)
- Accent: Orange (#FF6B35)
- Success: Green (#4CAF50)
- Background: Light Gray (#F5F5F5)

## Typography
- Headers: 20sp Bold
- Body: 16sp Regular
- Captions: 14sp Regular
- Time Display: 24sp Bold