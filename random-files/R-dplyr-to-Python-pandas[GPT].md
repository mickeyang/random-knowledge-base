> From: https://chatgpt.com/g/g-p-6a50d0ad67d88191acd0ad859172fff2-data-and-software-engineering/c/6a55f8df-d2f8-83e8-b637-fbf84e86a6fe

# you asked

message time: 2026-07-14 20:54:47

I have been using R programming language for data engineering and data analysis more often that using Python. Based on the attached conversation, generate a cheatsheet to help me convert my R dplyr brain to Python pandas brain.

---

# chatgpt response

Here is a **concise, data-engineering–focused cheatsheet** to translate your **R `dplyr` mental model → Python `pandas`**.

---

# Core Mental Model Shift

| Concept | dplyr (R) | pandas (Python) | Key Difference |
|--------|----------|----------------|---------------|
| Data object | tibble | DataFrame | Similar |
| Pipeline | `%>%` | method chaining (`.`) | No native pipe (use chaining) |
| Lazy eval | Yes | No | pandas executes immediately |
| NSE (non-standard eval) | Yes | No | pandas requires explicit column refs |
| Group state | persistent | temporary | pandas `groupby` is not sticky |

---

# 1. Select Columns

```r
df %>% select(col1, col2)
```

```python
df[['col1', 'col2']]
```

---

# 2. Filter Rows

```r
df %>% filter(col1 == "A")
```

```python
df[df['col1'] == 'A']
# or (recommended)
df.loc[df['col1'] == 'A']
```

From your file example, this is the correct fix:
```python
df[df['NHI_Number'] == 'ABC1234']
```

---

# 3. Mutate (Create/Transform Columns)

```r
df %>% mutate(new_col = col1 + col2)
```

```python
df['new_col'] = df['col1'] + df['col2']
```

Chain version:
```python
df.assign(new_col=lambda x: x['col1'] + x['col2'])
```

---

# 4. Arrange (Sort)

```r
df %>% arrange(col1)
df %>% arrange(desc(col1))
```

```python
df.sort_values('col1')
df.sort_values('col1', ascending=False)
```

---

# 5. Rename Columns

```r
df %>% rename(new = old)
```

```python
df.rename(columns={'old': 'new'})
```

---

# 6. Group By + Summarise

```r
df %>%
  group_by(col1) %>%
  summarise(n = n(), avg = mean(col2))
```

```python
df.groupby('col1').agg(
    n=('col2', 'count'),
    avg=('col2', 'mean')
).reset_index()
```

Key difference:
- pandas requires explicit column mapping
- `.reset_index()` is usually needed

---

# 7. Distinct

```r
df %>% distinct(col1)
```

```python
df[['col1']].drop_duplicates()
```

---

# 8. Join

```r
left_join(df1, df2, by = "id")
```

```python
df1.merge(df2, on='id', how='left')
```

---

# 9. Case When

```r
df %>%
  mutate(flag = case_when(
    col1 > 10 ~ "A",
    TRUE ~ "B"
  ))
```

```python
import numpy as np

df['flag'] = np.where(df['col1'] > 10, 'A', 'B')
```

Multiple conditions:
```python
df['flag'] = np.select(
    [df['col1'] > 10, df['col1'] > 5],
    ['A', 'B'],
    default='C'
)
```

---

# 10. Pipe vs Chaining

```r
df %>%
  filter(col1 > 10) %>%
  mutate(x = col2 * 2)
```

```python
(df
 .loc[df['col1'] > 10]
 .assign(x=lambda x: x['col2'] * 2)
)
```

Tip: Always wrap chained pandas in parentheses.

---

# 11. String Operations

```r
df %>% mutate(x = str_to_upper(col1))
```

```python
df['x'] = df['col1'].str.upper()
```

---

# 12. Handling Missing Values

```r
df %>% filter(!is.na(col1))
```

```python
df[df['col1'].notna()]
```

---

# 13. Count Shortcut

```r
df %>% count(col1)
```

```python
df['col1'].value_counts().reset_index()
```

---

# 14. Across (apply multiple columns)

```r
df %>% mutate(across(starts_with("col"), mean))
```

```python
df.filter(like='col').mean()
```

Or:
```python
df.apply(lambda x: x.mean() if x.name.startswith('col') else x)
```

---

# 15. Row-wise Operations (important difference)

```r
df %>% rowwise() %>% mutate(x = sum(c_across(a:c)))
```

```python
df['x'] = df[['a','b','c']].sum(axis=1)
```

---

# Critical Differences (High Impact)

## 1. Column Referencing
R:
```r
filter(col1 == 1)
```

Python:
```python
df['col1'] == 1
```

No implicit column access.

---

## 2. No Trailing Comma Indexing

Your mistake:
```python
df[df.NHI_Number == 'ABC1234',]  # wrong
```

Correct:
```python
df[df['NHI_Number'] == 'ABC1234']
```

---

## 3. Assignment Pitfall (SettingWithCopy)

Avoid:
```python
df[df['col1'] > 0]['col2'] = 1  # risky
```

Use:
```python
df.loc[df['col1'] > 0, 'col2'] = 1
```

---

## 4. groupby is NOT persistent

R:
```r
df %>% group_by(col1) %>% mutate(...)
```

Python:
```python
df.groupby('col1')['col2'].transform(...)
```

You must reapply grouping every time.

---

# Data Engineering Tips (Based on Your Workflow)

## Use `.loc` as default
```python
df.loc[condition, columns]
```

## Prefer explicit over implicit
- Always use `df['col']`, not `df.col`

## Use raw strings for paths (from your file)
```python
pd.read_csv(r"C:\data\file.csv")
```

## Use `.query()` for readability
```python
df.query("NHI_Number == 'ABC1234'")
```

---

# Quick Mapping Summary

| dplyr | pandas |
|------|--------|
| filter | df[...] / .loc |
| select | df[[...]] |
| mutate | df[...] = |
| summarise | groupby().agg() |
| arrange | sort_values |
| distinct | drop_duplicates |
| left_join | merge |
| case_when | np.select / np.where |
| %>% | chaining |

---

If you want next step:  (production-grade, not toy examples).

---
Powered by [AI Exporter](https://saveai.net)