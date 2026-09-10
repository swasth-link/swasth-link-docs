# User Authentication API

This document defines the API endpoints for **User Authentication** in the Swasth Link project.

## Base Path

```text
/api/user
```

---

## API Overview

| # | Method | Endpoint                    | Authentication | Description             |
| - | ------ | --------------------------- | -------------- | ----------------------- |
| 1 | `POST` | `/api/user/register`        | No             | Register a new user     |
| 2 | `POST` | `/api/user/login`           | No             | Login user              |
| 3 | `POST` | `/api/user/otp/verify`      | No             | Verify OTP              |
| 4 | `POST` | `/api/user/otp/resend`      | No             | Resend OTP              |
| 5 | `POST` | `/api/user/password/forgot` | No             | Start password recovery |
| 6 | `POST` | `/api/user/password/reset`  | No             | Reset password          |
| 7 | `POST` | `/api/user/logout`          | Yes            | Logout user             |

---

# 1. User Registration

### Endpoint

```http
POST /api/user/register
```

### Description

Creates a new user account and sends an OTP for verification.

### Request Body

```json
{
  "fullName": "Rahul Kumar",
  "dateOfBirth": "2004-08-10",
  "gender": "MALE",
  "mobile": "9876543210",
  "email": "rahul@example.com",
  "password": "UserPassword@123"
}
```

### Request Fields

| Field         | Type   | Required | Description          |
| ------------- | ------ | -------- | -------------------- |
| `fullName`    | String | Yes      | User's full name     |
| `dateOfBirth` | Date   | Yes      | User's date of birth |
| `gender`      | String | Yes      | User's gender        |
| `mobile`      | String | Yes      | User's mobile number |
| `email`       | String | No       | User's email address |
| `password`    | String | Yes      | Account password     |

### Success Response

**Status:** `201 Created`

```json
{
  "success": true,
  "message": "Registration successful. OTP sent.",
  "userId": "USR10001"
}
```

### Error Responses

| Status | Meaning                         |
| ------ | ------------------------------- |
| `400`  | Invalid request data            |
| `409`  | Mobile/email already registered |
| `500`  | Internal server error           |

### Flow

```text
Register
   ↓
Validate Data
   ↓
Create User
   ↓
Send OTP
   ↓
OTP Verification
```

---

# 2. User Login

### Endpoint

```http
POST /api/user/login
```

### Description

Authenticates a registered user.

### Request Body

```json
{
  "mobile": "9876543210",
  "password": "UserPassword@123"
}
```

### Success Response

**Status:** `200 OK`

```json
{
  "success": true,
  "message": "Login successful",
  "accessToken": "JWT_ACCESS_TOKEN",
  "refreshToken": "JWT_REFRESH_TOKEN",
  "user": {
    "userId": "USR10001",
    "fullName": "Rahul Kumar"
  }
}
```

### Error Responses

| Status | Meaning                         |
| ------ | ------------------------------- |
| `400`  | Invalid request                 |
| `401`  | Invalid mobile/password         |
| `403`  | Account not verified or blocked |
| `500`  | Internal server error           |

---

# 3. OTP Verification

### Endpoint

```http
POST /api/user/otp/verify
```

### Description

Verifies the OTP sent to the user's registered mobile number.

### Request Body

```json
{
  "mobile": "9876543210",
  "otp": "123456",
  "purpose": "REGISTRATION"
}
```

### Request Fields

| Field     | Type   | Required | Description              |
| --------- | ------ | -------- | ------------------------ |
| `mobile`  | String | Yes      | Registered mobile number |
| `otp`     | String | Yes      | OTP received by user     |
| `purpose` | String | Yes      | Reason for OTP           |

### Supported OTP Purposes

```text
REGISTRATION
FORGOT_PASSWORD
```

### Success Response

**Status:** `200 OK`

```json
{
  "success": true,
  "message": "OTP verified successfully"
}
```

### Error Responses

