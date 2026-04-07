## The Complete OIDC Flow — From Click to Verified Identity

There are 4 actors in every OIDC flow. Remember these:

```
┌─────────────┐   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│    User     │   │  Your App   │   │     IdP     │   │    JWKS     │
│  (Browser)  │   │  (Backend)  │   │  (Google)   │   │  Endpoint   │
└─────────────┘   └─────────────┘   └─────────────┘   └─────────────┘
```

### Phase 1: Setup (Done Once, Before Any User Logs In)

Before a single user touches your login button, a *developer registers the app with the IdP*. This is where trust is established.

**How does developer registers the app with the IdP**

You go to Google's developer console (or Okta, GitHub, whatever IdP you're using), fill out a small form that says:
- Here's my app's name
- Here's the URL I want users redirected back to after login (the `redirect_uri`)

Google gives you back:
- `client_id` — public, identifies your app
- `client_secret` — private, stays on your server only

```
Developer Action (one time)
─────────────────────────────────────────────────────────

Your App                              Google IdP
   │                                      │
   │   "I want to use OIDC login"         │
   │  ─────────────────────────────────►  │
   │                                      │
   │  ◄─────────────────────────────────  │
   │   Here's your:                       │
   │   ● client_id     = abc123           │
   │   ● client_secret = xyzSECRET        │
   │   ● authorized redirect_uri          │
   │                                      │

Your app stores these in config.
client_secret NEVER leaves your backend.
```

At this point, Google also publishes its public signing keys at:

```
https://accounts.google.com/.well-known/openid-configuration

                    ↓ (leads to)

https://www.googleapis.com/oauth2/v3/certs   ← JWKS endpoint
```

Your app doesn't fetch these yet. That comes later.

---

### Phase 2: User Clicks "Sign in with Google"

```
User                  Your App (Backend)
 │                          │
 │   clicks login button    │
 │  ──────────────────────► │
 │                          │
 │  ◄──────────────────────  │
 │   302 Redirect to:        │
 │                          │
 │   https://accounts.google.com/o/oauth2/auth
 │     ?client_id=abc123
 │     &redirect_uri=https://yourapp.com/callback
 │     &response_type=code
 │     &scope=openid email profile
 │     &state=randomString789        ← CSRF protection
 │     &nonce=randomNonce456         ← replay protection
```

What each parameter means:

```
client_id      → tells Google which app is asking
redirect_uri   → where to send the user back after login
scope=openid   → this is what makes it OIDC (not just OAuth2)
state          → random value your app generates, stored in session
nonce          → random value your app generates, will appear in JWT later
```

---

### Phase 3: Google Authenticates the User

```
User                         Google IdP
 │                               │
 │   browser lands on Google     │
 │  ───────────────────────────► │
 │                               │
 │  ◄───────────────────────────  │
 │   shows login page            │
 │                               │
 │   enters email + password     │
 │  ───────────────────────────► │
 │   (+ MFA if enabled)          │
 │                               │
 │  ◄───────────────────────────  │
 │   302 Redirect back to:       │
 │   https://yourapp.com/callback
 │     ?code=AUTHCODE_xyz        ← short-lived, one-time code
 │     &state=randomString789    ← same state you sent
```

**Key point**: Your app never saw the user's password. Google handled that entirely.

The `code` is not the token. It's a one-time voucher, valid for ~60 seconds, that your backend will exchange for the real tokens.

---

### Phase 4: Backend Exchanges Code for Tokens

```
Your App (Backend)                    Google IdP
        │                                  │
        │   POST /token                    │
        │   client_id=abc123               │
        │   client_secret=xyzSECRET        │
        │   code=AUTHCODE_xyz              │
        │   redirect_uri=...               │
        │  ──────────────────────────────► │
        │                                  │
        │  ◄──────────────────────────────  │
        │   {                              │
        │     "access_token": "...",       │
        │     "id_token": "eyJ...",        │
        │     "expires_in": 3600           │
        │   }                              │
```

You get back two tokens:

```
access_token → used to call Google APIs (get user's profile etc.)
               NOT for your app to identify the user

id_token     → THIS is what you use to know who the user is
               It's a JWT. This is the OIDC part.
```

---

### Phase 5: What the ID Token Actually Looks Like

The `id_token` is a JWT — three base64 chunks separated by dots:

```
eyJhbGciOiJSUzI1NiIsImtpZCI6ImFiYzEyMyJ9         ← HEADER
.
eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20   ← PAYLOAD
iLCJzdWIiOiIxMjM0NTY3ODkwIiwiZW1haWwiOiJzaWRk...
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c       ← SIGNATURE
```

```
Decoded:

HEADER
{
  "alg": "RS256",     ← signed using RSA + SHA256
  "kid": "abc123"     ← which key pair Google used to sign this
}

PAYLOAD
{
  "iss": "https://accounts.google.com",  ← who issued this
  "sub": "1234567890",                   ← unique user ID (stable forever)
  "email": "sidd@example.com",           ← can change, don't use as PK
  "aud": "abc123",                       ← must match YOUR client_id
  "iat": 1712345678,                     ← issued at (unix timestamp)
  "exp": 1712349278,                     ← expires at (1 hour later)
  "nonce": "randomNonce456"              ← matches what you sent in step 2
}

SIGNATURE
= RS256(base64(header) + "." + base64(payload), Google's_Private_Key)
```

