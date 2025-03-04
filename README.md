# DineFlex API Documentation

## Introduction

This document outlines the DineFlex API, which enables users to discover restaurants, view availability, and make bookings for early bird and last-minute dining offers.

## Base URL

All endpoints are relative to the base URL:

```
https://api.dineflex.ie
```

## API Endpoints

### Restaurant Endpoints

#### List All Restaurants

Returns a list of all restaurants participating in the DineFlex programme.

```
GET /restaurants
```

**Response (200 OK)**

```json
[
  {
    "id": "12345",
    "name": "Restaurant Name",
    "thumbnailUrl": "https://example.com/image.jpg",
    "location": "Dublin",
    "cuisine": "Italian",
    "hasEarlyBird": true,
    "hasLastMinute": false
  }
]
```

#### Get Restaurant Details

Returns detailed information about a specific restaurant.

```
GET /restaurants/:id
```

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string | Unique identifier of the restaurant |

**Response (200 OK)**

```json
{
  "id": "12345",
  "name": "Restaurant Name",
  "description": "Detailed description of the restaurant",
  "images": ["https://example.com/image1.jpg"],
  "address": "123 Main Street, Dublin",
  "phone": "+353 1 234 5678",
  "openingHours": "Mon-Fri: 12:00-22:00, Sat-Sun: 13:00-23:00",
  "cuisine": "Italian",
  "earlyBirdOffers": [
    {
      "id": "offer123",
      "title": "Early Dinner Special",
      "description": "3 courses for €25",
      "availableTimes": "17:00-19:00"
    }
  ],
  "lastMinuteAvailable": false
}
```

#### Get Restaurant Availability

Returns available time slots for a specific restaurant on a given date.

```
GET /restaurants/:id/availability
```

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string | Unique identifier of the restaurant |

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| date | string | Date in YYYY-MM-DD format |

**Response (200 OK)**

```json
{
  "restaurantId": "12345",
  "date": "2023-10-15",
  "availableSlots": [
    {
      "time": "17:30",
      "type": "earlyBird",
      "offerId": "offer123"
    },
    {
      "time": "18:00",
      "type": "earlyBird",
      "offerId": "offer123"
    },
    {
      "time": "19:30",
      "type": "regular"
    },
    {
      "time": "20:30",
      "type": "lastMinute",
      "discount": "20%"
    }
  ]
}
```

### Booking Endpoints

#### Create Booking

Creates a new booking for a restaurant.

```
POST /bookings
```

**Request Body**

```json
{
  "restaurantId": "12345",
  "date": "2023-10-15",
  "time": "18:00",
  "partySize": 4,
  "customerName": "John Smith",
  "customerEmail": "john@example.com",
  "customerPhone": "+353 87 123 4567"
}
```

**Response (200 OK)**

```json
{
  "id": "booking789",
  "status": "confirmed",
  "restaurantId": "12345",
  "restaurantName": "Restaurant Name",
  "date": "2023-10-15",
  "time": "18:00",
  "partySize": 4,
  "customerName": "John Smith",
  "confirmationCode": "DINE12345"
}
```

For validation errors, the response will include details about each invalid field:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "The request contains invalid data",
  "details": {
    "fields": [
      {
        "field": "email",
        "code": "INVALID_FORMAT",
        "message": "Email address format is invalid"
      },
      {
        "field": "partySize",
        "code": "OUT_OF_RANGE",
        "message": "Party size must be between 1 and 12"
      }
    ]
  }
}
```

#### Get Booking Details

Returns details about a specific booking.

```
GET /bookings/:id
```

**Path Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| id | string | Unique identifier of the booking |

**Response (200 OK)**

```json
{
  "id": "booking789",
  "status": "confirmed",
  "restaurantId": "12345",
  "restaurantName": "Restaurant Name",
  "date": "2023-10-15",
  "time": "18:00",
  "partySize": 4,
  "customerName": "John Smith",
  "confirmationCode": "DINE12345"
}
```

## Error Handling

The API uses standard HTTP status codes to indicate the general category of response:

| Status Code | Description |
|-------------|-------------|
| 200 | Success |
| 400 | Bad Request - Client-side input errors |
| 401 | Unauthorized - Authentication required |
| 403 | Forbidden - Insufficient permissions |
| 404 | Not Found - Resource doesn't exist |
| 409 | Conflict - Resource conflict (e.g., double booking) |
| 422 | Unprocessable Entity - Validation failures |
| 500 | Server Error - Unexpected errors |

All error responses follow this format:

```json
{
  "code": "ERROR_CODE",
  "message": "Human-readable error message",
  "details": {
    // Additional context (varies by error type)
  }
}
```

## Error Codes Reference

### General Errors

| Error Code | Description |
|------------|-------------|
| `INVALID_REQUEST` | Request is missing required fields or has an invalid format |
| `SERVER_ERROR` | Unexpected server error occurred |

### Restaurant Errors

| Error Code | Description |
|------------|-------------|
| `RESTAURANT_NOT_FOUND` | Restaurant with the specified ID was not found |
| `RESTAURANT_CLOSED` | Restaurant is closed on the requested date |

### Booking Errors

| Error Code | Description |
|------------|-------------|
| `BOOKING_NOT_FOUND` | Booking with the specified ID was not found |
| `BOOKING_TIME_UNAVAILABLE` | The requested time slot is no longer available |
| `BOOKING_PARTY_TOO_LARGE` | Party size exceeds restaurant capacity |
| `BOOKING_PAST_DATE` | Cannot book for a date in the past |

### Validation Errors

| Error Code | Description |
|------------|-------------|
| `VALIDATION_ERROR` | One or more fields contain invalid data |
| `VALIDATION_INVALID_DATE_FORMAT` | Date format is invalid (should be YYYY-MM-DD) |

#### Field-Specific Validation Codes

| Error Code | Description |
|------------|-------------|
| `VALIDATION_INVALID_FORMAT` | Field format is invalid |
| `VALIDATION_REQUIRED_FIELD` | Field is required but was not provided |
| `VALIDATION_OUT_OF_RANGE` | Numeric value is outside accepted range |
