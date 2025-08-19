# 🔐 Authentication Flows in a 4-Page Web Application

Your application has 4 main pages:

1. **Login/Signup Page**
2. **Dashboard / Home**
3. **Other Functional Pages**
4. **APIs (protected, require authentication)**

It supports **two login options**:

* **Normal Login (Email + Password / OTP)**
* **Google Auth (OAuth 2.0 / OpenID Connect)**

Both flows ultimately generate **Access & Refresh Tokens**, which are used to secure all subsequent API calls.

---

## 1️⃣ Normal Login Flow (Email + Password/OTP)

### 🔹 Steps

1. **User Registration (Signup)**

   * User enters **email + password** (or OTP-based signup).
   * Backend hashes the password (e.g., bcrypt).
   * User record stored in **database**.

2. **Login**

   * User submits **email + password** (or OTP).
   * Backend validates credentials:

     * If valid → backend generates:

       * **Access Token** (short-lived, e.g., 15 mins).
       * **Refresh Token** (long-lived, e.g., 7 days).
   * Tokens returned to frontend (usually stored in memory or HTTP-only cookies).

3. **API Calls**

   * Frontend attaches **Access Token** (in `Authorization: Bearer <token>`).
   * Backend validates token signature & expiry before responding.

4. **Token Expiry**

   * If access token expires, frontend silently requests a new one using **Refresh Token**.
   * Refresh Token validated → new Access Token issued.

---

### 🔹 Flow Diagram (Normal Login)

```
[User] → enters email/password → [Frontend] → [Backend API]
    → verify user in DB
    → issue Access + Refresh Tokens
    → store tokens in frontend
    → use Access Token for future API calls
```

---

## 2️⃣ Google Authentication Flow (OAuth 2.0 / OpenID Connect)

### 🔹 Steps

1. **User Chooses Google Login**

   * On login page, user clicks **"Sign in with Google"**.
   * Frontend redirects to **Google Auth Server** with `client_id`, `redirect_uri`, and `scopes`.

2. **Google Authentication**

   * User signs in with Google account.
   * Google asks user to consent (e.g., access email, profile).
   * After success → Google redirects back to your app’s `redirect_uri` with an **Authorization Code**.

3. **Token Exchange**

   * Frontend sends **Authorization Code** to your backend.
   * Backend exchanges it with Google’s **Token Endpoint**:

     * Receives:

       * **ID Token (JWT, user identity)**
       * **Access Token (Google API access)**
       * **Refresh Token (optional, if requested)**

4. **Backend Issues App Tokens**

   * Backend checks Google’s ID Token (verify signature, expiry).
   * If valid → create/find user in your app’s database.
   * Issue your own:

     * **Access Token** (15 mins, for your APIs).
     * **Refresh Token** (7 days, for your APIs).

5. **API Calls**

   * Frontend uses **your app’s Access Token** (not Google’s).
   * API validates tokens just like in normal login.

---

### 🔹 Flow Diagram (Google Auth)

```
[User] → clicks "Login with Google" → [Google Auth Server]
    → user signs in + consents
    → returns Authorization Code → [Frontend] → [Backend]
    → backend exchanges with Google
    → gets ID Token + Access Token
    → backend verifies & issues app tokens
    → Access Token used for API calls
```

---

## 3️⃣ Key Differences

| Aspect                 | Normal Login                           | Google Auth (OAuth)                               |
| ---------------------- | -------------------------------------- | ------------------------------------------------- |
| User Credentials       | Email + Password/OTP stored in DB      | Google handles authentication                     |
| Token Issuer           | Your backend                           | Google (ID Token) + Your backend (App Tokens)     |
| User Verification      | Manual (DB lookup, bcrypt hash check)  | Google verifies identity, backend trusts ID Token |
| Security               | Depends on password strength & hashing | Stronger (backed by Google’s infra)               |
| UX                     | User must remember password/OTP        | One-click login with Google                       |
| Integration Complexity | Simple (just DB + JWTs)                | Higher (OAuth flow, Google APIs, callbacks)       |

---

## 4️⃣ Best Practices

* 🔐 **Always use HTTPS** for token exchange.
* ⚡ **Access Tokens** should be **short-lived** (e.g., 15 mins).
* 🔄 **Refresh Tokens** should be **rotated** and revocable.
* 🍪 Prefer **HTTP-only secure cookies** for tokens over localStorage.
* 🛑 **Logout** should revoke refresh tokens.
* 📌 If using Google Auth, **link Google user ID** to your app’s user record.

---

✅ **Conclusion**

* **Normal Login** → full control, but requires secure password management.
* **Google Auth** → easier for users, more secure identity, but adds OAuth complexity.
* Both end up with the same outcome: **your backend issues Access + Refresh tokens that protect your APIs.**
