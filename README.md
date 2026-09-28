# RideLink Backend Microservices

**IT3130 – Application Development Group Assignment**  
Zero-to-Submission Implementation for **Account Service** and **Driver & Vehicle Service**.

---

## 1. Project Overview

RideLink is a backend-only microservices architecture providing ride-hailing core functionalities.  
This phase implements:
1. **Account Service (`port 8081`)**: User identity, passenger/driver registration, BCrypt password hashing, JWT authentication, role-based authorization, profile management, and account status administration.
2. **Driver & Vehicle Service (`port 8082`)**: Driver operational profiles, fleet vehicle management, availability status machine, simulated GPS location updates, and querying eligible available drivers for ride matching.

Both services maintain **independent databases** with zero direct cross-service database access.

---

## 2. Prerequisites

- **Java**: JDK 17 (or Java 21)
- **Build Tool**: Apache Maven 3.9+
- **Database**: MySQL Server 8.x (running on port `3306`)
- **Testing**: Postman, Browser (for Swagger UI)

---

## 3. Repository Structure

```
Ridelink_Backend/
├── account-service/
│   ├── src/main/java/com/ridelink/account/
│   │   ├── controller/      # AuthController, UserController
│   │   ├── service/         # AuthService, UserService
│   │   ├── repository/      # UserRepository
│   │   ├── entity/          # User, Role, AccountStatus
│   │   ├── dto/             # Request & Response DTOs, ErrorResponse
│   │   ├── security/        # JwtService, JwtAuthenticationFilter, SecurityConfig, UserPrincipal
│   │   ├── exception/       # GlobalExceptionHandler & custom exceptions
│   │   └── config/          # OpenApiConfig
│   └── src/main/resources/application.properties
│
├── driver-vehicle-service/
│   ├── src/main/java/com/ridelink/driver/
│   │   ├── controller/      # DriverController, VehicleController
│   │   ├── service/         # DriverService, VehicleService
│   │   ├── repository/      # DriverRepository, VehicleRepository
│   │   ├── entity/          # Driver, Vehicle, AvailabilityStatus, DriverStatus, VehicleType
│   │   ├── dto/             # Driver and Vehicle DTOs, ErrorResponse
│   │   ├── security/        # JwtService, JwtAuthenticationFilter, SecurityConfig, UserPrincipal
│   │   ├── exception/       # GlobalExceptionHandler & custom exceptions
│   │   └── config/          # OpenApiConfig
│   └── src/main/resources/application.properties
│
├── postman/
│   ├── RideLink_Account_Service.postman_collection.json
│   └── RideLink_Driver_Vehicle_Service.postman_collection.json
│
└── README.md
```

---

## 4. Database Setup

Log into MySQL and execute:

```sql
CREATE DATABASE IF NOT EXISTS ridelink_account;
CREATE DATABASE IF NOT EXISTS ridelink_driver;
```

---

## 5. Environment Variables & Configuration

Each service can be configured via environment variables or `application.properties`:

| Variable | Default Value | Description |
| :--- | :--- | :--- |
| `DB_URL` (Account) | `jdbc:mysql://localhost:3306/ridelink_account` | Account DB connection URL |
| `DB_URL` (Driver) | `jdbc:mysql://localhost:3306/ridelink_driver` | Driver DB connection URL |
| `DB_USERNAME` | `root` | MySQL username |
| `DB_PASSWORD` | `mumthas` | MySQL password |
| `JWT_SECRET` | `404E635266556A586E3272357538782F413F4428472B4B6250645367566B5970` | 256-bit secure secret key |
| `JWT_EXPIRATION_MS` | `3600000` (1 hour) | Token validity duration |

---

## 6. How to Build & Run

### A. Run Automated Unit & Integration Tests
```powershell
# Account Service (28 tests)
cd d:\3.1\Ridelink_Backend\account-service
mvn test

# Driver & Vehicle Service (20 tests)
cd d:\3.1\Ridelink_Backend\driver-vehicle-service
mvn test
```

### B. Start Services

Open two terminal windows:

**Terminal 1 — Account Service (Port 8081):**
```powershell
cd d:\3.1\Ridelink_Backend\account-service
mvn spring-boot:run
```

**Terminal 2 — Driver & Vehicle Service (Port 8082):**
```powershell
cd d:\3.1\Ridelink_Backend\driver-vehicle-service
mvn spring-boot:run
```

---

## 7. Interactive API Documentation (Swagger UI)

- **Account Service Swagger UI**: [http://localhost:8081/swagger-ui.html](http://localhost:8081/swagger-ui.html)  
  *(OpenAPI Spec: [http://localhost:8081/v3/api-docs](http://localhost:8081/v3/api-docs))*
- **Driver & Vehicle Service Swagger UI**: [http://localhost:8082/swagger-ui.html](http://localhost:8082/swagger-ui.html)  
  *(OpenAPI Spec: [http://localhost:8082/v3/api-docs](http://localhost:8082/v3/api-docs))*

---

## 8. Sample Test Data & End-to-End Workflow

1. **Register Driver**:
   - `POST http://localhost:8081/api/auth/register`
   - Body: `{"firstName":"Kasun", "lastName":"Perera", "email":"driver@ridelink.com", "password":"DriverPassword123!", "phone":"+94771234567", "role":"DRIVER"}`
2. **Login Driver**:
   - `POST http://localhost:8081/api/auth/login`
   - Body: `{"email":"driver@ridelink.com", "password":"DriverPassword123!"}`
   - Save the returned `token` and `userId`.
3. **Create Driver Profile**:
   - `POST http://localhost:8082/api/drivers`
   - Header: `Authorization: Bearer <token>`
   - Body: `{"userId": <userId>, "licenseNumber":"B1234567", "serviceArea":"Colombo"}`
4. **Add Vehicle**:
   - `POST http://localhost:8082/api/drivers/<driverId>/vehicles`
   - Header: `Authorization: Bearer <token>`
   - Body: `{"vehicleNumber":"WP-CAB-1234", "vehicleType":"CAR", "model":"Toyota Prius", "color":"White", "year":2022}`
5. **Update Location**:
   - `PATCH http://localhost:8082/api/drivers/<driverId>/location`
   - Body: `{"latitude": 6.9271, "longitude": 79.8612}`
6. **Set Available**:
   - `PATCH http://localhost:8082/api/drivers/<driverId>/availability`
   - Body: `{"availability":"AVAILABLE"}`
7. **Find Eligible Drivers (Used by Ride Management)**:
   - `GET http://localhost:8082/api/drivers/available?area=Colombo`

---

## 9. Postman Collections

Import both JSON collections from the `postman/` directory:
1. `postman/RideLink_Account_Service.postman_collection.json`
2. `postman/RideLink_Driver_Vehicle_Service.postman_collection.json`
