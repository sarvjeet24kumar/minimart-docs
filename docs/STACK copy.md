# Technical Stack & Infrastructure

MiniMart is built using a modern, asynchronous Python stack optimized for real-time performance and multi-tenant security.

---

## 1. Core Backend Technologies

| Component | Technology | Version | Purpose |
|-----------|------------|---------|---------|
| **Language** | Python | >= 3.11 | Core logic and type safety. |
| **Framework** | FastAPI | >= 0.128.4 | Asynchronous web framework & input validation. |
| **Database** | PostgreSQL | (Latest) | Relational store with `asyncpg` driver. |
| **ORM** | SQLAlchemy | >= 2.0.46 | Async database mapping and scoped queries. |
| **Migrations**| Alembic | >= 1.15.1 | Database schema versioning. |
| **Cache/PubSub**| Redis | >= 5.2.1 | JWT blacklisting and WebSocket messaging. |
| **Task Queue**| Celery | >= 5.6.2 | Email delivery and maintenance crons. |

---

## 2. Security & Utilities

- **Authentication:** `python-jose` for JWT (RS256/HS256 tokens).
- **Hashing:** `argon2-cffi` for secure, state-of-the-art password storage.
- **Validation:** `Pydantic v2` for strict runtime data validation.
- **IDs:** `uuid6` for database-friendly, chronologically sortable UUIDv7 primary keys.
- **Email:** `aiosmtplib` for non-blocking SMTP communication.

---

## 3. Environment Configuration

MiniMart uses `pydantic-settings` to manage configuration via a `.env` file.

| Variable | Type | Description |
|----------|------|-------------|
| `APP_ENV` | String | `development` or `production`. |
| `DATABASE_URL` | URL | Async connection string for PostgreSQL. |
| `REDIS_URL` | URL | Connection string for Redis. |
| `JWT_SECRET_KEY` | String | Secret for signing access tokens. |
| `SMTP_HOST/PORT` | Mixed | Configuration for outgoing email alerts. |
| `USER_DATA_RETENTION_DAYS` | Int | Days before soft-deleted data is erased. |
| `TENANT_RETENTION_DAYS` | Int | Days before soft-deleted tenants are wiped. |

---

## 4. Infrastructure Requirements

### A. Persistent Storage
- **Primary DB:** PostgreSQL 15+ recommended.
- **Cache:** Redis 6+ recommended for Pub/Sub support.

### B. Worker Nodes
The application requires a **Celery Worker** and a **Celery Beat** instance to handle:
1. Asynchronous Email dispatch.
2. Daily Maintenance tasks (Cleanup of expired data).

### C. Networking
- **Port 8000:** Default API port.
- **WebSockets:** Infrastructure must support long-lived TCP connections and `Upgrade` headers.

---

**Last Updated:** February 17, 2026
