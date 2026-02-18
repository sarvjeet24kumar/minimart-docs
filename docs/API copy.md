# API Documentation: MiniMart System

## Overview

MiniMart is a high-performance, multi-tenant shopping list management system built with FastAPI. The API follows RESTful principles, uses JSON for data exchange, and implements JWT for secure, stateless authentication.

**Base URL:** `http://localhost:8000/api/v1/`

**API Version:** v1

---

## Authentication

MiniMart uses **JWT (JSON Web Token)** authentication. 

### Authentication Flow

1. **Signup** → User registers with credentials.
2. **Email Verification** → User verifies email with OTP sent to their registered address.
3. **Login** → User provides credentials (email + password) and receives Access & Refresh tokens.
4. **Token Refresh** → Use the refresh token to obtain a new access token when it expires.

### Request Headers

Each authenticated request must include the `Authorization` header:

```http
Authorization: Bearer <access_token>
```

---

## Multi-Tenancy

Each organization (Tenant) in MiniMart is isolated. Most API requests require a tenant context.

### Tenant Header

Include the `Tenant-ID` header (UUID format) for organization-scoped requests:

```http
Tenant-ID: 550e8400-e29b-41d4-a716-446655440000
```

**Tenant Isolation:**
- Regular users and Tenant Admins can only access data within their organization.
- Super Admins have platform-level access to manage multiple tenants.

---

## User Roles & Permissions

### Role Hierarchy

```
SUPER_ADMIN (Platform Management - No private data access)
     │
     ▼
TENANT_ADMIN (Organization Admin - Full control over tenant resources)
     │
     ▼
   USER (Regular Member - Access to shared lists and own content)
```

| Role | Description | Access Level |
|------|-------------|--------------|
| **USER** | Regular member | Can create/join shopping lists and items. |
| **TENANT_ADMIN** | Organization Admin | Can manage all users, lists, and items in their tenant. |
| **SUPER_ADMIN** | Platform Admin | Manages tenants and organization-level accounts. |

---

## Common Response Formats

### Success Response

```json
{
  "message": "Operation successful"
}
```

### Paginated Response

```json
{
  "total": 100,
  "page": 1,
  "size": 10,
  "pages": 10,
  "items": [...]
}
```

---

## API Endpoints

### Authentication Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 1 | `POST` | `/auth/signup` | Public | Register new account (Requires Tenant-ID) |
| 2 | `POST` | `/auth/verify-email` | Public | Verify email with 6-digit OTP |
| 3 | `POST` | `/auth/resend-otp` | Public | Resend verification OTP |
| 4 | `POST` | `/auth/login` | Public | Login and receive JWT tokens |
| 5 | `POST` | `/auth/refresh` | Refresh | Get new access token |
| 6 | `POST` | `/auth/logout` | Auth | Revoke access and refresh tokens |
| 7 | `POST` | `/auth/forgot-password`| Public | Request password reset link |
| 8 | `POST` | `/auth/reset-password` | Public | Reset password with JWT token |
| 9 | `POST` | `/auth/change-password`| Auth | Change password for active user |

### User Management Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 10 | `GET` | `/users` | Admin, SuperAdmin | List users in organization |
| 11 | `POST` | `/users` | Admin, SuperAdmin | Create new user |
| 12 | `GET` | `/users/me` | Auth | Get current user's profile |
| 13 | `GET` | `/users/{id}` | Admin, SuperAdmin| Get specific user details |
| 14 | `PATCH` | `/users/{id}` | Admin, SuperAdmin| Update user information |
| 15 | `DELETE`| `/users/{id}` | Admin, SuperAdmin| Soft-delete user account |

### Tenant Management Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 16 | `GET` | `/tenants` | SuperAdmin | List all organizations |
| 17 | `POST` | `/tenants` | SuperAdmin | Create new organization |
| 18 | `GET` | `/tenants/{id}` | SuperAdmin | Get organization details |
| 19 | `PATCH` | `/tenants/{id}` | SuperAdmin | Update organization settings |
| 20 | `DELETE`| `/tenants/{id}` | SuperAdmin | Deactivate organization |

### Shopping List Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 21 | `GET` | `/shopping-lists` | Auth | List accessible shopping lists |
| 22 | `POST` | `/shopping-lists` | Auth | Create a new shopping list |
| 23 | `GET` | `/shopping-lists/{id}`| Member | Get list details and members |
| 24 | `PATCH`| `/shopping-lists/{id}`| Owner, Admin | Update list information |
| 25 | `DELETE`| `/shopping-lists/{id}`| Owner, Admin | Soft-delete shopping list |

### List Member & Invitation Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 26 | `POST` | `/shopping-lists/{id}/members` | Owner, Admin | Invite member to list |
| 27 | `DELETE`| `/shopping-lists/{id}/members/{user_id}` | Owner, Admin | Remove member from list |
| 28 | `POST` | `/invitations/accept` | Auth | Accept list invitation token |
| 29 | `POST` | `/invitations/reject` | Auth | Reject list invitation token |

### Item Management Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 30 | `GET` | `/shopping-lists/{id}/items` | Member | List items in a list |
| 31 | `POST` | `/shopping-lists/{id}/items` | Member (can_add)| Add item to list |
| 32 | `PATCH`| `/shopping-lists/items/{item_id}`| Member (can_update)| Update item status/details |
| 33 | `DELETE`| `/shopping-lists/items/{item_id}`| Member (can_delete)| Remove item from list |

### Chat & Notification Endpoints

| # | Method | Endpoint | Access | Description |
|---|--------|----------|--------|-------------|
| 34 | `GET` | `/shopping-lists/{id}/chat/messages` | Member | Get chat history |
| 35 | `POST` | `/shopping-lists/{id}/chat/messages` | Member | Send chat message (REST) |
| 36 | `GET` | `/notifications` | Auth | List user notifications |
| 37 | `PATCH`| `/notifications/read` | Auth | Mark all notifications as read |

---

## Access Definitions

| Role | Meaning |
|------|---------|
| **Public** | No authentication or header required. |
| **Auth** | Valid JWT Bearer token + appropriate `Tenant-ID` header. |
| **Member** | User must be an accepted member of the specific list. |
| **Owner** | User must be the designated owner of the shopping list. |
| **Admin** | Role must be `TENANT_ADMIN`. |
| **SuperAdmin** | Role must be `SUPER_ADMIN`. |

---

## HTTP Status Codes

| Status Code | Description |
|-------------|-------------|
| `200 OK` | Successful retrieval or update. |
| `201 Created` | Successful creation of a resource. |
| `204 No Content`| Successful deletion (no body returned). |
| `400 Bad Request`| Validation error or business logic violation. |
| `401 Unauthorized`| Missing or invalid JWT token. |
| `403 Forbidden` | Insufficient role or cross-tenant violation. |
| `404 Not Found` | Resource does not exist or access is restricted. |

---

**Last Updated:** February 17, 2026
**API Version:** v1.0.0
