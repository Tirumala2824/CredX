# CredX frontend

The CredX web interface is built with [Astro](https://astro.build/) and TypeScript. It provides the browser UI for the portfolio dashboard, watchlist, customer details, data uploads, and intervention workflows.

## Requirements

- Node.js (a currently supported LTS release)
- npm (included with Node.js)
- The CredX FastAPI backend running locally or at a configured API URL

## Install

From the repository root:

```bash
cd Frontend
npm ci
```

## Configure the API URL

The frontend API client uses the deployed CredX API as its default outside local development. To point the UI at a different backend, create a local `.env` file in `Frontend/`:

```dotenv
PUBLIC_API_URL=http://127.0.0.1:8000
```

For example, copy the tracked template:

```bash
cp .env.example .env
```

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

Astro exposes variables prefixed with `PUBLIC_` to browser code. Put only public configuration here; never put API keys, passwords, signing secrets, or private credentials in a `PUBLIC_*` variable.

When using the local backend, make sure its CORS configuration permits the frontend origin (`http://localhost:4321`). See the root [README](../README.md) and [backend configuration example](../backend/.env.example).

## Run locally

Start the backend separately by following the root README's backend setup. Then, from `Frontend/`:

```bash
npm run dev
```

Open the local URL printed by Astro (normally http://localhost:4321).

## Available commands

| Command | Description |
| --- | --- |
| `npm ci` | Install exact dependencies from `package-lock.json` |
| `npm run dev` | Start the local development server |
| `npm run build` | Build the production site into `dist/` |
| `npm run preview` | Preview the production build locally |
| `npm run astro -- --help` | Show Astro CLI help |

Before submitting frontend changes, run:

```bash
npm run build
```

## Project structure

```text
Frontend/
├── public/             Static assets and brand images
└── src/
    ├── components/     Shared UI and dashboard components
    ├── layouts/        Page layouts
    ├── lib/            API client and EWS helpers
    ├── pages/          Dashboard, portfolio, watchlist, upload, etc.
    └── styles/         Global, component and page styles
```

For the overall architecture, API endpoints, sample data, and backend setup, see the [repository root README](../README.md).
