# Architecture & Data Isolation

MiniMart is designed as a secure, multi-tenant system that prioritizes data privacy and real-time responsiveness. This document outlines the technical layers that enforce isolation and system integrity.

---

## 1. Multi-Tenant Strategy

MiniMart utilizes a **Shared Database, Scoped Query** architecture. Unlike separate databases, this approach offers high scalability while maintaining strict isolation through logic.

### Data Isolation Layers
- **Database Schema:** Every tenant-scoped table (Users, Shopping Lists, items, Chat) contains a `tenant_id` column.
- **Foreign Key Constraints:** All relations are strictly tied to the tenant hierarchy.
- **Soft Deletes:** Deleting a tenant (`deleted_at`) triggers a cascade that renders all associated data logically inaccessible before the retention period expires.

---

## 2. Request Lifecycle & Middleware

Every HTTP request undergoes a multi-stage validation process before reaching the business logic.

### A. Logging Middleware
- **Tracing:** Assigns a unique `X-Request-ID` to every request.
- **Privacy:** Automatically redacts PII (Emails, Passwords) and sensitive identifiers from logs.

### B. Tenant Identification
The system identifies the organization context via the `Tenant-ID` header:
```http
Tenant-ID: <organization_uuid>
```
- For unauthenticated requests (e.g., Signup), the header is mandatory.
- For authenticated requests, the system cross-references the header with the `tenant_id` embedded in the user's JWT.

### C. Authentication & RBAC
- **JWT Layer:** Validates the token and extracts the `sub` (User ID) and `role`.
- **Role Verification:** Fast-fail logic prevents unauthorized roles (e.g., regular users) from accessing admin routes.

---

## 3. Service Level Access Control (SLAC)

Access control is enforced at the service layer through the `BaseListService`.

### The Access Gate (`_get_list_with_access`)
This central method coordinates three critical checks for every shopping list operation:
1. **Tenant Check:** Blocks any attempt to access a list belonging to another organization.
2. **Super Admin Block:** Explicitly prevents platform-level admins from viewing private shopping data.
3. **Membership Check:** Ensures the user is an accepted member of the list or a Tenant Admin with override permissions.

---

## 4. Real-Time Architecture

The real-time layer is decoupled from the REST API to ensure high concurrency.

- **Connection Scoping:** Users connect to specific "scopes" (Global or Chat). 
- **Broadcast Filtering:** Events published from the service layer are only routed to users who pass the membership check for that specific resource.
- **Redis Integration:** Used as a message broker for WebSockets and as a result backend for background tasks.

---

## 5. Background Processing & Maintenance

MiniMart offloads heavy or delayed operations to **Celery Workers**:
- **Email Delivery:** Sending OTPs and invitations.
- **Data Cleanup:** Daily crons that execute the retention policies (hard-deleting expired tenants and unverified users).

---

## 6. System Components

| Component | Responsibility |
|-----------|----------------|
| **FastAPI** | High-performance API and WebSocket management. |
| **SQLAlchemy (Async)** | Asynchronous database operations with scoped querying. |
| **PostgreSQL** | Primary persistent store for all relational data. |
| **Redis** | In-memory store for JWT blacklisting and WebSocket pub/sub. |
| **Celery Beat** | Scheduler for periodic maintenance tasks. |

---

**Last Updated:** February 17, 2026
