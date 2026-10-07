[![Watch the Video](https://img.youtube.com/vi/WIntbYKzASk/maxresdefault.jpg)](https://www.youtube.com/watch?v=WIntbYKzASk)
# k6 load testing a real API (FastAPI + JWT auth)

Companion code for the video **"How do you load test a real API with login? Your first k6 script in 15 minutes"**.

A small but realistic backend — signup, login (JWT), users and items — and k6 scripts that test it the way real users use it.

## Stack

- **API:** Python 3.10+, FastAPI, SQLAlchemy 2, SQLite (WAL), bcrypt, PyJWT
- **Load testing:** [k6](https://grafana.com/docs/k6/latest/) v2.x
- **Manual testing:** Postman collection in `postman/`

## 1. Run the API

```bash
python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\activate
pip install -r requirements.txt

fastapi dev app/main.py            # development (auto-reload)
# or, when load testing:
fastapi run app/main.py            # no reload, closer to production
```

Open http://localhost:8000/docs for interactive Swagger docs.
Endpoint reference: [docs/API.md](docs/API.md).

## 2. Try it in Postman

Import `postman/k6-demo-api.postman_collection.json` and run the requests top to bottom.
Signup generates a fresh email, Login saves the token, and every protected request uses it automatically.

## 3. Install k6

```bash
brew install k6                    # macOS
winget install k6 --source winget  # Windows
# Linux / other: https://grafana.com/docs/k6/latest/set-up/install-k6/
k6 version
```

## 4. Run the k6 scripts

```bash
k6 run k6/01-smoke.js          # 1 user hits /health for 10s
k6 run k6/02-user-journey.js   # 10 users: signup → login → me → create → update → get
k6 run k6/03-load-test.js      # ramp to 20 users, log in once per user, per-endpoint thresholds

# Point at another environment:
k6 run -e BASE_URL=https://staging.example.com k6/02-user-journey.js

# See the live web dashboard while the test runs (http://localhost:5665):
K6_WEB_DASHBOARD=true k6 run k6/03-load-test.js
```

## Config (env vars)

| Variable | Default | Notes |
|---|---|---|
| `DATABASE_URL` | `sqlite:///./app.db` | Any SQLAlchemy URL |
| `SECRET_KEY` | dev value | **Change in any real deployment** |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | `30` | |
| `BCRYPT_ROUNDS` | `10` | 12 is a common production value; higher = slower login |

Reset the data: stop the server and delete `app.db*`.

> ⚠️ Only load test systems you own or have permission to test.
