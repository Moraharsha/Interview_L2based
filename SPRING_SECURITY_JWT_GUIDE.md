# Spring Security & JWT Complete Guide (Beginner to Interview Level)

## Table of Contents

1. What is Security?
2. Why Do We Need Spring Security?
3. Authentication vs Authorization
4. How Login Works Traditionally
5. What is JWT?
6. Structure of JWT
7. What is Tamper-Proof?
8. JWT Login Flow
9. Spring Security Architecture
10. SecurityContext
11. AuthenticationManager
12. UserDetails & UserDetailsService
13. PasswordEncoder & BCrypt
14. JWT Utility Class Explained
15. JWT Filter Explained
16. SecurityFilterChain Explained
17. Stateless Authentication
18. Common HTTP Status Codes
19. Interview Questions & Answers
20. Complete Request Flow

---

# 1. What is Security?

Security means protecting our application from:

- Unauthorized users
- Data theft
- Password leaks
- Fake requests
- Access to restricted resources

Example:

```text
Bank Application

Customer -> Can see own account
Admin -> Can manage customers
Anonymous User -> Cannot access anything
```

Without security:

```text
Anyone can access everything
```

With security:

```text
Only authorized users can access resources
```

---

# 2. Why Do We Need Spring Security?

Without Spring Security:

We manually write:

- Login logic
- Password validation
- Session handling
- Role checking
- URL protection

Example:

```java
if(user == null){
   return "Access Denied";
}
```

for every controller.

This becomes difficult to maintain.

Spring Security provides:

- Authentication
- Authorization
- Password Encryption
- Session Management
- JWT Integration
- CSRF Protection
- Security Filters

---

# 3. Authentication vs Authorization

This is one of the most common interview questions.

---

## Authentication

Authentication means:

> Who are you?

Example:

```text
Username: vardhan
Password: admin123
```

System verifies credentials.

Result:

```text
User authenticated
```

---

## Authorization

Authorization means:

> What are you allowed to do?

Example:

```text
ROLE_ADMIN
```

Allowed:

```text
Create Product
Delete Product
Update Product
```

Example:

```text
ROLE_USER
```

Allowed:

```text
View Products
Place Orders
```

Not allowed:

```text
Delete Products
```

---

## Simple Example

Authentication:

```text
Show your ID card
```

Authorization:

```text
Which rooms can you enter?
```

---

# 4. Traditional Login Flow

Before JWT:

```text
User Login
    ↓
Server validates
    ↓
Session Created
    ↓
Session ID stored in browser
    ↓
Server stores session data
```

Problem:

```text
Large memory usage
Hard scaling
```

For microservices:

```text
Not recommended
```

---

# 5. What is JWT?

JWT stands for:

```text
JSON Web Token
```

A compact token used for authentication.

Example:

```text
eyJhbGciOiJIUzI1NiJ9.
eyJzdWIiOiJ2YXJkaGFuIn0.
abcxyz123
```

JWT contains user information and can prove identity.

---

# 6. Structure of JWT

JWT contains 3 parts.

```text
Header.Payload.Signature
```

Example:

```text
xxxxx.yyyyy.zzzzz
```

---

## Header

Contains algorithm information.

Example:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

---

## Payload

Contains claims.

Example:

```json
{
  "sub": "vardhan",
  "iat": 1747829400,
  "exp": 1747915800
}
```

Meaning:

```text
sub = username
iat = issued at
exp = expiration
```

---

## Signature

Generated using secret key.

Example:

```text
HMACSHA256(
header + payload + secretKey
)
```

Used to verify token integrity.

---

# 7. What is Tamper-Proof?

Very important interview question.

---

Suppose original token payload:

```json
{
   "sub":"vardhan"
}
```

Attacker changes:

```json
{
   "sub":"admin"
}
```

Payload changed.

But signature still belongs to original payload.

Now verification fails.

Result:

```text
Invalid JWT
```

Request rejected.

---

## Why?

Because signature is generated using:

```text
Header + Payload + SecretKey
```

When payload changes:

```text
Signature changes
```

Old signature no longer matches.

Therefore JWT is:

```text
Tamper-Proof
```

Meaning:

```text
Data cannot be modified without detection
```

---

# 8. JWT Login Flow

```text
User Login
    ↓
Username + Password
    ↓
Auth Service
    ↓
Validate Credentials
    ↓
Generate JWT
    ↓
Return Token
    ↓
Client Stores Token
    ↓
Every Request
Authorization: Bearer <token>
    ↓
JWT Validation
    ↓
Access Granted
```

---

# 9. Spring Security Architecture

```text
Client Request
       ↓
Security Filters
       ↓
Authentication
       ↓
Authorization
       ↓
Controller
       ↓
Response
```

Every request passes through Security Filters first.

---

# 10. SecurityContext

SecurityContext stores information about the currently logged-in user.

Example:

