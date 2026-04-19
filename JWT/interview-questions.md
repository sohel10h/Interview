# JWT Interview Questions

These are common JWT questions collected from recurring interview-prep sources and aligned with ASP.NET Core JWT guidance.

## Easy

1. What is JWT?
   Answer: JWT is a compact, URL-safe token format used to transfer claims between parties. It is commonly used in APIs for authentication and authorization.
2. What are the three parts of a JWT?
   Answer: Header, payload, and signature.
3. Is JWT encrypted by default?
   Answer: No. A standard JWT is usually signed, not encrypted. Anyone can decode the header and payload, but they should not be able to change them without breaking the signature.
4. What is the difference between authentication and authorization?
   Answer: Authentication checks who the user is. Authorization checks what the user is allowed to do.
5. What are claims in JWT?
   Answer: Claims are pieces of information stored in the token, such as user id, role, issuer, or expiry. They can be registered, public, or private claims.
6. What do `iss`, `aud`, and `exp` mean?
   Answer: `iss` means issuer, `aud` means audience, and `exp` means expiration time.
7. Where is a JWT usually sent in an HTTP request?
   Answer: Usually in the `Authorization` header as `Bearer <token>`.
8. What is the difference between `401 Unauthorized` and `403 Forbidden`?
   Answer: `401` means the request is not authenticated, usually because the token is missing, invalid, or expired. `403` means the token is valid, but the user does not have permission.
9. Why are JWTs useful for APIs and microservices?
   Answer: They are useful because they support stateless authentication, work well across distributed services, and carry claims that downstream services can use.
10. What is the role of the signature in JWT?
    Answer: The signature proves the token was issued by a trusted party and that the header and payload were not modified.

## Medium

1. What is the difference between registered, public, and private claims?
   Answer: Registered claims are standard names defined by the JWT spec, like `iss` and `exp`. Public claims are custom but should avoid collisions. Private claims are app-specific claims agreed on by the systems using them.
2. What is the difference between access token and refresh token?
   Answer: An access token is a short-lived token used to call APIs. A refresh token is used to obtain a new access token without forcing the user to log in again.
3. What are the advantages and disadvantages of JWT compared to session-based authentication?
   Answer: JWT scales well because it is stateless and easy to share across services. The downside is that immediate revocation is harder than with server-side sessions.
4. How do you validate JWT in ASP.NET Core?
   Answer: In ASP.NET Core, you usually configure `AddJwtBearer` and validate issuer, audience, lifetime, and signing key through `TokenValidationParameters`.
5. What happens when a JWT expires?
   Answer: Validation fails and the API should reject it, usually with `401 Unauthorized`. The client then needs a refresh token or a new login.
6. How should JWT be stored on the client side?
   Answer: It depends on the client type, but storage should minimize theft risk. For browsers, security tradeoffs matter, and many teams prefer secure, HTTP-only cookies over exposing tokens to JavaScript.
7. What is the difference between symmetric and asymmetric signing algorithms?
   Answer: Symmetric signing uses the same secret key to sign and validate. Asymmetric signing uses a private key to sign and a public key to validate.
8. What is the purpose of `nbf`, `iat`, and `jti`?
   Answer: `nbf` defines when the token becomes valid, `iat` shows when it was issued, and `jti` is a unique token id often used for replay protection or revocation tracking.
9. Can JWT carry roles or permissions?
   Answer: Yes. Roles, scopes, or permissions can be stored as claims, but the token still has to be validated before those claims are trusted.
10. What does `JwtBearerHandler` do in ASP.NET Core?
    Answer: It reads the bearer token, validates it, and creates the authenticated user principal from the claims if validation succeeds.

## Hard

1. How would you design JWT revocation in a mostly stateless system?
   Answer: Use short-lived access tokens, store refresh tokens server-side, support refresh-token rotation, and keep a revocation or blacklist mechanism for high-risk cases.
2. What are common JWT security vulnerabilities?
   Answer: Common issues include weak signature validation, poor client-side storage, replay attacks, trusting claims before validation, and insecure handling of algorithms or keys.
3. Why must audience validation matter in multi-service systems?
   Answer: Audience validation prevents a token meant for one service from being accepted by a different service.
