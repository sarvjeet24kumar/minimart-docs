# Functional Use Case Documentation: MiniMart System

---

## Use Case 1: User Registration & Login (Tenant Context)

**Description:**  
Allows a user to create an account within a specific organization (Tenant) and authenticate via secure JWT.

**Steps:**
- User provides registration details (Email, Username, Password, Name).
- System validates data (email format, password strength, unique username/email within the specific tenant).
- System validates that the `Tenant-ID` provided exists and is active.
- System stores the user profile linked to the relevant `Tenant`.
- System triggers an asynchronous email verification OTP.
- Upon verification, user logs in and system generates JWT tokens (Access & Refresh).
- **Security Check:** Middleware validates on every request that the user's `Tenant` is active and not soft-deleted.

**Post-condition:**  
User is registered, verified, and authenticated. All subsequent requests are scoped to their organization.

**Exceptions:**
- Duplicate username/email for the same tenant.
- Invalid or inactive Tenant-ID provided in header.
- Password does not meet security requirements.

---

## Use Case 2: Collaborative Shopping List Management

**Description:**  
Allows users to create, share, and collaborate on shared shopping lists with members of their tenant.

**Steps:**
- User creates a new shopping list (becomes the `OWNER`).
- User invites other organization members to the list via Email/User-ID.
- System generates a secure, stateless invitation token.
- Invited user accepts the invitation and is added as a `MEMBER`.
- **Real-time Sync:** All members are notified of the new list or membership change via WebSockets.

**Post-condition:**  
A shared shopping list is established with specific members who can collaborate in real-time.

**Exceptions:**
- Inviting a user from a different organization (Blocked by Tenant Isolation).
- Attempting to invite a user who is already a member.
- Invitation token expired or revoked.

---

## Use Case 3: Item Tracking & Status Synchronization

**Description:**  
Allows members of a shopping list to add items and track their purchase status.

**Steps:**
- Member with `can_add_item` permission adds a product with name and quantity.
- System validates the member has sufficient permissions for the specific list.
- User updates item status between `PENDING` and `PURCHASED`.
- **Real-time Sync:** System broadcasts the change to all connected list subscribers immediately.
- System updates the `updated_at` timestamp and tracks the actor (last updated by).

**Post-condition:**  
The shopping list state is updated across all member devices simultaneously.

**Exceptions:**
- Member lacks specific permission for adding/updating items.
- Item does not exist or has been deleted by another member.

---

## Use Case 4: Real-Time Collaborative Chat

**Description:**  
Allows list members to communicate in real-time through a dedicated chat room for each shopping list.

**Steps:**
- User connects to the chat-specific WebSocket endpoint for a list.
- User sends a message via the WebSocket or REST fallback.
- System persists the message to the database for history retrieval.
- System broadcasts the message to all members currently in the list's chat scope.

**Post-condition:**  
Instant communication is achieved, with history available for offline members.

**Exceptions:**
- User is not a member of the list (Connection rejected).
- Message length exceeds the maximum limit (e.g., 2000 characters).

---

## Use Case 5: Organization Administrative Management

**Description:**  
Allows tenant administrators to oversee their organization's users and resources.

**Steps:**
- Admin creates new user accounts for the organization.
- Admin manages user active/inactive status.
- Admin oversees all shopping lists within the tenant (Elevated access).
- Admin modifies list member permissions (Elevated override).

**Post-condition:**  
The organization's users and content are managed according to the administrator's actions.

**Exceptions:**
- Admin attempts to access/manage data from other tenants (Blocked by Tenant Isolation).
- Admin attempts to delete the primary owner of a list without transferring ownership.

---

## Use Case 6: Platform-Level Supervision

**Description:**  
Platform-wide control over tenants and global system auditing for Super Admins.

**Steps:**
- Super Admin creates or updates Tenants (Organizations).
- Super Admin performs "Soft Delete" on inactive or non-paying tenants.
- Super Admin manages `TENANT_ADMIN` accounts globally.
- System provides cross-tenant health and readiness checks.

**Post-condition:**  
The platform ecosystem is maintained, and tenant lifecycles are controlled.

**Exceptions:**
- Super Admin attempts to access PII (Private member data) within a tenant list (Limited by design).
- Attempting to reactive a tenant that has been significantly purged.

---

## Use Case 7: Secure Permission & Real-Time Enforcement

**Description:**  
Ensures all resources are accessed by authenticated users with appropriate roles and tenant context.

**Steps:**
- **Middleware Check:** Every request validates the JWT, checks for blacklisted tokens, and refreshes Tenant/User status.
- **RBAC Enforcement:** System checks platform roles (`USER`, `TENANT_ADMIN`, `SUPER_ADMIN`).
- **ABAC/List Enforcement:** System checks list-level role (`OWNER`, `MEMBER`) and granular permission flags (`can_add_item`, etc.).
- **WebSocket Security:** System immediately disconnects users from all scopes if their account is deactivated or their membership is revoked.

**Post-condition:**  
Absolute security and data isolation are maintained at both REST and WebSocket layers.

**Exceptions:**
- Access token is revoked (logged out) or expired.
- User account deactivated during an active session (Immediate kick).
- Tenant marked as inactive (Immediate block for all organization users).
