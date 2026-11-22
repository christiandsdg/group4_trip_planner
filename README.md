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

| # | Module | Owner | Route | Main entity | Calculated result |
|---|--------|-------|-------|-------------|-------------------|
| 1 | Travelers | Vince | `/api/travelers` | Traveler (name, email) | Total paid by each traveler |
| 2 | Trips | Jesus | `/api/trips` | Trip (name, destination, start and end date), plus which travelers are on it | Trip length and days until departure |
| 3 | Bookings | Maria | `/api/bookings` | Booking (trip, type, description, cost) | Total booked cost per trip |
| 4 | Expenses & settle-up | Christian | `/api/expenses` | Expense (trip, paid by, amount, date), split equally between the trip's travelers | Net balance per traveler and who owes whom |
| 5 | Currency | Nargiz | `/api/currency` | Exchange rate (from currency, to currency, rate) | An amount converted to another currency |
| 6 | Budget | Ben | `/api/budgets` | Budget (trip, limit) | Budget left = limit − bookings − expenses |

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
