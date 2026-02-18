# Authentication & Security

MiniMart implements a secure, tenant-scoped authentication system using JWT tokens and OTP-based email verification for onboarding and password recovery.

---

## Registration & Email Verification

### Signup Flow

1. User sends credentials to `POST /api/v1/auth/signup` with the `Tenant-ID` header.
2. System validates:
   - Tenant exists and is active.
   - Email is unique within the tenant.
   - Username is unique within the tenant.
   - Password meets strength requirements (min 8 chars, mixed case, at least one digit).
3. User is created with `is_email_verified=False` and `is_active=False`.
4. A 6-digit OTP is generated and stored in Redis (`otp:{tenant_id}:{email}`).
5. An asynchronous task (via BackgroundTasks) dispatches the verification email.
6. Response: `"Signup successful. Please verify your email."`

### Email Verification

1. User submits the OTP to `POST /api/v1/auth/verify-email`.
   - Requires `email` and `otp`.
   - Requires `Tenant-ID` header.
2. System validates the OTP against the value stored in Redis.
3. On success:
   - User is marked `is_email_verified=True` and `is_active=True`.
   - OTP is cleared from Redis.
4. The account is now active and ready for login.

### Resend Verification OTP

1. User requests a new code via `POST /api/v1/auth/resend-otp`.
2. Requires `email` and the `Tenant-ID` header.
3. System generates a new 6-digit OTP and resets the TTL in Redis (`OTP_EXPIRE_MINUTES`).
4. A fresh verification email is sent.

---

## Login Flow

MiniMart uses a single-factor (password-based) login flow that issues secure JWT tokens.

### Step 1: Credential Validation

1. User sends credentials to `POST /api/v1/auth/login`.
   - Requires `email` and `password`.
   - `Tenant-ID` header is optional for SuperAdmins but required for Tenant Admins and Users.
2. System validates:
   - User exists in the specified tenant (or is a SuperAdmin).
   - Account is not soft-deleted (`deleted_at` is null).
   - Email is verified (`is_email_verified=True`).
   - Account is active (`is_active=True`).
   - Tenant is active.
   - Password is correct.
3. On success:
   - System issues an `access_token` and a `refresh_token`.
4. Response contains both tokens and their expiration details.

---

## Password Management

### Forgot Password

1. User submits a request to `POST /api/v1/auth/forgot-password`.
   - Requires `email`.
2. System validates user existence and generates a secure reset JWT.
3. The token JTI is cached in Redis (`password_reset:{user_id}`) to ensure only one active reset link per user.
4. An email is sent with a secure reset link.
5. Response is always generic: `"If an account with that email exists, a password reset link has been sent."` to prevent email enumeration.

### Reset Password

1. User submits new credentials to `POST /api/v1/auth/reset-password`.
   - Requires `token`, `new_password`, and `confirm_password`.
2. System validates:
   - Token is a valid `password_reset` JWT and has not expired.
   - Token JTI matches the one stored in Redis.
   - Passwords match and meet strength requirements.
3. System updates the hashed password in the database.
4. Reset record is cleared from Redis.

### Change Password

1. Authenticated user calls `POST /api/v1/auth/change-password`.
2. Required fields:
   - `current_password`: Must match the existing password.
   - `new_password`: Must meet requirements and differ from the current one.
3. System verifies the current password and updates to the new hashed version.

---

## Authentication Mechanism

### JWT and Token Security

- **Access Token:** Short-lived token used for API authorization. Contains `sub` (user_id), `tenant_id`, `role`, and `email`.
- **Refresh Token:** Long-lived token used to obtain new access tokens.
- **Blacklisting:** 
    - On **Logout**, the access token JTI is added to a Redis-based blacklist.
    - The refresh token is added to the `blacklisted_tokens` database table.

### Dependencies & Middleware

All protected routes use the `get_current_user` dependency, which:
1. Extracts the JWT from the `Authorization: Bearer <token>` header.
2. Validates the signature and expiration.
3. Checks if the token JTI is in the Redis blacklist.
4. Verifies the user and their associated tenant are both active.
5. Injects the authenticated `User` model into the request context.
