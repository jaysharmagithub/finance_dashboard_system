Overview
This project is a backend system designed to manage users, roles, and financial transactions. 
The goal was to build something that reflects real-world backend practices, with a focus on data integrity, security, and performance.
The system follows a layered architecture where controllers handle requests, services manage business logic, and repositories interact 
with the database. The API is designed to be predictable, secure, and scalable.


Setup and Running the Application
To run this project locally, make sure you have Java 17, Maven, and MySQL installed.
First, configure your database connection in application.yml or application.properties with your MySQL credentials.
Then run:
mvn clean install
mvn spring-boot:run
The application will start at:
http://localhost:8080
API documentation is available at:
http://localhost:8080/swagger-ui.html


Tech Stack
This project is built using a modern Java backend stack:
Java 17
Spring Boot 3.x
Spring Data JPA (Hibernate)
Spring Security
MySQL
Bean Validation (Jakarta Validation)
OpenAPI (springdoc)
Lombok
Testcontainers for integration testing


Project Structure
The project follows a layered structure to keep responsibilities clear:
controller handles HTTP requests and responses
service contains business logic
repository interacts with the database
dto defines request and response models
entity represents database tables
exception handles global error management
This separation helps keep the code clean and maintainable.


Authentication and Authorization
The application uses Spring Security for authentication and access control.
Users log in using their email and password, where email acts as the unique identifier. User details are loaded through a custom UserDetailsService.
Authorization is handled using roles such as ROLE_USER and ROLE_ADMIN. Endpoints are protected using role-based access rules, ensuring users can only 
access what they are allowed to.
The current implementation uses session-based authentication.


Validation Strategy
Input validation is handled using annotation-based validation.
Common validations include:
email format validation
password constraints
mobile number validation
required fields
This ensures invalid data is rejected before reaching business logic.


Database Design
The database follows a normalized relational structure to maintain consistency and scalability.
Users and roles have a many-to-many relationship, allowing flexible role-based access control. Transactions belong to a single user, 
ensuring clear ownership and data isolation.
Each transaction is linked to a category, which enables structured tracking and analytics.
Transactions use two identifiers: an internal database ID and a UUID-based identifier exposed externally. This avoids exposing internal database structure.
Idempotency is handled using a unique request key to prevent duplicate transactions.
All entities include auditing fields such as createdAt and updatedAt. A soft delete approach is used to preserve historical data.


API Overview
The API is divided into four main areas: users, transactions, roles, and dashboard analytics.
User APIs handle registration, updates, deletion, and retrieval of user data. Admin endpoints support paginated user listing.
Transaction APIs manage financial records, including creation, updates, and summaries like income, expenses, and balance.
Role APIs manage role creation, assignment, and removal.
Dashboard APIs provide aggregated insights such as totals, category breakdown, and growth metrics.
All APIs follow a consistent request and response structure.


API Versioning
APIs can be versioned using a pattern like /api/v1/... to support future changes without breaking existing clients.


Sample Requests and Responses
Login
Request
POST /api/auth/login

{
  "email": "john@example.com",
  "password": "password123"
}
Response
{
  "status": 200,
  "message": "Login successful",
  "data": "john@example.com",
  "timeStamp": "2026-04-06T17:33:13.514738600Z",
  "path": "/api/auth/login"
}

User APIs
Get User by Email
Request
GET /api/users/john@example.com
Response
{
  "status": 200,
  "message": "User details retrieved successfully",
  "data": {
    "firstName": "John",
    "lastName": "Doe",
    "mobileNo": "9876543210",
    "emailId": "john@example.com",
    "roles": [
      "ROLE_USER"
    ]
  },
  "timeStamp": "2026-04-06T17:38:21.851799200Z",
  "path": "/api/users/john@example.com"
}

Get All Users (Paginated)
Request
GET /api/users
Response
{
  "status": 201,
  "message": "All Transactions retrieved successfully",
  "data": {
    "content": [
      {
        "firstName": "Admin",
        "lastName": "User",
        "mobileNo": "9999999999",
        "emailId": "admin@finance.com",
        "roles": [
          "ROLE_ADMIN"
        ]
      },
      {
        "firstName": "John",
        "lastName": "Doe",
        "mobileNo": "9876543210",
        "emailId": "john@example.com",
        "roles": [
          "ROLE_USER"
        ]
      }
    ],
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 11,
    "totalPages": 2,
    "last": false
  },
  "timeStamp": "2026-04-06T17:42:23.182097100Z",
  "path": "/api/users"
}

