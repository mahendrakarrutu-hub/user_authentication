# User Auth App

A simple username/password authentication system with a Django REST Framework backend and a Flutter client. Supports registration, login, token-based auth, and profile view/update.

## Stack

- **Backend:** Django 6.0, Django REST Framework, Token Authentication
- **Frontend:** Flutter (Material), `http` package
- **Database:** SQLite (dev default)

## Project Structure

```
backend/
├── user_auth/            # Project settings, root urls
│   ├── settings.py
│   └── urls.py
└── user/                  # Auth app
    ├── serializers.py
    ├── views.py
    └── urls.py

frontend/
├── main.dart              # Login, Register, Profile screens
└── api_service.dart       # HTTP client wrapping the API
```

## Backend Setup

1. Create a virtual environment and install dependencies:
   ```bash
   python -m venv venv
   source venv/bin/activate   # Windows: venv\Scripts\activate
   pip install django djangorestframework django-cors-headers
   ```

2. Run migrations:
   ```bash
   python manage.py migrate
   ```

3. Start the dev server:
   ```bash
   python manage.py runserver
   ```
   API will be available at `http://127.0.0.1:8000/api/auth/`.

## API Endpoints

| Method | Endpoint | Auth required | Description |
|--------|----------|:---:|-------------|
| POST | `/api/auth/register/` | No | Create a new user, returns token + user |
| POST | `/api/auth/login/` | No | Authenticate, returns token + user |
| GET | `/api/auth/profile/` | Yes | Get current user's profile |

Authenticated requests must include:
```
Authorization: Token <token>
```

### Example: Register
```bash
curl -X POST http://127.0.0.1:8000/api/auth/register/ \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "email": "alice@example.com", "password": "hunter22"}'
```

### Example: Login
```bash
curl -X POST http://127.0.0.1:8000/api/auth/login/ \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "password": "hunter22"}'
```

### Example: Get Profile
```bash
curl http://127.0.0.1:8000/api/auth/profile/ \
  -H "Authorization: Token <your_token>"
```

## Frontend Setup

1. Add the HTTP dependency to `pubspec.yaml`:
   ```yaml
   dependencies:
     http: ^1.2.0
   ```

2. Update the base URL in `api_service.dart` if your backend isn't running locally:
   ```dart
   static const String baseUrl = 'http://127.0.0.1:8000/api/auth';
   ```
   > Note: Android emulators can't reach `127.0.0.1` on the host machine — use `10.0.2.2` instead. iOS simulators and web can use `127.0.0.1` directly.

3. Run the app:
   ```bash
   flutter pub get
   flutter run
   ```

## App Flow

1. **Login screen** — enter credentials, or navigate to Register.
2. **Register screen** — create an account with username/email/password.
3. **Profile screen** — view username/email, edit first/last name, log out.

## Screenshots

### Profile Screen
![Profile screen](./profile_screenshot.png)

Shows the logged-in user's username and email (read-only), plus editable First Name / Last Name fields and a Save Profile button.

## File Reference

| File | Purpose |
|------|---------|
| `user_auth/settings.py` | Django project settings (apps, middleware, DRF/CORS config, DB) |
| `user_auth/urls.py` | Root URL configuration, mounts the `user` app |
| `user/serializers.py` | `RegisterSerializer`, `LoginSerializer`, `ProfileSerializer` |
| `user/views.py` | `RegisterView`, `LoginView`, `ProfileView` |
| `user/urls.py` | App-level routes: `register/`, `login/`, `profile/` |
| `main.dart` | Flutter UI: `LoginScreen`, `RegisterScreen`, `ProfileScreen` |
| `api_service.dart` | Flutter HTTP client: `register()`, `login()`, `getProfile()`, `updateProfile()` |

## Known Limitations / Before Production

- `SECRET_KEY` is hardcoded in `settings.py` — move to an environment variable.
- `CORS_ALLOW_ALL_ORIGINS = True` is dev-only — restrict to known origins in production.
- Root `urls.py` currently includes `user.urls` twice (under both `/api/` and `/api/auth/`) — remove the duplicate unless intentional.
- No rate limiting on login/register — add DRF throttling to prevent brute-force attempts.
- Password strength only enforces a 6-character minimum — Django's built-in `AUTH_PASSWORD_VALIDATORS` aren't currently wired into the serializer.
- No logout/token-revocation endpoint yet — tokens are valid indefinitely once issued.
- `DEBUG = True` and empty `ALLOWED_HOSTS` must be set correctly before deploying.
- The Flutter app calls `PUT /api/auth/profile/` to save first/last name (`updateProfile()` in `api_service.dart`), but `ProfileView` on the backend is a `RetrieveAPIView` — it only handles `GET`. Saving a profile will currently fail with a 405. Change `ProfileView` to `RetrieveUpdateAPIView` and add `first_name`/`last_name` to `ProfileSerializer` to fix this.

## License

Add your license here.
