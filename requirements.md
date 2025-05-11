# Requirement Specifications for Core Backend Features

## User Authentication & Management

### Functional Requirements

- Register as guest/host with email/password or OAuth (Google)
- Login with credentials + JWT generation
- Update profile (name, phone, profile photo)
- Deactivate account

### Technical Specifications

**API Endpoints**:

```
POST /api/auth/register
POST /api/auth/login
GET /api/users/{userId}
PATCH /api/users/{userId}
DELETE /api/users/{userId}
```

**Input Validation:**
| **Field** | **Rules** |
|:---|:---|
| email | Valid format, unique |
| password | Min 8 chars, 1 special char |
| phone | E.164 format (e.g., +254...) |
| role | Enum: guest, host, admin |

**Performance:**

- Handle 1000 req/min with <500ms latency
- JWT expiration: 24 hours

## Property Management

### Functional Requirements

- Hosts create/update/delete listings
- Search properties with filters (location, price, amenities)
- View listing details with availability calendar
- Technical Specifications

### Technical Specifications

**API Endpoints:**

```
POST /api/properties
GET /api/properties?location=...&minPrice=...
PATCH /api/properties/{propertyId}
DELETE /api/properties/{propertyId}
```

**Data Model:**

```json
{
  "title": "Beach House",
  "description": "3-bedroom villa...",
  "locationId": "uuid",
  "pricePerNight": 150.0,
  "amenities": ["wifi", "pool"],
  "availability": ["2024-01-01", "2024-01-05"]
}
```

**Validation:**

- Location must exist in locations table
- Price > $10/night
- Availability dates not overlapping

**Performance:**

- Search returns <1s for 10k properties
- Geo queries use spatial index on (lat,lng)

## Booking System

### Functional Requirements

- Reserve properties for specific dates
- Prevent double-booking
- Calculate total price dynamically
- Cancel bookings with policy enforcement

### Technical Specifications

**API Endpoints:**

```
POST /api/bookings
GET /api/bookings/{bookingId}
PATCH /api/bookings/{bookingId}/cancel
```

**Booking Flow:**

- Check availability for dates
- Lock dates temporarily (Redis)
- Process payment
- Confirm booking

**Validations:**

```json
{
  "propertyId": "uuid",
  "startDate": "YYYY-MM-DD",
  "endDate": "YYYY-MM-DD",
  "guests": 2
}
```

**Performance:**

- Handle 500 concurrent bookings/min
- Booking confirmation <2s
