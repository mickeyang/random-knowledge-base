> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0QS9lEUFmDo

# you asked

message time: 2026-09-15 16:50:13

inSnowflake, how to restored a table back to 10 minutes ago, table name is master_monthly_snapshot

---

# yuanbao response

In Snowflake, 10 minutes ago is well within the default 1‑day Time Travel window (Standard) and up to 90 days on Enterprise permanent tables, as long as `DATA_RETENTION_TIME_IN_DAYS` isn’t 0. [citation:1][citation:14]

Replace `db.schema` with your real names.

### 1) Preview the table as it was 10 minutes ago
```sql
-- by offset (seconds; -60*10 = 10 minutes)
SELECT * FROM db.schema.master_monthly_snapshot AT(OFFSET => -60*10);

-- or by timestamp
SELECT * FROM db.schema.master_monthly_snapshot
AT(TIMESTAMP => CURRENT_TIMESTAMP() - INTERVAL '10 minutes');

-- row count check
SELECT COUNT(*) FROM db.schema.master_monthly_snapshot AT(OFFSET => -60*10);
```
Use `BEFORE(...)` instead of `AT(...)` if you want the state immediately before a specific statement/query ID. [citation:1][citation:7]

### 2) Restore it safely (recommended: clone, verify, swap)
Don’t overwrite the live table until you’ve checked the historical copy.

```sql
-- create a zero-copy clone of the 10-min-ago state
CREATE TABLE db.schema.master_monthly_snapshot_restored
CLONE db.schema.master_monthly_snapshot AT(OFFSET => -60*10);

-- verify
SELECT COUNT(*) FROM db.schema.master_monthly_snapshot_restored;
-- compare key totals vs current if needed
```

Then swap names atomically so ongoing queries aren’t broken:

```sql
ALTER TABLE db.schema.master_monthly_snapshot
SWAP WITH db.schema.master_monthly_snapshot_restored;
```

After you confirm everything is good, drop the old/restored one:

```sql
DROP TABLE db.schema.master_monthly_snapshot_restored;
```

`CLONE ... AT` is zero‑copy initially, so it’s fast and cheap until you modify either table. [citation:1][citation:8]

### 3) If you only want to undo part of the changes
Instead of swapping the whole table, pull the old rows/columns and merge back:

```sql
-- example: reinsert rows missing now (use real key columns)
INSERT INTO db.schema.master_monthly_snapshot
SELECT s.* FROM db.schema.master_monthly_snapshot AT(OFFSET => -60*10) s
WHERE NOT EXISTS (
  SELECT 1 FROM db.schema.master_monthly_snapshot t
  WHERE t.<primary_key> = s.<primary_key>
);
```

### 4) If the table was dropped (not just mutated)
10 minutes ago doesn’t matter much; use UNDROP within retention:

```sql
SHOW TABLES HISTORY IN SCHEMA db.schema LIKE 'master_monthly_snapshot';
UNDROP TABLE db.schema.master_monthly_snapshot;
```

If a new table already exists with that name, rename the current one first, then UNDROP. [citation:12]

### Checks / gotchas
- Confirm retention:
  ```sql
  SHOW TABLES LIKE 'MASTER_MONTHLY_SNAPSHOT' IN SCHEMA db.schema;
  -- or
  SELECT table_name, data_retention_time_in_days
  FROM db.information_schema.tables
  WHERE table_name='MASTER_MONTHLY_SNAPSHOT';
  ```
- If retention is 0, or 10 minutes ago is outside the retention window, Time Travel/clone will fail; beyond Time Travel you’d need Fail‑safe support (permanent tables only, not self‑service). [citation:9][citation:14]
- `AT` includes changes exactly at that timestamp; `BEFORE(STATEMENT=>...)` is best when you know the exact bad INSERT/UPDATE/DELETE query ID from Query History.

Want the exact commands for your DB/schema and whether the table is permanent/transient/temporary? Send the fully qualified name and whether the issue was DELETE/UPDATE/TRUNCATE/DDL/drop.