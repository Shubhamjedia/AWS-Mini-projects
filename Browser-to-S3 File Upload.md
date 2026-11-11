# AWS S3 Direct File Upload Project Report

## 1. Executive Summary
This project demonstrates the implementation of a secure, serverless direct file upload pipeline from a browser-based frontend to an Amazon S3 bucket. By leveraging Identity Federation with Google OAuth 2.0 / OpenID Connect (OIDC) and AWS Security Token Service (STS), the system completely eliminates hardcoded AWS access keys and avoids proxying file traffic through a backend server.

> **Key Security Takeaway:** AWS STS `assumeRoleWithWebIdentity` exchanges verified Google ID Tokens for short-lived AWS temporary credentials, granting browser-level execution strictly aligned with Zero-Trust cloud security standards.

---

## 2. System Architecture & Components
The solution brings together three core service pillars:

| Layer / Component | Technology | Role & Responsibility |
| :--- | :--- | :--- |
| **Identity Provider (IdP)** | Google OIDC | Authenticates end-users and issues cryptographically signed JWT ID tokens. |
| **Token Exchange & Security** | AWS STS & IAM OIDC IdP | Verifies Google JWT token authenticity and returns temporary AWS access credentials. |
| **Cloud Object Storage** | Amazon S3 | Provides target storage (`user-data-11223344`) secured with fine-grained CORS rules. |

---

## 3. Architecture Flowchart & Working

```text
+-------------------------------------------------------+
|                    Browser / Web UI                   |
|  (HTML5, Vanilla JS, Google Sign-In Client SDK)       |
+-------------------------------------------------------+
                           |
                           | (1) User clicks "Sign in with Google"
                           v
+-------------------------------------------------------+
|                    Google OIDC IdP                    | ---> Issues cryptographically signed JWT ID Token
+-------------------------------------------------------+
                           |
                           | (2) Returns ID Token back to Frontend UI
                           v
+-------------------------------------------------------+
|                    Browser / Web UI                   |
+-------------------------------------------------------+
                           |
                           | (3) Calls `STS.assumeRoleWithWebIdentity()` with Google JWT
                           v
+-------------------------------------------------------+
|                        AWS STS                        |
+-------------------------------------------------------+
                           |
                           | (4) Validates token signature & audience
                           v
+-------------------------------------------------------+
|                 AWS IAM OIDC Provider                 | ---> Confirms trust with `accounts.google.com`
+-------------------------------------------------------+
                           |
                           | (5) Verifies Trust Policy & Maps Permissions
                           v
+-------------------------------------------------------+
|                     AWS IAM Role                      | ---> Assigns temporary session credentials (`s3:PutObject`)
+-------------------------------------------------------+
                           |
                           | (6) Returns Ephemeral Access Keys & Session Token
                           v
+-------------------------------------------------------+
|                    Browser / Web UI                   |
+-------------------------------------------------------+
                           |
                           | (7) Direct Secure Upload (PUT Request with CORS)
                           v
+-------------------------------------------------------+
|                   Amazon S3 Bucket                    | ---> File safely stored in `google-users-uploads/`
+-------------------------------------------------------+
