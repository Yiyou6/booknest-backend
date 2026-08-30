# StayBooking

StayBooking is a backend service for a short-term stay / rental booking platform (think Airbnb). Hosts publish listings with photos and a location; guests search for nearby places within a given radius and book them for a date range.

The service is built with **Spring Boot 4**, **Java 21**, **PostgreSQL + PostGIS**, and **JWT-based authentication**.

---

## Features

- **Authentication & authorization** — register and log in with JWT. Two roles: `ROLE_HOST` and `ROLE_GUEST`.
- **Listing management** — hosts create, list, and delete their own listings.
- **Image storage** — listing photos are uploaded to Google Cloud Storage.
- **Geocoding** — a street address is converted to a latitude/longitude via the Google Maps Geocoding API.
- **Location-based search** — guests search by center point + radius (in meters), check-in/check-out dates, and guest count using PostGIS geospatial queries. Results automatically exclude listings that already have overlapping bookings.
- **Booking management** — guests create/list/delete their bookings; hosts view bookings for their listings. Overlapping-booking conflicts are rejected.

---

## Tech Stack

| Area            | Technology                                                              |
| --------------- | ----------------------------------------------------------------------- |
| Language        | Java 21                                                                 |
| Framework       | Spring Boot 4.1.0 (Web MVC, Data JPA, Security)                         |
| Build           | Gradle                                                                  |
| Database        | PostgreSQL + PostGIS (geospatial extension)                             |
| ORM / Spatial   | Hibernate ORM + Hibernate Spatial + JTS                                |
| Auth            | Spring Security + JJWT (JWT)                                            |
| Storage         | Google Cloud Storage                                                    |
| Geocoding       | Google Maps Services (Geocoding API)                                    |

---

## Prerequisites

- JDK 21
- Docker & Docker Compose (for the local PostGIS database)
- A Google Cloud Platform project with:
  - A Cloud Storage bucket (for listing images)
  - A service account key file (`credentials.json`)
  - A Geocoding API key

---

## Getting Started

### 1. Start the database

A PostGIS-enabled PostgreSQL container is provided via Docker Compose:

```bash
docker compose up -d
```

This starts PostgreSQL on `localhost:5432` with database `postgres` and password `secret`.

### 2. Configure secrets

The application reads its external configuration from environment variables. Set the following:

| Variable             | Description                                          |
| -------------------- | ---------------------------------------------------- |
| `DATABASE_URL`       | Database host (default `localhost`)                  |
| `DATABASE_PORT`      | Database port (default `5432`)                       |
| `DATABASE_USERNAME`  | Database user (default `postgres`)                   |
| `DATABASE_PASSWORD`  | Database password (default `secret`)                 |
| `GCS_BUCKET`         | Google Cloud Storage bucket name for listing images  |
| `GEOCODING_KEY`      | Google Maps Geocoding API key                        |
| `JWT_SECRET_KEY`     | Secret key used to sign JWTs                         |

Additionally, place your GCP service account key at:

```
src/main/resources/credentials.json
```

### 3. Run the application

```bash
./gradlew bootRun
```

The service starts on `http://localhost:8080`.

> **Note:** On startup, `DevRunner` seeds the database with sample users, listings, and bookings. JPA is configured with `ddl-auto: create-drop`, so schema and data are recreated on every restart (suitable for development only).

---

## API Overview

All endpoints (except authentication) require a `Bearer` token in the `Authorization` header.

### Authentication

| Method | Endpoint         | Role   | Description             |
| ------ | ---------------- | ------ | ----------------------- |
| POST   | `/auth/register` | Public | Register a new user     |
| POST   | `/auth/login`    | Public | Log in, returns a JWT   |

### Listings (host)

| Method | Endpoint                        | Role       | Description                          |
| ------ | ------------------------------- | ---------- | ------------------------------------ |
| GET    | `/listings`                     | `ROLE_HOST` | List the host's own listings         |
| POST   | `/listings`                     | `ROLE_HOST` | Create a listing (multipart form)    |
| DELETE | `/listings/{listingId}`         | `ROLE_HOST` | Delete a listing                     |
| GET    | `/listings/{listingId}/bookings`| `ROLE_HOST` | List bookings for a listing          |

### Search (guest)

| Method | Endpoint          | Role        | Description                            |
| ------ | ----------------- | ----------- | -------------------------------------- |
| GET    | `/listings/search`| `ROLE_GUEST` | Search listings by location and dates  |

Query parameters: `lat`, `lon`, `checkin_date`, `checkout_date`, `guest_number`, and optional `distance` (meters, defaults to `500000`).

### Bookings (guest)

| Method | Endpoint             | Role        | Description                    |
| ------ | -------------------- | ----------- | ------------------------------ |
| GET    | `/bookings`          | `ROLE_GUEST` | List the guest's bookings      |
| POST   | `/bookings`          | `ROLE_GUEST` | Create a booking               |
| DELETE | `/bookings/{bookingId}` | `ROLE_GUEST` | Cancel a booking               |

---

## Project Structure

```
src/main/java/com/laioffer/staybooking/
├── authentication/   # Register / login service & controller
├── booking/          # Booking service, controller, exceptions
├── listing/          # Listing service, controller, exceptions
├── location/         # Geocoding (address → lat/lng)
├── model/            # JPA entities, DTOs, request/response records
├── repository/       # Spring Data JPA repositories
├── security/         # JWT filter, handler, UserDetailsService
├── storage/          # Google Cloud Storage image upload
├── AppConfig.java    # Security chain, beans, GCS/geocoding config
├── DevRunner.java    # Seeds sample data on startup
└── StaybookingApplication.java
```

---

## Configuration Reference

Key settings live in `src/main/resources/application.yaml`:

- `spring.datasource.*` — PostgreSQL connection (driven by environment variables).
- `spring.jpa.hibernate.ddl-auto: create-drop` — recreate schema on each run (dev only).
- `spring.servlet.multipart.max-file-size: 10MB` — image upload limit.
- `spring.sql.init.schema-locations: postgis_extension.sql` — enables the PostGIS extension at startup.

---

## Tests

```bash
./gradlew test
```

---

## License

This project is provided for educational purposes.
