# CYLIX V2 - Code Quality Samples

## 💻 Real Examples from the Codebase

These are actual code snippets demonstrating the quality and professionalism of the CYLIX V2 codebase.

---

## 🎨 Frontend Code (React + TypeScript)

### **Sample 1: App.tsx - Main Application Structure**

```typescript
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { Provider } from 'react-redux';
import { store } from './store';
import { useEffect, useState } from 'react';
import './index.css'; // Global styles with design system

// Import components
import Login from './features/auth/Login';
import Register from './features/auth/Register';
import AuthGuard from './features/auth/AuthGuard';
import Dashboard from './features/dashboard/Dashboard';
import AIInteractions from './features/ai-interactions/AIInteractions';
import PurpleTeaming from './features/purple-teaming/PurpleTeaming';
import Settings from './features/settings/Settings';

// Theme provider component for dark mode management
const ThemeProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [isDarkMode, setIsDarkMode] = useState(() => {
    const savedTheme = localStorage.getItem('cylix-theme');
    if (savedTheme) return savedTheme === 'dark';
    return window.matchMedia('(prefers-color-scheme: dark)').matches;
  });

  useEffect(() => {
    document.documentElement.setAttribute('data-theme', isDarkMode ? 'dark' : 'light');
    localStorage.setItem('cylix-theme', isDarkMode ? 'dark' : 'light');
  }, [isDarkMode]);

  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    const handleChange = (e: MediaQueryListEvent) => {
      const savedTheme = localStorage.getItem('cylix-theme');
      if (!savedTheme) setIsDarkMode(e.matches);
    };
    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  return (
    <div className={`min-h-screen transition-colors duration-300 ${isDarkMode ? 'dark' : ''}`}>
      {children}
    </div>
  );
};
```

---

## ⚙️ Backend Code (Python + FastAPI)

### **Sample 2: Auth Login Endpoint (apps/web_ui_backend/api/v1/endpoints/auth/fastapi_impl.py)**

```python
@router.post("/login")
async def login(
    request: Request,
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: Session = Depends(get_db),
) -> dict[str, Any]:
    """Authenticate user and return an access token.

    Accepts application/x-www-form-urlencoded as used by
    OAuth2PasswordRequestForm with fields: username, password.
    """
    # Accept JSON or form payload
    username: str | None = None
    password: str | None = None
    try:
        if request.headers.get("content-type", "").lower().startswith("application/json"):
            body = await request.json()
            username = str(body.get("username")) if body.get("username") else None
            password = str(body.get("password")) if body.get("password") else None
    except Exception:
        pass

    if not username or not password:
        username = form_data.username
        password = form_data.password

    # DEV bypass: allow login without DB when DEV_NO_AUTH is set
    if str(os.environ.get("DEV_NO_AUTH", "")).lower() in {"1", "true", "yes", "on"}:
        subject = username or form_data.username
        if not subject:
            raise HTTPException(status_code=400, detail="username required")
        mfa_required = bool(getattr(settings, "MFA_REQUIRED", False))
        access_token = create_access_token(
            subject=subject,
            expires_delta=timedelta(minutes=60 * 24),
            additional_claims={"roles": ["admin"], "mfa_required": mfa_required, "mfa_verified": (not mfa_required)},
        )
        return {"access_token": access_token, "token_type": "bearer", "expires_in": 60 * 60 * 24}

    # Normal path using DB user
    user = (
        db.query(User)
        .filter((User.username == username) | (User.email == username))
        .first()
    )
    if not user or not user.is_active or not verify_password(password, user.password_hash):
        raise HTTPException(status_code=401, detail="Invalid credentials")

    mfa_required = bool(getattr(settings, "MFA_REQUIRED", False))
    access_token = create_access_token(
        subject=user.username,
        expires_delta=timedelta(minutes=60 * 24),
        additional_claims={"roles": user.roles or [], "mfa_required": mfa_required, "mfa_verified": (not mfa_required)},
    )
    return {"access_token": access_token, "token_type": "bearer", "expires_in": 60 * 60 * 24}
```

### **Sample 3: Demo JWT Auth Server (mock_backend_auth.py)**

```python
app = FastAPI(title="CYLIX V2 Security API", version="2.0.0")
SECRET_KEY = secrets.token_urlsafe(32)
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 15
security = HTTPBearer()

class LoginRequest(BaseModel):
    username: str
    password: str

@app.post("/api/v2/auth/login", response_model=TokenResponse)
async def login(request: LoginRequest):
    user_data = MOCK_USERS.get(request.username)
    if not user_data or user_data["password"] != request.password:
        raise HTTPException(status_code=401, detail="Incorrect username or password")
    access_token = create_access_token(data={"sub": request.username})
    refresh_token = create_refresh_token(data={"sub": request.username})
    return TokenResponse(access_token=access_token, refresh_token=refresh_token, user=User(**user_data))
```

**Highlights:**
- ✅ Modern FastAPI patterns, async endpoints
- ✅ JWT auth flows (login/refresh/verify)
- ✅ Typed responses and Pydantic models
- ✅ Optional DEV_NO_AUTH bypass for local testing
- ✅ Structured error handling and HTTP status codes

---

## 📊 Code Organization

```text
frontend/src/
  features/
  store/
  services/
  components/

apps/web_ui_backend/api/v1/endpoints/auth/
  fastapi_impl.py  # implemented auth endpoints

mock_backend_auth.py  # demo API with JWT auth for UI demos
```

---

## 🧪 Testing Structure (example)

```python
@pytest.fixture
def client():
    return TestClient(app)

def test_health_endpoint(client):
    response = client.get("/health")
    assert response.status_code == 200
```

---

## ✨ Key Takeaways

- ✅ Professional, typed Python and TypeScript
- ✅ Enterprise patterns and structure
- ✅ Clear separation of demo vs production concerns
- ✅ Honest documentation of current capabilities

For full details, see **[CURRENT_STATE_ASSESSMENT.md](./CURRENT_STATE_ASSESSMENT.md)**.
