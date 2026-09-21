# Football Analytics Platform

An end-to-end football data project built using Python, Azure SQL, GitHub Actions and Power BI.

The project collects football data from API-Football, loads it into Azure SQL, transforms it into reporting tables, and uses Power BI for league, team and player analysis.

The final dashboard covers the top three leagues in England, Spain, France and Germany, plus the top two leagues in Italy.

---

## Project Goal

The aim of this project was to build more than a one-off dashboard.

I wanted to create a small data platform that could:

- Pull data from a REST API
- Handle pagination and API limits
- Load raw data into staging tables
- Transform data into fact and dimension tables
- Update existing records instead of creating duplicates
- Run ingestion through GitHub Actions
- Feed a Power BI model from Azure SQL

---

## Architecture

```text
API-Football
     ↓
Python Ingestion
     ↓
Azure SQL Staging
     ↓
SQL MERGE / UPSERT
     ↓
Fact + Dimension Tables
     ↓
Power BI
```

---

## Data Ingestion

The ingestion layer is split into separate Python modules for different parts of the API, including:

- Leagues
- Teams
- Fixtures
- Standings
- Players
- Player match statistics
- Match events
- Lineups
- Injuries
- Transfers

Each API response is transformed with pandas and loaded into staging tables before being merged into the main reporting tables.

Because the full dataset covers multiple leagues and several detailed endpoints, the API request allowance was not enough to load everything in one run.

I therefore loaded the historical data in stages by changing the configured leagues and seasons between runs, while keeping the same ingestion and transformation process.

This let me build a larger dataset without changing the overall pipeline design.

---

## Data Model

The SQL layer separates data by grain rather than storing everything in one large table.

Examples include:

```text
dim_competition
dim_team
dim_player
dim_date

fact_match
fact_standing
fact_player_season
fact_player_match
fact_player_event
```

This made it easier to build relationships in Power BI and work with league, match and player-level data in the same model.

SQL `MERGE` statements are used to update existing records where possible.

This is useful for football data because fixtures, standings and player statistics change throughout a season.

---

## Power BI Dashboard

The dashboard is split into three main areas.

### Overview

A league-level view showing areas such as:

- League standings
- Top scorers and assists
- Player ratings
- Cards
- Team goals
- Passing
- Shooting

### Team Breakdown

Allows a selected team to be analysed in more detail, including:

- Results
- Points progression
- Squad statistics
- Player appearances
- Player performance

### Player Comparison

Allows two players to be compared across areas such as:

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

- **Python** - API ingestion and orchestration
- **Pandas** - transforming API responses
- **Requests** - API communication
- **SQLAlchemy / PyODBC** - database connectivity
- **Azure SQL Database** - cloud data storage
- **SQL** - modelling and upserts
- **GitHub Actions** - scheduled ingestion
- **Power BI** - reporting and analysis
- **Git / GitHub** - source control

---

## Project Structure

```text
Football-Analytics-Platform/
│
├── .github/
│   └── workflows/
│
├── dashboard/
│   └── Football Analytics Dashboard.pbix
│
├── db/
│   └── SQL schema and merge scripts
│
├── ingestion/
│   └── Python ingestion modules
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

Install the required packages:

```bash
pip install -r ingestion/requirements.txt
```

Set the required environment variables for the API and Azure SQL connection, then run the ingestion pipeline:

```bash
python -m ingestion.daily_ingest
```

API keys and database credentials are kept outside the repository.

---

## What I Learned

This project taught me a lot about working with APIs compared with static files.

### API limits affect pipeline design

The biggest challenge was the amount of data I wanted to collect.

Once player stats, events, lineups, injuries and transfers were included across several leagues, the number of API requests became too large for one load.

I had to break the ingestion into stages by league and season.

This helped me understand that the limits of a source system can change how a pipeline needs to be designed.

### Data grain matters

Football data exists at several different levels.

A fixture, a league table row and a player season record all represent different things.

Separating these into different fact and dimension tables made the model easier to understand and made the Power BI relationships more reliable.

### Staging tables make debugging easier

Loading API data into staging tables first gave me a clear point where I could check what had been extracted before updating the main tables.

This was useful when dealing with changing API responses or failed loads.

### Incremental updates are better than rebuilding everything

Football data changes regularly.

Fixtures move from scheduled to completed, standings update after every matchday and player totals change throughout the season.

Using upserts helped me keep records current without repeatedly inserting duplicates.

### Automation is different from running a script manually

Moving ingestion into GitHub Actions meant the project had to work without relying on my local machine.

That made me think more about environment variables, dependencies, secrets and database connectivity.

---

## Future Improvements

- Add automated data quality checks
- Add better pipeline logging
- Add tests for transformation logic
- Add failure notifications
- Add more historical seasons
- Improve Power BI measures with more per-90 statistics
- Publish a public-facing version of the dashboard

---

## Summary

This project combines API ingestion, Python, Azure SQL, SQL transformations, GitHub Actions and Power BI in one end-to-end workflow.

The main thing I gained from building it was a better understanding of how API limits, database design and automation all affect the way a data pipeline needs to be built.
