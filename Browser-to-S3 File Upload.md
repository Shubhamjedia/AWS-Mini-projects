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

## 3. Implementation Workflow & Deployment Screenshots

### Step 1: User Authentication via Google Sign-In
The user authenticates on the client web interface using Google OAuth 2.0. Upon successful login, the application acquires a valid Google ID token and updates the UI status

<img width="686" height="448" alt="login" src="https://github.com/user-attachments/assets/718a53af-9234-4fa9-b0a2-422bffcfa067" />

*Figure 1: Client web page showing successful Google authentication status.*

---

### Step 2: STS Token Exchange & Direct S3 File Upload
The client passes the Google ID token to AWS STS via `assumeRoleWithWebIdentity` to obtain temporary keys. The selected file (`modified sop.docx`) is then directly uploaded to S3, triggering a browser confirmation popup[cite: 7].

<img width="1362" height="521" alt="s3" src="https://github.com/user-attachments/assets/f3f88e34-ce01-4e32-b6cd-0b496deb66ea" />

*Figure 2: Browser popup confirming successful direct file upload to S3 using temporary AWS keys[cite: 7].*

---

### Step 3: Verification of S3 Directory Structure
Verification inside the Amazon S3 Management Console for bucket `user-data-11223344` shows the dynamically created prefix folder `google-users-uploads/`[cite: 8].

<img width="1912" height="816" alt="s3 verify" src="https://github.com/user-attachments/assets/00646acf-e64b-4b03-bd03-96b5073a5305" />

*Figure 3: AWS S3 Bucket Console verifying the `google-users-uploads/` folder[cite: 8].*

---

### Step 4: Verification of Uploaded File Object
Navigating inside `google-users-uploads/` confirms that the file `1794399910854_modified sop.docx` (229.0 KB) was saved successfully[cite: 9].
<img width="1917" height="812" alt="s3 verify 2" src="https://github.com/user-attachments/assets/e17273e0-b2cc-4728-b510-e78b72a09c5e" />

*Figure 4: Detailed view inside Amazon S3 confirming uploaded file object and timestamp[cite: 9].*

## 4. Architecture Flowchart & Working

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