Transaction APIs
Get All Transactions (Paginated)
Request
GET /transactions
Response
{
  "status": 201,
  "message": "All Transactions retrieved successfully",
  "data": {
    "content": [
      {
        "amount": 5000.0000,
        "description": "Monthly Salary",
        "username": "john@example.com",
        "type": "income",
        "category": "Salary"
      },
      {
        "amount": 5000.0000,
        "description": "Monthly Salary",
        "username": "john@example.com",
        "type": "income",
        "category": "Salary"
      }
    ],
    "pageNumber": 0,
    "pageSize": 10,
    "totalElements": 1279,
    "totalPages": 128,
    "last": false
  },
  "timeStamp": "2026-04-06T17:51:17.150068400Z",
  "path": "/transactions"
}

Dashboard APIs
Get Dashboard Summary
Request
GET /api/dashboard/summary
Response
{
  "status": 200,
  "message": "Financial summary retrieved successfully",
  "data": {
    "totalIncome": 0.0000,
    "totalExpenses": 0.0000,
    "netBalance": 0.0000,
    "topExpenseCategory": {
      "name": "No expenses yet",
      "total": 0
    },
    "monthlyGrowth": 0.0,
    "yearlyGrowth": 0.0,
    "categoryBreakdown": [],
    "recentTransactions": [],
    "deficit": false
  },
  "timeStamp": "2026-04-06T17:45:20.774820400Z",
  "path": "/api/dashboard/summary"
}


Service Layer Design
The service layer handles business logic and enforces rules.
When creating a transaction, the system first checks idempotency, then validates the user and category before saving. 
A UUID identifier is generated for external use.
Updates re-validate related entities instead of trusting incoming data.
Financial calculations are performed using database queries instead of loading data into memory, improving performance.
Pagination is applied to large datasets such as users and transactions.


Repository Design
The repository layer is optimized using custom queries where needed.
Aggregation operations like sum and grouping are performed directly in the database instead of Java memory. 
This improves performance and reduces unnecessary data loading.
Queries are designed to fetch only required data and map results directly into DTOs.


DTO and Response Design
DTOs are used to separate internal entities from external API contracts.
Request DTOs validate incoming data, and response DTOs shape output without exposing internal models.
All responses follow a consistent structure, including status, message, timestamp, and request path. 
Pagination responses also include metadata such as page number and total elements.


Exception Handling
All exceptions are handled centrally using a global exception handler.
Different error types are mapped to appropriate HTTP statuses:
404 for missing resources
400 for validation and business errors
409 for conflicts
403 for unauthorized access
500 for unexpected system errors
Internal errors are not exposed to clients, ensuring better security.


Logging Strategy
Logging is used to track system behavior and issues.
info for normal operations
warn for expected issues such as validation failures or duplicates
error for system-level failures
This helps with debugging and monitoring.


Performance Considerations
Performance is handled by pushing heavy operations to the database.
Aggregations are done using database queries
Pagination prevents large data loads
Lazy and eager loading are used based on use case
Key fields like email and userId are expected to be indexed


Testing
The project includes basic testing support.
Spring Boot testing framework is used
Testcontainers is used to run MySQL in isolation
Tests can be executed using:
mvn test


Key Design Decisions
Several important decisions were made:
Database-level aggregation instead of in-memory processing
UUID-based identifiers to avoid exposing database IDs
Idempotency to prevent duplicate transactions
Pagination for scalability
Centralized exception handling
DTO-based API design


Tradeoffs Considered
Using database queries improves performance but requires custom query writing.
UUIDs improve security but add slight complexity.
Session-based authentication is simpler but less scalable than token-based approaches.
Centralized exception handling keeps code clean but requires proper categorization.
Strict validation improves reliability but increases development effort.


Assumptions
Email uniquely identifies a user and is used for login.
Each transaction belongs to one user and must have a category.
Categories exist before transactions are created.
Users should not access other users’ data.
Duplicate requests may occur and must be handled safely.


Edge Cases Covered
Duplicate transaction requests are handled using idempotency keys.
Duplicate user registration is prevented using unique constraints.
Missing resources return proper error responses.
Empty results return safe defaults like zero.
Unauthorized access is restricted using roles.
Bulk operations are validated properly.


Limitations
The system uses session-based authentication, which may not scale well.
Caching is not implemented.
Rate limiting is not present.
No asynchronous processing for heavy operations.
Audit data is stored but not exposed through a UI.


Final Note
The focus of this design is to build a clean, secure, and scalable backend system. 
The goal was not just to make the application work, but to handle real-world concerns such as data consistency, duplicate requests, and access control in a structured way.
