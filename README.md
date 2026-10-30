# Cultural X-Ray

Cultural X-Ray helps small businesses make ads that speak to their customers' culture. It reads what fills a business's customers' lives (the podcasts, shows, music, creators and places they love, from Qloo's taste graph) and checks an ad against it before the owner pays to run it. It then suggests re-cuts, lets the owner edit and export them in the browser, and writes creative briefs.

This repository ties the product together. The code lives in two repositories, linked here as git submodules:

| Folder | Repository | What it is |
|---|---|---|
| [`web/`](web) | [Cultural-X-ray-Web](https://github.com/Brian-Mwangi-developer/Cultural-X-ray-Web) | The web app: Next.js 16, React, Tailwind 4, shadcn (Base UI) |
| [`backend/`](backend) | [Cultural-X-ray-Backend](https://github.com/Brian-Mwangi-developer/Cultural-X-ray-Backend) | The API and worker: Express 5, Better Auth, Prisma 7 on Postgres, Redis, RabbitMQ |

Both repositories are private, so you need access to them to clone their code.

## What you need

- **Node.js 22** or newer, and **pnpm 11** (`corepack enable` sets it up).
- **Docker**, for Redis and RabbitMQ (and, optionally, Prometheus and Grafana).
- A **Postgres** database. The project uses Neon; any Postgres 15+ works.
- Optional service keys, each switching a feature on:

  | Feature | Settings (in `backend/.env` unless noted) | Without it |
  |---|---|---|
  | Culture maps and lookups | `QLOO_API_KEY` | Maps can't be built |
  | Ad check (reading videos) | `GEMINI_PROVIDER=vertex`, `GOOGLE_CLOUD_PROJECT`, `GOOGLE_VERTEX_API_KEY` | `GEMINI_PROVIDER=practice` gives scripted answers at no cost |
  | Ask X-Ray (the assistant) | `ANTHROPIC_API_KEY` | `ASK_PRACTICE=true` streams scripted answers |
  | Talking to X-Ray (voice) | The same Vertex settings, plus `VOICE_*` | Voice is unavailable |
  | Video and image uploads | `BLOB_READ_WRITE_TOKEN`, `BLOB_ACCESS` (Vercel Blob) | Ad checks say they aren't set up |
  | Emails | `EMAIL_PROVIDER=ses` and AWS keys | `EMAIL_PROVIDER=log` (the default) prints emails, confirmation links included, to the API's log |
  | Google sign-in | `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET` | Email and password only |
  | Maps of places | `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` and `NEXT_PUBLIC_GOOGLE_MAPS_MAP_ID` in `web/.env` | The "Open map" button is hidden |

## Get the code

```bash
git clone --recurse-submodules https://github.com/Brian-Mwangi-developer/Cultural-X-ray.git
cd Cultural-X-ray
```

If you cloned without `--recurse-submodules`, run `git submodule update --init`.

## Run it locally

### 1. The backend (API on :4000, worker on :4001)

```bash
cd backend
pnpm install
pnpm env:init        # writes .env with fresh Redis, RabbitMQ and ops credentials
```

Open `backend/.env` and set at least:

- `DATABASE_URL`: your Postgres connection string.
- `BETTER_AUTH_SECRET`: any random string of 32+ characters (`openssl rand -base64 32`).
- `BETTER_AUTH_URL=http://localhost:3000` and `PUBLIC_WEB_URL=http://localhost:3000`.

Then add whichever optional keys from the table above you have. `backend/.env.example` lists every setting.

```bash
pnpm db:migrate      # creates the tables
pnpm dev             # starts Redis, RabbitMQ and monitoring in Docker, then the API and worker
```

`pnpm dev:lite` does the same without Prometheus and Grafana, which is lighter on a laptop.

### 2. The web app (:3000)

```bash
cd web
pnpm install
```

Create `web/.env`:

```bash
BACKEND_URL=http://localhost:4000
NEXT_PUBLIC_BETTER_AUTH_URL=http://localhost:3000
# Optional, for maps of places:
# NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=
# NEXT_PUBLIC_GOOGLE_MAPS_MAP_ID=
```

```bash
pnpm dev
```

Open <http://localhost:3000>. The web app forwards `/api/auth/*` and `/api/v1/*` to the backend and holds no database or auth secrets of its own.

### 3. First run

1. Sign up at <http://localhost:3000/signup>. With `EMAIL_PROVIDER=log`, the confirmation link is printed in the backend's terminal.
2. Pick a goal in the welcome room, find your business by name or by shop and town, and build your culture map.
3. Try the ad check with your own video (MP4, MOV or WebM, up to 60 seconds).

## Useful commands

| Where | Command | What it does |
|---|---|---|
| `backend` | `pnpm test` | Runs the test suite (Vitest) |
| `backend` | `pnpm check` | Type-checks |
| `backend` | `pnpm stack up` / `pnpm stack down` | Starts or stops the Docker services on their own |
| `backend` | `pnpm db:studio` | Opens Prisma Studio on the database |
| `web` | `pnpm lint` | Lints the web app |
| `web` | `pnpm build && pnpm start` | Production build of the web app |

| Service | Port |
|---|---|
| Web | 3000 |
| API / worker | 4000 / 4001 |
| Redis | 6390 |
| RabbitMQ (and its admin page) | 5682 (15682) |
| Prometheus / Grafana | 9095 / 3005 |

More detail on how the backend is built, its endpoints and its runbook is in [`backend/Architecture.md`](backend/Architecture.md).

## Keeping the submodules up to date

The submodules track each repository's `main` branch:

```bash
git submodule update --remote          # move web/ and backend/ to the latest main
git add web backend && git commit -m "Update web and backend"
```

## Licence

This repository is licensed under the [Apache License 2.0](LICENSE). The `web` and `backend` repositories are separate and private, and their code is covered by their own terms.
