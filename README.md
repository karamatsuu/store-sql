# Store — Retail Sales Model (SQL Server / T-SQL)

A **re-runnable** T-SQL script (`Store_schema.sql`) that builds a `Store`
database modelling stores, the regions they belong to, and their employees.

## What it creates

- **Database** `Store` (created only if missing).
- **Tables** `REGION`, `EMPLOYEE`, `STORE` with primary and foreign keys from
  the reference diagram, plus all the sample rows.
- **Sequence** `dbo.EMP_CODE_SEQ` — `EMPLOYEE.EMP_CODE` is filled with
  `NEXT VALUE FOR dbo.EMP_CODE_SEQ`, never typed by hand.
- **Default** — `EMPLOYEE.Store_code` defaults to `3`.
- **Check** — `STORE.store_ytd_sales` must be `> 0` (no negatives or zero).

## The circular foreign key

`STORE.EMP_CODE` points to the employee who manages the store, while
`EMPLOYEE.STORE_CODE` points to the store the employee works in — a cycle.
Loading data with both FKs active would deadlock, so the script:

1. creates `EMPLOYEE` **without** the FK to `STORE`;
2. loads `REGION`, `EMPLOYEE`, then `STORE`;
3. adds `FK_EMPLOYEE_STORE` at the end, once every referenced row exists.

## Re-runnability

The cleanup block drops `FK_EMPLOYEE_STORE` first (breaking the cycle), then the
tables and the sequence, before recreating everything. Re-running the script —
or running it on a fresh server — completes without errors, and the sequence
restarts at 1.

## Run it

Open `Store_schema.sql` in **SSMS** / **Azure Data Studio** and execute, or:

```bash
sqlcmd -S localhost -U sa -P <password> -i Store_schema.sql
```

Check the data:

```sql
SELECT * FROM EMPLOYEE ORDER BY EMP_CODE;
SELECT * FROM STORE    ORDER BY STORE_CODE;
SELECT * FROM REGION   ORDER BY REGION_CODE;
```

## Notes

- A rollback-wrapped block at the end demonstrates the `Store_code = 3` default
  without leaving test data behind.
- A commented-out insert shows the `store_ytd_sales > 0` check rejecting a
  negative value.
