# INCA Database Schema

## Overview

This document describes the database schema for the INCA platform.

## Entity Relationship Diagram

```
┌──────────────┐         ┌──────────────┐         ┌──────────────┐
│    Users     │         │  Documents   │         │   Reviews    │
├──────────────┤         ├──────────────┤         ├──────────────┤
│ id (PK)      │         │ id (PK)      │         │ id (PK)      │
│ username     │         │ title        │         │ document_id  │
│ email        │    ┌────│ submitter_id │         │ reviewer_id  │
│ password     │    │    │ status       │────┐    │ status       │
│ full_name    │◄───┘    │ version      │    └───►│ comments     │
│ role         │         │ created_at   │         │ created_at   │
│ department   │         │ updated_at   │         │ updated_at   │
│ active       │         └──────────────┘         └──────────────┘
│ created_at   │                │
│ updated_at   │                │
└──────────────┘                │
       │                        │
       │                        │
       │                        ▼
       │              ┌──────────────────┐
       │              │  DocumentFiles   │
       │              ├──────────────────┤
       │              │ id (PK)          │
       │              │ document_id (FK) │
       │              │ filename         │
       │              │ file_path        │
       │              │ file_size        │
       │              │ mime_type        │
       │              │ uploaded_at      │
       │              └──────────────────┘
       │
       │              ┌──────────────────┐
       └─────────────►│   AuditLog       │
                      ├──────────────────┤
                      │ id (PK)          │
                      │ user_id (FK)     │
                      │ action           │
                      │ entity_type      │
                      │ entity_id        │
                      │ changes          │
                      │ timestamp        │
                      └──────────────────┘
```

## Tables

### Users
Stores user account information.

| Column       | Type         | Constraints                    | Description                        |
|--------------|--------------|--------------------------------|------------------------------------|
| id           | UUID         | PRIMARY KEY                    | Unique user identifier             |
| username     | VARCHAR(50)  | UNIQUE, NOT NULL               | Login username                     |
| email        | VARCHAR(255) | UNIQUE, NOT NULL               | Email address                      |
| password     | VARCHAR(255) | NOT NULL                       | Hashed password                    |
| full_name    | VARCHAR(255) | NOT NULL                       | User's full name                   |
| role         | ENUM         | NOT NULL                       | User role (see roles below)        |
| department   | VARCHAR(100) | NULL                           | User's department                  |
| active       | BOOLEAN      | DEFAULT TRUE                   | Account active status              |
| created_at   | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP      | Account creation date              |
| updated_at   | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP      | Last update date                   |

**User Roles:**
- ADMIN - System administrator
- SUBMITTER - Can submit documents
- REVIEWER - Can review documents
- APPROVER - Can approve/reject documents
- VIEWER - Read-only access

### Documents
Stores document metadata and status.

| Column        | Type         | Constraints                    | Description                        |
|---------------|--------------|--------------------------------|------------------------------------|
| id            | UUID         | PRIMARY KEY                    | Unique document identifier         |
| title         | VARCHAR(255) | NOT NULL                       | Document title                     |
| description   | TEXT         | NULL                           | Document description               |
| document_type | VARCHAR(50)  | NOT NULL                       | Type of document                   |
| submitter_id  | UUID         | FOREIGN KEY (Users.id)         | User who submitted document        |
| project_code  | VARCHAR(50)  | NOT NULL                       | Metro project code                 |
| status        | ENUM         | NOT NULL                       | Document status (see below)        |
| version       | INTEGER      | DEFAULT 1                      | Document version number            |
| created_at    | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP      | Document creation date             |
| updated_at    | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP      | Last update date                   |

**Document Status:**
- DRAFT - Being prepared
- SUBMITTED - Submitted for review
- IN_REVIEW - Under review
- PENDING_REVISION - Needs changes
- APPROVED - Approved
- REJECTED - Rejected
- ARCHIVED - Archived

### Reviews
Stores review information and feedback.

| Column       | Type      | Constraints                    | Description                        |
|--------------|-----------|--------------------------------|------------------------------------|
| id           | UUID      | PRIMARY KEY                    | Unique review identifier           |
| document_id  | UUID      | FOREIGN KEY (Documents.id)     | Document being reviewed            |
| reviewer_id  | UUID      | FOREIGN KEY (Users.id)         | User performing review             |
| status       | ENUM      | NOT NULL                       | Review status                      |
| comments     | TEXT      | NULL                           | Review comments                    |
| rating       | INTEGER   | CHECK (1-5)                    | Review rating (1-5)                |
| created_at   | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP      | Review creation date               |
| updated_at   | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP      | Last update date                   |

**Review Status:**
- PENDING - Awaiting review
- IN_PROGRESS - Review in progress
- COMPLETED - Review completed
- APPROVED - Approved
- REJECTED - Rejected with comments

### DocumentFiles
Stores uploaded file information.

| Column       | Type         | Constraints                    | Description                        |
|--------------|--------------|--------------------------------|------------------------------------|
| id           | UUID         | PRIMARY KEY                    | Unique file identifier             |
| document_id  | UUID         | FOREIGN KEY (Documents.id)     | Associated document                |
| filename     | VARCHAR(255) | NOT NULL                       | Original filename                  |
| file_path    | VARCHAR(500) | NOT NULL                       | Storage path                       |
| file_size    | BIGINT       | NOT NULL                       | File size in bytes                 |
| mime_type    | VARCHAR(100) | NOT NULL                       | File MIME type                     |
| uploaded_at  | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP      | Upload timestamp                   |

### AuditLog
Tracks all system actions for audit purposes.

| Column       | Type         | Constraints                    | Description                        |
|--------------|--------------|--------------------------------|------------------------------------|
| id           | UUID         | PRIMARY KEY                    | Unique log entry identifier        |
| user_id      | UUID         | FOREIGN KEY (Users.id)         | User who performed action          |
| action       | VARCHAR(50)  | NOT NULL                       | Action performed                   |
| entity_type  | VARCHAR(50)  | NOT NULL                       | Type of entity affected            |
| entity_id    | UUID         | NOT NULL                       | ID of affected entity              |
| changes      | JSON         | NULL                           | Details of changes made            |
| ip_address   | VARCHAR(45)  | NULL                           | IP address of user                 |
| timestamp    | TIMESTAMP    | DEFAULT CURRENT_TIMESTAMP      | Action timestamp                   |

## Indexes

Performance-critical indexes to be added:

- `idx_documents_submitter` on Documents(submitter_id)
- `idx_documents_status` on Documents(status)
- `idx_reviews_document` on Reviews(document_id)
- `idx_reviews_reviewer` on Reviews(reviewer_id)
- `idx_auditlog_user` on AuditLog(user_id)
- `idx_auditlog_timestamp` on AuditLog(timestamp)

## Constraints

- CASCADE DELETE on DocumentFiles when parent Document is deleted
- RESTRICT DELETE on Documents if Reviews exist
- Prevent user deletion if they have submitted documents

## Future Enhancements

- Add notifications table
- Add workflow states table
- Add document templates table
- Add project metadata table
