# EventHub
# EventHub API

A Django REST Framework backend API for a simplified event ticketing platform. Users can browse events, reserve seats, and cancel reservations.

## Requirements

- Python 3.8+
- Django 4.2+
- Django REST Framework 3.14+

## Setup / Run

From the repo root (where you created your virtualenv):

```powershell
$env:PYTHONUTF8=1
.\.venv\Scripts\python.exe -m pip install -r .\eventhub\requirements.txt
cd .\eventhub
.\.venv\Scripts\python.exe manage.py makemigrations
.\.venv\Scripts\python.exe manage.py migrate
.\.venv\Scripts\python.exe manage.py runserver 127.0.0.1:8000
```

Then open:

- `http://127.0.0.1:8000/api/`

## Data Models

### Event
- `title`: Event name
- `venue`: Location
- `date`: Event date
- `total_seats`: Total capacity
- `available_seats`: Seats currently available
- `status`: upcoming, ongoing, completed, cancelled
- `created_at`: Timestamp

### Reservation
- `event`: ForeignKey to Event
- `attendee_name`: Name of the attendee
- `attendee_email`: Email of the attendee
- `seats_reserved`: Number of seats booked
- `status`: confirmed, cancelled
- `created_at`: Timestamp

## Endpoints

Base URL:

- `http://127.0.0.1:8000/api/`

### Event Endpoints

- `GET /api/events/`  
  List all events.

- `GET /api/events/?status=upcoming`  
  Filter events by status.

- `GET /api/events/?venue=Bangalore`  
  Filter events by venue (case-insensitive).

- `POST /api/events/`  
  Create a new event.

Request JSON:

```json
{
  "title": "PyCon India 2025",
  "venue": "NIMHANS Convention Centre, Bangalore",
  "date": "2025-09-20",
  "total_seats": 500,
  "available_seats": 500,
  "status": "upcoming"
}
```

- `GET /api/events/{id}/`  
  Retrieve a single event.

- `PUT /api/events/{id}/`  
  Update an event.

- `DELETE /api/events/{id}/`  
  Delete an event.

### Reservation Endpoints

- `GET /api/reservations/`  
  List all reservations.

- `GET /api/reservations/?event_id=1`  
  Filter reservations by event.

- `POST /api/reservations/`  
  Create a reservation (deducts seats from event).

Request JSON:

```json
{
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2
}
```

Expected response (201):

```json
{
  "id": 1,
  "event": 1,
  "attendee_name": "Priya Sharma",
  "attendee_email": "priya@example.com",
  "seats_reserved": 2,
  "status": "confirmed",
  "created_at": "2025-03-30T10:00:00Z"
}
```

Overbooking failure (400):

```json
{
  "non_field_errors": ["Only 1 seat(s) available."]
}
```

- `POST /api/reservations/{id}/cancel/`  
  Cancel a reservation (restores seats to event).

No request body needed. Returns the updated reservation with `status: cancelled`.

- `GET /api/reservations/{id}/`  
  Retrieve a single reservation.

- `PUT /api/reservations/{id}/`  
  Update a reservation.

- `DELETE /api/reservations/{id}/`  
  Delete a reservation.

## Middleware

`RequestLoggingMiddleware` logs every request to the console with:

- HTTP method
- Path
- Status code
- Duration

Example log:

```
INFO POST /api/reservations/ - 201 - 0.12s
```

## Design Decision

**Seat deduction in ReservationSerializer.create()**  
I chose to deduct `available_seats` and create the `Reservation` inside the same `create()` method in the serializer. This keeps both writes in one place and ensures atomicity for this assignment’s scope. In a production system with high concurrency, I would use `transaction.atomic()` to prevent race conditions, but the brief explicitly notes this as a future topic.

## Testing Checklist

- Create an event
- List all events
- Filter events by status
- Filter events by venue
- Create a reservation (seats deducted from event)
- Overbooking attempt returns 400
- Cancel a reservation (seats restored to event)
- Filter reservations by event_id
- Middleware logs appear in the terminal for every request