| Status | Meaning               |
| ------ | --------------------- |
| `400`  | Invalid OTP           |
| `410`  | OTP expired           |
| `429`  | Too many attempts     |
| `500`  | Internal server error |

---

# 4. Resend OTP

### Endpoint

```http
POST /api/user/otp/resend
```

### Description

Sends a new OTP to the user's registered mobile number.

### Request Body

```json
{
  "mobile": "9876543210",
  "purpose": "REGISTRATION"
}
```

### Success Response

**Status:** `200 OK`

```json
{
  "success": true,
  "message": "OTP sent successfully"
}
```

### Error Responses

| Status | Meaning               |
| ------ | --------------------- |
| `400`  | Invalid request       |
| `429`  | Too many OTP requests |
| `500`  | Internal server error |

---

# 5. Forgot Password

### Endpoint

```http
POST /api/user/password/forgot
```

### Description

Starts the password recovery process by sending an OTP.

### Request Body

```json
{
  "mobile": "9876543210"
}
```

### Success Response

**Status:** `200 OK`

```json
{
  "success": true,
  "message": "If the account exists, an OTP has been sent."
}
```

> A generic response should be used so that the API does not reveal whether a particular mobile number is registered.

---

# 6. Reset Password

### Endpoint

```http
POST /api/user/password/reset
```

### Description

Resets the user's password after successful OTP verification.

### Request Body

```json
{
  "mobile": "9876543210",
  "otp": "123456",
  "newPassword": "NewPassword@123"
}
```

### Success Response

**Status:** `200 OK`

```json
{
  "success": true,
  "message": "Password reset successfully"
}
```

### Error Responses

| Status | Meaning                    |
| ------ | -------------------------- |
| `400`  | Invalid request            |
| `401`  | Invalid or expired OTP     |
| `422`  | Password validation failed |
| `500`  | Internal server error      |

---

# 7. User Logout

### Endpoint

```http
POST /api/user/logout
```

### Description

Logs out the currently authenticated user.

### Request Headers

```http
Authorization: Bearer <ACCESS_TOKEN>
```

### Request Body

```json
{}
```

### Success Response

**Status:** `200 OK`

```json
{
  "success": true,
  "message": "Logout successful"
}
```

### Error Responses

| Status | Meaning                  |
| ------ | ------------------------ |
| `401`  | Invalid or expired token |
| `500`  | Internal server error    |

---

# User Authentication Flow

## Registration Flow

```text
User
 ↓
Register
 ↓
OTP Sent
 ↓
Verify OTP
 ↓
Account Verified
 ↓
Login
```

## Forgot Password Flow

```text
User
 ↓
Forgot Password
 ↓
OTP Sent
 ↓
Verify OTP
 ↓
Reset Password
 ↓
Login
```

## Login Flow

```text
User
 ↓
Login
 ↓
Credentials Valid
 ↓
Access Token
 ↓
Authenticated User
```

---

# Authentication Header

Authenticated APIs use:

```http
Authorization: Bearer <ACCESS_TOKEN>
```

Example:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

---

# Standard Response Format

All APIs should follow a consistent response structure.

### Success

```json
{
  "success": true,
  "message": "Operation successful",
  "data": {}
}
```

### Error

```json
{
  "success": false,
  "message": "Invalid request",
  "errorCode": "INVALID_REQUEST"
}
```

---

# Development Responsibility

### Backend Developer

* Implement API endpoints
* Request validation
* Authentication and authorization
* Password hashing
* OTP generation and verification
* JWT/token management
* Error handling
* API documentation

### Frontend Developer

* Registration UI
* Login UI
* OTP UI
* Forgot Password UI
* Reset Password UI
* API integration
* Token/session handling
* Display API errors

---

# Endpoint Summary

```text
POST /api/user/register
POST /api/user/login
POST /api/user/otp/verify
POST /api/user/otp/resend
POST /api/user/password/forgot
POST /api/user/password/reset
POST /api/user/logout
```
