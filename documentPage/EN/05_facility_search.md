# Facility Search and Filter Design Document

## Overview

The facility search and filter feature helps users quickly find charging stations that meet their needs, including map display, search functionality, and multi-dimensional filtering.

## 1. Map Main Interface

### Page Layout
- **Map Display Area**:
  - Full-screen map background
  - User's current location marker (blue dot)
  - Charging station location markers (green icons)
  - Street name labels

- **Facility Information Card**:
  - Located at bottom, swipeable up/down
  - Shows selected charging station information

- **Action Buttons**:
  - Bottom right: Menu button (three horizontal lines)
  - Bottom right: Location button (arrow icon)

### Facility Information Display
- **Title**: Kitayama Kanden Facility
- **Availability Status**: Available (Quick: 1/1) - green dot indicator
- **Distance**: 57m from current location
- **Business Status**: Open | 8:00～23:00
- **Reservation Options**:
  - Now (immediate) - plug icon
  - Hold - 1h clock icon
  - 1-Day Reservation - calendar icon

### API Interfaces

#### Get Nearby Stations
```
GET /api/v1/stations/nearby
```
**Query Parameters:**
- `latitude`: 35.6762
- `longitude`: 139.6503
- `radius`: 5000 (meters)

**Response:**
```json
{
  "stations": [
    {
      "id": 1,
      "name": "Kitayama Kanden Facility",
      "location": {
        "latitude": 35.6765,
        "longitude": 139.6510,
        "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1"
      },
      "distance": 57,
      "status": "available",
      "chargers": {
        "quick": {
          "available": 1,
          "total": 1
        },
        "standard": {
          "available": 0,
          "total": 0
        }
      },
      "business_hours": {
        "open": "08:00",
        "close": "23:00",
        "is_open": true
      },
      "reservation_types": ["immediate", "hold", "all_day"]
    }
  ]
}
```

#### Get Station Details
```
GET /api/v1/stations/{station_id}
```
**Response:**
```json
{
  "id": 1,
  "name": "Kitayama Kanden Facility",
  "location": {
    "latitude": 35.6765,
    "longitude": 139.6510,
    "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1"
  },
  "chargers": [
    {
      "id": "chrg_001",
      "type": "quick",
      "power": "50kW",
      "status": "available",
      "price_per_15min": 500
    }
  ],
  "amenities": ["toilet", "convenience_store"],
  "payment_methods": ["credit_card", "ic_card", "app"],
  "photos": ["station_photo1.jpg", "station_photo2.jpg"]
}
```

## 2. Search Interface

### Search Box Design
- **Input Prompt**: "Where would you like to go?"
- **Search Suggestion List**:
  - Kitayama Kanden Facility
    - Quick: 1/1 (green status)
    - 57m | Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1
    - Open | 8:00～23:00
  - SBPS Test
    - Standard: 0/1 (red status)
    - 510m | Tokyo, Shinagawa-ku, Osaki 2-1-1
    - Open | 6:00～22:00

### Keyboard Interaction
- Supports Japanese input
- Real-time search suggestions
- Quick input options

### API Interfaces

#### Search Stations
```
POST /api/v1/stations/search
```
**Request:**
```json
{
  "query": "Kitayama",
  "location": {
    "latitude": 35.6762,
    "longitude": 139.6503
  },
  "max_results": 10
}
```
**Response:**
```json
{
  "results": [
    {
      "id": 1,
      "name": "Kitayama Kanden Facility",
      "match_score": 0.95,
      "location": {
        "address": "Tokyo, Shinagawa-ku, Kitashinagawa 5-3-1",
        "distance": 57
      },
      "availability": {
        "quick": "1/1",
        "standard": "0/0"
      },
      "business_hours": {
        "display": "8:00～23:00",
        "is_open": true
      }
    },
    {
      "id": 2,
      "name": "SBPS Test",
      "match_score": 0.75,
      "location": {
        "address": "Tokyo, Shinagawa-ku, Osaki 2-1-1",
        "distance": 510
      },
      "availability": {
        "quick": "0/0",
        "standard": "0/1"
      },
      "business_hours": {
        "display": "6:00～22:00",
        "is_open": true
      }
    }
  ]
}
```

