# 🔐 JWT Authentication Bypass via Algorithm Confusion

## Introduction

JWTs (JSON Web Tokens) are a popular way to handle authentication in modern web applications, often using asymmetric algorithms like **RS256** to ensure integrity and authenticity. However, a classic yet overlooked vulnerability—**algorithm confusion**—can allow attackers to **bypass authentication entirely** when servers fail to enforce proper algorithm verification.

In this **PortSwigger Expert-level lab**, we exploit **algorithm confusion** by crafting a JWT that tricks the server into verifying an RS256 token as HS256, using the **server's own public key** as the HMAC secret. This vulnerability arises when developers accept tokens signed using both symmetric (HMAC) and asymmetric (RSA) algorithms **without proper validation of the `alg` header**.

In short: we become an **admin without ever knowing the private key**. Let's break it down.

---

## 🎯 Lab Objective

**Lab**: [JWT authentication bypass via algorithm confusion](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-kid-header-path-traversal)
**Difficulty**: Expert
**Goal**: Exploit algorithm confusion in JWT validation to gain admin access and delete user `carlos`.

---

## 🧪 Step-by-Step Exploit (PoC)

> **Background**: Install the **JWT Editor extension** from Burp Suite’s BApp store for seamless token manipulation.

![JWTEditor](https://github.com/user-attachments/assets/1d3474f0-4947-46a4-b37e-a4e02ed49909) <br/>

### 1. Access the Lab and Log In

Visit the lab and log in using:

```
Username: wiener  
Password: peter
```

### 2. Try Accessing /admin

Change the path to `/admin` and note the message:

```
Admin interface only available if logged in as an administrator.
```

### 3. Send /admin to Repeater

Capture the `/admin` request and forward it to **Burp Repeater**.

### 4. Locate the Server's Public Key

In your browser, visit the **standard JWK endpoint**:

```
https://<your-lab-id>.web-security-academy.net/jwks.json
```

Copy the key object (excluding the outer array):

```json
{"kty":"RSA","e":"AQAB","use":"sig","kid":"8660d963-1b64-4e93-96b2-1bb9fb258620","alg":"RS256","n":"rM3x..."}
```

### 5. Generate a Malicious Signing Key

In **JWT Editor → Keys tab**:

* Click **New RSA Key**
* Paste the copied JWK
* Save → Right-click → **Copy Public Key as PEM**

### 6. Base64 Encode the Public Key

Use **Decoder** in Burp Suite:

* Paste the PEM
* Encode to **Base64**
* Copy the resulting string

### 7. Create Symmetric Key with PEM as Secret

Back in **JWT Editor → Keys tab**:

* Click **New Symmetric Key**
* Replace the `k` value with the **Base64-encoded PEM**
* Save

### 8. Modify JWT for Admin Access

Back in **Repeater → JSON Web Token** tab:

* Set `alg` to `HS256`
* Change `sub` to `administrator`
* Click **Sign**
* Choose your symmetric key and **Don’t modify header**

### 9. Send the Token

Hit **Send** in Repeater.

🎉 **Success!** You've bypassed authentication and accessed `/admin` as the administrator.

### 10. Delete Carlos

Change URL to:

```
/admin/delete?username=carlos
```

Send the request.

### 11. Confirm the Lab is Solved

Right-click → **Show in browser** → Open the URL.
You’ll see the lab marked as **solved** ✅.

---

## 🔍 Technical Insight

This attack is possible due to the **JWT algorithm field (`alg`) being trusted blindly**. By switching the algorithm from `RS256` to `HS256`, and using the **server’s public key as the HMAC secret**, we forge a valid token without ever knowing the private key.

### 💣 The server expects:

* `RS256`: RSA with public/private keypair

### 🚨 We send:

* `HS256`: HMAC with the public key as the symmetric secret

The server verifies the token with the public key and... accepts it. 🤯

---

## 💡 Mitigations

1. **Do not allow multiple JWT algorithms** unless absolutely necessary.
2. **Validate the `alg` field** on the server side and enforce strict expected values.
3. Use mature JWT libraries that are **explicitly configured for RS256 only**.
4. **Rotate signing keys regularly** and monitor for anomalies in JWT handling.

---

## 🤝 Final Thoughts

JWTs are powerful, but **power comes with responsibility**. One subtle misconfiguration, like accepting multiple algorithms or mismanaging key usage, can open the gates to full admin access.

**Keep auditing your JWT handling logic, enforce strict cryptographic standards, and stay up-to-date with token-based vulnerabilities.**

---

> ✨ **Happy Hacking** and may your payloads always execute!
> Written by: Aditya Bhatt | InfoSec WriteUps Contributor
> 🐦 [Follow me on Medium](https://medium.com/@yourhandle) | 🔒 Ethical Hacking | 🧠 Cybersecurity Researcher
