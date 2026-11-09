# Contributing to CredX

Thank you for taking the time to improve CredX. Contributions should make the project more correct, understandable, secure, and easier to maintain.

## Before you start

1. Read the [project README](README.md) to understand the architecture and current limitations.
2. Search existing pull requests and repository discussions (where enabled) before starting a large change.
3. For a significant feature or architectural change, describe the problem and proposed approach to the maintainer before investing in implementation.
4. Do not assume this project is suitable for production credit decisions. See [SECURITY.md](SECURITY.md) and the security limitations in the README.

## Development setup

Follow the setup instructions in the root README. In brief:

- Backend: Python 3.14, `uv sync --locked`, environment initialization, and Alembic migrations from `backend/`.
- Frontend: Node.js and npm, then `npm ci` from `Frontend/`.

## Branch and pull request workflow

1. Fork the repository and create a focused branch from `main`.
2. Keep each pull request limited to one logical change.
3. Add or update tests for behavior changes.
4. Update documentation when commands, configuration, API behavior, or user-facing workflows change.
5. Run relevant checks before opening the pull request.
6. Explain the motivation, implementation, verification performed, and any remaining limitations.

Suggested branch names:

- `feature/short-description`
- `fix/short-description`
- `docs/short-description`

## Checks

Backend (run inside `backend/`):

```bash
uv run pytest -q
uv run ruff check .
uv run ruff format --check .
```

Frontend (run inside `Frontend/`):

```bash
npm ci
npm run build
```

Run the checks relevant to your change. Do not claim a check passed unless you ran it and observed a successful result.

## Data, secrets, and generated files

- Use synthetic or properly authorized data only.
- Never commit `.env` files, API keys, JWT secrets, passwords, access tokens, private keys, or deployment credentials.
- Never add real customer, borrower, account, repayment, or other personal financial information.
- Do not commit local databases, temporary exports, logs, caches, test artifacts, or generated files unless a change explicitly requires a reviewed fixture.
- Keep model artifacts and datasets clearly documented, including provenance and usage restrictions.

## Code quality

- Prefer explicit types, small functions, and clear names.
- Preserve existing API contracts unless the pull request intentionally changes them and documents migration impact.
- Handle incomplete or missing data explicitly; do not silently interpret unknown data as low risk.
- Avoid adding dependencies unless they are needed and compatible with the supported Python/Node versions.
- Include reproducible steps for bugs and regression tests where practical.

## License

No license file is currently present. Until the project owner adds and publishes a license, contributors should not assume that the code may be reused, redistributed, or modified under an open-source license.
