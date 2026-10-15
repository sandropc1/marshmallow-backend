# MarshMallowAPI Backend

A REST API for a simple order-management domain (users, products, categories, orders, order items and payments), built with Spring Boot and Spring Data JPA. It runs on an in-memory H2 database that is seeded with sample data at startup, and allows CORS from a local Vite frontend (`http://localhost:5173`).

## Tech stack

- Java 21
- Spring Boot 4.0 (Web MVC, Data JPA)
- H2 in-memory database with H2 console (PostgreSQL driver is on the classpath but not configured)
- Maven (wrapper included)

## Domain model

| Entity | Table | Relationships |
|---|---|---|
| `User` | `tb_user` | One user has many orders (as client) |
| `Order` | `tb_order` | Belongs to a user; has many order items; optional one-to-one payment. Exposes a computed `total` |
| `OrderItem` | `tb_order_item` | Composite key (order + product) via `OrderItemPK`; `quantity`, `price`, computed `subTotal` |
| `Product` | `tb_product` | Many-to-many with categories (`tb_product_category`) |
| `Category` | `tb_category` | Many-to-many with products |
| `Payment` | `tb_payment` | One-to-one with order, sharing its id (`@MapsId`) |

`OrderStatus` values: `WAITING_PAYMENT`, `PAID`, `SHIPPED`, `DELIVERED`, `CANCELED` (stored as an integer code).

## API endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/users` | List users |
| GET | `/users/{id}` | Get a user |
| POST | `/users` | Create a user (returns `201` with `Location` header) |
| PUT | `/users/{id}` | Update a user's name, email and phone |
| DELETE | `/users/{id}` | Delete a user (`204`) |
| GET | `/products` | List products |
| GET | `/products/{id}` | Get a product |
| GET | `/categories` | List categories |
| GET | `/categories/{id}` | Get a category |
| GET | `/orders` | List orders |
| GET | `/orders/{id}` | Get an order (with items, payment and total) |

### Error handling

User operations translate failures into a JSON `StandardError` body (`timestamp`, `status`, `error`, `message`, `path`):

- `ResourceNotFoundException` -> `404 Resource not found`
- `DatabaseException` (data integrity violation on delete) -> `400 Database error`

## Project structure

```
src/main/java/com/sandropc1/webservicesjava/
├── config/          # CorsConfig, TestConfig (seeds sample data under the "test" profile)
├── entities/        # JPA entities, enums/OrderStatus, pk/OrderItemPK
├── repositories/    # Spring Data JPA repositories
├── services/        # business logic + custom exceptions
└── resources/       # REST controllers + exception handler
```

## Getting started

### Prerequisites

- JDK 21

### Run

```bash
./mvnw spring-boot:run        # Windows: mvnw.cmd spring-boot:run
```

The app starts on `http://localhost:8080`. The `test` profile is active by default (`application.properties`), which configures the H2 in-memory database (`application-test.properties`) and runs `TestConfig` to seed sample users, products, categories, orders and a payment.

The H2 console is available at `http://localhost:8080/h2-console` (JDBC URL `jdbc:h2:mem:testdb`, user `sa`, empty password). No environment variables are required.

## Running tests

```bash
./mvnw test
```

The test suite currently contains a single Spring context-load test.
