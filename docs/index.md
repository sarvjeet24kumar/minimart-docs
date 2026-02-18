# MiniMart – Multi-Tenant Shopping List Platform



## Overview

**MiniMart** is a multi-tenant REST API platform built with FastAPI that enables multiple organizations (tenants) to manage shared shopping lists with complete data isolation and real-time updates via WebSockets.

### Key Goals

- **Multi-tenant** – Serve multiple organizations from a single deployment with strict isolation.
- **Real-time Synchronization** – Live updates for items, chat, and status via WebSockets.
- **Platform Roles** – Super Admins, Tenant Admins, and Regular Users.
- **List Permissions** – List-level roles (Owner/Member) with specific action flags (Add, Update, Delete).
- **Embedded Collaboration** – Integrated real-time chat within every shopping list.
- **Security** – JWT-based authentication with OTP verification, token blacklisting, and rate limiting.

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


### Authentication & Security
- Email-based signup with async OTP verification.
- Secure JWT authentication with refresh token rotation.
- Access token blacklisting on logout.
- Rate limiting on endpoints.

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

**Real-time**

- WebSockets (Bi-directional live communication)

---
