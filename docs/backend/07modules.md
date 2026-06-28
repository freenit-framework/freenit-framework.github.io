# Backend Modules

Backend is the Freenit framework's server-side component: an async **FastAPI** service that exposes a modular REST API under `/api/v1`. It supports two storage backends—**SQL** via [Oxyde](https://github.com/oxyde/oxyde) (SQLite by default, configurable) and **LDAP** via [Bonsai](https://github.com/noirello/bonsai)—selected at runtime through configuration. Beyond authentication and user/role management, it bundles project kanban, an LMS, mailing lists, Git repository hosting, and proxies for Stalwart mail services.

The package is published as `freenit` on PyPI and provides a `freenit` console script that scaffolds full-stack projects.

---

## Table of Contents

- [Directory Layout](#directory-layout)
- [Features](#features)
- [Core Modules](#core-modules)
- [API Modules](#api-modules)
- [Data Models](#data-models)
- [Utility Modules](#utility-modules)
- [CLI and Scripts](#cli-and-scripts)
- [Project Template](#project-template)
- [Testing](#testing)
- [Configuration](#configuration)

---

## Directory Layout

```
services/backend/
├── freenit/                 # Main Python package
│   ├── api/                 # FastAPI route modules
│   ├── auth.py              # JWT and password utilities
│   ├── base_config.py       # Default configuration classes
│   ├── cli.py               # Console script entry point
│   ├── config.py            # Environment-based config resolver
│   ├── decorators.py        # @route / FreenitAPI decorator
│   ├── encrypt.py           # Per-user Fernet encryption
│   ├── git/                 # Git repository backend
│   ├── mail.py              # SMTP helper
│   ├── mailinglist/         # Mailing-list worker and stores
│   ├── migration.py         # Alembic migration runner
│   ├── models/              # SQL and LDAP models
│   ├── modules.py           # Module registry and resolver
│   ├── permissions.py       # Pre-built permission dependencies
│   ├── stalwart.py          # Stalwart mail HTTP/JMAP client
│   └── app.py               # FastAPI application factory
├── bin/                     # Development/build helper scripts
├── migrations/              # Oxyde/Alembic migration files
├── tests/                   # pytest test suite
├── freenit/project/         # Scaffold copied by `freenit` CLI
├── ldap/                    # LDAP schemas for mailing lists / OMEMO
├── ansible/                 # Reggae/Ansible provisioning stubs
├── pyproject.toml           # Hatchling build configuration
├── main.py                  # uvicorn entry point
├── migrate.py               # Migration helper
├── oxyde_config.py          # Oxyde migration configuration
└── README.md
```

---

## Features

- **Modular FastAPI service** — Features are declared in `freenit/modules.py` and resolved at import time, including circular dependencies.
- **Dual storage backend** — Switch between SQL (Oxyde/SQLAlchemy) and LDAP (Bonsai) by changing model paths in config.
- **Auth / users / roles** — Registration, login/logout, JWT access/refresh cookies, role-based permissions.
- **Project kanban** — Projects, boards, columns, tasks, project groups, and permissions.
- **LMS** — Courses, lectures, lecture states (draft / public / private), course groups, and permissions.
- **Mailing lists** — Public/private lists with Stalwart principals, double-opt-in subscribe/unsubscribe, moderation queue, and archives.
- **Git hosting** — Repository metadata, permissions, push logs, read-only browser, smart HTTP transport, and hooks.
- **Stalwart proxies** — JMAP, CalDAV/CardDAV/WebDAV, ManageSieve, and XMPP chat config endpoints.
- **OMEMO support** — Encrypted per-user bundle storage.
- **CLI scaffolding** — `freenit` command generates backend, frontend, and project skeletons.

---

## Core Modules

The backend is organized around a module registry in `freenit/modules.py`. Each `Module` declares its name, dependencies, SQL model paths, and API module path.

| Module | Dependencies | Models | API | Description |
|--------|--------------|--------|-----|-------------|
| `auth` | `user`, `role` | `freenit.models.sql.base` | `freenit.api.auth` | Login, logout, register, verify, refresh, token endpoints. |
| `user` | `role` | — | `freenit.api.user` | User CRUD and profile endpoints. |
| `role` | `user` | — | `freenit.api.role` | Role CRUD and user assignment. |
| `project` | `user` | `freenit.models.sql.project` | `freenit.api.project` | Kanban projects, boards, columns, tasks, groups. |
| `lms` | `user` | `freenit.models.sql.lms` | `freenit.api.lms` | Courses, lectures, course groups. |
| `mailinglist` | `user` | `freenit.models.sql.mailinglist` | `freenit.api.mailinglist` | Mailing lists, subscriptions, moderation. |
| `domain` | `user` | — | `freenit.api.domain` | LDAP domain and POSIX group management. |
| `dav` | `user` | — | `freenit.api.dav` | CalDAV/CardDAV/WebDAV proxy. |
| `mail` | `user` | — | `freenit.api.mail` | JMAP proxy to Stalwart. |
| `sieve` | `user` | — | `freenit.api.sieve` | ManageSieve script management. |
| `chat` | `user` | — | `freenit.api.chat` | XMPP WebSocket config endpoint. |
| `omemo` | `user` | — | `freenit.api.omemo` | OMEMO bundle storage. |
| `git` | `user` | `freenit.models.sql.git` | `freenit.api.git` | Git repo metadata and HTTP transport. |

`resolve_modules(requested)` returns the transitive closure of a module list, handling circular dependencies by including every module in a cycle once any member is requested. `get_api_modules()` and `get_models()` return API and SQL model paths for migrations.

---

## API Modules

### Router (`freenit/api/router.py`)

Defines the main `FastAPI` instance and the `route` decorator used throughout the backend. The decorator turns class-based endpoints with `get`, `post`, `patch`, and `delete` static methods into FastAPI routes. `description()` provides optional operation descriptions.

### Auth (`freenit/api/auth/__init__.py`)

- `POST /auth/login` — sets `access` and `refresh` HttpOnly cookies.
- `POST /auth/logout`
- `POST /auth/register` — creates an inactive SQL user or LDAP entry.
- `POST /auth/verify` — activates account from a JWT.
- `POST /auth/refresh` — refreshes the access cookie.
- `GET /auth/token` — returns a fresh JWT.

### Users (`freenit/api/user/sql.py`, `freenit/api/user/ldap.py`)

- `GET /users`, `GET|PATCH /users/{id}`, `GET|PATCH /profile`
- LDAP variants use DN/uid-based lookups instead of integer IDs.

### Roles (`freenit/api/role/sql.py`, `freenit/api/role/ldap.py`)

- `GET|POST /roles`, `GET|PATCH|DELETE /roles/{id}`
- `POST|DELETE /roles/{role_id}/{user_id}` — add/remove user from role.

### Project (`freenit/api/project.py`)

- `GET|POST /projects`, `GET|PATCH|DELETE /projects/{id}`
- `GET|POST /projects/{id}/boards`, `GET|PATCH|DELETE /boards/{id}`
- `GET|POST /boards/{id}/columns`, `GET|PATCH|DELETE /columns/{id}`
- `GET|POST /columns/{id}/tasks`, `GET|PATCH|DELETE /tasks/{id}`
- `GET|POST /projects/{id}/groups`, `GET|PATCH|DELETE /project-groups/{id}`
- `GET|POST /project-groups/{id}/members`, `DELETE /project-groups/{id}/members/{user_id}`

### LMS (`freenit/api/lms.py`)

- `GET|POST /courses`, `GET|PATCH|DELETE /courses/{id}`
- `GET|POST /courses/{id}/lectures`, `GET|PATCH|DELETE /lectures/{id}`
- Course groups and permissions mirror project endpoints.
- Lecture visibility is enforced via `LectureState` (draft / published_public / published_private).

### Mailing List (`freenit/api/mailinglist.py`)

- `GET|POST /mailinglists`, `GET|PATCH|DELETE /mailinglists/{id}`
- `GET /mailinglists/public`, `/mailinglists/domains`
- `GET /mailinglists/{id}/subscribers`, `/mailinglists/{id}/archive`, `/mailinglists/{id}/archive/{msg_id}`
- `POST /mailinglists/{id}/subscribe`, `GET /mailinglists/{id}/confirm/{token}`
- `POST /mailinglists/{id}/unsubscribe`, `GET /mailinglists/{id}/unsubscribe/{token}`
- `GET /mailinglists/{id}/moderation`, `POST .../approve|reject|process`

### Git (`freenit/api/git.py`, `freenit/api/git_http.py`)

- `GET|POST /git/repos`, `GET|PATCH|DELETE /git/repos/{id}`, `GET /git/repos/public`
- `GET|POST /git/repos/{id}/permissions`, `DELETE /git/repos/{id}/permissions/{perm_id}`
- `GET /git/repos/{id}/push-log`, `POST /git/repos/{id}/hooks/push`, `GET /git/repos/{id}/hooks`
- Read-only browser: `/git/repos/{id}/refs`, `/commits`, `/tree`, `/blob`, `/readme`, `/clone-url`, `/task-refs`
- Smart HTTP transport mounted at app root: `GET /git/{repo}/info/refs`, `POST /git/{repo}/git-upload-pack`, `POST /git/{repo}/git-receive-pack`

### Stalwart Proxies

| File | Endpoints | Purpose |
|------|-----------|---------|
| `freenit/api/mail.py` | `/mail/jmap`, `/mail/jmap/session`, `/mail/jmap/upload/...`, `/mail/jmap/download/...`, WS `/mail/jmap/ws` | JMAP proxy with admin impersonation. |
| `freenit/api/dav.py` | `/cal[/{path}]`, `/card[/{path}]`, `/file[/{path}]`, `POST /cal/fetch-ical` | CalDAV/CardDAV/WebDAV proxy plus iCal fetch with SSRF checks. |
| `freenit/api/sieve.py` | `/mail/sieve/scripts`, `/mail/sieve/scripts/{name}`, `/mail/sieve/scripts/{name}/active` | ManageSieve over TCP (RFC 5804). |
| `freenit/api/chat.py` | `/chat/config` | Returns configured XMPP WebSocket URL. |
| `freenit/api/omemo.py` | `/profile/omemo` | Stores/retrieves OMEMO bundle. |

---

## Data Models

### SQL Models (`freenit/models/sql/`)

Base class `OxydeBaseModel` in `freenit/models/sql/base.py` wraps `oxyde.Model` and provides:

- `dbtype() == "sql"`
- `pk` property
- `patch()` method for partial updates
- `load_all()` helper

**User / Role / UserRole** (`base.py`):
- `User`: email, password (pbkdf2), fullname, active, admin, omemo_bundle, M2M `roles` via `UserRole`.
- `BaseRole`: unique `name`, M2M `users`.
- `CustomRoleList` enables `await user.roles.add(role)`.

**Project** (`project.py`): `Project`, `Board`, `Column`, `Task`, `ProjectGroup`, `ProjectGroupPermission`, `ProjectMember`.

**LMS** (`lms.py`): `Course`, `Lecture`, `LectureState`, `CourseGroup`, `CourseGroupPermission`, `CourseMember`.

**Mailing List** (`mailinglist.py`): `MailingList`, `PendingSubscriber`, `ModerationMessage`.

**Git** (`git.py`): `GitRepo`, `GitPermission`, `GitPushLog`, `GitCommitTaskRef`.

### LDAP Models (`freenit/models/ldap/`)

Base class `LDAPBaseModel` in `freenit/models/ldap/base.py` is a Pydantic model with `dn` and `dbtype() == "ldap"`. Helpers manage UID/GID/MLID/Git-ID counters and LDAP filters.

- **User** (`user.py`): `UserSafe` / `User` with bind/login, registration, DN/uid/email lookup, group filling, and OMEMO storage.
- **Role** (`role.py`): `groupOfUniqueNames`-style role with member add/remove.
- **Group** (`group.py`): POSIX group under a domain OU.
- **Domain** (`domain.py`): Creates/saves role-base and user-base OUs.
- **Git** (`git.py`): Pydantic views for repo, permission, push log, and commit-task-ref data.
- **Mailing List** (`mailinglist.py`): Pydantic views for list, pending subscriber, and moderation message data.

### Model Dispatch

`freenit/models/{user,role,project,lms,mailinglist,git}.py` import the configured backend module path from `BaseConfig`. This lets the rest of the codebase use backend-agnostic imports like `freenit.models.user` while the actual implementation is determined by config.

---

## Utility Modules

| File | Purpose |
|------|---------|
| `freenit/config.py` | Imports `DevConfig`/`TestConfig`/`ProdConfig` from `local_config.py` if present, otherwise `base_config.py`; `getConfig()` returns the env-selected singleton. |
| `freenit/base_config.py` | Defines `Auth`, `Mail`, `XMPP`, `LDAP`, and `BaseConfig` classes. Default DB is SQLite; default active modules are `["auth"]`. |
| `freenit/local_config.py` | Local override file (not tracked) used for real service credentials and LDAP backend. |
| `freenit/modules.py` | `Module` dataclass, `MODULES` registry, `resolve_modules()`, `get_api_modules()`, `get_models()`. |
| `freenit/auth.py` | JWT encode/decode with PyJWT, `authorize()`, `permissions()`, `verify()`/`encrypt()` using `pbkdf2_sha256` and `config.secret`. |
| `freenit/permissions.py` | Pre-built permission callables: `domain_perms`, `git_perms`, `lms_perms`, `mailinglist_perms`, etc. |
| `freenit/decorators.py` | `FreenitAPI` / `route` class decorator and `description()` helper. |
| `freenit/encrypt.py` | Per-user Fernet encryption (`encrypt_for_user`, `decrypt_for_user`) for OMEMO bundles. |
| `freenit/mail.py` | Plain SMTP `sendmail()` helper. |
| `freenit/stalwart.py` | HTTP/JMAP client for Stalwart: create/delete/patch principals, list domains, fetch/destroy emails. |
| `freenit/migration.py` | Alembic migration runner used by `oxyde_config.py`. |
| `freenit/cli.py` | Entry point for the `freenit` console script; shells out to `bin/freenit.sh`. |
| `freenit/app.py` | FastAPI app factory with lifespan (runs migrations, connects/disconnects DB) and optional Git HTTP router mount. |

---

## CLI and Scripts

### CLI

`pyproject.toml` defines:

```toml
[project.scripts]
freenit = "freenit.cli:main"
```

`freenit/cli.py` invokes `bin/freenit.sh`, which scaffolds projects.

### Backend Scripts (`services/backend/bin/`)

| Script | Purpose |
|--------|---------|
| `freenit.sh` | Project/backend/frontend scaffolding generator. |
| `common.sh` | Virtualenv/setup/migration helper. |
| `devel.sh` | Installs deps and runs `python main.py`. |
| `test.sh` | Runs `pytest -v --ignore=freenit/project/`. |
| `security.sh` | Runs `bandit` over the package. |
| `build.sh` | Builds the package with hatchling. |
| `publish.sh` | Publishes the package with twine. |
| `shell.sh` | Launches IPython with `PYTHONPATH` set. |
| `init.sh` | One-time project initialization. |

---

## Project Template

`freenit/project/` is the scaffold copied by `freenit.sh backend`:

- `main.py` — uvicorn entry point.
- `bin/{common,init,devel,test,shell,build}.sh` — per-project lifecycle scripts.
- `tests/` — starter test client, factories, and conftest.
- `project/` — application package with `app.py`, `config.py`, `base_config.py`, models, and API.
- `ansible/roles/devel/` — FreeBSD package provisioning role.
- `templates/site.yml.tpl` — Ansible playbook template.
- `ldap/` — LDAP schemas for OMEMO.

---

## Testing

- **Runner**: `pytest` with `pytest-asyncio` and `pytest-factoryboy`.
- **Config**: `tests/conftest.py` sets `FREENIT_ENV=test`, creates a temporary SQLite DB, overrides `config.database`, and calls `migrate.db_setup()`.
- **Client**: `tests/client.py` subclasses `fastapi.testclient.TestClient`, builds full URLs, and provides `login()`.
- **Factories**: `tests/factories.py` uses `factory.Factory` for `User`, `Role`, `Project`, `Board`, `Column`, `Task`, project groups, courses, and lectures.

### Test Files

| File | Coverage |
|------|----------|
| `tests/test_auth.py` | Login, register, refresh. |
| `tests/test_user.py` | Profile and user admin CRUD. |
| `tests/test_role.py` | Role CRUD and user assignment. |
| `tests/test_permission.py` | JWT encode/decode and role-based permission checks. |
| `tests/test_project.py` | Full project/board/column/task/group/member lifecycle. |
| `tests/test_lms.py` | Course/lecture/group/permission visibility tests. |
| `tests/test_mailinglist.py` | List creation, subscribe flow, domain listing. |
| `tests/test_modules.py` | Module resolution and discovery endpoint. |

Run tests with `bin/test.sh` or `pytest -v --ignore=freenit/project/`.

---

## Configuration

Configuration is layered through `freenit/base_config.py` and optional `freenit/local_config.py`:

- `BaseConfig` defines defaults: SQLite DB, port 5000, `api_root = "/api/v1"`, active modules `["auth"]`, and placeholders for mail/LDAP/Stalwart.
- `DevConfig` enables debug mode and insecure auth cookies.
- `TestConfig` uses a separate SQLite file and enables all modules for test coverage.
- `ProdConfig` sets a stronger secret and enables `Mail()`.

`freenit/config.py` imports the three config classes from `local_config.py` if it exists, otherwise from `base_config.py`. `getConfig()` reads `FREENIT_ENV` (default `prod`) and returns the matching singleton.

Key environment variables:

- `FREENIT_ENV` — `dev`, `test`, or `prod`.
- `FREENIT_DBURL` / `DATABASE_URL` — database URL for migrations.
- `SYSPKG` — use system packages instead of a virtualenv.
- `OFFLINE` — skip package installation.

---

## Notable Supporting Files

| File | Purpose |
|------|---------|
| `oxyde_config.py` | Oxyde migration configuration; reads `FREENIT_DBURL`/`DATABASE_URL` and exports `MODELS`, `MIGRATIONS_DIR`, `DATABASES`. |
| `migrations/` | Seven migrations from initial auth through Git tables. |
| `freenit/git/hooks.py` | Executable hook helper for bare repos (`pre-receive`, `update`, `post-receive`, `run-tests`). |
| `freenit/git/repo.py` | Dulwich-based read-only Git browser and commit-message task-ref extractor. |
| `freenit/git/webhook.py` | Webhook dispatch for Git push events. |
| `freenit/mailinglist/worker.py` | Polls list inbox via JMAP and distributes or moderates messages. |
