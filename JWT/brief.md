# JWT Brief for C# and .NET Core

## What JWT Is

JWT stands for JSON Web Token. It is a compact, URL-safe token format used to transfer claims between two parties. In real .NET interviews, JWT is usually discussed in the context of API authentication and authorization.

Important point: JWT is not encryption by default. A normal JWT is usually signed so the server can verify integrity, but the payload can still be decoded by anyone holding the token.

## JWT Structure

A JWT usually has 3 parts separated by dots:

`header.payload.signature`

- Header: metadata like token type and signing algorithm
- Payload: claims such as user id, role, issuer, audience, expiry
- Signature: proves the token was issued by a trusted source and was not changed

## Common Claim Types

JWT claims are usually grouped into:

- Registered claims
- Public claims
- Private claims

Important registered claims:

- `iss`: issuer
- `sub`: subject
- `aud`: audience
- `exp`: expiration time
- `nbf`: not before
- `iat`: issued at
- `jti`: unique token id

## JWT in ASP.NET Core

In ASP.NET Core, JWT is commonly used with bearer authentication for APIs.

Typical setup:

```csharp
builder.Services
    .AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://your-auth-server";
        options.Audience = "your-api";
    });
```

The `JwtBearerHandler` validates the token and builds the user identity from claims.

## How the Server Knows the JWT Is Ours

This is a very common interview question.

When your server creates a JWT, it signs the token using a key.

With a symmetric algorithm like `HS256`:

- your app has a secret key
- the header and payload are Base64Url-encoded
- the server creates a signature from `header.payload` using HMAC-SHA256 and the secret key
- the final token becomes `header.payload.signature`

When the token comes back to the API:

1. The API reads the token
2. It separates header, payload, and signature
3. It uses the same secret key to generate the signature again
4. It compares the newly generated signature with the signature inside the token
5. If both match, the token was signed by someone who knows the secret key and the payload was not changed

Important detail: if even one character in the payload changes, the generated signature changes too, so validation fails.

This is how the server knows the JWT is from a trusted issuer that owns the secret key.

In ASP.NET Core, this happens through token validation settings such as:

```csharp
options.TokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuer = true,
    ValidateAudience = true,
    ValidateLifetime = true,
    ValidateIssuerSigningKey = true,
    IssuerSigningKey = new SymmetricSecurityKey(
        Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
};
```

## Symmetric vs Asymmetric Validation

- Symmetric signing: same secret key is used to sign and validate
- Asymmetric signing: private key signs, public key validates

Interview note:

- If you generate tokens inside your own app, symmetric signing is common in small systems
- In larger production systems or identity providers, asymmetric signing is often preferred because APIs can validate with a public key without storing the private signing key

## What Must Be Validated

In interviews, this is one of the most important parts.

Your API should validate:

- Signature
- Issuer
- Audience
- Expiration
- Signing key / issuer signing key

If signature validation fails, the server should treat the token as untrusted even if the payload looks correct.

If token validation fails, the API should return `401 Unauthorized`.

If the token is valid but the user does not have enough permission, the API should return `403 Forbidden`.

## Authentication vs Authorization

- Authentication: verifies who the user is
- Authorization: decides what the user can do

JWT helps with both:

- the token proves identity after validation
- claims like roles or scopes help authorize access

## Access Token vs Refresh Token

- Access token: short-lived token sent with API calls
- Refresh token: used to request a new access token without logging in again

Good interview answer: access tokens should usually be short-lived, and refresh tokens need stronger protection and revocation handling.

## Advantages

- Stateless for APIs
- Good for distributed systems and microservices
- Easy to send in `Authorization: Bearer <token>`
- Claims travel with the token

## Limitations

- Harder to revoke immediately than server-side sessions
- Token size can grow if too much data is added
- Sensitive data should not be placed in payload
- Bad storage on the client can create security issues

## Common Security Best Practices

- Always use HTTPS
- Keep access tokens short-lived
- Validate `iss`, `aud`, `exp`, and signature every time
- Do not trust claims without validation
- Do not put passwords or sensitive secrets in the payload
- Prefer standard auth flows such as OAuth 2.0 / OpenID Connect
- Prefer asymmetric keys in production when possible
- Be careful with browser storage; avoid unsafe token storage patterns
- Support refresh-token rotation or revocation when needed

## Common Interview Mistakes

- Saying JWT is encrypted by default
- Confusing authentication with authorization
- Forgetting audience and issuer validation
- Returning `403` for an invalid token instead of `401`
- Treating JWT as a replacement for all session-based approaches
- Ignoring token revocation strategy

## Quick Interview Summary

If asked for a short answer:

"JWT is a signed token format used mostly for stateless authentication in APIs. In ASP.NET Core, we usually configure it with `AddJwtBearer`, then validate signature, issuer, audience, and expiry. A bad token gives `401`, while a valid token with insufficient permission gives `403`."

If asked how the server knows the token is ours:

"Because the token is signed. The server recomputes the signature using the configured signing key and compares it with the token signature. If they match, and issuer/audience/expiry are valid, the token is trusted."

## References

- [RFC 7519 - JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519)
- [RFC 8725 - JWT Best Current Practices](https://www.rfc-editor.org/rfc/rfc8725)
- [Microsoft Learn - Configure JWT bearer authentication in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/configure-jwt-bearer-authentication?view=aspnetcore-10.0)
