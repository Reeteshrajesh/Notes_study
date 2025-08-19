# Keycloak – Detailed Documentation

## 1. Introduction

Keycloak is an **open-source Identity and Access Management (IAM)** solution. It provides features like:

* **Single Sign-On (SSO)**
* **User Federation**
* **Identity Brokering** (Google, GitHub, Microsoft, etc.)
* **Role-Based Access Control (RBAC)**
* **Multi-Factor Authentication (MFA)**

I set up Keycloak locally to understand how it works. This document will help me (and others) for future reference.

---

## 2. Why Keycloak? (Benefits)

* ✅ **Centralized Authentication** → No need to manage login in each app.
* ✅ **Single Sign-On (SSO)** → Login once, access multiple applications.
* ✅ **SSO with Identity Providers** → Google, GitHub, Azure AD, etc.
* ✅ **Role/Group Management** → Control which users see which apps.
* ✅ **Secure Token-based Authentication** (OIDC, SAML).
* ✅ **Restrict access to domains** (e.g., only `@xyz.com` and `@abc.com` users allowed).
* ✅ **Self-hosted & Open Source** → No vendor lock-in, runs in Docker or Kubernetes.

---

## 3. High-Level Flow

Example workflow:

1. User clicks **Login with Google**.
2. Google authenticates → Keycloak gets identity.
3. Keycloak checks **roles / groups**.
4. User is redirected to application with a **SSO token (JWT)**.
5. App verifies token (via Keycloak public keys) → User logged in.

🔑 Example:

* Reetesh → Access to Grafana + Jitsu.
* Manthan → Access to Retool + Metabase.

---

## 4. Key Concepts

* **Realm** → Isolated security domain (like a tenant).
* **Client** → Applications (Grafana, Retool, etc.).
* **User** → Employees / Students who log in.
* **Role / Group** → Defines access levels.
* **Identity Provider (IdP)** → Google, GitHub, etc.
* **Token (JWT / SAML)** → Secure way apps validate login.

---

## 5. Restriction by Domain

* You can configure Keycloak to only allow logins from specific email domains.
  Example: only `@xyz.com` and `@abc.com` users can log in.

This ensures external users cannot access the system.

---

## 6. Local Setup (Docker-Compose)

I created a local setup with **Docker-Compose** to test Keycloak.

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15
    restart: always
    environment:
      POSTGRES_DB: keycloak
      POSTGRES_USER: keycloak
      POSTGRES_PASSWORD: keycloakpass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  keycloak:
    image: quay.io/keycloak/keycloak:26.0.2
    command: start --optimized
    restart: always
    environment:
      KC_DB: postgres
      KC_DB_URL_HOST: postgres
      KC_DB_URL_PORT: 5432
      KC_DB_URL_DATABASE: keycloak
      KC_DB_USERNAME: keycloak
      KC_DB_PASSWORD: keycloakpass

      # Admin credentials
      KEYCLOAK_ADMIN: admin
      KEYCLOAK_ADMIN_PASSWORD: adminpass

      # Hostname (important in production)
      KC_HOSTNAME: auth.mydomain.com
      KC_PROXY: edge
    ports:
      - "8080:8080"
    depends_on:
      - postgres

volumes:
  postgres_data:

```

🔹 Run with:

```bash
docker-compose up -d
```

🔹 Access at:

```
http://localhost:8080
```

(Admin console → user: `admin`, pass: `adminpass`)

👉 If anyone needs my docker-compose file, let me know.

---

## 7. Integration with Apps

To integrate an app:

1. Create a **Client** in Keycloak.
2. Configure client protocol → `openid-connect`.
3. Add app redirect URLs.
4. App (e.g., Grafana, Retool) uses Keycloak’s OIDC endpoint.
5. Keycloak issues a JWT after login.

---

## 8. Advanced Features

* MFA (OTP, SMS, Authenticator apps).
* Social Login (Google, GitHub, LinkedIn).
* LDAP/AD Federation.
* User Self-service (password reset, profile update).
* Session & Token Management.

---

## 9. Use Cases for Future

* **Organization Portal** → Central dashboard for employees, login via Keycloak.
* **Restrict Access** → Only company email logins.
* **Observability Tools** → Grafana, Loki, Jitsu access controlled via Keycloak.
* **Microservices Authentication** → Issue JWT tokens for API security.
* **Production Setup** → Deploy Keycloak in Kubernetes with HA setup.

---

## 10. Conclusion

* I successfully set up **Keycloak locally** using Docker-Compose.
* Tested Google login, roles, and access control.
* Next step → deploy in **production (EKS / Kubernetes)** for real-world usage.
