# 🔐 Authentication Flow Documentation

## Overview
This document describes the complete authentication flow implementation including login, registration, token refresh, logout, and security mechanisms.

## 📋 Table of Contents
- [Login Flow](#-login-flow)
- [Registration Flow](#-registration-flow)
- [Token Refresh Flow](#-token-refresh-flow)
- [Logout Flow](#-logout-flow)
- [Token Rotation Security](#-token-rotation-security)

---

## 🔑 Login Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant Ctrl as Controller
    participant AM as AuthenticationManager
    participant DB as Database
    participant SC as SecurityContext

    C->>Ctrl: Login Request (username/password)
    Ctrl->>AM: Authenticate credentials
    AM-->>Ctrl: Authentication result
    
    alt Authentication Failed
        Ctrl-->>C: Exception + Login failed message
    else Authentication Success
        Ctrl->>DB: Generate Access Token (AT) & Refresh Token (RT)
        Ctrl->>C: Set RT in Cookie + AT in Response
        Note over C: Store AT in Local Storage
        Ctrl->>SC: Save Authentication to SecurityContextHolder
        Ctrl-->>C: Success Response
        C->>C: Redirect to Home
    end
```

### Implementation Steps:
1. **Client** sends login request with username/password to **Controller**
2. **Controller** calls **AuthenticationManager** to validate credentials
3. **Process Result**:
   - ❌ **Invalid**: Throw exception → Login failed notification
   - ✅ **Valid**: 
     - Generate Access Token (AT) and Refresh Token (RT)
     - Store RT in HTTP-only cookie
     - Return AT to client (stored in local storage)
     - Save Authentication in SecurityContextHolder
     - Redirect client to Home page

---

## 📝 Registration Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant Ctrl as Controller
    participant DB as Database
    participant SC as SecurityContext

    C->>Ctrl: Registration Request
    Ctrl->>DB: Check if user exists
    
    alt User Exists
        Ctrl-->>C: Exception + Registration failed message
    else User Not Exists
        Ctrl->>DB: Save user data
        Ctrl->>SC: Create & save Authentication
        Ctrl->>DB: Generate AT & RT
        Ctrl->>C: Set RT in Cookie + AT in Response
        Note over C: Store AT in Local Storage
        Ctrl-->>C: Success Response
        C->>C: Redirect to Home
    end
```

### Implementation Steps:
1. **Client** sends registration request to **Controller**
2. **Controller** checks if user already exists:
   - ❌ **Exists**: Throw exception → Registration failed notification
   - ✅ **Valid**:
     - Save user data to database
     - Create and save Authentication in SecurityContextHolder
     - Generate Access Token and Refresh Token
     - Store tokens (RT in cookie, AT in local storage)
     - Redirect client to Home page

---

## 🔄 Token Refresh Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant Ctrl as Controller
    participant DB as Database

    C->>Ctrl: Refresh Token Request
    Ctrl->>Ctrl: Extract RT from Cookie
    Ctrl->>DB: Validate Refresh Token
    
    alt RT Invalid
        Ctrl-->>C: Exception + Token invalid
    else RT Valid
        Note over Ctrl,DB: Token Rotation Process
        Ctrl->>DB: Revoke old RT (mark as used)
        Ctrl->>DB: Generate new AT & RT
        Ctrl->>C: Set new RT in Cookie + new AT in Response
        Note over C: Store new AT in Local Storage
        Ctrl-->>C: Success Response
    end
```

### Implementation Steps:
1. **Client** sends refresh token request
2. **Server** extracts RT from HTTP-only cookie
3. **Validate RT**:
   - ❌ **Invalid**: Throw exception
   - ✅ **Valid**: Execute rotation process:
     - Revoke old RT and mark as used (prevents replay attacks)
     - Generate new Access Token and Refresh Token
     - Store new tokens (RT in cookie, AT in local storage)
     - Return success response → Client continues with new AT

---

## 🚪 Logout Flow

```mermaid
sequenceDiagram
    participant C as Client
    participant Ctrl as Controller
    participant DB as Database

    C->>Ctrl: Logout Request
    Ctrl->>Ctrl: Extract RT from Cookie
    Ctrl->>Ctrl: Clear RT Cookie
    Ctrl->>DB: Revoke RT in Database
    Note over C: Clear AT from Local Storage
    Ctrl-->>C: Logout Success
    Note over C: User completely logged out
```

### Implementation Steps:
1. **Client** initiates logout request
2. **Server** processes logout:
   - Extract RT from cookie
   - Clear RT from HTTP-only cookie
   - Revoke RT in database (invalidate token)
3. **Client** clears Access Token from local storage
4. **Result**: User is completely logged out, all tokens are invalidated

---

## 🔒 Token Rotation Security

### Why Token Rotation?
Token rotation is a critical security mechanism that ensures each Refresh Token can only be used once.

### How It Works:
```mermaid
graph TD
    A[RT Used for Refresh] --> B[Revoke Old RT]
    B --> C[Mark as 'Used' in DB]
    C --> D[Generate New RT]
    D --> E[Send New RT to Client]
    E --> F[Old RT Cannot Be Reused]
```

### Security Benefits:
- **🛡️ Replay Attack Prevention**: Each RT is single-use only
- **🔍 Breach Detection**: Reuse of revoked tokens indicates compromise
- **⏰ Limited Exposure Window**: Stolen tokens have minimal lifetime
- **🔄 Automatic Invalidation**: Regular token renewal reduces risk

### Implementation Details:
1. **Before Rotation**: Validate current RT
2. **During Rotation**: 
   - Revoke old RT (set status to 'used')
   - Generate new RT with fresh expiration
3. **After Rotation**: Old RT becomes permanently invalid

---

## 🏗️ Architecture Components

| Component | Responsibility |
|-----------|---------------|
| **Controller** | Handle HTTP requests/responses |
| **AuthenticationManager** | Validate user credentials |
| **SecurityContextHolder** | Manage authentication state |
| **Token Service** | Generate/validate/revoke tokens |
| **Database** | Store user data and token metadata |

---
