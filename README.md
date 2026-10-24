# EventHub API

EventHub is a lightweight, backend RESTful API built with Django and Django REST Framework (DRF) for a simplified event ticketing platform. It allows users to browse events, purchase ticket reservations, and manage cancellations while ensuring proper tracking of seat availability metrics.

---

## How to Run the Project

Follow these steps sequentially to set up and start the application on your local machine:

### 1. Project Initialization & Environment Setup
Clone or navigate to the root directory where the application code resides:
```bash
# Navigate to the project root directory
cd eventhub_project2

### 2. Dependency Management
Install the core application frameworks and export standard freeze manifests:
```bash
# Install framework dependencies
pip install django djangorestframework

# Generate project requirements manifest
pip freeze > requirements.txt
```

### 3. Database Schema Provisioning
Run the built-in migration manager to define local structural tables:
```bash
# Formulate structural database schemas
python manage.py makemigrations

# Execute transactional schema setup alterations
python manage.py migrate
```

### 4. Boot Up the Server Instance
Launch the application development instance server:
```bash
python manage.py runserver
```
The active interface framework will default to running on: `http://127.0.0.1:8000`

---

## System Endpoints & API Specifications

### Event Endpoints (`/api/events/`)

*   **`POST /api/events/`**
    *   **Description:** Creates a new event record.
    *   **Validation Constraint:** Ensures `available_seats` does not exceed `total_seats`.
*   **`GET /api/events/`**
    *   **Description:** Retrieves all events ordered chronologically by date.
    *   **Query Parameters:**
        *   `status`: Filters events by an exact state matching key (`upcoming`, `ongoing`, `completed`, `cancelled`).
        *   `venue`: Performs a case-insensitive search filter matching text against the venue name (`venue__icontains`).
*   **`GET /api/events/<id>/`**
    *   **Description:** Retrieves granular event configuration properties including a computed `reservations_count` metric tracking confirmed seats.
*   **`PUT/PATCH /api/events/<id>/`**
    *   **Description:** Modifies existing descriptive attributes for a specific event entry.
*   **`DELETE /api/events/<id>/`**
    *   **Description:** Purges an event configuration row completely from active records.

### Reservation Endpoints (`/api/reservations/`)

*   **`POST /api/reservations/`**
    *   **Description:** Processes a new booking reservation and automatically deducts the requested seats from the event's availability profile.
    *   **Validation Constraints:** 
        *   Ensures that `seats_reserved` is at least 1.
        *   Ensures the target event status is either `upcoming` or `ongoing`.
        *   Validates that the event has sufficient `available_seats` remaining.
        <img width="940" height="607" alt="image" src="https://github.com/user-attachments/assets/848fcd3f-460f-4144-be35-c1b3bbcefdef" />

        <img width="940" height="450" alt="image" src="https://github.com/user-attachments/assets/e579f7a5-06d3-49bc-86d2-695e0a4fa7da" />


*   **`GET /api/reservations/`**
    *   **Description:** Lists all reservation instances ordered by newest creation date (`-created_at`).
    *   **Query Parameters:**
        *   `event_id`: Filters the entire list to return only reservations tied to a specific event ID.
*   **`GET /api/reservations/<id>/`**
    *   **Description:** Details structural information for a distinct reservation instance.
*   **`POST /api/reservations/<id>/cancel/`**
    *   **Description:** A custom action route that shifts the target reservation status metadata flag to `cancelled` and increments the parent event's `available_seats` count by the value of the seats refunded. Returns an HTTP 400 Bad Request if the reservation was already marked as cancelled.
    <img width="940" height="370" alt="image" src="https://github.com/user-attachments/assets/6484c401-236d-4f67-ac11-9575b7694bdd" />

---

## Architectural & Design Decisions

### Strategic Design Selection: Encapsulated Atomic Mutations in `Serializer.create()`

**Decision:** The system couples structural model verification checks alongside data persistence mutations (deducting `available_seats` from the parent `Event` object) entirely inside the `create()` layer method within the `ReservationSerializer`, rather than separating them across decoupled view layers or post-save signal hooks.
