# EducPlus Backend

Backend API for **EduPlus**, an educational platform backend built with Go.

The application provides authentication, event management, email subscriptions, email notifications, and PostgreSQL persistence through GORM.

## Features

* JWT-based authentication
* User authentication with bcrypt password verification
* Access and refresh tokens
* Event management
* Email subscriptions
* Email notifications for events
* PostgreSQL database integration
* GORM-based database access
* REST API built with Gin
* Protected API routes
* HTML 404 error page

## Tech Stack

| Technology                                | Purpose                      |
| ----------------------------------------- | ---------------------------- |
| [Go](https://go.dev/)                     | Backend programming language |
| [Gin](https://gin-gonic.com/)             | HTTP web framework           |
| [GORM](https://gorm.io/)                  | ORM and database access      |
| [PostgreSQL](https://www.postgresql.org/) | Relational database          |
| JWT                                       | Authentication               |
| bcrypt                                    | Password verification        |
| SMTP                                      | Email delivery               |
| Go Mail                                   | Email message construction   |
| Resend                                    | Email-related dependency     |

## Project Structure

```text
educPlus-backend/
├── database/
│   └── ...                 # Database access and configuration
│
├── mailsender/
│   └── ...                 # Email sending functionality
│
├── models/
│   └── models.go           # Application data models
│
├── server/
│   └── server.go           # HTTP server, routes and authentication
│
├── templates/
│   └── ...                 # HTML templates
│
├── educplus.sql            # Database initialization script
├── go.mod                  # Go module definition
├── go.sum                  # Dependency checksums
├── main.go                 # Application entry point
└── .gitignore
```

## Architecture

The backend is organized around the HTTP server, database layer, models, and email sender.

```text
                    ┌─────────────────┐
                    │     Client      │
                    └────────┬────────┘
                             │
                             │ HTTP
                             ▼
                    ┌─────────────────┐
                    │      Gin        │
                    │   HTTP Server   │
                    └────────┬────────┘
                             │
                ┌────────────┼────────────┐
                │            │            │
                ▼            ▼            ▼
          Authentication   Events     Subscription
                │            │            │
                └────────────┼────────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │      GORM       │
                    │  Database Layer │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   PostgreSQL    │
                    └─────────────────┘

                             │
                             ▼
                    ┌─────────────────┐
                    │  Mail Sender    │
                    │      SMTP       │
                    └─────────────────┘
```

The application starts from `main.go`, which starts the HTTP server through the server package.

## Authentication

EduPlus uses JWT tokens for authentication.

After successful authentication, the backend generates:

* An access token
* A refresh token

The access token is used to access protected routes, while the refresh token can be used to obtain a new access token.

### Authentication Flow

```text
Client
  │
  │ POST /api/login
  ▼
Backend
  │
  ├── Validate credentials
  │
  ├── Verify password
  │
  └── Generate JWT tokens
       │
       ├── accessToken
       └── refreshToken
```

Passwords are verified using bcrypt.

## API Endpoints

### Authentication

#### `POST /api/login`

Authenticates a user and returns JWT tokens.

Example request:

```json
{
  "username": "username",
  "password": "password"
}
```

Example response:

```json
{
  "accessToken": "<access-token>",
  "refreshToken": "<refresh-token>"
}
```

### `GET /api/newToken/:refresh`

Generates a new access token using a refresh token.

Example:

```text
GET /api/newToken/<refresh-token>
```

Example response:

```json
{
  "new_access_token": "<new-access-token>"
}
```

### `GET /api/login`

Validates the JWT supplied in the request and returns the authenticated username.

The request requires an authorization header:

```http
Authorization: Bearer <access-token>
```

### Events

#### `POST /api/newEvent`

Creates an event and triggers the email notification workflow.

The endpoint receives event information including:

* `title`
* `description`

### `GET /api/events`

Retrieves events from the database.

### Email Subscription

#### `POST /api/subscription`

Registers an email address for event notifications.

Example request:

```json
{
  "email": "user@example.com"
}
```

Example response:

```json
{
  "hello": "subscription successful"
}
```

### Protected Route

#### `GET /protected`

Protected endpoint requiring a valid JWT access token.

Example request:

```http
Authorization: Bearer <access-token>
```

## Data Models

The application defines several GORM models.

### User

```text
User
├── ID
├── CreatedAt
├── UpdatedAt
├── DeletedAt
├── Username
└── Password
```

### Event

```text
Event
├── ID
├── CreatedAt
├── UpdatedAt
├── DeletedAt
├── Title
├── Description
├── EventType
├── StartDate
├── EndDate
└── ImagePath
```

### Mail

```text
Mail
├── ID
├── CreatedAt
├── UpdatedAt
├── DeletedAt
└── EmailAddress
```

### Content

```text
Content
├── ID
├── CreatedAt
├── UpdatedAt
├── DeletedAt
└── Path
```

## Database

The backend uses PostgreSQL through GORM.

The repository contains an `educplus.sql` file for database initialization.

The database includes tables related to events and administrators, while the application also defines GORM models for users, events, mail subscriptions, and content.

## Getting Started

### Prerequisites

Install:

* Go
* PostgreSQL
* Git

Check the Go installation:

```bash
go version
```

### Clone the repository

```bash
git clone https://github.com/mandrindraa/educPlus-backend.git
cd educPlus-backend
```

### Install dependencies

```bash
go mod download
```

### Create the database

Create a PostgreSQL database named `educplus`:

```sql
CREATE DATABASE educplus;
```

The repository also provides an SQL initialization file:

```bash
psql -U postgres -f educplus.sql
```

### Run the application

```bash
go run .
```

The server runs on:

```text
http://localhost:8080
```

## Testing the API

The API can be tested with tools such as cURL, Postman, or Insomnia.

Example login request:

```bash
curl -X POST http://localhost:8080/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "password"
  }'
```

Example protected request:

```bash
curl http://localhost:8080/protected \
  -H "Authorization: Bearer <access-token>"
```

## Email Notifications

The `mailsender` package handles email notifications.

The backend retrieves subscribed email addresses and sends event notifications through SMTP.

```text
New Event
    │
    ▼
Retrieve subscribed emails
    │
    ▼
Build email
    │
    ▼
SMTP server
    │
    ▼
Subscribers
```

## License

No license is currently specified in the repository.

## Author

**Mandrindra Antonnio**

GitHub: [@mandrindraa](https://github.com/mandrindraa)

---

Built with Go, Gin, GORM and PostgreSQL.
