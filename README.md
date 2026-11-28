# Trip Planner

## Concept

Trip Planner is a small web application for planning a group trip and managing its money. A group of friends uses it to record who is going, where and when, what has been booked, and what each person has paid along the way. The app compares the spending with the trip's budget, converts amounts between currencies, and works out who owes whom at the end of the trip.

Every module links its data to a traveler, so travelers are the app's user management: they define who uses the application and which data belongs to whom.

**Tech stack:** one FastAPI application with a router per module, one shared database, a Streamlit frontend that calls the API, Docker Compose, GitHub Actions for continuous integration, and a deployment on Azure.

## Team

| Member    | Module                | Shared job                 |
|-----------|-----------------------|----------------------------|
| Vince     | Travelers             | Database                   |
| Jesus     | Trips                 | Frontend shell & dashboard |
| Maria     | Bookings              | Documentation & board      |
| Christian | Expenses & settle-up  | Cloud deployment           |
| Nargiz    | Currency              | Continuous integration     |
| Ben       | Budget                | Containers & configuration |

## Modules

Each module has its own package, API router (under `/api/<module>`) and Streamlit page, with one main entity stored in the database and one calculated result.

| # | Module | Owner | Route | Main entity                                                                                   | Calculated result |
|---|--------|-------|-------|-----------------------------------------------------------------------------------------------|-------------------|
| 1 | Travelers | Vince | `/api/travelers` | Traveler (name, email)                                                                        | Total paid by each traveler |
| 2 | Trips | Jesus | `/api/trips` | Trip (name, destination, start and end date), plus which travelers are on it                  | Trip length and days until departure |
| 3 | Bookings | Maria | `/api/bookings` | Booking (trip, type, description, cost)                                                       | Total booked cost per trip |
| 4 | Expenses & settle-up | Christian | `/api/expenses` | Expense (trip, paid_by_traveler_id, amount, date), split equally between the trip's travelers | Net balance per traveler and who owes whom |
| 5 | Currency | Nargiz | `/api/currency` | Exchange rate (from currency, to currency, rate)                                              | An amount converted to another currency |
| 6 | Budget | Ben | `/api/budgets` | Budget (trip, limit_amount)                                                                   | Budget left = limit − bookings − expenses |

### Cross-module feature

A **trip dashboard** that combines the modules on one page: the trip's travelers, its bookings total, the budget left and who owes whom. It is built on the other modules' endpoints.

## Shared jobs

| Shared job | Responsible for | Member |
|------------|-----------------|--------|
| Database | The shared database setup, the models' common base and the sample data | Vince |
| Frontend shell & dashboard | The common layout, navigation and the dashboard that combines the modules | Jesus |
| Documentation & board | The README, the architecture diagram, and the issues and project board | Maria |
| Cloud deployment | The Azure resources, the deployment and its cost | Christian |
| Continuous integration | The GitHub Actions workflow: Ruff and the tests on every pull request | Nargiz |
| Containers & configuration | The Dockerfile, Docker Compose, settings from environment variables, `.env.example` | Ben |

## Conventions

**Stack:** the tech stack in the Concept section above, built with these tools: Python 3.14 with uv, FastAPI with Pydantic for the request and response models, SQLAlchemy for the database (PostgreSQL, run locally with Docker Compose and on Azure later), Streamlit for the frontend, Ruff for linting and formatting, pytest for tests.

**Package names:** all code lives in `src/group4_trip_planner/`. Each module has its own package with the same files:

```text
src/group4_trip_planner/
├── db.py            # shared: engine, Base, Money type, sessions
├── main.py          # shared: the FastAPI app, includes every module's router
└── <module>/
    ├── __init__.py
    ├── models.py    # the module's table(s)
    └── router.py    # the module's endpoints
```

The module packages are `travelers`, `trips`, `bookings`, `expenses`, `currency` and `budgets`.

**API routes:** each module has its own FastAPI router under `/api/<module>/`, with the route in the Modules table above (for example `/api/budgets/`). `POST /api/<module>/` creates an item, `GET /api/<module>/` lists them. Request and response bodies are JSON.

**Money:** `Decimal` in Python, sent and returned in the API as a decimal string ("12.50"), stored as whole cents (an integer) in the database with the shared `Money` column type from `db.py`. Never a `float`.

**Currency:** all amounts are stored in EUR. An amount entered in another currency is converted to EUR with the Currency module before it is saved. Exchange rates are not money: they are `Decimal` in Python and `Numeric(12, 6)` in the database (six decimal places), not the `Money` type.

**Dates:** `datetime.date` in Python, sent and returned as `YYYY-MM-DD` in the API.

**IDs:** every table has an integer primary key `id`. A link to another table is named `<entity>_id` (for example `trip_id`, `traveler_id`) and is a foreign key.

## Git rules

1. **Nobody pushes to `main`.** It is protected: a change reaches `main` only through a pull request with one approval and, once CI is set up, passing checks.
2. **One branch per task**, created from an up-to-date `main` and named `feature/...`, `fix/...` or `docs/...` (for example `feature/budgets-api`).
3. **Small pull requests**, reviewed and approved by one teammate, merged with a merge commit within a day or two. The description says what changes and links the issue (`Closes #<issue>`).
4. **Bring `main` into your branch often** (`git pull origin main`) and resolve conflicts on your branch, before merging.
5. **Shared files have an owner**, and changes to them go through that person: `db.py` and `main.py` (Vince, Database), configuration and `.env.example` (Ben, Containers & configuration), the CI workflow (Nargiz, Continuous integration).

## Running the API

```bash
cp .env.example .env
docker compose up -d db
uv sync
uv run fastapi dev src/group4_trip_planner/main.py
```
**Docker must be running.** `docker compose up -d db` starts the PostgreSQL database in the background; `docker compose down` stops it.

Open the address the terminal prints (normally http://127.0.0.1:8000). The interactive documentation, where every endpoint can be tried, is at http://127.0.0.1:8000/docs.
