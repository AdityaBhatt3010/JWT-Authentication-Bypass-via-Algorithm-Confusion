# 🔐 JWT Authentication Bypass via Algorithm Confusion

## Introduction

JWTs (JSON Web Tokens) are a popular way to handle authentication in modern web applications, often using asymmetric algorithms like **RS256** to ensure integrity and authenticity. However, a classic yet overlooked vulnerability—**algorithm confusion**—can allow attackers to **bypass authentication entirely** when servers fail to enforce proper algorithm verification.

In this **PortSwigger Expert-level lab**, we exploit **algorithm confusion** by crafting a JWT that tricks the server into verifying an RS256 token as HS256, using the **server's own public key** as the HMAC secret. This vulnerability arises when developers accept tokens signed using both symmetric (HMAC) and asymmetric (RSA) algorithms **without proper validation of the `alg` header**.

In short: we become an **admin without ever knowing the private key**. Let's break it down.

![JWT_NewNewNew_Cover](https://github.com/user-attachments/assets/2dd898a2-b17b-4c31-8883-fa12cd90e247) <br/>

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

![1](https://github.com/user-attachments/assets/d81a5a94-6366-4db8-9ccf-bcd20006d8cc) <br/>

### 2. Try Accessing /admin

Change the path to `/admin` and note the message:

```
Admin interface only available if logged in as an administrator.
```

![2](https://github.com/user-attachments/assets/d27a8a61-a350-48ba-b9c9-40f3ac3cf367) <br/>

### 3. Send /admin to Repeater

Capture the `/admin` request and forward it to **Burp Repeater**.

![3](https://github.com/user-attachments/assets/db325f1d-cf94-4300-b7d5-7f4d671c697a) <br/>

### 4. Locate the Server's Public Key

In your browser, visit the **standard JWK endpoint**:

```
https://<your-lab-id>.web-security-academy.net/jwks.json
```

Copy the key object (excluding the outer array):

```json
{"kty":"RSA","e":"AQAB","use":"sig","kid":"8660d963-1b64-4e93-96b2-1bb9fb258620","alg":"RS256","n":"rM3x..."}
```

![4](https://github.com/user-attachments/assets/1be8467c-0d07-4735-9a84-b36a502b1bca) <br/>

### 5. Generate a Malicious Signing Key

In **JWT Editor → Keys tab**:

* Click **New RSA Key**
* Paste the copied JWK
* Save → Right-click → **Copy Public Key as PEM**

![5](https://github.com/user-attachments/assets/514cdf23-5b1d-4d6c-9c51-c3002bb28f5d) <br/>

### 6. Base64 Encode the Public Key

Use **Decoder** in Burp Suite:

* Paste the PEM
* Encode to **Base64**
* Copy the resulting string

![6](https://github.com/user-attachments/assets/12b5fe26-8818-4818-9d65-1e1c426e9921) <br/>

### 7. Create Symmetric Key with PEM as Secret

Back in **JWT Editor → Keys tab**:

* Click **New Symmetric Key**
* Replace the `k` value with the **Base64-encoded PEM**
* Save

![7](https://github.com/user-attachments/assets/173f6991-a53e-4f60-b03a-473224e60b1a) <br/>

### 8. Modify JWT for Admin Access

Back in **Repeater → JSON Web Token** tab:

* Set `alg` to `HS256`
* Change `sub` to `administrator`
* Click **Sign**
* Choose your symmetric key and **Don’t modify header**

![8](https://github.com/user-attachments/assets/8dcb33bf-b857-48a7-a846-4eb57fde702e) <br/>

### 9. Send the Token

Hit **Send** in Repeater.

🎉 **Success!** You've bypassed authentication and accessed `/admin` as the administrator.

![9](https://github.com/user-attachments/assets/8fc7498c-2f94-4d3e-849e-d36268c96e16) <br/>

### 10. Delete Carlos

Change URL to:

```
/admin/delete?username=carlos
```

Send the request.

![10](https://github.com/user-attachments/assets/1414fa2a-1a6d-4b1e-b021-edf358090397) <br/>

### 11. Confirm the Lab is Solved

Right-click → **Show in browser** → Open the URL.

![11](https://github.com/user-attachments/assets/faa68ebb-d61a-4fe1-8b5b-b086451d9fed) <br/>

You’ll see the lab marked as **solved** ✅.

![12](https://github.com/user-attachments/assets/b552ac51-5985-4429-a69c-3e9013884a55) <br/>

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

> ✨ **Happy Hacking** and may your payloads always execute! <br/>
> Written by: Aditya Bhatt | InfoSec WriteUps Contributor <br/>
> 🐦 [Follow me on Medium](https://medium.com/@yourhandle) | 🔒 Ethical Hacking | 🧠 Cybersecurity Researcher <br/>
