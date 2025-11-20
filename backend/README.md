# INCA Backend API

Backend services and API for the INCA document management platform.

## Overview

This directory will contain the backend API implementation, including:

- RESTful API endpoints
- Database models and migrations
- Authentication and authorization services
- Business logic and workflows
- Integration services

## Structure

```
backend/
├── src/                  # Source code
│   ├── controllers/      # API controllers
│   ├── models/           # Database models
│   ├── services/         # Business logic
│   ├── middleware/       # Express middleware
│   ├── routes/           # API routes
│   └── utils/            # Utility functions
├── tests/                # Test files
├── config/               # Configuration files
└── README.md            # This file
```

## API Endpoints (Planned)

### Authentication
- `POST /api/auth/login` - User login
- `POST /api/auth/logout` - User logout
- `POST /api/auth/refresh` - Refresh token

### Documents
- `GET /api/documents` - List documents
- `POST /api/documents` - Create document
- `GET /api/documents/:id` - Get document details
- `PUT /api/documents/:id` - Update document
- `DELETE /api/documents/:id` - Delete document

### Reviews
- `GET /api/reviews` - List reviews
- `POST /api/reviews` - Create review
- `GET /api/reviews/:id` - Get review details
- `PUT /api/reviews/:id` - Update review status

### Users
- `GET /api/users` - List users
- `POST /api/users` - Create user
- `GET /api/users/:id` - Get user details
- `PUT /api/users/:id` - Update user

## Development

Instructions will be added as the backend is implemented.

## Testing

Testing guidelines will be provided.
