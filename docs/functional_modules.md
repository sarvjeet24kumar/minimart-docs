# Functional Documentation: MiniMart Multi-Tenant System

---

## 1. Authentication & Multi-Tenancy Module

- **Tenant-Aware Authentication**: Secure JWT-based registration and login that automatically identifies the user's organization (Tenant) via the `Tenant-ID` header.
- **Tenant Isolation**: All requests are validated against the tenant status before processing.
- **Secure Credentials**: Password hashing and validation using hashing algorithm.
- **Role-Based Access (Platform)**: Specialized platform roles (**Super Admin**, **Tenant Admin**, **User**) with distinct global permissions.

---

## 2. Tenant Management Module (Platform Level)

- **Lifecycle Management**: Super Admins can create and manage multiple tenants, controlling their active status and visibility.
- **Soft-Delete Architecture**: Tenants and their associated data (Users, Lists, Items) support soft-deletion, marking records as deleted without immediate physical removal.
- **Data Protection**: Support for restoring recently deleted tenants via administrative override before permanent database maintenance.

---

## 3. Shopping List & Member Management Module

- **Collaboration Context**: Management of shared shopping lists and member roles within a tenant's isolated environment.
- **Data Integrity**: Enforces unique constraints per tenant for usernames and list participation rules.
- **Tenant-Isolated Lists**: Users and Tenant Admins only see and collaborate on shopping lists belonging to their organization.

---

## 4. Item Status Tracking Module

- **Actionable Lists**: Members can add items to lists and track their specific fulfillment progress.
- **Predefined Statuses**:
  - **PENDING**: Added items that are yet to be acquired.
  - **PURCHASED**: Items that have been fulfilled and verified by a member.
- **Audit Tracking**: Automatic recording of timestamps and the specific user (actor) who performed the status change.

---

## 5. Access Control & Permission Module

- **Ownership Enforcement**: List Owners have absolute control over their lists, including deletion and member management.
- **Member Permission Sets**: Granular, list-specific flags for members:
  - **Can Add Item**: Permission to expand the list.
  - **Can Update Item**: Permission to modify item details or status.
  - **Can Delete Item**: Permission to remove items from the list.
- **Cross-Tenant Security**: Guaranteed prevention of unauthorized access between different organizations at the API and database levels.

---

## 6. Real-Time Synchronization & Sync Module

- **Instant Updates**: Integrated WebSocket server providing bi-directional communication for live list synchronization.
- **Scoped Broadcasting**: 
  - **Global Scope**: For notifications and organizational alerts.
  - **Chat Scope**: For real-time communication within specific list chat rooms.