4. When would you choose opaque tokens instead of JWT?
   Answer: Opaque tokens are a better fit when you want server-side introspection, simpler revocation, or you do not want claims visible to the client.
5. How would you secure refresh tokens?
   Answer: Store them securely, rotate them on use, track them server-side, give them limited lifetime, and revoke them on logout or suspicious activity.
6. What is the difference between signing and encryption in JWT-based systems?
   Answer: Signing protects integrity and authenticity. Encryption protects confidentiality. A signed token can still be readable.
7. How would you support multiple token issuers in one ASP.NET Core API?
   Answer: Configure multiple authentication schemes or a policy scheme, then validate tokens against the expected issuer-specific settings.
8. Why should you avoid putting sensitive business data into JWT payload?
   Answer: Because the payload is usually only encoded, not encrypted. Sensitive data can be exposed, and large payloads also make tokens harder to manage.
9. How would you explain JWT replay attack prevention?
   Answer: Use HTTPS, short token lifetimes, unique token ids like `jti`, strong refresh-token controls, and server-side detection or revocation where needed.
10. In production, why is a standard OAuth 2.0 / OpenID Connect flow preferred over hand-rolled JWT login?
    Answer: Because standard flows are battle-tested and provide safer token issuance, validation, rotation, and identity management than a custom login flow.

## Very Common JWT Trust Question

1. How does the server know this JWT was generated by our system?
   Answer: The server validates the token signature using the configured signing key. With symmetric signing, the same secret key used to generate the token is used again during validation. If the recomputed signature matches the token signature, the token was signed by someone who knows that secret key and the payload was not changed.
2. What is the actual process when using a secret key?
   Answer: The server takes the encoded header and payload, combines them as `header.payload`, and runs the signing algorithm such as HMAC-SHA256 with the secret key. That creates the signature. Later, during validation, the API repeats the same calculation with the same secret key. If the result matches the signature in the JWT, the token is considered authentic.
3. Is matching the signature alone enough?
   Answer: No. The server should also validate issuer, audience, expiry, and any other required rules before trusting the token.

## .NET Core Follow-Up Questions

These often appear after the basic JWT questions:

1. How do you configure JWT bearer authentication in `Program.cs`?
   Answer: Use `AddAuthentication` with `JwtBearerDefaults.AuthenticationScheme`, then call `AddJwtBearer` and configure validation settings such as issuer, audience, lifetime, and signing key.
2. What does `[Authorize]` do with JWT bearer auth?
   Answer: It requires the request to be authenticated. If the JWT is missing or invalid, access is denied. It can also enforce roles or policies.
3. How do you read claims from `HttpContext.User`?
   Answer: You can read them from `HttpContext.User.Claims` or use helpers like `FindFirst`.
4. How do you implement role-based authorization with JWT in ASP.NET Core?
   Answer: Put role claims into the token and use `[Authorize(Roles = "Admin")]` or policy-based authorization.
5. How do you return proper `401` and `403` responses in an API?
   Answer: Let the authentication middleware reject invalid tokens with `401`, and let authorization rules reject insufficient permissions with `403`.
6. What is `TokenValidationParameters` used for?
   Answer: It defines how ASP.NET Core should validate the JWT, including issuer, audience, lifetime, clock skew, and signing key rules.

## Recommended Answer Style

For interview answers:

- Start with the definition
- Explain why it is used
- Mention one real ASP.NET Core setup detail
- Mention one security concern or tradeoff

That answer pattern usually sounds stronger than giving only theory.

## Sources

Recurring question patterns were taken from:

- [LearnThatStack - JWT and Token-Based Auth Interview Questions](https://www.learnthatstack.com/interview-questions/security/jwt)
- [CLIMB - 20 JWT Token Interview Questions and Answers](https://climbtheladder.com/jwt-token-interview-questions/)
- [JavaInUse - Top JWT Interview Questions](https://www.javainuse.com/misc/jwt-interview-questions)

Technical alignment and validation points were checked against:

- [Microsoft Learn - Configure JWT bearer authentication in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication?view=aspnetcore-10.0)
- [RFC 7519 - JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519)
- [RFC 8725 - JWT Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725)
