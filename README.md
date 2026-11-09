# CredX

**Early-warning intelligence for credit portfolios.**

CredX is an open-source project exploring how lenders and credit-risk teams can monitor portfolio health, identify customers whose repayment or financial behaviour may be deteriorating, and prioritize follow-up using explainable risk signals.

Rather than treating a credit score as a complete decision, CredX combines repayment patterns, credit exposure, cash-flow signals, configurable rules, and model outputs to help analysts investigate changes over time.

> **Project status:** Experimental / development-stage software. CredX is not a banking core, credit bureau, loan-origination system, or production-ready decision engine. It does not approve or reject credit applications. Outputs are analytical signals that require human review.

## What it includes

- **Early-warning assessment:** combines repayment-delay, missed-payment, exposure, data-completeness, and cash-flow signals.
- **Risk monitoring:** portfolio overview, watchlist, customer detail, assessment history, and prioritisation views.
- **Explainability:** exposes indicators and contributing signals behind an assessment; an optional Nebius-compatible model endpoint can be configured for generated explanations.
- **Intervention workflow:** records analyst alert classifications and reasons against an assessment.
- **Data ingestion:** accepts CSV, Excel, and JSON uploads for customers, loans, repayments, exposure, and cash-flow records.
- **Authentication API:** registration, login, refresh-token rotation, logout, and authenticated profile access.
- **Web interface:** an Astro frontend for dashboard, portfolio, watchlist, customer, upload, and intervention workflows.

The repository contains rule-based scoring and saved ML model artifacts. The exact scoring path can fall back when a trained artifact is unavailable; inspect the API response's score-source and signal metadata rather than assuming every assessment uses an ML model.

## Architecture

```text
CredX/
├── Frontend/                 Astro web application
├── backend/                  FastAPI application and API
│   ├── app/
│   │   ├── api/v1/           Authentication, EWS and monitoring routes
│   │   ├── models/            SQLAlchemy persistence models
│   │   ├── schemas/           Request and response schemas
│   │   └── services/          Scoring, explainability, history and prioritisation
│   ├── docs/                  API and deployment documentation
│   ├── migrations/            Alembic migrations
│   ├── scripts/               Setup, data and deployment utilities
│   └── tests/                 Backend tests
└── core/
    ├── rules/                 Early-warning rule implementations
    └── models/                Feature engineering and ML model code/artifacts
```

### Technology

| Layer | Technologies |
| --- | --- |
| Frontend | Astro 4, TypeScript, HTML/CSS |
| API | Python 3.14, FastAPI, Uvicorn |
| Persistence | SQLAlchemy, SQLite, Alembic |
| Risk analytics | pandas, NumPy, scikit-learn, XGBoost, joblib |
| Optional LLM explanation | OpenAI-compatible API configuration (Nebius example in backend settings) |
| Deployment tooling | Docker, GitHub Actions, Zoho Catalyst deployment scripts |

## Quick start

