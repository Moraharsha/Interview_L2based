# Why Doesn't Auth Service Participate in Every Request After JWT Creation?

## My Question

> Once the JWT token is created, why doesn't the Auth Service come into the middle again for every request?

This is a very common question when learning JWT and Microservices.

---

# Short Answer

**Because JWT is self-contained.**

Once the Auth Service creates the token, the token itself contains all the information needed to verify the user.

Therefore:

- Auth Service creates the token once.
- Client stores the token.
- API Gateway or Microservices validate the token locally.
- Auth Service is not contacted again for normal requests.

---

# Step 1: User Logs In

The user sends credentials:

```http
POST /auth/login
```

Request:

```json
{
  "username": "vardhan",
  "password": "12345"
}
```

Flow:

```text
Client
   ↓
Auth Service
   ↓
Verify Username & Password
   ↓
Generate JWT
   ↓
Return JWT
```

Example JWT:

```text
eyJhbGciOiJIUzI1NiJ9...
```

---

# Step 2: What Does JWT Contain?

JWT contains information like:

```json
{
  "sub": "vardhan",
  "role": "USER",
  "exp": 1780000000
}
```

and a digital signature.

---

# Why Is the Signature Important?

The signature proves:

```text
Nobody modified the token
```

If somebody changes:

```json
{
  "role": "USER"
}
```

to:

```json
{
  "role": "ADMIN"
}
```

the signature becomes invalid.

The request will be rejected.

---

# Step 3: User Calls Another API

Example:

```http
GET /products
Authorization: Bearer eyJhbGciOi...
```

Flow:

```text
Client
   ↓
API Gateway
   ↓
Product Service
```

Notice:

```text
Auth Service is NOT involved
```

---

# Why Doesn't Gateway Ask Auth Service?

Because Gateway can verify the token itself.

Gateway checks:

```text
Is signature valid?
Is token expired?
```

using the same secret key used by Auth Service.

Example:

```text
Auth Service Secret
       =
Gateway Secret
```

If valid:

```text
Forward Request
```

If invalid:

```text
401 Unauthorized
```

---

# Real Flow

## Login Request

```text
Client
   ↓
Auth Service
   ↓
Generate JWT
   ↓
Return Token
```

---

## Product Request

```text
Client
   ↓
API Gateway
   ↓
Validate JWT
   ↓
Product Service
```

No Auth Service.

---

## Order Request

```text
Client
   ↓
API Gateway
   ↓
Validate JWT
   ↓
Order Service
```

No Auth Service.

---

# Then Why Create Auth Service?

Auth Service is responsible for:

## Registration

```http
POST /auth/register
```

## Login

```http
POST /auth/login
```

## Token Refresh

```http
POST /auth/refresh
```

## Password Reset

```http
POST /auth/forgot-password
```

After issuing the JWT, its main job is finished.

---

# Movie Ticket Example

Think of JWT like a movie ticket.

## Ticket Counter

```text
Ticket Counter
      ↓
Issue Ticket
```

This is:

```text
Auth Service
```

---

## Security Guard

```text
Security Guard
      ↓
Check Ticket
      ↓
Allow Entry
```

This is:

```text
API Gateway
```

---

### Important Observation

The security guard does NOT call the ticket counter every time.

Instead:

```text
Look at Ticket
      ↓
Verify Ticket
      ↓
Allow Entry
```

JWT works exactly the same way.

---

# What Is JWT Validation?

Gateway receives:

```http
Authorization: Bearer eyJhbGciOi...
```

It checks:

```text
1. Token present?
2. Signature valid?
3. Not expired?
```

If all are true:

```text
Request Allowed
```

Otherwise:

```text
401 Unauthorized
```

---

# Where Is JWT Validated?

There are two common approaches.

---

## Approach 1 (Recommended)

Validate JWT at API Gateway.

```text
Client
   ↓
Gateway
   ↓
JWT Validation
   ↓
Forward Request
```

Advantages:

- Centralized validation
- Less duplicate code
- Better performance

---

## Approach 2

Every microservice validates JWT.

```text
Client
   ↓
Gateway
   ↓
Product Service JWT Filter
```

```text
Client
   ↓
Gateway
   ↓
Order Service JWT Filter
```

Disadvantages:

- Duplicate code
- Harder maintenance

---

# Final Architecture

```text
                Login
                  ↓
            Auth Service
                  ↓
             Generate JWT
                  ↓
                Client
                  ↓
          Bearer Token Sent
                  ↓
            API Gateway
                  ↓
           Validate JWT
                  ↓
    ┌─────────┬─────────┬─────────┐
    ↓         ↓         ↓
Product   Order     User Service
Service   Service
```

---

# Key Interview Answer

**Q: Once JWT is generated, why doesn't Auth Service participate in every request?**

**Answer:**

JWT is a self-contained token that already contains user information and a digital signature. After login, the client sends the JWT with each request. The API Gateway or Microservices validate the token locally using the secret key without contacting the Auth Service again. Therefore, Auth Service is mainly responsible for registration, login, token generation, refresh tokens, and password management, while normal requests are authenticated using the JWT itself.
