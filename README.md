# 🏡 StayBooking

**Your very own Airbnb — starting from the backend.** StayBooking is the server-side engine behind a short-stay rental platform: hosts list their places with photos and a pin on the map, guests find a great spot nearby, pick their dates, and book it.

Picture this: a tree house by the Golden Gate Bridge, a beachfront villa, a cozy mountain retreat — all living in a database, matched automatically to travelers at the right time and the right distance. That's what StayBooking does.

Built on **Spring Boot 4**, **Java 21**, **PostgreSQL / PostGIS**, and **JWT authentication**.

---

## ✨ What it does

- 🔐 **Register & log in, done** — stateless JWT auth with two roles: `ROLE_HOST` (landlord) and `ROLE_GUEST` (traveler).
- 🏠 **Listings on your terms** — hosts create, view, and delete their own places.
- 🖼️ **Photos in the cloud** — listing images go straight to Google Cloud Storage with UUID names, publicly readable.
- 📍 **Address → coordinates** — type a street address, Google Maps Geocoding turns it into lat/lng.
- 🔎 **Search nearby** — guests draw a circle (center + radius in meters), add check-in/check-out dates and headcount, and PostGIS does the spatial heavy lifting. **Listings with date clashes quietly disappear** from results, so you never chase a place that's already taken.
- 📅 **Bookings without the drama** — guests book, cancel, and track their trips; hosts see bookings for their listings. **Overlapping dates are rejected on the spot**, so a room never gets double-sold.

---

## 🧰 Tech stack

| Area        | Technology                                                       |
| ----------- | ---------------------------------------------------------------- |
| Language    | Java 21                                                          |
| Framework   | Spring Boot 4.1.0 (Web MVC / Data JPA / Security)                |
| Build       | Gradle                                                           |
| Database    | PostgreSQL + PostGIS (geospatial extension)                      |
| ORM / GIS   | Hibernate ORM + Hibernate Spatial + JTS                          |
| Auth        | Spring Security + JJWT (JWT)                                     |
| Storage     | Google Cloud Storage                                             |
| Geocoding   | Google Maps Services (Geocoding API)                             |

---

## 🧩 Prerequisites

- JDK 21
- Docker & Docker Compose (for the local PostGIS database)
- A Google Cloud Platform project with:
  - A Cloud Storage bucket (for listing photos)
  - A service account key file (`credentials.json`)
  - A Geocoding API key

---

## 🚀 Getting started

### 1. Fire up the database

One command and PostGIS-flavored PostgreSQL is up:

```bash
docker compose up -d
```

The database runs on `localhost:5432`, database `postgres`, password `secret`.

### 2. Configure your secrets

The app reads external config from environment variables. Fill these in:

| Variable             | Description                                          |
| -------------------- | ---------------------------------------------------- |
| `DATABASE_URL`       | Database host (default `localhost`)                  |
| `DATABASE_PORT`      | Database port (default `5432`)                       |
| `DATABASE_USERNAME`  | Database user (default `postgres`)                   |
| `DATABASE_PASSWORD`  | Database password (default `secret`)                 |
| `GCS_BUCKET`         | GCS bucket name for listing photos                   |
| `GEOCODING_KEY`      | Google Maps Geocoding API key                        |
| `JWT_SECRET_KEY`     | Secret key used to sign JWTs                         |

Also drop your GCP service account key at:

```
src/main/resources/credentials.json
```

### 3. Run it

```bash
./gradlew bootRun
```

The service wakes up at `http://localhost:8080`.

> 💡 **Heads up:** on startup, `DevRunner` seeds a bunch of sample users, listings, and bookings so you can play right away. JPA is set to `ddl-auto: create-drop`, so the schema is rebuilt on every restart (development only).

---

## 📡 API overview

Every endpoint except authentication requires a `Bearer` token in the `Authorization` header.

### Authentication

| Method | Endpoint         | Access | Description            |
| ------ | ---------------- | ------ | ---------------------- |
| POST   | `/auth/register` | Public | Register a new user    |
| POST   | `/auth/login`    | Public | Log in, returns a JWT  |

### Listings (host)

| Method | Endpoint                        | Access      | Description                    |
| ------ | ------------------------------- | ----------- | ------------------------------ |
| GET    | `/listings`                     | `ROLE_HOST` | List the host's own listings   |
| POST   | `/listings`                     | `ROLE_HOST` | Create a listing (multipart)   |
| DELETE | `/listings/{listingId}`         | `ROLE_HOST` | Delete a listing               |
| GET    | `/listings/{listingId}/bookings`| `ROLE_HOST` | List bookings for a listing    |

### Search (guest)

| Method | Endpoint           | Access       | Description                     |
| ------ | ------------------ | ------------ | ------------------------------- |
| GET    | `/listings/search` | `ROLE_GUEST` | Search listings by location & dates |

Query params: `lat`, `lon`, `checkin_date`, `checkout_date`, `guest_number`, and optional `distance` (meters, defaults to `500000`).

### Bookings (guest)

| Method | Endpoint                   | Access       | Description             |
| ------ | -------------------------- | ------------ | ----------------------- |
| GET    | `/bookings`                | `ROLE_GUEST` | List the guest's bookings |
| POST   | `/bookings`                | `ROLE_GUEST` | Create a booking        |
| DELETE | `/bookings/{bookingId}`    | `ROLE_GUEST` | Cancel a booking        |

---

## 🗂️ Project structure

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

## ⚙️ Configuration cheat sheet

Core settings live in `src/main/resources/application.yaml`:

- `spring.datasource.*` — PostgreSQL connection (driven by environment variables).
- `spring.jpa.hibernate.ddl-auto: create-drop` — rebuild schema on every run (dev only).
- `spring.servlet.multipart.max-file-size: 10MB` — image upload limit.
- `spring.sql.init.schema-locations: postgis_extension.sql` — enables the PostGIS extension at startup.

---

## 🧪 Tests

```bash
./gradlew test
```

---

## 📄 License

This project is for educational purposes.
