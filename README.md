# Football Analytics Platform

An end-to-end football data platform built using Python, Azure SQL, GitHub Actions and Power BI.

The project ingests football data from API-Football, stages and transforms it in Azure SQL, and presents the final data through an interactive Power BI dashboard covering league, team and player analysis.

The aim was to build more than a one-off dashboard and create a small data pipeline that could ingest changing data, update existing records and run automatically.

---

## Project Goal

This project was built to demonstrate:

- REST API ingestion with Python
- Handling paginated API responses and rate limits
- Staging raw data before loading reporting tables
- Fact and dimension-style SQL modelling
- Incremental updates using SQL `MERGE`
- Cloud database hosting with Azure SQL
- Scheduled ingestion with GitHub Actions
- Power BI modelling and dashboard development

The project is currently focused on Premier League data, with the ingestion structure designed so additional leagues and seasons can be added.

---

## Architecture

```text
API-Football
     ↓
Python Ingestion
     ↓
Azure SQL Staging Tables
     ↓
SQL MERGE / UPSERT
     ↓
Fact + Dimension Tables
     ↓
Power BI
```

GitHub Actions is used to run the core ingestion process on a schedule.

---

## Data Pipeline

The ingestion layer is split into separate modules for:

- Leagues
- Teams
- Fixtures
- Standings
- Players

Each API response is transformed into a pandas DataFrame and loaded into staging tables in Azure SQL.

SQL `MERGE` statements then update the reporting tables. This is useful for football data because records change over time. Fixtures move from scheduled to completed, standings change after each matchday and player season totals continue to increase.

Player ingestion is handled separately because the endpoint is paginated and requires multiple API calls. The script works through each page while adding delays and retry handling to stay within API limits.

---

## Data Model

The SQL layer uses separate tables for different levels of the data, including:

```text
dim_competition
dim_team
dim_date

fact_match
fact_standing
```

Player data is also modelled for player and player-season analysis.

The main grains are:

- **Match** – one row per fixture
- **Standing** – one row per team, competition and season
- **Player** – one row per player
- **Player Season** – one row per player, competition and season

Separating the data by grain makes it easier to build reliable relationships and measures in Power BI.

---

## Power BI Dashboard

The reporting layer contains three main pages.

### Overview

A league-level view containing:

- League standings
- Top scorers and assists
- Player ratings
- Cards
- Team goals
- Passing statistics
- Shooting statistics

### Team Breakdown

Allows a team to be selected and analysed in more detail, including:

- Results
- Points progression
- Squad statistics
- Player appearances
- Individual player performance

### Player Comparison

Allows two players to be compared across areas such as:

- Appearances and minutes
- Goals and assists
- Ratings
- Passing
- Shooting
- Dribbling
- Duels
- Interceptions
- Discipline

The Power BI file is stored in the `dashboard` folder.

---

## Tech Stack

- **Python** – API ingestion and pipeline orchestration
- **Pandas** – transforming API responses
- **Requests** – REST API communication
- **SQLAlchemy / PyODBC** – Azure SQL connectivity
- **Azure SQL Database** – cloud data storage
- **SQL** – schema design and incremental upserts
- **GitHub Actions** – scheduled ingestion
- **Power BI** – analytics and visualisation
- **Git / GitHub** – source control

---

## Project Structure

```text
Football-Analytics-Platform/
│
├── .github/
│   └── workflows/
│       └── ingest.yml
│
├── dashboard/
│   └── Football Analytics Dashboard.pbix
│
├── db/
│   ├── schema.sql
│   └── merge_upserts.sql
│
├── ingestion/
│   ├── config.py
│   ├── daily_ingest.py
│   ├── fetch_fixtures.py
│   ├── fetch_leagues.py
│   ├── fetch_players.py
│   ├── fetch_standings.py
│   ├── fetch_teams.py
│   ├── requirements.txt
│   └── utils.py
│
└── README.md
```

---

## Running Locally

Clone the repository:

```bash
git clone https://github.com/ThomasMillar/Football-Analytics-Platform.git
cd Football-Analytics-Platform
```

Install dependencies:

```bash
pip install -r ingestion/requirements.txt
```

Configure the required environment variables:

```env
API_FOOTBALL_KEY=your_api_key
AZURE_SQL_CXN=your_connection_string
LEAGUE_ID=39
SEASON=2024
```

Run the core ingestion pipeline:

```bash
python -m ingestion.daily_ingest
```

Player ingestion can be run separately when required.

---

## What I Learned

This project taught me different lessons from my UK Housing Data Platform because the source data comes from a live external API rather than large static files.

### Designing around API limitations

I had to account for authentication, pagination, rate limits, retries and failed requests rather than assuming the entire dataset would always be available at once.

The player endpoint in particular showed me how API limits can directly affect pipeline design.

### Understanding data grain

Football data exists at different levels. Matches, standings and player statistics cannot all be treated as the same type of dataset.

This project helped me understand why defining the grain of a fact table before loading data is important.

### Using staging tables

Loading API responses into staging tables before updating the final model made the pipeline easier to debug and gave me a clear separation between extracted data and reporting data.

### Incremental loading

Using SQL `MERGE` statements helped me understand how to update records that change over time rather than repeatedly inserting duplicates or rebuilding the whole dataset.

### Moving from scripts to automation

Running ingestion through GitHub Actions exposed problems that were easy to miss when running scripts manually, particularly around configuration, credentials, dependencies and database connectivity.

It helped turn the project from a set of Python scripts into a more complete pipeline.

---

## Future Improvements

- Add player ingestion to the main scheduled pipeline
- Expand to additional leagues and seasons
- Add automated data quality checks
- Add pipeline logging and monitoring
- Add automated tests for transformation logic
- Improve Power BI measures with more per-90 statistics
- Add historical season comparisons
- Publish a public-facing version of the dashboard

---

## Summary

This project combines API ingestion, Python, Azure SQL, incremental SQL transformations, scheduled workflows and Power BI in one end-to-end platform.

The main focus was learning how the different parts of a data pipeline work together, particularly how API limitations affect ingestion, how database design affects reporting and how automation changes the requirements of a pipeline.