---

### Phase 6: Your App Verifies the JWT

This is where the trust mechanism kicks in. Your app has never seen this specific token before. Here's how it knows it's real:

```
Your App (Backend)                         JWKS Endpoint
        │                                       │
        │  1. decode header → kid = "abc123"    │
        │                                       │
        │  2. GET /certs (fetch Google's        │
        │     public keys — cached after        │
        │     first fetch)                      │
        │  ───────────────────────────────────► │
        │                                       │
        │  ◄───────────────────────────────────  │
        │  {                                    │
        │    "keys": [                          │
        │      { "kid": "abc123", "n": "...",   │
        │        "e": "AQAB", "alg": "RS256" }  │
        │      { "kid": "xyz789", ... }         │
        │    ]                                  │
        │  }                                    │
        │                                       │
        │  3. find key where kid = "abc123"     │
        │                                       │
        │  4. verify signature using            │
        │     that public key                   │
        │                                       │
        │     signature valid? ──► YES          │
        │     token untampered? ──► YES         │
```

Then check the claims:

```
✓  iss == "https://accounts.google.com"  → right issuer
✓  aud == "abc123"                       → meant for my app
✓  exp > now()                           → not expired
✓  nonce == "randomNonce456"             → matches session, not replayed
─────────────────────────────────────────────────────
All pass → User is authenticated. Trust the claims.
```

---

### Phase 7: App Creates a Session

```
Your App (Backend)                    User (Browser)
        │                                  │
        │  JWT verified ✓                  │
        │  extract: sub, email, name       │
        │                                  │
        │  create session / set cookie     │
        │  ─────────────────────────────►  │
        │  Set-Cookie: session=abc...      │
        │                                  │
        │  (stop using the JWT now)        │
```

The JWT is done its job. Your app now runs on its own session. You don't pass the JWT around internally.

---

### The Full Picture in One Diagram

```
USER          YOUR APP          GOOGLE IdP         JWKS
 │                │                  │               │
 │  click login   │                  │               │
 │ ─────────────► │                  │               │
 │                │                  │               │
 │ ◄────────────  │                  │               │
 │  redirect →    │                  │               │
 │                                   │               │
 │  ─────────────────────────────►   │               │
 │  GET /auth?client_id&scope=openid │               │
 │                                   │               │
 │  Google shows login page          │               │
 │  user enters password             │               │
 │                                   │               │
 │  ◄─────────────────────────────   │               │
 │  redirect → /callback?code=xyz    │               │
 │                                   │               │
 │ ─────────────► │                  │               │
 │  code=xyz      │                  │               │
 │                │  POST /token     │               │
 │                │  code + secret   │               │
 │                │ ───────────────► │               │
 │                │                  │               │
 │                │ ◄───────────────  │               │
 │                │  id_token (JWT)  │               │
 │                │                  │               │
 │                │  fetch pub keys  │               │
 │                │ ──────────────────────────────►  │
 │                │ ◄──────────────────────────────  │
 │                │  JWKS (pub keys) │               │
 │                │                  │               │
 │                │  verify JWT sig  │               │
 │                │  check claims    │               │
 │                │  ✓ all pass      │               │
 │                │                  │               │
 │ ◄────────────  │                  │               │
 │  session cookie│                  │               │
 │  logged in ✓   │                  │               │
```

---

### Why Can't Anyone Fake a Token?

```
ATTACK 1: Forge a token with fake claims
─────────────────────────────────────────
Attacker writes:
{ "sub": "admin", "email": "admin@yourapp.com" }

Problem: They don't have Google's private key.
They can't produce a valid signature.
→ Verification step fails. Token rejected.


ATTACK 2: Steal a token and reuse it elsewhere
────────────────────────────────────────────────
Attacker captures a real JWT.

Problem: aud = "abc123" (your client_id only)
         exp = 1 hour from issue time
         nonce = bound to original session
→ Rejected by any other app. Expires quickly.


ATTACK 3: Tamper with the payload
───────────────────────────────────
Attacker changes email claim in the payload.

Problem: Signature was computed over the
         original payload. Any change breaks it.
→ Signature mismatch. Token rejected.
```

---

### Where the Trust Actually Lives

```
TRUST CHAIN
─────────────────────────────────────────────────────

You trust Google's domain (accounts.google.com)
         │
         └─► You fetched JWKS over HTTPS
                      │
                      └─► HTTPS cert proves it's really Google
                                   │
                                   └─► Public key from JWKS
                                       verifies JWT signature
                                                │
                                                └─► JWT claims
                                                    are trustworthy

The ONLY moment of trust = the HTTPS fetch of the JWKS.
Everything else is pure math.
```
