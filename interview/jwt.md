---
title: JWT Explained - Your Digital Passport for Secure Web Authentication
description: Unlock the secrets of JSON Web Tokens (JWTs)! We dive into their structure (Header, Payload, Signature), how they enable stateless authentication, common workflows, benefits, and crucial security considerations.
date: 2024-06-08
draft: false
slug: /pensieve/jwt
tags:
  - interview
  - SDE
---

Hey everyone, and welcome back to the blog! In the world of modern web and mobile applications, especially as we build more distributed systems and APIs, managing user identity securely and efficiently is paramount. One of the most popular and powerful tools in our arsenal for this is the **JSON Web Token**, or **JWT** (often pronounced "jot").

If you've ever wondered how you stay logged into multiple services after signing in once, or how applications securely transmit information about your identity without needing to hit a database every single time, JWTs are often playing a key role. Let's decode these digital passports!

## What is a JSON Web Token (JWT)? Your Digital Passport

A **JSON Web Token (JWT)** is an open standard (RFC 7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. Think of it like a special, digitally sealed box or a passport. This information can be verified and trusted because it is **digitally signed**.

One of the standout features often associated with JWTs is their ability to facilitate **stateless authentication**. Since a JWT can contain user claims and its integrity is verifiable through its signature, the server often doesn't need to store session information about the user. The token itself carries the necessary proof of authentication.

## Deconstructing a JWT: The Three-Part Harmony

A JWT looks like a long, somewhat cryptic string of characters. However, it's actually composed of three distinct parts, separated by dots (`.`):

`xxxxx.yyyyy.zzzzz`

Each of these parts is Base64Url encoded:

1.  **Header (The Label):**
    * This first part contains metadata about the token itself.
    * It typically consists of two pieces of information:
        * `typ`: The type of the token, which is invariably `JWT`.
        * `alg`: The signing algorithm being used to generate the signature, such as HMAC SHA256 (e.g., `HS256`) or RSA.
    * An example of the JSON for a header would be:
        ```json
        {
          "alg": "HS256",
          "typ": "JWT"
        }
        ```
    * This JSON object is then Base64Url encoded to form the first part of the JWT.

2.  **Payload (The Message/Claims):**
    * This second part contains the "claims". Claims are statements about an entity (typically the user) and additional data. Think of this as the actual information or message you want to share.
    * There are three types of claims:
        * **Registered Claims:** These are a set of predefined claims which are not mandatory but are recommended for providing a set of useful, interoperable claims. Examples include:
            * `iss` (Issuer): Who issued the token.
            * `exp` (Expiration Time): When the token expires (vital for security!).
            * `sub` (Subject): Who the token is about (e.g., user ID).
            * `aud` (Audience): Who the token is intended for.
            * `iat` (Issued At): When the token was issued.
        * **Public Claims:** These are claims that can be defined at will by those using JWTs. To avoid collisions, they should either be defined in the IANA JSON Web Token Registry or be namespaced (e.g., using a URI).
        * **Private Claims:** These are custom claims created to share information between parties that have agreed on using them. They are specific to your application.
    * An example JSON payload might look like:
        ```json
        {
          "sub": "user123",
          "name": "Aditi Rao",
          "admin": true,
          "iat": 1516239022,
          "exp": 1516242622
        }
        ```
    * This JSON payload is also Base64Url encoded to form the second part of the JWT.

3.  **Signature (The Tamper-Proof Seal):**
    * This is what makes the JWT secure and allows its authenticity and integrity to be verified. It's like a special seal that only the sender (or someone with the secret/private key) knows how to create.
    * The signature ensures that the header and payload haven't been tampered with since the token was issued. If the JWT is signed with an asymmetric key pair (public/private keys), it also verifies that the sender of the JWT is who it says it is.
    * **How it's created:**
        To create the signature, you take:
        * The Base64Url encoded header.
        * A dot (`.`).
        * The Base64Url encoded payload.
        * And then sign this combined string using the algorithm specified in the header (`alg`) along with a secret (for HMAC algorithms like HS256) or a private key (for asymmetric algorithms like RS256 or ES256).
        * For example, for HMAC SHA256:
            `HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`

## The JWT Workflow: How It's Used in Authentication

Here’s a typical flow for how JWTs are used in an authentication system:

1.  **Login Request:** The user provides their credentials (e.g., username and password) to the client application (web or mobile). The client sends these credentials to an authentication server.
2.  **Credential Validation & JWT Issuance:** The authentication server validates the user's credentials. If successful, the server generates a JWT. It constructs the header and payload, then signs it using a secret key (for symmetric algorithms) or its private key (for asymmetric algorithms). Critically, because the relevant user information and permissions (claims) can be embedded in the JWT, the server often **doesn't need to store any session information** for this token.
3.  **JWT Returned to Client:** The server sends the generated JWT back to the client application.
4.  **Client Stores JWT:** The client application receives the JWT and typically stores it locally. Common storage locations include:
    * `localStorage` or `sessionStorage` in web browsers.
    * Secure storage on mobile devices.
    * Sometimes as an HTTP-only cookie (to mitigate XSS risks of accessing it via JavaScript).
5.  **Subsequent API Requests:** For every subsequent request to a protected API route or resource, the client includes the JWT. This is most commonly done by sending it in the `Authorization` HTTP header using the `Bearer` schema:
    `Authorization: Bearer <your_jwt_token>`
6.  **Server Verifies JWT:** When the API server receives a request containing a JWT, it must first validate the token before processing the request:
    * **Verify Signature:** The server uses the shared secret (for HMAC) or the corresponding public key (for RSA/ECDSA) to verify that the signature is valid and that the header and payload haven't been tampered with.
    * **Validate Claims:** The server checks standard claims like `exp` (expiration time) to ensure the token is still valid, `iss` (issuer), and `aud` (audience) if they are being used.
7.  **Access Granted/Denied:**
    * If the token is valid (signature is correct, not expired, etc.), the server trusts the claims within the JWT. It can then use the information in the payload (like user ID or roles) to authorize the request and process it.
    * If the token is invalid, the server rejects the request, typically with a `401 Unauthorized` or `403 Forbidden` status code.

## Why JWTs? Key Benefits

JWTs have become popular for several good reasons:

* **Statelessness:** Because the token is self-contained and includes user information (claims) and is signed, the server often doesn't need to maintain session state or look up session data in a database for every request. This simplifies backend architecture and improves scalability, as any server instance can validate a token.
* **Compactness:** JWTs are relatively small (especially compared to older XML-based tokens like SAML), which means they are efficiently transmitted in HTTP headers or URL parameters.
* **Self-Contained:** The payload can carry necessary user details and permissions, potentially reducing the need for extra database lookups on the server after initial token validation.
* **Widely Adopted Standard (RFC 7519):** Being an open standard promotes interoperability between different systems and languages.
* **Versatility:** Can be used for authentication and secure information exchange.

## Security Considerations for JWTs

While powerful, JWTs must be handled correctly to maintain security:

* **Always Use HTTPS:** JWTs should always be transmitted over an encrypted HTTPS connection to protect them from being intercepted by man-in-the-middle attacks.
* **Secure Secret/Key Management:**
    * For HMAC algorithms (like HS256), the `secret` used for signing must be kept confidential. If compromised, anyone can forge tokens.
    * For asymmetric algorithms (like RS256), the `private key` used for signing must be kept highly secure, while the `public key` can be distributed for verification.
* **Token Expiry (`exp` claim):** *Always* include an expiration claim (`exp`) in your JWTs and enforce it on the server. Short-lived access tokens are generally more secure. For longer user sessions, implement a refresh token mechanism.
* **Avoid Storing Sensitive Data in Payload (Unless Encrypted):** The payload of a JWT is Base64Url encoded, *not encrypted* by default. This means anyone who gets hold of the token can easily decode the payload and read its contents. **Do not store highly sensitive information (like passwords or credit card numbers) directly in the payload.** If sensitive data must be included, consider using JSON Web Encryption (JWE) to encrypt the payload itself.
* **Algorithm Choice (`alg` claim):** Always validate the `alg` claim in the header on the server-side and ensure you are using strong, vetted signing algorithms. Critically, never trust the `alg` header from an incoming token to determine how to validate it without checking it against your expected algorithm(s). Specifically, ensure that the `none` algorithm (which means no signature) is disabled in production environments, as it was a historical attack vector.
* **Token Storage on Client-Side:**
    * Storing JWTs in `localStorage` makes them accessible via JavaScript, which can be a risk if your site has Cross-Site Scripting (XSS) vulnerabilities.
    * Storing JWTs in HTTP-only cookies can offer better protection against XSS, as JavaScript cannot access them. However, this makes them susceptible to Cross-Site Request Forgery (CSRF) attacks unless CSRF protection measures are also implemented. Choose your storage mechanism carefully based on your application's security posture.

## Key Takeaways

* JSON Web Tokens (JWTs) are a compact, URL-safe standard for representing claims to be transferred between two parties, commonly used for authentication and information exchange.
* A JWT consists of three Base64Url-encoded parts: Header, Payload, and Signature, separated by dots.
* The signature ensures the token's integrity and authenticity.
* JWTs enable stateless authentication, as the server can verify the token without needing to store session state.
* While offering many benefits, it's crucial to implement JWTs with security best practices in mind, including using HTTPS, managing keys securely, setting token expirations, and being cautious about payload content.

JWTs are a powerful and flexible tool in the modern developer's toolkit for building secure and scalable applications.
