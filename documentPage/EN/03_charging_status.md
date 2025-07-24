# Charging Status Management Design Document

## Overview

The charging status management module provides users with real-time charging status monitoring and control functions, including remaining time display, charging termination options, and charging completion reminders.

## Charging Screen 1: Charging in Progress

### Page Title
- "Charging Available..."

### Main Features
1. **Remaining Time Display**
   - Large font showing remaining time: "Remaining Time 0:27"
   - Clock icon for visual support
   - Orange circular progress bar indicating charging progress

2. **Terminate Charging Button**
   - White button: "End Usage"
   - Help link: "How to end usage"

3. **Usage Status Information**
   - Usage Start Time: 19:12
   - Usage End Time: 19:40
   - Usage Duration: 1 minute (rounded up if less than 1 minute)
   - Free Time Included: -1 minute (as preparation time)

### Design Features
- Clean interface design with focus on remaining time
- Circular progress bar for intuitive charging progress display
- Gray background to reduce visual fatigue

### API Interfaces

#### Get Charging Status
```
GET /api/v1/charging/status/{session_id}
```
**Response:**
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

#### Real-time Charging Data (WebSocket)
```
WS /api/v1/charging/realtime/{session_id}
```
**Message:**
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

## Charging Screen 2: Termination Confirmation Dialog

### Dialog Title
- "How to End Usage?"

### Dialog Content
1. **Operation Instruction Illustration**
   - Shows charger removal steps
   - User operation diagram

2. **Text Instructions**
   - "You can end usage by unplugging the charger and leaving the parking space, or by pressing the 'End Usage' button."

3. **Action Button**
   - "OK" confirmation button

### Interaction Design
- Semi-transparent overlay background
- Clear operation guidance
- Simple confirmation process

## Charging Screen 3: Exit Reminder Dialog

### Dialog Title
- "After usage ends, please exit from the EV parking space."

### Dialog Content
1. **Warning Icon**
   - "CAUTION!" warning sign
   - Orange emphasis color

2. **Important Reminder**
   - "Please end usage after you are fully prepared."

3. **Fee Notice**
   - "※If you remain parked in the EV space after ending, penalty fees may apply or your account may be suspended."

4. **Action Button**
   - "End Usage" orange button

### Design Elements
- Prominent warning notice
- Clear fee explanation
- Prevention of user errors

### API Interfaces

#### Terminate Charging
```
POST /api/v1/charging/terminate
```
**Request:**
```json
{
  "session_id": "sess_abc123",
  "termination_type": "user_initiated",
  "confirm": true
}
```
**Response:**
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

#### Get Termination Status
```
GET /api/v1/charging/termination-status/{session_id}
```
**Response:**
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

## Charging Complete Screen

### Page Title
- "Thank you for using our service."

### Main Content
1. **Completion Illustration**
   - Animation of vehicle leaving charging station
   - "THANK YOU" message
   - Friendly ending experience

2. **Fee Summary**
   - Total Fee: ¥500
   - Orange highlighting for amount

3. **Usage Details**
   - Charging Station: SBPS Test
   - Usage Time: 2024/2/15(Thu) 19:12～19:13
   - Usage Fee: ¥0
   - Reservation Fee: ¥500
   - Unit Price: ¥100/15min
   - Charging Service Time: 0min (excluding 5min free time)

4. **Action Options**
   - "OK" confirmation button
   - "Issue Receipt" link

### User Experience Optimization
- Friendly completion screen
- Clear fee breakdown
- Convenient receipt issuance

### API Interfaces

#### Get Charging Completion Details
```
GET /api/v1/charging/completion/{session_id}
```
**Response:**
```json
{
  "session_id": "sess_abc123",
  "station": {
    "name": "SBPS Test",
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
    "unit_price": "¥100/15min"
  },
  "energy_delivered": {
    "kwh": 0.1,
    "average_power_kw": 6
  },
  "receipt_available": true
}
```

## Design Specifications

### Color Usage
- Primary: Teal (#00BCD4) for titles and important information
- Accent: Orange (#FF6B35) for buttons and progress display
- Background: Light Gray (#F5F5F5)
- Text: Dark Gray (#333333)

### Interaction Principles
1. Real-time charging status updates
2. Secondary confirmation for important operations
3. Transparent fee information
4. Simple and intuitive operation flow

### Safety Considerations
- Confirmation steps for charging termination
- Clear fee and penalty notices
- Design to prevent misoperation

## Technical Requirements

### Update Frequency
- Remaining time: Every 1 second
- Progress bar: Every 10 seconds
- Status changes: Real-time

### State Management
- Active charging state
- Paused state
- Completed state
- Error state

### Notification System
- 5 minutes before completion warning
- Completion notification
- Overtime warning

### Additional API Interfaces

#### Send Charging Notification
```
POST /api/v1/charging/notifications
```
**Request:**
```json
{
  "session_id": "sess_abc123",
  "notification_type": "5_minute_warning",
  "delivery_method": ["push", "email"]
}
```
**Response:**
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

#### Get Charging History
```
GET /api/v1/charging/history
```
**Query Parameters:**
- `user_id`: "user123"
- `limit`: 10
- `offset`: 0

**Response:**
```json
{
  "total_count": 25,
  "sessions": [
    {
      "session_id": "sess_abc123",
      "date": "2024-02-15",
      "station_name": "SBPS Test",
      "duration_minutes": 1,
      "energy_kwh": 0.1,
      "cost": 500,
      "status": "completed"
    }
  ]
}
```

#### Report Charging Issue
```
POST /api/v1/charging/report-issue
```
**Request:**
```json
{
  "session_id": "sess_abc123",
  "issue_type": "unable_to_terminate",
  "description": "Terminate button not responding",
  "urgency": "high"
}
```
**Response:**
```json
{
  "success": true,
  "ticket_id": "ticket_789",
  "support_contact": "+81-3-1234-5678",
  "estimated_response_time": "5 minutes"
}
```