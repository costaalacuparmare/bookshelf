# BookShelf API

A book catalog and review service built with FastAPI and SQLite. This repo is the running project for the [Production Systems Engineering](https://edi33416.github.io/prod-eng/) course. It is built up lab by lab, and each lab (one part of the course) is split into chapters. Lab 1 (Part I, Development Foundations) is done and covers project setup, tooling and workflow, and a working API.

## Requirements

- Python 3.11 or newer
- Git

## Getting started

```bash
python3.11 -m venv .venv
source .venv/bin/activate
pip install -e ".[dev]"
pre-commit install
```

The editable install is required. The package lives under `src/`, so `import bookshelf` only works after installing.

Check the install:

```bash
python -c "import bookshelf; print(bookshelf.__version__)"
```

## Running the API

```bash
uvicorn bookshelf.main:app --reload
```

- Swagger UI: http://127.0.0.1:8000/docs
- Health check: http://127.0.0.1:8000/health

The SQLite database is created at `src/bookshelf/bookshelf.db` on first start. It is git-ignored. Do not use `--reload` in production.

### Endpoints

| Method | Path | Description |
| ------ | ---- | ----------- |
| GET | `/health` | Liveness check |
| POST | `/books/` | Create a book (409 on duplicate ISBN) |
| GET | `/books/` | List books (`offset`, `limit`) |
| GET | `/books/search?q=` | Search by title or author |
| GET | `/books/{id}` | Get one book (404 if missing) |
| PUT | `/books/{id}` | Update a book's fields |
| DELETE | `/books/{id}` | Delete a book and its reviews |
| POST | `/books/{id}/reviews` | Add a review (rating 1-5) |
| GET | `/books/{id}/reviews` | List a book's reviews |

Invalid input, such as a bad ISBN, a future year or a rating outside 1-5, returns 422.

Example:

```bash
curl -s -X POST http://localhost:8000/books/ \
  -H "Content-Type: application/json" \
  -d '{"title": "The Pragmatic Programmer", "author": "David Thomas", "isbn": "9780135957059", "year": 1999}'
```

## Quality checks

```bash
ruff check src/
ruff format --check src/
mypy src/
pre-commit run --all-files
```

Ruff and mypy are configured in `pyproject.toml`. Pre-commit runs Ruff and basic file hygiene hooks on every commit.

## Project layout

```text
src/bookshelf/
  main.py          FastAPI app, lifespan, /health
  models.py        Pydantic request and response models
  database.py      SQLite connection and schema setup
  routers/
    books.py       Book endpoints
    reviews.py     Review endpoints
tests/             Added in Lab 2
pyproject.toml     Build, dependencies, Ruff, mypy, pytest config
requirements*.lock Pinned dependency versions
.pre-commit-config.yaml
.editorconfig
```

## Lab progress

### Lab 1 (Part I): Development Foundations (done)

All chapters are complete.

#### Python Project Structure

- Used the `src` layout.
- Added `pyproject.toml` with the build system, project metadata, dependencies, a `dev` extra, and tool config.
- Added `__version__` to `src/bookshelf/__init__.py`.
- Created a venv and installed with `pip install -e ".[dev]"`.

#### Dependencies and Virtual Environments

- Installed runtime and dev dependencies.
- Generated lockfiles with pip-tools:
  - `pip-compile --output-file=requirements.lock pyproject.toml`
  - `pip-compile --extra=dev --output-file=requirements-dev.lock pyproject.toml`
- Added `.gitignore` entries for the venv and tool caches.

#### Linters, Formatters and Type Checking

- Configured Ruff (rules `E,F,I,UP,B,S`) and strict mypy in `pyproject.toml`.
- Added `.editorconfig`.

#### Pre-commit Hooks

- Added `.pre-commit-config.yaml` with Ruff, Ruff format and the pre-commit-hooks set.
- Ran `pre-commit install` to enable the hooks locally.

#### Git Workflow for Teams

- Trunk-based development, with short-lived feature branches off `main`.
- Branch prefixes: `feature/`, `fix/`, `chore/`, `docs/`, `refactor/`.
- Conventional Commits, such as `feat(books): add search endpoint`.
- Every change goes through a PR, squash-merged.
- Did the exercise: pushed `feature/bookshelf-api`, opened a PR and squash-merged it into `main`.

#### Building the BookShelf API

- Pydantic models with ISBN and year validation.
- SQLite storage using parameterised queries.
- Book and review routers, wired up in `main.py`.
- Verified by hand against every endpoint and the validation cases.

## Notes

- SQLite is for development only. Lab 4 moves the project to PostgreSQL in Docker.
- Swagger UI at `/docs` should be restricted in production.
