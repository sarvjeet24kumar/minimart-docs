# Tenant Onboarding Guide

## What is a Tenant?

A tenant represents an organization or organization-unit using the MiniMart platform. Each tenant has its own completely isolated environment with separate users, shopping lists, and data.

---

## Onboarding Process

### Step 1: Tenant Creation (By Super Admin)

The platform Super Admin initializes the tenant with:

- **Name**: Organization's display name.
- **Slug**: Unique URL identifier (e.g., `acme-corp`), used for identification.
- **Initial Status**: Created as `Active` by default.

Upon creation, the tenant receives a unique `ID` (UUIDv7) which must be included in the `Tenant-ID` header for all subsequent organization-scoped API requests.

---

### Step 2: Admin Account Setup

The Super Admin creates the first Administrative user for the new tenant:

1. Super Admin provides:
   - Admin's email address.
   - Username.
   - Initial secure password.
   - Role set as `TENANT_ADMIN`.
2. System actions:
   - Creates the user record linked to the `tenant_id`.
   - Dispatches an email verification OTP.
3. Admin Action:
   - Verifies email via `POST /api/v1/auth/verify-email`.
   - The account is now active and ready for organization management.

---

### Step 3: Initial Configuration

The Tenant Admin prepares the organization for its users:

| Task | Description |
|------|-------------|
| **Invite Users** | Create user accounts directly or share invitation tokens. |
| **Manage Members** | Set up initial shopping lists and define member permission sets. |
| **Review Settings** | Ensure the organization slug and name are correctly set. |

---

### Step 4: User Signup

Regular users join the specific organization:

1. User registers via `POST /api/v1/auth/signup` providing the `Tenant-ID` header.
2. System validates the tenant is active and not soft-deleted.
3. User receives an email verification OTP.
4. User verifies their account to gain access to the organization's resources.

---

## Tenant Lifecycle

### Active State
- Default state upon creation.
- All users can login and collaborate on shopping lists.

### Inactive State
- Tenant Admins or Super Admins can deactivate a tenant (`is_active=False`).
- **Effect:** All logins and API requests for that tenant are immediately blocked with a `403 Tenant Inactive` error.
- All WebSocket connections for the tenant's users are terminated.

### Deletion (Soft-Delete)
- Super Admins can delete a tenant.
- **Effect:** Marks `deleted_at`, deactivates the tenant, and hides it from standard lookups. Data remains for audit/recovery but is inaccessible.

---

## Data Isolation

MiniMart ensures absolute data isolation between organizations:

- **Scoped Queries:** All service-layer operations automatically filter by `tenant_id`.
- **Identity Isolation:** Users are unique within their tenant; cross-tenant authentication is strictly forbidden.
- **Resource Ownership:** Shopping lists and items are strictly owned by a single tenant. No resource can be shared across tenant boundaries.

---

## Quick Reference

### For Super Admin
1. Create tenant via `POST /api/v1/tenants`.
2. Create the first `TENANT_ADMIN` for that organization.
3. Provide the `Tenant-ID` to the organization's representative.

### For Tenant Admin
1. Verify email and login.
2. Invite and manage team members via `User Management`.
3. Oversee all shopping lists and items within the organization.

### For Users
1. Register using the provided `Tenant-ID` header.
2. Complete registration and email verification.
3. Start creating and collaborating on shopping lists.

---

## Support

- **Tenant Issues:** Handled by the Super Admin.
- **User/List Issues:** Handled by the Tenant Admin within the organization.
- **System Outages:** Contact platform support.
