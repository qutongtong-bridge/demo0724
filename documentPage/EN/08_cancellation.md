# Cancellation Function Design Document

## Overview

The cancellation function provides users with the operation to cancel charging space reservations, including clear visual feedback and operation confirmation process.

## Page Design

### Visual Design
- **Background**: Full-screen white background
- **Central Content**: Cancellation confirmation information

### Main Elements

#### 1. Title Text
- **Content**: "Hold has been cancelled"
- **Color**: Teal (#00BCD4)
- **Font Size**: 20sp
- **Position**: Top of page

#### 2. Status Illustration
- **Icon Elements**:
  - Speech bubble: "CANCEL!"
  - Charging station icon
  - Cancel mark (X)
  - Dotted circle indicating cancelled status
- **Color Scheme**:
  - Teal speech bubble
  - Gray dotted line
  - Black charging station

#### 3. Message Text
- **Content**: "We look forward to your next visit."
- **Font Size**: 16sp
- **Color**: Dark Gray (#333333)
- **Position**: Below illustration

#### 4. Action Button
- **Text**: "Return to Top"
- **Style**:
  - Orange background (#FF6B35)
  - White text
  - Rounded button
  - Width: 80% of screen width
- **Position**: Bottom of page

## Interaction Flow

### 1. Trigger Scenarios
- User actively cancels reservation
- Reservation timeout automatic cancellation
- System error causing cancellation

### 2. Cancellation Process
1. User clicks cancel on reservation details page
2. Cancellation confirmation dialog appears
3. User confirms cancellation operation
4. Cancellation success page displays
5. Click return to top button to go back to main interface

### API Interfaces

#### Get Reservation Details
```
GET /api/v1/reservations/{reservation_id}
```
**Response:**
```json
{
  "reservation_id": "res_123456",
  "type": "hold",
  "status": "active",
  "station": {
    "id": 1,
    "name": "Kitayama Kanden Facility",
    "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1"
  },
  "start_time": "2024-02-15T19:15:00Z",
  "end_time": "2024-02-15T20:15:00Z",
  "cancellation_policy": {
    "fee": 200,
    "free_cancellation_before": "2024-02-15T19:00:00Z"
  }
}
```

#### Cancel Reservation
```
POST /api/v1/reservations/{reservation_id}/cancel
```
**Request:**
```json
{
  "reservation_id": "res_123456",
  "cancellation_reason": "change_of_plans",
  "confirm": true
}
```
**Response:**
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

### 3. Animation Effects
- Page entry: Fade-in effect
- Illustration display: Scale animation
- Button click: Press effect

## Design Specifications

### Layout Specifications
- **Content Vertical Center** alignment
- **Element Spacing**:
  - Title to illustration: 40dp
  - Illustration to message text: 32dp
  - Message text to button: 48dp
- **Margins**: 16dp on left and right

### Illustration Specifications
- **Size**: 200dp x 200dp
- **Style**: Flat design
- **Colors**: Consistent with brand colors

### Copy Guidelines
- **Tone**: Friendly, positive
- **Wording**: Concise and clear
- **Emotion**: Reduce negative feelings

## User Experience Considerations

### 1. Emotional Design
- Friendly illustration reduces negative feelings from cancellation
- Positive copy encourages future use
- Bright colors avoid heavy feeling

### 2. Operation Guidance
- Clear return path
- Single operation option to avoid confusion
- Prominent button design

### 3. Information Communication
- Clear notification of successful cancellation
- Friendly farewell message
- Quick return to main flow

## Extended Feature Suggestions

### 1. Cancellation Reason Collection
- Optional cancellation reason selection
- Helps improve service

### 2. Recommend Other Time Slots
- Display nearby available charging stations
- Recommend other reservation time slots

### 3. Compensation Benefits
- Provide discount coupon for next use
- Points compensation mechanism

## Technical Implementation

### State Management
- Cancellation pending
- Cancellation successful
- Cancellation failed
- Error handling

### Analytics
- Track cancellation reasons
- Monitor cancellation rates
- User behavior patterns

### Performance
- Quick page load
- Smooth animations
- Immediate response

### Additional API Interfaces

#### Get Cancellation Reasons
```
GET /api/v1/cancellation/reasons
```
**Response:**
```json
{
  "reasons": [
    {
      "id": "change_of_plans",
      "label": "Change of plans",
      "category": "user"
    },
    {
      "id": "found_better_option",
      "label": "Found a better option",
      "category": "user"
    },
    {
      "id": "emergency",
      "label": "Emergency",
      "category": "emergency"
    },
    {
      "id": "other",
      "label": "Other reason",
      "category": "other",
      "requires_comment": true
    }
  ]
}
```

#### Submit Cancellation Feedback
```
POST /api/v1/cancellation/feedback
```
**Request:**
```json
{
  "reservation_id": "res_123456",
  "reason_id": "change_of_plans",
  "comment": "Meeting time changed",
  "rating": 4
}
```
**Response:**
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

#### Get Alternative Suggestions
```
GET /api/v1/reservations/alternatives
```
**Query Parameters:**
- `original_reservation_id`: "res_123456"
- `latitude`: 35.6762
- `longitude`: 139.6503

**Response:**
```json
{
  "alternatives": [
    {
      "station_id": 2,
      "station_name": "South Hill Charging Station",
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