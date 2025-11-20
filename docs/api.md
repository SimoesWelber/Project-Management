# API Documentation

## INCA REST API

Base URL: `http://localhost:3000/api/v1`

## Authentication

All authenticated endpoints require a JWT token in the Authorization header:

```
Authorization: Bearer <token>
```

### Login
```http
POST /api/v1/auth/login
Content-Type: application/json

{
  "username": "user@example.com",
  "password": "password"
}

Response:
{
  "success": true,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "uuid",
      "username": "user@example.com",
      "role": "SUBMITTER"
    }
  }
}
```

### Logout
```http
POST /api/v1/auth/logout
Authorization: Bearer <token>

Response:
{
  "success": true,
  "message": "Logged out successfully"
}
```

## Documents

### List Documents
```http
GET /api/v1/documents
Authorization: Bearer <token>

Query Parameters:
- status (optional): Filter by status
- page (optional): Page number (default: 1)
- limit (optional): Items per page (default: 20)

Response:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "Engineering Design Document",
      "status": "SUBMITTED",
      "submitter_id": "uuid",
      "created_at": "2025-11-20T10:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100
  }
}
```

### Get Document by ID
```http
GET /api/v1/documents/:id
Authorization: Bearer <token>

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Engineering Design Document",
    "description": "Detailed description",
    "document_type": "Engineering Design",
    "status": "SUBMITTED",
    "submitter": {
      "id": "uuid",
      "full_name": "John Doe"
    },
    "files": [
      {
        "id": "uuid",
        "filename": "design.pdf",
        "file_size": 1024000
      }
    ],
    "created_at": "2025-11-20T10:00:00Z",
    "updated_at": "2025-11-20T10:00:00Z"
  }
}
```

### Create Document
```http
POST /api/v1/documents
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "New Engineering Document",
  "description": "Document description",
  "document_type": "Engineering Design",
  "project_code": "LINE-6-STATION-A"
}

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "New Engineering Document",
    "status": "DRAFT",
    "created_at": "2025-11-20T10:00:00Z"
  }
}
```

### Update Document
```http
PUT /api/v1/documents/:id
Authorization: Bearer <token>
Content-Type: application/json

{
  "title": "Updated Title",
  "description": "Updated description"
}

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Updated Title",
    "updated_at": "2025-11-20T11:00:00Z"
  }
}
```

### Delete Document
```http
DELETE /api/v1/documents/:id
Authorization: Bearer <token>

Response:
{
  "success": true,
  "message": "Document deleted successfully"
}
```

### Upload File to Document
```http
POST /api/v1/documents/:id/files
Authorization: Bearer <token>
Content-Type: multipart/form-data

Form Data:
- file: <binary file>

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "filename": "design.pdf",
    "file_size": 1024000,
    "uploaded_at": "2025-11-20T10:00:00Z"
  }
}
```

## Reviews

### List Reviews
```http
GET /api/v1/reviews
Authorization: Bearer <token>

Query Parameters:
- document_id (optional): Filter by document
- status (optional): Filter by status

Response:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "document_id": "uuid",
      "reviewer": {
        "id": "uuid",
        "full_name": "Jane Reviewer"
      },
      "status": "IN_PROGRESS",
      "created_at": "2025-11-20T10:00:00Z"
    }
  ]
}
```

### Create Review
```http
POST /api/v1/reviews
Authorization: Bearer <token>
Content-Type: application/json

{
  "document_id": "uuid",
  "reviewer_id": "uuid"
}

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "document_id": "uuid",
    "status": "PENDING",
    "created_at": "2025-11-20T10:00:00Z"
  }
}
```

### Update Review
```http
PUT /api/v1/reviews/:id
Authorization: Bearer <token>
Content-Type: application/json

{
  "status": "COMPLETED",
  "comments": "Review completed. Document approved with minor suggestions.",
  "rating": 4
}

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "COMPLETED",
    "updated_at": "2025-11-20T12:00:00Z"
  }
}
```

## Users

### List Users (Admin only)
```http
GET /api/v1/users
Authorization: Bearer <token>

Response:
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "username": "user@example.com",
      "full_name": "John Doe",
      "role": "REVIEWER",
      "active": true
    }
  ]
}
```

### Create User (Admin only)
```http
POST /api/v1/users
Authorization: Bearer <token>
Content-Type: application/json

{
  "username": "newuser@example.com",
  "email": "newuser@example.com",
  "password": "SecurePassword123",
  "full_name": "New User",
  "role": "SUBMITTER",
  "department": "Engineering"
}

Response:
{
  "success": true,
  "data": {
    "id": "uuid",
    "username": "newuser@example.com",
    "role": "SUBMITTER"
  }
}
```

## Error Responses

All error responses follow this format:

```json
{
  "success": false,
  "error": "Error message",
  "code": "ERROR_CODE"
}
```

### Common Error Codes

- `400 Bad Request` - Invalid request data
- `401 Unauthorized` - Missing or invalid authentication
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource not found
- `409 Conflict` - Resource conflict (e.g., duplicate)
- `422 Unprocessable Entity` - Validation error
- `500 Internal Server Error` - Server error

## Rate Limiting

API requests are limited to:
- 100 requests per 15 minutes per IP address
- Authenticated users: 1000 requests per hour

Rate limit headers:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1234567890
```

## Pagination

List endpoints support pagination:

```
?page=1&limit=20
```

Response includes pagination metadata:
```json
{
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "pages": 5
  }
}
```

## Versioning

API version is included in the URL: `/api/v1/`

Future versions will be available at `/api/v2/`, etc.