```java
Authentication auth =
SecurityContextHolder
.getContext()
.getAuthentication();
```

Contains:

```text
Username
Roles
Authorities
Authentication Status
```

Think of it as:

```text
Current User Information Holder
```

---

# 11. AuthenticationManager

AuthenticationManager performs authentication.

Example:

```java
authenticationManager.authenticate(
new UsernamePasswordAuthenticationToken(
username,
password
));
```

Process:

```text
Receive credentials
     ↓
Check database
     ↓
Verify password
     ↓
Return authenticated user
```

---

# 12. UserDetails & UserDetailsService

## UserDetails

Represents logged-in user information.

Contains:

```java
username
password
authorities
```

---

## UserDetailsService

Loads user from database.

Example:

```java
@Override
public UserDetails loadUserByUsername(
String username)
```

Flow:

```text
Username
   ↓
Database Query
   ↓
User Found
   ↓
Return UserDetails
```

---

# 13. PasswordEncoder & BCrypt

Never store passwords like:

```text
admin123
```

Store encrypted passwords.

Example:

```java
passwordEncoder.encode("admin123");
```

Result:

```text
$2a$10$hfjfjjdjdjjd...
```

This is BCrypt hash.

---

## Password Verification

```java
passwordEncoder.matches(
"admin123",
storedHash
);
```

Result:

```text
true
```

---

# 14. JWT Utility Class Explained

Responsibilities:

### Generate Token

```java
generateToken(username)
```

Creates JWT.

---

### Extract Username

```java
extractUsername(token)
```

Returns:

```text
vardhan
```

---

### Validate Token

```java
validateToken(token)
```

Checks:

- Signature
- Expiration
- Format

---

### Extract Claims

```java
extractClaims(token)
```

Returns:

```json
{
 "sub":"vardhan",
 "iat":123,
 "exp":456
}
```

---

# 15. JWT Filter Explained

JWT Filter executes for every request.

Example:

```text
GET /products
Authorization: Bearer xyz
```

Filter:

```text
Extract Token
     ↓
Validate Token
     ↓
Extract Username
     ↓
Set Authentication
     ↓
Continue Request
```

---

# 16. SecurityFilterChain Explained

Modern Spring Security configuration.

Example:

```java
@Bean
public SecurityFilterChain securityFilterChain(
HttpSecurity http)
```

Responsibilities:

```text
Permit URLs
Protect URLs
Disable CSRF
Configure Sessions
Register Filters
```

---

Example:

```java
http
.authorizeHttpRequests(auth ->
auth
.requestMatchers("/auth/**")
.permitAll()
.anyRequest()
.authenticated()
);
```

Meaning:

```text
/auth/**
Open to everyone

Others
Require authentication
```

---

# 17. Stateless Authentication

JWT applications are usually:

```java
SessionCreationPolicy.STATELESS
```

Meaning:

```text
Server stores no session
```

Every request must contain JWT.

Benefits:

- Better scaling
- Better microservices support
- Less memory usage

---

# 18. Common HTTP Status Codes

## 200

```text
OK
```

Success.

---

## 201

```text
Created
```

Resource created.

---

## 400

```text
Bad Request
```

Invalid input.

---

## 401

```text
Unauthorized
```

Authentication missing.

---

## 403

```text
Forbidden
```

Authenticated but not allowed.

---

## 404

```text
Not Found
```

Resource doesn't exist.

---

## 500

```text
Internal Server Error
```

Server-side issue.

---

# 19. Important Interview Questions

### What is Spring Security?

Framework providing authentication and authorization.

---

### Difference between Authentication and Authorization?

Authentication:

```text
Who are you?
```

Authorization:

```text
What can you access?
```

---

### Why JWT?

Stateless authentication suitable for distributed systems and microservices.

---

### Why BCrypt?

Secure password hashing with salt.

---

### What is SecurityContext?

Stores authenticated user information.

---

### Why disable CSRF in JWT?

Because JWT authentication is stateless and doesn't depend on server sessions.

---

### What is Tamper-Proof JWT?

If payload is modified, signature validation fails and token becomes invalid.

---

# 20. Complete Request Flow

```text
Client Login
    ↓
POST /auth/login
    ↓
AuthenticationManager
    ↓
UserDetailsService
    ↓
Database Validation
    ↓
PasswordEncoder.matches()
    ↓
Generate JWT
    ↓
Return Token
    ↓
Client Stores Token
    ↓
Authorization: Bearer Token
    ↓
JWT Filter
    ↓
Validate JWT
    ↓
SecurityContext Updated
    ↓
Controller Access Granted
    ↓
Response Returned
```

---

# Summary

Spring Security provides:

- Authentication
- Authorization
- Password Encryption
- Security Filters
- JWT Integration
- Session Management

JWT provides:

- Stateless Authentication
- Scalability
- Security
- Tamper-Proof Verification

Together they form the standard security architecture used in modern Spring Boot microservices.