You need **Python 3.14**, **Node.js and npm**, and **Git**. For the backend, [uv](https://docs.astral.sh/uv/) is the recommended dependency manager. The commands below are for a local development setup.

### 1. Clone the repository

```bash
git clone https://github.com/Tirumala2824/CredX.git
cd CredX
```

### 2. Start the backend API

In Terminal 1:

```bash
cd backend
```

Install uv if it is not already installed:

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows PowerShell
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Install the locked backend dependencies:

```bash
uv sync --locked
```

Create the local environment file and generate a development JWT secret:

```bash
uv run python scripts/init_env.py
```

This creates `backend/.env` if it does not already exist. The file is ignored by Git. Never commit it or share its secret values.

Apply database migrations:

```bash
uv run alembic upgrade head
```

Start the API:

```bash
uv run uvicorn app.main:create_app --factory --reload --host 127.0.0.1 --port 8000
```

The backend should now be available at:

- API base: http://127.0.0.1:8000
- Interactive API docs: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc
- Health endpoint: http://127.0.0.1:8000/health

### 3. Start the frontend

Open Terminal 2 from the repository root:

```bash
cd CredX/Frontend
npm ci
npm run dev
```

Astro prints the local URL; the default is http://localhost:4321.

The frontend API client currently has a development/provisioned backend URL fallback. To use a different backend, set `PUBLIC_API_URL` in the frontend environment (for example, create `Frontend/.env` with `PUBLIC_API_URL=http://127.0.0.1:8000`). Ensure the backend CORS policy allows the exact frontend origin before browser requests are made. Do not put secrets in `PUBLIC_*` variables; they are exposed to browser code.

### Windows PowerShell quick reference

```powershell
git clone https://github.com/Tirumala2824/CredX.git
cd CredX\backend
uv sync --locked
uv run python scripts/init_env.py
uv run alembic upgrade head
uv run uvicorn app.main:create_app --factory --reload --host 127.0.0.1 --port 8000
```

In a second PowerShell window:

```powershell
cd CredX\Frontend
npm ci
npm run dev
```

## Configuration

Backend configuration is documented in [backend/.env.example](backend/.env.example) and validated in `backend/app/core/config.py`.

| Variable | Purpose |
| --- | --- |
| `JWT_SECRET` | Required signing secret; generated by `scripts/init_env.py` for local development |
| `DATABASE_URL` | SQLite database location; defaults to the backend data database |
| `JWT_ISSUER`, `JWT_AUDIENCE` | JWT validation settings |
| `ACCESS_TOKEN_MINUTES` | Access-token lifetime |
| `REFRESH_TOKEN_DAYS` | Refresh-session lifetime |
| `LOGIN_MAX_ATTEMPTS`, `LOGIN_LOCK_SECONDS` | Per-account failed-login lockout |
| `CORS_ORIGINS` | Intended list of allowed browser origins; verify deployed middleware behavior before relying on it |
| `NEBIUS_API_KEY` | Optional key for the configured OpenAI-compatible explanation provider |
| `NEBIUS_BASE_URL`, `NEBIUS_MODEL` | Optional provider endpoint and model |

Do not copy production credentials into local files, commits, issues, screenshots, or pull requests. Use synthetic data for development.

## Working with portfolio data

The EWS upload endpoint supports five data types. Uploads are made to `POST /api/v1/ews/upload?data_type=<type>` as multipart form data with a file field named `file`.

| Data type | Required columns |
| --- | --- |
| `customers` | `customer_id`, `customer_type` |
| `loans` | `customer_id`, `loan_id`, `product_type`, `sanctioned_amount`, `outstanding_balance` |
| `repayments` | `customer_id`, `loan_id`, `due_date`, `amount_due`, `amount_paid`, `repayment_status` |
| `exposure` | `customer_id`, `facility_type`, `outstanding_balance` |
| `cashflow` | `customer_id`, `month`, `monthly_inflow` |

For field details and endpoint payloads, see [API reference](backend/docs/API.md). Sample-data documentation lives in [backend/sample_data_v2](backend/sample_data_v2/README.md) and [backend/sample_data_v3](backend/sample_data_v3/README.md).

Typical workflow:

1. Start the API and open its Swagger page at `/docs`.
2. Register and log in to obtain a bearer token.
3. Upload the customer and facility data, then repayments, exposure, and cash-flow records.
4. Run an assessment using `POST /api/v1/ews/assess`.
5. Review portfolio/watchlist and customer detail endpoints in Swagger or the frontend.
6. Inspect indicators, score source, data sufficiency and assessment history before taking any action.

Do not treat a missing-data or provisional result as a low-risk outcome.

## API overview

The API is versioned under `/api/v1`. Common routes include:

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/health` | Process health check |
| POST | `/api/v1/auth/register` | Register a user |
| POST | `/api/v1/auth/login` | Log in and receive tokens |
| POST | `/api/v1/auth/refresh` | Rotate refresh token |
| POST | `/api/v1/auth/logout` | Revoke the current session |
| GET | `/api/v1/users/me` | Current user profile |
| POST | `/api/v1/ews/upload` | Upload portfolio data |
| POST | `/api/v1/ews/assess` | Run risk assessment |
| GET | `/api/v1/ews/watchlist` | Query latest customer assessments |
| GET | `/api/v1/ews/portfolio` | Portfolio summary |
| GET | `/api/v1/ews/customer/{customer_id}` | Customer detail and history |
| POST | `/api/v1/ews/explain/{customer_id}` | Explain latest customer assessment |

See [backend/docs/API.md](backend/docs/API.md) for the complete API contract and [backend/docs/openapi.json](backend/docs/openapi.json) for an importable OpenAPI schema.

## Development and tests

Backend checks:

```bash
cd backend
uv run pytest -q
uv run ruff check .
uv run ruff format --check .
uv run pip-audit -r requirements.txt --disable-pip --no-deps
```

Frontend production build:

```bash
cd Frontend
npm ci
npm run build
```

For reproducible backend dependencies, use `uv sync --locked`. The project also keeps hash-pinned pip requirement exports; see [backend/README.md](backend/README.md) for the alternative pip workflow and dependency-maintenance commands.

## Deployment

Deployment-specific notes are maintained separately:

- [Zoho Catalyst AppSail development deployment](backend/docs/DEPLOYMENT_CATALYST.md)
- [Backend README and pip alternative](backend/README.md)

The Catalyst workflow deploys the backend only. It does not deploy the Astro frontend. Review the deployment document before configuring secrets or running deployment workflows.

## Security and responsible use

CredX handles credit-related information. The current repository is a development project and must not be used to make consequential lending decisions without substantial engineering, model-risk, security, legal and compliance review.

Important limitations documented by the implementation include:

- SQLite is the current supported database. The Catalyst AppSail deployment uses instance-local/seeded SQLite and is not a durable multi-instance data store.
- The current API includes authentication, but this should not be interpreted as a complete production banking security posture. Review authorization on every data and monitoring endpoint before exposing the service.
- The frontend currently stores bearer/refresh tokens in browser local storage. This is not a recommended production token-storage design for banking or other sensitive applications.
- Model outputs, thresholds and rule scores need independent validation, calibration, bias/fairness testing, monitoring and governance on representative, lawfully obtained data.
- No KYC, credit-bureau integration, loan approval, payment, or money-transfer functionality is provided.

Use synthetic or properly authorized data only. Do not upload real customer financial or personally identifiable information to a public demo or development deployment.

## Contributing

Contributions are welcome. Before opening a pull request:

1. Open an issue describing the bug, proposal or question.
2. Keep changes focused and include tests for behavior changes.
3. Run the relevant backend checks and frontend build.
4. Never include secrets, customer data, database files, generated logs or unrelated model artifacts in a contribution.
5. Describe configuration changes and any migration or compatibility impact.

Please include reproducible steps, expected behavior and actual behavior in bug reports. For security-sensitive issues, do not publish exploit details or personal/customer data in a public issue; contact the maintainers privately through the repository owner.

## License

No license file is currently present. Until a license is added, do not assume the code is licensed for reuse, redistribution or modification. Add an explicit open-source license before describing the repository as open source in the legal sense.

---

**CredX is an evolving engineering project.** Feedback, careful reviews and focused contributions are appreciated.
