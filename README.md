# Placement Portal

A full-stack **campus placement management system** that connects students, recruiters and the Training & Placement Office (TPO) on a single platform. Students browse and apply to jobs they are eligible for, companies post openings and shortlist candidates, and admins approve companies and track placement statistics.

> **Status:** 🚧 Under active development. Authentication and the core data model are done; the remaining modules are being built phase by phase (see the [Roadmap](#roadmap)).

---

## Features

### Student
- Register and log in securely
- Create and update a profile (branch, CGPA, backlogs, skills, graduation year)
- Browse jobs, apply, and track application status
- Upload a resume (PDF)

### Company / Recruiter
- Register and wait for admin approval
- Post, edit and delete job openings with eligibility criteria (minimum CGPA, allowed branches, maximum backlogs, deadline)
- View applicants for their own jobs and move them through the hiring pipeline

### Admin / TPO
- Approve or reject company registrations
- View all students and applications
- Placement statistics (overall, branch-wise, company-wise)

### Business rules
- A student can apply only if they meet the job's CGPA, branch and backlog criteria
- No duplicate applications and no applications after the deadline
- Only approved companies can post jobs
- A company can only see applicants of its own jobs
- Application status follows a defined flow: `APPLIED → SHORTLISTED → INTERVIEW → SELECTED / REJECTED`

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Java 24, Spring Boot 4, Spring Web MVC, Spring Data JPA (Hibernate) |
| Security | Spring Security, JWT (jjwt), BCrypt password hashing, role-based access control |
| Database | MySQL 8 |
| Frontend | React (Vite), React Router, Axios |
| Build tools | Maven, npm |
| Others | Lombok, Bean Validation |

---

## Architecture

The backend follows a layered architecture. Controllers never touch repositories directly, and entities are never returned from the API (DTOs are used instead).

```
Controller  →  Service  →  Repository  →  MySQL
   (REST)      (business     (Spring Data
                rules)          JPA)
```

### Project structure

```
placement-portal/
├── backend/
│   └── src/main/java/org/satyam/placementportal/
│       ├── config/        Security and CORS configuration
│       ├── controller/    REST endpoints
│       ├── service/       Business logic
│       ├── repository/    Spring Data JPA repositories
│       ├── entity/        JPA entities
│       ├── dto/           Request / response objects
│       ├── security/      JWT utility, filter, user details service
│       ├── exception/     Custom exceptions and global handler
│       └── enums/         Role, ApplicationStatus, JobType
└── frontend/
    └── src/
        ├── api/           Axios instance and API calls
        ├── components/    Reusable UI components
        ├── context/       Auth context
        ├── pages/         auth, student, company, admin pages
        ├── routes/        Route definitions and protected routes
        └── utils/
```

### Data model

| Entity | Description |
|---|---|
| `User` | Login account (email, hashed password, role) |
| `StudentProfile` | Student details, one-to-one with `User` |
| `Company` | Company details and approval flag, one-to-one with `User` |
| `JobPosting` | Job details and eligibility criteria, belongs to a `Company` |
| `Application` | A student's application to a job with status; unique per (student, job) |

---

## API Overview

Base URL: `http://localhost:8081/api`

| Area | Endpoint | Access | Status |
|---|---|---|---|
| Auth | `POST /auth/register/student` | Public | ✅ Done |
| Auth | `POST /auth/register/company` | Public | ✅ Done |
| Auth | `POST /auth/login` | Public | ✅ Done |
| Student | `GET, PUT /student/profile` | Student | 🚧 In progress |
| Company | `GET, PUT /company/profile` | Company | 🚧 In progress |
| Jobs | `GET /jobs`, `GET /jobs/{id}` | Authenticated | 📋 Planned |
| Jobs | `POST /jobs/{id}/apply` | Student | 📋 Planned |
| Company | `POST, PUT, DELETE /company/jobs` | Approved company | 📋 Planned |
| Company | `GET /company/jobs/{id}/applicants` | Company | 📋 Planned |
| Company | `PATCH /company/applications/{id}/status` | Company | 📋 Planned |
| Admin | company approvals, students, applications, stats | Admin | 📋 Planned |

Protected endpoints expect a JWT in the header: `Authorization: Bearer <token>`.

---

## Getting Started

### Prerequisites
- JDK 24 (or the version set in `backend/pom.xml`)
- MySQL 8
- Node.js (LTS) and npm

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/placement-portal.git
cd placement-portal
```

### 2. Run the backend
Set your credentials as environment variables (do not commit them):

```bash
# Windows (PowerShell)
$env:DB_PASSWORD="your_mysql_password"
$env:JWT_SECRET="a-long-random-string-of-at-least-32-characters"

# macOS / Linux
export DB_PASSWORD=your_mysql_password
export JWT_SECRET=a-long-random-string-of-at-least-32-characters
```

Then start the app:

```bash
cd backend
./mvnw spring-boot:run        # Windows: .\mvnw.cmd spring-boot:run
```

The database `placement_portal` and all tables are created automatically on first run. The API starts on `http://localhost:8081`.

Key settings in `backend/src/main/resources/application.properties`:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/placement_portal?createDatabaseIfNotExist=true
spring.datasource.username=root
spring.datasource.password=${DB_PASSWORD}
spring.jpa.hibernate.ddl-auto=update
server.port=8081
app.jwt.secret=${JWT_SECRET}
app.jwt.expiration-ms=86400000
```

### 3. Run the frontend
```bash
cd frontend
npm install
```

Create a `frontend/.env` file:

```
VITE_API_URL=http://localhost:8081/api
```

```bash
npm run dev
```

The app opens on `http://localhost:5173`.

---

## Quick API Test

Register a student:

```http
POST /api/auth/register/student
Content-Type: application/json

{
  "email": "student1@test.com",
  "password": "pass1234",
  "name": "Student One",
  "rollNo": "CS001",
  "branch": "CSE",
  "cgpa": 8.2,
  "backlogs": 0,
  "phone": "9999999999",
  "graduationYear": 2027
}
```

Response (`201 Created`):

```json
{ "token": "<jwt>", "role": "STUDENT", "email": "student1@test.com" }
```

---

## Security

- Passwords are hashed with BCrypt and never stored or returned in plain text
- Stateless authentication with signed JWTs
- Role-based access (`STUDENT`, `COMPANY`, `ADMIN`) enforced on API routes
- Request validation with Bean Validation and consistent JSON error responses
- Secrets are read from environment variables, not from source code

---

## Roadmap

- [x] Project setup and layered package structure
- [x] Database design and JPA entities
- [x] Authentication: registration, login, JWT, role-based security
- [x] Global exception handling and validation
- [ ] Student and company profile management
- [ ] Job posting CRUD with eligibility fields
- [ ] Job browsing with filters and pagination
- [ ] Apply flow with eligibility checks and status updates
- [ ] Admin: company approval, student and application views, statistics
- [ ] Resume upload (PDF validation, safe file storage)
- [ ] React frontend: auth, dashboards, protected routes
- [ ] Unit and integration tests, Swagger / OpenAPI docs
- [ ] Email notifications, Docker Compose setup, CSV export

---

## Author

**Satyam**

If you find this project useful, consider giving it a ⭐.