## 3. Filter Function

### Filter Dimensions

#### 3.1 Charging Type
- **Options** (Multi-select):
  - ✓ Quick Charging
  - ✓ Standard Charging

#### 3.2 Usage Type
- **Options** (Multi-select):
  - ✓ Hold
  - ✓ 1-Day Reservation
  - ✓ Use without reservation
- **Help Link**: "What is Hold?"

#### 3.3 Facility Type
- **Options** (Multi-select):
  - ✓ Show Plugo chargers
  - ✓ Show other chargers

#### 3.4 Other Conditions
- **Options** (Multi-select):
  - ✓ Show planned installation chargers

### Filter Operations
- **Reset Function**: "Show All" - orange text
- **Apply Filter**: "13746 Facilities" - blue button
- **Close Button**: X in top right corner

### Interaction Design
- Checkboxes support multi-selection
- Real-time update of filtered result count
- Filter conditions can be saved
- One-click reset all filters

### API Interfaces

#### Apply Filters
```
POST /api/v1/stations/filter
```
**Request:**
```json
{
  "location": {
    "latitude": 35.6762,
    "longitude": 139.6503,
    "radius": 10000
  },
  "filters": {
    "charging_types": ["quick", "standard"],
    "usage_types": ["hold", "all_day", "no_reservation"],
    "facility_types": ["plugo", "others"],
    "other_conditions": ["show_planned"]
  }
}
```
**Response:**
```json
{
  "total_count": 13746,
  "stations": [
    {
      "id": 1,
      "name": "Kitayama Kanden Facility",
      "location": {
        "latitude": 35.6765,
        "longitude": 139.6510,
        "distance": 57
      },
      "charger_types": ["quick"],
      "usage_types": ["hold", "all_day", "no_reservation"],
      "facility_type": "plugo",
      "status": "operational"
    }
  ],
  "applied_filters": {
    "charging_types": 2,
    "usage_types": 3,
    "facility_types": 2,
    "other_conditions": 1
  }
}
```

#### Get Filter Options
```
GET /api/v1/stations/filter-options
```
**Response:**
```json
{
  "charging_types": [
    {
      "id": "quick",
      "name": "Quick Charging",
      "count": 8523
    },
    {
      "id": "standard", 
      "name": "Standard Charging",
      "count": 12456
    }
  ],
  "usage_types": [
    {
      "id": "hold",
      "name": "Hold",
      "count": 9876,
      "help_text": "What is Hold?"
    },
    {
      "id": "all_day",
      "name": "1-Day Reservation",
      "count": 7654
    },
    {
      "id": "no_reservation",
      "name": "Use without reservation",
      "count": 13746
    }
  ],
  "facility_types": [
    {
      "id": "plugo",
      "name": "Plugo chargers",
      "count": 5432
    },
    {
      "id": "others",
      "name": "Other chargers",
      "count": 8314
    }
  ]
}
```

## 4. Design Specifications

### Color Specifications
- Available Status: Green (#4CAF50)
- Unavailable Status: Red (#F44336)
- Selected State: Orange (#FF6B35)
- Primary Color: Teal (#00BCD4)

### Icon Design
- Charging Station Icon: Green background + white charging symbol
- Status Indicator: Dot (Green = available, Red = occupied)
- Function Icons: Linear icon style

### Information Hierarchy
1. Facility name and status (most important)
2. Distance and address (secondary)
3. Business hours (auxiliary information)

### User Experience Considerations
- Map and list viewing options
- Quick filter for common conditions
- Real-time charging station status display
- One-click navigation function
- Support for multiple reservation methods

## Technical Features

### Map Integration
- Real-time location tracking
- Smooth zoom and pan
- Clustering for multiple stations
- Custom marker designs

### Search Algorithm
- Fuzzy matching support
- Location-based prioritization
- Recent search history
- Predictive suggestions

### Filter Performance
- Instant filter application
- Persistent filter selections
- Filter combination logic
- Result count optimization