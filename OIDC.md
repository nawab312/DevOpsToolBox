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
