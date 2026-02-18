# MiniMart – Multi-Tenant Shopping List Platform

A real-time, scalable platform for collaborative shopping list management across multiple organizations with granular access control and live synchronization.

---

## Overview

**MiniMart** is a multi-tenant REST API platform built with FastAPI that enables multiple organizations (tenants) to manage shared shopping lists with complete data isolation and real-time updates via WebSockets.

### Key Goals

- **Multi-tenant** – Serve multiple organizations from a single deployment with strict isolation.
- **Real-time Synchronization** – Live updates for items, chat, and status via WebSockets.
- **3-Tier Platform Roles** – Super Admins, Tenant Admins, and Regular Users.
- **Granular List Permissions** – List-level roles (Owner/Member) with specific action flags (Add, Update, Delete).
- **Embedded Collaboration** – Integrated real-time chat within every shopping list.
- **Security** – JWT-based authentication with OTP verification, token blacklisting, and rate limiting.

---

## Multi-Tenant Architecture

### Isolation Mechanism

- **Database-level:** All core entities (Users, Lists) are tagged with `tenant_id` for scoping.
- **Header-enforced:** `TenantMiddleware` (via FastAPI dependencies) enforces the `Tenant-ID` header on all authoritative requests.
- **Query Scoping:** Service-layer logic automatically applies tenant filters to all CRUD operations.
- **Status Validation:** Middleware blocks access for inactive or soft-deleted tenants.

---

## Role-Based Access Control

### Role Hierarchy

```
SUPER_ADMIN - Manages platform tenants and cross-tenant administrators.
    
TENANT_ADMIN - Full control within a specific organization (User & List management).
    
USER - Standard collaborative access; manages personal and shared lists.
```

---

## Key Features

### Platform & Multi-tenancy
- Infrastructure for complete organization isolation.
- Platform-wide tenant management for Super Admins.
- Soft-delete pattern for all data entities.

### Authentication & Security
- Email-based signup with async OTP verification.
- Secure JWT authentication with refresh token rotation.
- Access token blacklisting on logout (Redis-backed).
- Rate limiting on sensitive endpoints (Auth, Invitations).

### Shopping & Collaboration
- **Real-time Sync:** Live item additions, updates, and purchases.
- **Mini-Chat:** Dedicated WebSocket chat rooms for every list.
- **Granular Permissions:** Owners can toggle "Can Add", "Can Update", or "Can Delete" for members.
- **Stateless Invitations:** JWT-based invite tokens with configurable TTL and auto-expiry.

---

## Technology Stack

**Backend**
- FastAPI (Async ASGI framework)
- SQLAlchemy 2.0 (Async Postgres ORM)
- PostgreSQL (Primary persistence)
- Redis (OTP storage, Token Blacklist, and WebSocket tracking)
- Celery (Asynchronous background tasks/emails)

**Security**
- Argon2 (State-of-the-art password hashing)
- python-jose (JWT management)
- Stateless OTP system (Email-based verification)

**Real-time**
- WebSockets (Bi-directional live communication)
- Scoped Broadcasting (Global vs. List-specific messaging)

---
