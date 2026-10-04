> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0Qfbzbz69PE

# you asked

message time: 2026-09-23 19:24:09

in Snowflake, what's the difference between 

"rlike(fm.measurement_description, 'height.*', 'i')"

and 

"rlike(fm.measurement_description, 'height%', 'i')"

---

# yuanbao response

In Snowflake, `RLIKE` uses **regular expressions** (regex), not SQL `LIKE` wildcards. This is the key to understanding the difference.

### 1. `rlike(fm.measurement_description, 'height.*', 'i')`
- **`.`** is a regex metacharacter meaning "any single character".
- **`*`** means "zero or more of the preceding element".
- Combined, **`.*`** means "zero or more of any character".
- The **`'i'`** flag makes the match case-insensitive.
- **What it matches:** Any string that contains `height` (in any case) followed by **anything** (or nothing).  
  ✅ Matches: `"Height"`, `"HEIGHT: 180 cm"`, `"height_in_inches"`, `"Patient height is 5'10"`  
  ❌ Does not match: `"weight"` (doesn't contain "height")

### 2. `rlike(fm.measurement_description, 'height%', 'i')`
- In regex, **`%` is NOT a wildcard**—it is a literal percent sign.
- The pattern looks for the exact characters `height` followed immediately by a literal `%` sign.
- The **`'i'`** flag still makes it case-insensitive for the letters.
- **What it matches:** Only strings that end with (or contain) `height` followed by a literal `%` symbol.  
  ✅ Matches: `"height%"`, `"HEIGHT%"`, `"Body height%"`  
  ❌ Does **not** match: `"height"`, `"height cm"`, `"height: 180"` (no literal `%` present)

---

### Summary Table

| Pattern | `height.*` | `height%` |
|---------|------------|-----------|
| **Wildcard meaning** | Regex `.*` = any characters | Literal `%` sign |
| **Matches "height" alone** | ✅ Yes | ❌ No (requires `%` after) |
| **Matches "height 180"** | ✅ Yes | ❌ No |
| **Matches "height%"** | ✅ Yes (`.*` includes `%`) | ✅ Yes |
| **Common use case** | "Find any description mentioning height" | "Find descriptions with a literal percent sign after 'height' (e.g., 'height%')" |

### ⚠️ Common Pitfall
If you intended `height%` to mean "height followed by anything" (like SQL `LIKE 'height%'`), you should use `height.*` instead. `RLIKE` does **not** interpret `%` as a wildcard—only `LIKE` and `ILIKE` do.

```sql
-- If you want SQL wildcard behavior, use ILIKE:
fm.measurement_description ILIKE 'height%'

-- If you want regex behavior, use RLIKE with .*:
rlike(fm.measurement_description, 'height.*', 'i')
```