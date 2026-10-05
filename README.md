# Employee Directory REST API

A RESTful API for creating and managing employee records. Built with Java 17, Spring Boot 3, Spring Data JPA, and MySQL, the application provides CRUD endpoints backed by a relational database.

## Tech stack

- **Java 17**
- **Spring Boot 3.3.5**
- **Spring Web** for REST endpoints
- **Spring Data JPA** and **Hibernate** for persistence and ORM
- **MySQL** for the database
- **Maven** for dependency management and builds

## Architecture

The application uses a layered structure to keep HTTP handling, business logic, and persistence responsibilities separate:

| Layer | Responsibility |
| --- | --- |
| Controller | Exposes `/api/employees` endpoints and HTTP response codes. |
| Service | Implements employee CRUD operations and missing-resource checks. |
| Repository | Uses Spring Data JPA to access employee records. |
| Entity | Maps the `Employee` model to the `employees` database table. |
| Exception handling | `ResourceNotFoundException` maps missing employees to HTTP `404 Not Found`. |

## Prerequisites

- JDK 17 or later
- MySQL Server running on `localhost:3306`
- Maven 3.6 or later
- Visual Studio Code (optional)

## Setup

### 1. Create the database

Connect to MySQL and create the application database:

```sql
CREATE DATABASE employee_db;
```

### 2. Configure the database connection

The connection is configured in `src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/employee_db?useSSL=false&serverTimezone=UTC
spring.datasource.username=${DB_USERNAME:root}
spring.datasource.password=${DB_PASSWORD:}
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

Set `DB_USERNAME` and `DB_PASSWORD` to match your MySQL account. If unset, the application uses `root` with an empty password. `ddl-auto=update` creates or updates the table schema at startup, and SQL statements are printed to the console.

For example, in PowerShell:

```powershell
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = "your-mysql-password"
```

### 3. Run the application

From the project root in a terminal:

```powershell
mvn spring-boot:run
```

Or open the project folder in VS Code, install the **Extension Pack for Java** if needed, open `src/main/java/com/example/employeedirectory/EmployeeDirectoryApplication.java`, and use the **Run** action above the `main` method. Make sure VS Code uses JDK 17 or later and has the database environment variables configured in its terminal.

The API is available at `http://localhost:8080`.

## API endpoints

All endpoints use the base URL `http://localhost:8080/api/employees`. Employee request bodies use JSON with `firstName`, `lastName`, `email`, `department`, and `salary` fields.

| HTTP method | Endpoint URL | Description | Sample request body | Response status codes |
| --- | --- | --- | --- | --- |
| `POST` | `/api/employees` | Create an employee. | `{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com","department":"Engineering","salary":120000.0}` | `201 Created` |
| `GET` | `/api/employees` | Get all employees. | None | `200 OK` |
| `GET` | `/api/employees/{id}` | Get an employee by ID. | None | `200 OK`, `404 Not Found` |
| `PUT` | `/api/employees/{id}` | Replace an employee's details. | `{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com","department":"Research","salary":125000.0}` | `200 OK`, `404 Not Found` |
| `DELETE` | `/api/employees/{id}` | Delete an employee; returns a confirmation message. | None | `200 OK`, `404 Not Found` |

Example employee JSON:

```json
{
  "firstName": "Ada",
  "lastName": "Lovelace",
  "email": "ada@example.com",
  "department": "Engineering",
  "salary": 120000.0
}
```

## Test the API

Start the application first. The following PowerShell commands use `Invoke-RestMethod` and save the created employee ID for subsequent requests:

```powershell
$baseUrl = "http://localhost:8080/api/employees"
$employee = @{
    firstName = "Ada"
    lastName = "Lovelace"
    email = "ada@example.com"
    department = "Engineering"
    salary = 120000.0
} | ConvertTo-Json

# Create (201 Created)
$created = Invoke-RestMethod -Method Post -Uri $baseUrl -ContentType "application/json" -Body $employee
$id = $created.id

# List all employees (200 OK)
Invoke-RestMethod -Method Get -Uri $baseUrl

# Get one employee (200 OK, or 404 Not Found for an unknown ID)
Invoke-RestMethod -Method Get -Uri "$baseUrl/$id"

# Update employee details (200 OK, or 404 Not Found)
$updatedEmployee = @{
    firstName = "Ada"
    lastName = "Lovelace"
    email = "ada@example.com"
    department = "Research"
    salary = 125000.0
} | ConvertTo-Json
Invoke-RestMethod -Method Put -Uri "$baseUrl/$id" -ContentType "application/json" -Body $updatedEmployee

# Delete employee (200 OK, or 404 Not Found)
Invoke-RestMethod -Method Delete -Uri "$baseUrl/$id"
```

Equivalent `curl` examples (use `curl.exe` in PowerShell):

```powershell
curl.exe -i -X POST "http://localhost:8080/api/employees" -H "Content-Type: application/json" -d '{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com","department":"Engineering","salary":120000.0}'
curl.exe -i "http://localhost:8080/api/employees"
curl.exe -i "http://localhost:8080/api/employees/1"
curl.exe -i -X PUT "http://localhost:8080/api/employees/1" -H "Content-Type: application/json" -d '{"firstName":"Ada","lastName":"Lovelace","email":"ada@example.com","department":"Research","salary":125000.0}'
curl.exe -i -X DELETE "http://localhost:8080/api/employees/1"
```

Replace `1` in the cURL GET, PUT, and DELETE examples with an ID returned by the POST request.

## Build and test

Run the Maven test suite from the project root:

```powershell
mvn test
```
