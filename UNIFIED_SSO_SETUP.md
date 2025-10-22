# CYLIX V2 – Unified SSO (Grafana + Cylix UI)

Unify authentication for Grafana and the Cylix UI using **OIDC SSO** with **Keycloak** and **oauth2-proxy**, behind **NGINX**. This plan keeps the UI anonymous-friendly, secure, and production-ready with standard components.

---

## 🧱 Architecture

```
            +-------------------+
            |     Keycloak      |  (IdP / OIDC)
            +---------+---------+
                      |
                OIDC / OAuth2
                      |
+---------+     +-----v------+      +----------------+
|  Users  | --> |  NGINX RP  | -->  |   oauth2-proxy | --> Cylix UI (Vite app)
+---------+     +------------+      +----------------+
                      \
                       \--> Grafana (OIDC plugin)
```

- **Keycloak**: Identity Provider (realm `cylix`) with two public clients: `grafana` and `cylix-ui`.
- **oauth2-proxy**: Protects the Cylix UI using OIDC login, issues session cookie.
- **Grafana**: Uses built-in Generic OAuth (OIDC) auth.
- **NGINX**: Reverse-proxy and TLS termination; shared cookie domain (e.g., `.cylix.local`).

---

## 🔐 Keycloak Setup

1. Create realm: `cylix`.
2. Create client `grafana`:
   - Access type: Public or Confidential (recommended: Confidential)
   - Redirect URIs: `https://grafana.cylix.local/*`
   - Web Origins: `https://grafana.cylix.local`
3. Create client `cylix-ui`:
   - Access type: Public or Confidential (recommended: Confidential)
   - Redirect URIs: `https://app.cylix.local/oauth2/callback`
   - Web Origins: `https://app.cylix.local`
4. Assign default roles/claims as needed (email, profile, roles).
5. Note issuer URL: `https://keycloak.cylix.local/realms/cylix`.

---

## ⚙️ Grafana OIDC (grafana.ini)

```ini
[auth]
disable_login_form = true

[auth.generic_oauth]
name = Keycloak
enabled = true
client_id = grafana
client_secret = <SECRET>
scopes = openid profile email
email_attribute_path = email
login_attribute_path = preferred_username
auth_url = https://keycloak.cylix.local/realms/cylix/protocol/openid-connect/auth
token_url = https://keycloak.cylix.local/realms/cylix/protocol/openid-connect/token
api_url = https://keycloak.cylix.local/realms/cylix/protocol/openid-connect/userinfo
allow_sign_up = true
```

- Set `root_url`, `serve_from_sub_path` if Grafana is under `/grafana`.

---

## 🛡️ oauth2-proxy for Cylix UI

Example (Kubernetes manifest fragment):

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oauth2-proxy
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: oauth2-proxy
        image: quay.io/oauth2-proxy/oauth2-proxy:v7.6.0
        args:
          - --provider=oidc
          - --oidc-issuer-url=https://keycloak.cylix.local/realms/cylix
          - --client-id=cylix-ui
          - --client-secret=$(CLIENT_SECRET)
          - --cookie-secret=$(COOKIE_SECRET)
          - --cookie-domain=.cylix.local
          - --cookie-secure=true
          - --http-address=0.0.0.0:4180
          - --upstream=http://cylix-ui:3001
          - --scope=openid email profile
          - --email-domain=*
        env:
        - name: CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: sso-secrets
              key: client-secret
        - name: COOKIE_SECRET
          valueFrom:
            secretKeyRef:
              name: sso-secrets
              key: cookie-secret
        ports:
        - containerPort: 4180
```

- Generate a strong `COOKIE_SECRET` (32-byte base64).
- Upstream points to your Vite/Node server service name.

---

## 🌐 NGINX Reverse Proxy

```nginx
server {
  listen 443 ssl;
  server_name app.cylix.local;

  # TLS config omitted for brevity

  location / {
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://oauth2-proxy:4180;
  }
}

server {
  listen 443 ssl;
  server_name grafana.cylix.local;

  location / {
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://grafana:3000;
  }
}
```

- Use a shared cookie domain `.cylix.local` across subdomains.
- Consider a single NGINX `server` with `/` → UI, `/grafana` → Grafana if subpath routing preferred.

---

## 🔁 Unified Logout

- Implement logout endpoints that clear oauth2-proxy session and redirect to Keycloak end-session:
  - `https://keycloak.cylix.local/realms/cylix/protocol/openid-connect/logout?post_logout_redirect_uri=https://app.cylix.local`
- Configure Grafana `signout_redirect_url` similarly.

---

## 🔒 Security Notes

- Enforce TLS end-to-end; set `cookie_secure=true` and `SameSite=Lax`.
- Store secrets in Kubernetes Secrets or Vault; never in images.
- Apply CSP headers in NGINX to reduce XSS risk.
- Set short token lifetimes and refresh via silent re-auth if needed.

---

## 🧪 Validation Checklist

- Login to `app.cylix.local` redirects to Keycloak → back to UI.
- Open `grafana.cylix.local` → SSO into Grafana without separate login.
- Roles/claims mapping verified (e.g., email/roles available).
- Logout clears both sessions and returns to UI.

---

## 🧭 Alternatives

- **Auth0**, **Okta**, or any OIDC IdP
- **Traefik ForwardAuth** instead of oauth2-proxy
- **Subpath** routing: `/` UI, `/grafana` Grafana with `serve_from_sub_path=true`

---

## 🧷 Appendix – Minimal Helm/K8s values (hints)

- `grafana.ini` with `[auth.generic_oauth]` (as above)
- Ingress rules for `app.cylix.local` and `grafana.cylix.local`
- Secret manifests for `client-secret` and `cookie-secret`

This plan provides a clear, production-ready path to unify Grafana and Cylix UI authentication.
