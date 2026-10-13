# test-manager-library-3145556-infra

> **Prueba técnica · Ficha ADSO 3145556**
> Entrega: **martes 13 de octubre de 2026, a las 5:00 p. m.** (17:00, hora Colombia).

Este repositorio es la **infraestructura** de *Manager Library*: contenedores, red, volúmenes y el motor de base de datos vacío. **Empieza vacío a propósito**: se construye en el fork de cada aprendiz.

## Instrucciones

Cada aprendiz debe **crear el fork** de los seis repositorios del proyecto y **resolver el proyecto
con el spec planteado**.

1. Hacer fork, a su cuenta de GitHub, de cada repositorio de la tabla del final.
2. Leer el spec en [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs).
   Se entrega en dos versiones: `spec-python/` y `spec-.net/`.
3. Desarrollar en los forks.

## El reto se desarrolla con React y PHP (Laravel)

El spec está escrito para Python y para .NET, pero el reto **no** se hace en esos lenguajes:

| Capa | Tecnología del reto |
|---|---|
| Frontend | React |
| Backend | PHP con Laravel |

Lo que el spec define sobre el negocio —historias, criterios de aceptación, reglas, contrato de la
API, modelo de datos— se respeta. Lo que define sobre la tecnología se traduce a React y Laravel.

## La prueba no consiste en escribir el código

El propósito principal es ver la **capacidad de desempeño con SDD** (*Spec-Driven Development*,
desarrollo guiado por especificación): cómo se lee, se interpreta y se aplica una especificación
para llevarla a un stack distinto. El código es el medio, no el fin.

## Entrega

Cuenta lo que quede **subido a los forks antes de las 5:00 p. m.** (17:00). Lo que llegue después no se revisa.
No hay prórroga: suban avances parciales con commits pequeños durante la prueba.

## Los seis repositorios

| Repositorio | Qué va ahí |
|---|---|
| [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs) | El spec: `spec-python/` y `spec-.net/` |
| [`test-manager-library-3145556-api`](https://github.com/code-sena/test-manager-library-3145556-api) | Backend en PHP (Laravel) |
| [`test-manager-library-3145556-app`](https://github.com/code-sena/test-manager-library-3145556-app) | Frontend en React |
| [`test-manager-library-3145556-page`](https://github.com/code-sena/test-manager-library-3145556-page) | Sitio público estático de presentación |
| [`test-manager-library-3145556-infra`](https://github.com/code-sena/test-manager-library-3145556-infra) | Contenedores, red, volúmenes y motor de base de datos vacío |
| [`test-manager-library-3145556-tool`](https://github.com/code-sena/test-manager-library-3145556-tool) | Utilidades: sembrador de datos de demostración |

---

# Manager Library — local environment

> The text above is the challenge brief. This section documents how to run the environment.

One command starts an **empty** PostgreSQL 16 engine, the API (Laravel), the frontend (React) and the public page. No schema lives in this
repository: the API owns it through its migrations (constitution Article V, ADR-001).

## What it starts

| Service | Image | URL | Notes |
|---|---|---|---|
| `db` | `postgres:16` | `127.0.0.1:5432` | Empty engine, data in the `library_pgdata` volume, reachable from localhost only |
| `api` | PHP 8.3 (built from `docker/php`) | http://localhost:8000/api/v1 | Runs the migrations, then `php artisan serve` |
| `app` | `node:22-alpine` | http://localhost:5173 | Vite dev server |
| `page` | `nginx:alpine` | http://localhost:8080 | Serves the static `page` repository |

## Prerequisites

Docker Desktop (Compose v2) and the four repositories cloned **side by side**:

```text
prueba/
├── test-manager-library-3145556-api
├── test-manager-library-3145556-app
├── test-manager-library-3145556-page
└── test-manager-library-3145556-infra     <- you are here
```

## Secrets: the `.env` rule

- Only `.env.example` is versioned, and it contains generic placeholders only.
- Your real `.env` is ignored by git (`.gitignore`) and must never be committed or pushed (constitution Article IX).
- `docker-compose.yml` has no default for any secret: if one is missing, Compose stops with a message naming it.
- Check at any time: `git check-ignore -v .env` prints the ignoring rule, and `git ls-files ".env*"` lists only `.env.example`.

## 1. Configure

```bash
# Windows cmd:        copy .env.example .env
# Windows PowerShell: Copy-Item .env.example .env
```

Open `.env` and replace every `your_...` placeholder with your own value.

## 2. Create the Laravel project (once)

The `api` repository starts without code. This creates the Laravel skeleton inside it, keeping its `README.md` and `.git`, and without
leaving any `.env` file behind:

```bash
docker compose run --rm --no-deps api sh -c "composer create-project laravel/laravel /tmp/laravel --no-interaction --prefer-dist && rm -f /tmp/laravel/.env && cp -an /tmp/laravel/. /app/"
```

Then generate the application key and paste it into `APP_KEY` in your `.env`:

```bash
docker compose run --rm --no-deps api php artisan key:generate --show
```

## 3. Start

```bash
docker compose up -d --build
docker compose ps
docker compose logs -f api
```

`api` and `app` print a clear message and stop if their repository has no code yet; `db` and `page` are unaffected.

## Everyday commands

| Goal | Command |
|---|---|
| Stop everything | `docker compose down` |
| Reset the database to empty (**deletes all data**) | `docker compose down -v` |
| Open `psql` | `docker compose exec db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"'` |
| Run the API tests / artisan | `docker compose run --rm api php artisan test` |
| Rebuild the PHP image | `docker compose build api` |

## Verification (evidence for T-20)

1. **One command:** `docker compose up -d --build` starts the whole environment.
2. **Empty engine:** with a fresh volume, before the API runs its migrations:
   `docker compose exec db sh -c 'psql -U "$POSTGRES_USER" -d "$POSTGRES_DB" -c "\dt"'` answers `Did not find any relations.`
3. **No schema here:** `git grep -n -i -E "create[[:space:]]+table"` returns nothing.
4. **No secrets versioned:** `git ls-files ".env*"` lists only `.env.example`.
