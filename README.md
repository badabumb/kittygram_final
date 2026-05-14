# Kittygram: Containers and CI/CD

Kittygram is an educational Django REST API and React application for sharing cat cards with photos, colors, birth years, and achievements. This repository contains the full containerized stack and a GitHub Actions workflow for linting, tests, Docker image builds, publishing to Docker Hub, and Telegram notification.

## Technologies

- Python 3.9, Django 3.2, Django REST Framework, Djoser
- PostgreSQL 13
- React 17, React Scripts
- Gunicorn
- Nginx
- Docker Compose
- GitHub Actions
- Ruff and pytest

## Environment

Create a local `.env` file from the example:

```bash
cp .env.example .env
```

Example values:

```env
SECRET_KEY=change-me
DEBUG=False
ALLOWED_HOSTS=localhost,127.0.0.1
TIME_ZONE=Europe/Moscow
POSTGRES_DB=kittygram
POSTGRES_USER=kittygram_user
POSTGRES_PASSWORD=kittygram_password
DB_NAME=kittygram
DB_HOST=db
DB_PORT=5432
```

For production, add your domain or server IP to `ALLOWED_HOSTS` and use a unique `SECRET_KEY`.

## Local Docker Run

Build and start all services:

```bash
docker compose up --build
```

The gateway publishes the application at:

```text
http://localhost:9000
```

Useful checks:

```bash
docker compose ps
docker compose logs backend
docker compose exec backend python manage.py check
```

The backend runs migrations and `collectstatic` on startup. The frontend container copies the production React build into the shared `static` volume. The gateway serves the React app, Django static files, media files, `/api/`, and `/admin/`.

## Production Compose

`docker-compose.production.yml` uses published Docker Hub images instead of local builds:

```bash
docker compose -f docker-compose.production.yml up -d
```

Set `DOCKERHUB_USERNAME` in the shell or `.env` so Compose can resolve these images:

```text
<dockerhub_username>/kittygram_backend
<dockerhub_username>/kittygram_frontend
<dockerhub_username>/kittygram_gateway
```

## Local Backend Run

The Django settings use PostgreSQL when `DB_HOST` is set. Without `DB_HOST`, they fall back to SQLite, which is convenient for local tests:

```bash
python3 -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip
pip install -r backend/requirements.txt
pytest backend/cats/tests.py
```

## Local Frontend Run

```bash
cd frontend
npm ci
npm start
```

To run frontend tests once:

```bash
npm test -- --watchAll=false
```

## CI/CD

The workflow is defined in `.github/workflows/main.yml` and duplicated as `kittygram_workflow.yml` for the educational checker.

On every push to any branch, GitHub Actions:

1. Installs backend dependencies.
2. Runs `ruff check backend`.
3. Runs backend tests.
4. Installs frontend dependencies.
5. Runs frontend tests.
6. Builds Docker images.

Only pushes to `main` or `master` also publish images to Docker Hub and send a Telegram success notification.

Required GitHub Secrets:

```text
DOCKERHUB_USERNAME
DOCKERHUB_TOKEN
TELEGRAM_TO
TELEGRAM_TOKEN
```

## Course Checker

For the Practicum tests, create `tests.yml` in the repository root:

```yaml
repo_owner: your_github_username
kittygram_domain: https://your-kittygram-domain.example
taski_domain: https://your-taski-domain.example
dockerhub_username: your_dockerhub_username
```

Then run:

```bash
pytest
```
