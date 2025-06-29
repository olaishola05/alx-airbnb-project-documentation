# Airbnb Clone Backend – Technical and Functional Requirements

This document defines the technical and functional requirements for core backend features, including API specifications, input/output formats, validation rules, and performance considerations.

## 1. User Authentication

### Functional Requirements

- Allow users to register as either guest or host.
- Allow users to log in with email and password.
- Issue JWT tokens upon successful login.
- Protect private routes using token-based authentication.
- Support role-based access control (RBAC).

### API Endpoints

`POST /api/auth/register`

**Description:** Register a new user.

**Request Body:**

```json
{
  "first_name": "John",
  "last_name": "Doe",
  "email": "john@example.com",
  "password": "securePassword123",
  "role": "guest"
}
```

**Validation Rules:**

- Email must be unique and valid.
- Password must be at least 8 characters.
- Role must be one of: guest, host.

**Response:**

- 201 Created with JWT token on success.
- 400 Bad Request for validation errors.

`POST /api/auth/login`

Description: Log in an existing user.

**Request Body:**

```json
{
  "email": "john@example.com",
  "password": "securePassword123"
}

```

**Response**

- 200 OK with JWT token on success.
- 401 Unauthorized for invalid credentials.

### Performance Criteria

- Passwords must be hashed using bcrypt or similar.
- Token verification must not exceed 100ms on average.

## Property Management

### Functional Requirements

- Hosts can create, update, and delete property listings.
- Properties should include title, description, location, price, and amenities.
- Hosts can upload and manage images.

### API Endpoints

`POST /api/properties`

Description: Create a new property listing (host only).

**Request Body:**

```json
{
  "name": "Cozy Cabin in the Woods",
  "description": "A lovely, quiet place to escape the city.",
  "location": "Asheville, NC",
  "price_per_night": 150,
  "amenities": ["wifi", "kitchen", "parking"]
}

```

**Validation Rules:**

- Name, description, location, and price_per_night are required.
- Price must be a positive decimal.

**Response:**

- 201 Created with property data.
- 403 Forbidden if user is not a host.

`PUT /api/properties/:id`

Description: Edit an existing property (host only).

Request Body: Same fields as POST, all optional for partial update.

**Response:**

- 200 OK on success.
- 404 Not Found if property does not exist or does not belong to the user.

`DELETE /api/properties/:id`

Description: Delete a property listing (host only).

**Response:**

- 204 No Content on success.

**Performance Criteria**

- Image uploads should complete in under 2 seconds.
- Property search must support pagination and filtering with latency < 500ms.

## Booking System

### Functional Requirements

- Guests can book available properties for a range of dates.
- Bookings must prevent overlapping reservations.
- Guests and hosts can cancel bookings based on defined rules.
- Bookings have statuses: pending, confirmed, canceled, completed.

API Endpoints
`POST /api/bookings`

Description: Create a new booking.

**Request Body:**

```json
{
  "property_id": "abc-123",
  "start_date": "2025-08-01",
  "end_date": "2025-08-05"
}
```

**Validation Rules:**

- Dates must be valid and not overlap with existing bookings.
- Start date must be before end date.
- Property must exist.

**Response:**

- 201 Created with booking details.
- 409 Conflict if dates are already booked.

`GET /api/bookings/:id`

Description: Retrieve booking details.

**Response:**

- 200 OK with booking object.
- 403 Forbidden if user does not own the booking.

`DELETE /api/bookings/:id`

Description: Cancel a booking.

**Response:**

- 200 OK with updated status.
- 403 Forbidden if user cannot cancel.
- 400 Bad Request if booking is not cancellable.

**Performance Criteria**

- Booking conflict check must complete in under 100ms.
- Booking confirmation should trigger async notification within 1 second.
