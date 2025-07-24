# Charging Check-in Flow Design Document

## Overview

The charging check-in is the first step for users to use the charging service. The entire process is divided into 4 steps, with a progress indicator at the top showing the current stage.

## Step 1: Vehicle Selection

### Page Title
- "Charging Preparation"

### Progress Indicator
- 4 dots with the 1st highlighted
- Step label: Vehicle Selection

### Main Content
- **Prompt Text**: "Please select a number."
- **Charging Station Information**:
  - Number: 1
  - Charging Specifications: Standard 6/3kW ¥~100/15min
  - Daily Maximum: ¥100,000
  - Cable Availability: Cable Available
- **Visual Elements**: Charging station icon display

### Interaction Details
- Users need to select charging station number
- Display detailed specifications of the charging station
- Help link at bottom: "I don't know which parking space"

### Design Features
- Clear numbered selection
- Prominent display of key information
- Visual representation of charging equipment

### API Interfaces

#### Get Available Charging Stations
```
GET /api/v1/stations/available
```
**Response:**
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

#### Select Charging Station
```
POST /api/v1/checkin/select-station
```
**Request:**
```json
{
  "station_id": 1,
  "user_id": "user123"
}
```
**Response:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "next_step": "power_selection"
}
```

## Step 2: Power Output Selection

### Page Title
- "Charging Preparation"

### Progress Indicator
- 4 dots with the 2nd highlighted
- Step label: Output Selection

### Main Content
- **Prompt Text**: "Please select output power."
- **Power Options**:
  1. **6kW**
     - Price: ¥100/15min
     - Daily Maximum: ¥100,000
  2. **3kW**
     - Price: ¥50/15min
     - Daily Maximum: ¥50,000

### Interaction Details
- Users must select charging power
- Different power levels have different pricing
- Selected option shows visual feedback

### Design Features
- Clear price comparison
- Radio button selection
- Highlighted selection state

### API Interfaces

#### Select Power Output
```
POST /api/v1/checkin/select-power
```
**Request:**
```json
{
  "session_id": "sess_abc123",
  "power_option": "6kW",
  "price_per_15min": 100
}
```
**Response:**
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

## Step 3: Content Confirmation

### Page Title
- "Charging Preparation"

### Progress Indicator
- 4 dots with the 3rd highlighted
- Step label: Content Confirmation

### Main Content
- **Prompt Text**: "Starting to use SBPS Test."
- **Confirmation Information**:
  - Test Name: SBPS Test
  - Address: Tokyo, Shinagawa-ku, Osaki 2-1-1
  - Payment badge
- **Charger Information**:
  - Standard 6kW (Cable Available)
  - Price: ¥100/15min (Daily Max ¥100,000)
- **Fee Details**:
  - Usage Fee: ¥100/15min
  - Estimated Total: ¥334
- **Important Notes**:
  - Actual usage time billing explanation
  - Free preparation time explanation
  - Separate parking fee notice
  - Overtime fee explanation

### Design Features
- Comprehensive information display
- Clear fee breakdown
- Important notices highlighted

### API Interfaces

#### Get Charging Summary
```
GET /api/v1/checkin/summary/{session_id}
```
**Response:**
```json
{
  "session_id": "sess_abc123",
  "station": {
    "name": "SBPS Test",
    "address": "Tokyo, Shinagawa-ku, Osaki 2-1-1",
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

#### Confirm Charging
```
POST /api/v1/checkin/confirm
```
**Request:**
```json
{
  "session_id": "sess_abc123",
  "user_confirmation": true,
  "payment_method": "default"
}
```
**Response:**
```json
{
  "success": true,
  "session_id": "sess_abc123",
  "charging_id": "chrg_xyz789",
  "next_step": "parking_guidance"
}
```

## Step 4: Parking Guidance

### Page Title
- "Charging Preparation"

### Progress Indicator
- 4 dots with the 4th highlighted
- Step label: Parking

### Main Content
- **Parking Guide Illustration**:
  - Charging station layout display
  - "PARKING" position marker
  - Vehicle parking diagram
- **Safety Instructions**:
  - "After the smart barrier is completely lowered, please confirm safety around you before parking."
- **Confirmation Button**: "OK"

### Interaction Details
- Final safety reminder
- User confirms to start charging

### Design Features
- Visual parking guidance
- Safety-first approach
- Clear call-to-action

### API Interfaces

#### Start Charging Session
```
POST /api/v1/charging/start
```
**Request:**
```json
{
  "charging_id": "chrg_xyz789",
  "station_id": 1,
  "user_confirmation": "parked"
}
```
**Response:**
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

#### Update Barrier Status (WebSocket)
```
WS /api/v1/charging/barrier-status
```
**Message:**
```json
{
  "charging_id": "chrg_xyz789",
  "barrier_status": "lowered",
  "safe_to_proceed": true
}
```

## Design Specifications

### Visual Design
- Clear step progress indication
- Important information highlighted
- Illustrations to aid understanding
- Consistent color system

### User Experience
- Clear, sequential steps
- Important information confirmed multiple times
- Transparent fees to avoid misunderstanding
- Comprehensive safety tips

### Interaction Design
- Each step requires active user confirmation
- Back function supports modifying selections
- Help information always accessible

## Color Scheme
- Primary: Teal (#00BCD4)
- Accent: Orange (#FF6B35)
- Background: Light Gray (#F5F5F5)
- Text: Dark Gray (#333333)

## Typography
- Headers: 20sp, Bold
- Body text: 16sp, Regular
- Captions: 14sp, Regular
- Button text: 16sp, Medium