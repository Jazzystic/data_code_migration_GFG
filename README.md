# Historical Data Migration — CSV to PostgreSQL

Python notebooks used to clean and load historical hospitality data from CSV exports into a PostgreSQL data warehouse hosted on Azure, as part of the GFG Real Estate Asset Management centralized database project.

## Notebooks

### `migración_histórico_comentarios_GFG.ipynb`
Migrates guest review history. Handles text sanitation (stripping line breaks that would break batch inserts), date coercion for posting and response timestamps, and null normalization before a batched load into the `raw` layer.

### `migración_histórico_consolidado_GFG.ipynb`
Migrates the consolidated weekly rankings dataset — roughly 57 columns spanning TripAdvisor, Google, Expedia and Booking metrics plus social media follower counts. Covers:

- Column mapping from spreadsheet headers (`TA.Reviews>`, `GG.Ratings>`) to SQL-safe identifiers
- Type coercion: integers for counts, decimals for ratings, `DATE` for week markers
- Duplicate detection against the target table's composite key, distinguishing true duplicates from rows sharing key columns but differing elsewhere
- Batched inserts via `psycopg2.extras.execute_batch`

### `migración_histórico_rankingstable_GFG.ipynb`
Migrates the weekly rankings table. Same pipeline shape as the others: column normalization, type coercion, null handling, batched load into the `raw` layer.

## Approach

Both notebooks follow the same shape: load CSV → normalize columns and types → convert pandas nulls (`NaT`, `pd.NA`) to SQL `NULL` → batch insert. The null conversion matters more than it looks: pandas nullable types don't translate to `psycopg2` parameters directly, and silent coercion produces rows that insert without error but hold wrong values.

## Configuration

Connection details are read from environment variables. Copy `.env.example` to `.env` and fill in your own:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `PGHOST` | PostgreSQL server hostname |
| `PGDATABASE` | Target database |
| `PGUSER` | Database user |
| `PGPASSWORD` | Database password |

## Requirements

`pandas`, `psycopg2-binary`

Originally developed and run in Google Colab.

## Notes

Notebook outputs are stripped — they contained operational data. Source CSVs are not included.

---

Julio López - Data Architect | julio@gfgam.com.mx
