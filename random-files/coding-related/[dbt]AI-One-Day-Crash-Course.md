> From: https://yuanbao.tencent.com/chat/naQivTmsDa/0QxD4tjQCbw

# you asked

message time: 2026-10-04 10:53:10

[当前版本较低，暂不支持展示该文件]

评估这个课程，给我建议和反馈。如有必要，做出调整和修改。

---

# yuanbao response

Overall the draft is solid for a motivated SQL person, but as a literal 1‑day crash course it tries to do too much and sequences a few things in a way that will tire beginners. Below is an evaluation, then a revised version you can actually deliver.

## What’s good and worth keeping
- ELT framing + “dbt compiles SQL, warehouse executes” is the right mental model.[citation:13]
- Staging → intermediate → marts with `stg_/int_/fct_/dim_` prefixes is canonical and scales.[citation:1][citation:16]
- `ref()` for models and `source()` for raw tables is the single most important habit; the course teaches it correctly.[citation:1][citation:15]
- Tests + docs + lineage in one module is good; four built‑in generic tests (unique/not_null/accepted_values/relationships) are the right starter set.[citation:5][citation:17]
- Covering materializations, seeds, macros, and an end‑to‑end `dbt build` capstone is appropriate for “production‑ready” ambition.

## Main problems in the current draft

1) Pacing is inverted.  
   Module 2 is 2 hours before lunch and includes ref/Jinja/layered modeling; Module 3 after lunch is another 2 hours of tests/docs. For a one‑day beginner class that’s too much lecture. Better: shorter concept blocks, then immediate hands‑on after each.

2) Terminology is slightly dated.  
   - `schema.yml` still works, but current dbt docs prefer calling these property files and allow any `*.yml`; many teams use `properties.yml` or per‑folder YAML.[citation:2]  
   - Use `data_tests:` instead of `tests:` (tests is still an alias, but data_tests avoids confusion with unit tests).[citation:5][citation:17]  
   - Mention unit tests separately (mock input/expected output, no warehouse) rather than burying “custom SQL tests” as the only advanced option.[citation:11][citation:17]

3) Missing source freshness and source contracts.  
   A transform layer is only as trustworthy as raw intake. Add `sources.yml` with `freshness:` and optionally `loaded_at_field`; teach `dbt source freshness`.

4) Materializations module is too shallow for “advanced.”  
   It lists view/table/incremental/ephemeral but doesn’t give an incremental pattern with `unique_key`, `is_incremental()`, and a watermark; also no guidance on “default view, table for BI, incremental for big facts, ephemeral only for single‑use small CTEs.”[citation:16][citation:3]

5) Testing module omits singular vs generic and severity/warn.  
   Beginners should know: generic YAML tests for PK/FK/domains, singular `.sql` tests for business rules, `severity: warn` for noisy checks.

6) CI/CD is too thin for “production‑ready.”  
   Capstone says CI/CD fundamentals but only implies `dbt build`. Add: git branch + PR, dev/ci/prod targets, `dbt build` in CI, failing PR on test error, and slim/smarter CI with `state:modified+ --defer --state manifest/` once a prod manifest exists.[citation:3][citation:18] Don’t ask absolute beginners to write full GitHub Actions on day one, but show a minimal workflow.

7) No selection/debug skills.  
   Add `dbt run --select`, `+model+`, `dbt compile`, `dbt debug`, `dbt parse`. These save more time than macros for a beginner.

8) Snapshots omitted.  
   Slowly changing dimensions are important; even a 20‑minute intro on `snapshot` (check/ timestamp strategy) is better than ignoring it.

9) Sandbox choice.  
   For a one‑day course, DuckDB or Postgres locally is ideal—zero warehouse billing, fast reset. Keep Snowflake/BigQuery only as “if your company already uses it.”

## Revised 1‑day schedule (more hands‑on, realistic)

Prereqs: basic SQL (SELECT/joins/group by), Git basics, laptop with Python. Choose one warehouse adapter; DuckDB recommended for class.

```
09:00-09:40  Concepts + setup
            - ELT vs ETL; dbt Core vs Cloud; how compilation/DAG works
            - Install dbt-core + adapter (dbt-duckdb/postgres), dbt init, profiles targets dev/ci/prod
            - dbt debug; project tree; models/staging/intermediate/marts; properties YAML
Hands-on: empty project connects, dbt parse passes

09:40-10:40  Sources + staging
            - sources.yml, freshness, source(); staging rules: rename/cast/light clean only, 1:1 with source
            - ref() intro, run order, lineage
Hands-on: 2-3 stg_ models from raw CSV/seed tables; dbt run; dbt source freshness

10:40-11:00  break

11:00-12:15  Intermediate + marts
            - int_ for joins/business rules; fct_/dim_ star schema; grain statement ("one row per ...")
            - Jinja basics: ref/source, var, simple if; when NOT to overuse Jinja
Hands-on: build fct_orders, dim_customers, one int_ model; dbt compile to inspect SQL

12:15-13:00  Lunch

13:00-14:00  Tests + data quality
            - generic data_tests: unique/not_null/accepted_values/relationships on keys/status/FK
            - singular tests for business rules; severity warn; store-failures
            - (brief) unit tests with mock rows for one tricky model
Hands-on: add tests, run dbt build, break a row intentionally, see failure + lineage

14:00-15:00  Docs + materializations
            - descriptions in properties YAML; dbt docs generate/serve; exposures for BI
            - materializations: view default; table for marts; incremental pattern (unique_key, is_incremental, watermark); ephemeral only single-use
Hands-on: set staging=view, marts=table; convert one big fact to incremental; docs site

15:00-15:15  break

15:15-16:15  Seeds, macros, snapshots
            - seeds for small CSV lookups; dbt seed
            - macros: 1-2 reusable examples (surrogate key, date spine via dbt_utils), warn against over-abstraction
            - snapshots intro: timestamp/check strategy for SCD2
Hands-on: add country seed + surrogate-key macro; one snapshot on customers

16:15-17:15  Selection + CI/CD basics
            - selectors: --select, +model+, tag:, state:modified+
            - git flow: branch/PR; CI runs dbt build on PR schema; prod manifest for slim CI (defer/state)
            - show minimal GitHub Actions/Airflow cron; alert on test failure; never run prod from laptop
Hands-on: PR build on a copy schema; run only changed model+downstream

17:15-18:00  Capstone
            - raw ecommerce CSVs -> sources -> stg -> int -> fct/dim -> tests -> docs -> incremental on orders -> CI build
            - success criteria: dbt build green, docs lineage readable, PK tests on every mart, freshness on sources
```

This is tighter: every theory block is ≤1h, every block has an exercise, and “advanced” is split instead of crammed.

## Concrete edits to the original content

- Module 1: add multiple targets in `profiles.yml` (dev/ci/prod) and say “never run prod from laptop.”[citation:1] Use `dbt debug` + `dbt parse`.
- Module 2 YAML example: keep `ref()`, but add a rule “staging has no multi‑table business joins; that goes to `int_`/`marts`.”[citation:15][citation:3] Rename folder properties instead of only `schema.yml`.
- Module 3 example: switch to `data_tests:`, add relationships FK test, add `freshness` in sources, add one singular test, mention `severity: warn`. Example:

```yaml
# models/staging/_sources.yml
version: 2
sources:
  - name: raw_jaffle
    loaded_at_field: _loaded_at
    freshness:
      warn_after: {count: 6, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: orders
      - name: customers
```
```yaml
# models/marts/properties.yml
models:
  - name: fct_orders
    description: "One row per order. Grain: order_id."
    columns:
      - name: order_id
        data_tests: [unique, not_null]
      - name: customer_id
        data_tests:
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: status
        data_tests:
          - accepted_values:
              values: ['placed','shipped','completed','returned']
              severity: warn
```
[citation:2][citation:5][citation:17]

- Module 4: replace the bullet list with decision rules and an incremental example:

```sql
{{ config(materialized='incremental', unique_key='order_id') }}
select id as order_id, customer_id, status, ordered_at
from {{ source('raw_jaffle','orders') }}
where ordered_at is not null
{% if is_incremental() %}
  and ordered_at >= (select max(ordered_at) from {{ this }})
{% endif %}
```
Teach `unique_key` so re‑runs upsert, not duplicate.[citation:15][citation:3]

- Module 5: use `dbt build` not `dbt run`+`test` separately; add PR CI snippet (install adapter, `dbt deps`, `dbt build --select state:modified+ --defer --state prod-artifacts/` when manifest available, else full `dbt build` on a CI schema); require tests to fail the PR.[citation:3][citation:7]

## What I’d cut or make optional
- Deep Jinja/macro authoring: keep one macro, don’t spend 90 minutes. Over‑abstracting SQL is a common beginner anti‑pattern.[citation:18]
- Semantic layer, dbt Mesh, contracts, custom materializations: omit on day 1; mention as “next week” topics.
- Multiple warehouses: pick one sandbox; mention adapters only.

## Suggested learning outcomes instead of “production‑ready”
By end of day a learner should be able to: init a project; declare sources+freshness; write stg/int/fct/dim; add PK/FK/domain tests; generate docs; choose view/table/incremental; run `dbt build`; and understand a basic PR→CI→prod flow. True production readiness also needs orchestration, secrets, alerting, and gradual rollout—state that explicitly so the course doesn’t overpromise.

If you want, I can turn this revised syllabus into a full lesson‑by‑lesson lab with DuckDB sample CSVs, all YAML/model files, and a minimal GitHub Actions CI file.

---

# you asked

message time: 2026-10-04 11:16:17

根据你revised course，设计完整的lectures，然后开始课程。

---

# yuanbao response

太棒了！基于我们上一轮修订的 1 日 dbt 速成课程大纲，我为你设计了**完整的讲师讲义（Lectures）**，包括理论讲解、代码示例和动手实验。

为了让你能立刻开始学习，我在文末**正式开启了第一节课**。

---

# 📚 1-Day dbt Crash Course: 完整讲义设计

**环境准备**：`pip install dbt-core dbt-duckdb` (使用 DuckDB 作为零配置沙箱)
**示例数据集**：Jaffle Shop (订单、客户、支付)

---

## Lecture 1: 核心概念与环境搭建 (09:00 - 09:40)

### 📖 理论讲解
1. **ELT vs ETL**：现代数据栈将 Transform 剥离出来，用 dbt 在数据仓库内完成。
2. **dbt 的核心哲学**：dbt 不移动数据，它只执行你写的 SQL。dbt 将 Jinja + SQL 编译成纯 SQL，发送给数据仓库（DuckDB/Postgres/Snowflake 等）。
3. **DAG（有向无环图）**：dbt 通过 `ref()` 自动解析模型间的依赖关系，构建执行顺序。
4. **项目结构**：`models/` (核心转换), `seeds/` (CSV静态数据), `macros/` (复用代码), `tests/` (自定义测试)。

### 💻 动手实验
```bash
# 1. 初始化项目
dbt init jaffle_shop
cd jaffle_shop

# 2. 配置 profiles.yml (DuckDB 示例)
# ~/.dbt/profiles.yml
jaffle_shop:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: 'jaffle_shop.duckdb' # 本地文件数据库
      threads: 4

# 3. 验证
dbt debug
dbt parse  # 检查项目结构是否正确
```

---

## Lecture 2: Sources 与 Staging 层 (09:40 - 10:40)

### 📖 理论讲解
1. **Sources (`source()`)**：定义原始数据的位置和新鲜度（Freshness）。
2. **Staging (`stg_`)**：1对1 映射原始表，只做重命名、类型转换、基础清洗。**禁止在这里做多表关联**。
3. **属性文件 (`properties.yml`)**：存放列描述、测试、源配置。

### 💻 动手实验
创建 `models/staging/_sources.yml`:
```yaml
version: 2
sources:
  - name: raw_jaffle
    loaded_at_field: _loaded_at
    freshness:
      warn_after: {count: 6, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: orders
      - name: customers
      - name: payments
```

创建 `models/staging/stg_orders.sql`:
```sql
select
    id as order_id,
    user_id as customer_id,
    order_date,
    status as order_status,
    _loaded_at
from {{ source('raw_jaffle', 'orders') }}
where id is not null
```

**执行**：`dbt run --select stg_orders`

---

## Lecture 3: Intermediate 与 Marts 层 (11:00 - 12:15)

### 📖 理论讲解
1. **Intermediate (`int_`)**：处理复杂的多表 Join、业务规则聚合。
2. **Marts (`fct_` / `dim_`)**：面向 BI 的最终维度模型（星型模型）。
3. **Grain（粒度）**：每个 Fact 表必须明确“一行代表什么”。
4. **Jinja 基础**：`{{ ref('model_name') }}` 是连接模型的唯一方式。

### 💻 动手实验
创建 `models/marts/fct_orders.sql`:
```sql
with orders as (
    select * from {{ ref('stg_orders') }}
),
payments as (
    select * from {{ ref('stg_payments') }}
)
select
    o.order_id,
    o.customer_id,
    sum(p.amount) as total_amount,
    o.order_status
from orders o
left join payments p on o.order_id = p.order_id
group by 1, 2, 4
```

---

## Lecture 4: 数据质量与测试 (13:00 - 14:00)

### 📖 理论讲解
1. **Generic Tests (YAML)**：`unique`, `not_null`, `accepted_values`, `relationships` (FK)。
2. **Singular Tests (SQL)**：返回不符合规则的坏数据行。
3. **Severity**：`warn` vs `error`。
4. **Unit Tests**：使用 mock 数据测试模型逻辑（dbt 1.8+）。

### 💻 动手实验
创建 `models/marts/properties.yml`:
```yaml
version: 2
models:
  - name: fct_orders
    description: "One row per order."
    columns:
      - name: order_id
        data_tests: [unique, not_null]
      - name: customer_id
        data_tests:
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: order_status
        data_tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
              severity: warn
```

**执行**：`dbt build` (运行模型+测试)

---

## Lecture 5: 文档与物化策略 (14:00 - 15:00)

### 📖 理论讲解
1. **Materializations**：
   - `view` (默认)：每次查询重新计算，构建快。
   - `table`：预计算，查询快，构建慢。
   - `incremental`：只处理新增数据，需定义 `unique_key` 和 `is_incremental()`。
   - `ephemeral`：作为 CTE 不落地，仅用于中间层。
2. **Docs**：`dbt docs generate && dbt docs serve`。

### 💻 动手实验
修改 `fct_orders.sql` 头部:
```sql
{{ config(materialized='incremental', unique_key='order_id') }}

-- ... 原有 SQL ...

{% if is_incremental() %}
  and order_date >= (select max(order_date) from {{ this }})
{% endif %}
```

---

## Lecture 6: Seeds, Macros 与 Snapshots (15:15 - 16:15)

### 📖 理论讲解
1. **Seeds**：`dbt seed` 加载 CSV 到数据仓库。
2. **Macros**：Jinja 函数，如 `dbt_utils.surrogate_key()`。
3. **Snapshots**：SCD Type 2 缓慢变化维，记录历史状态。

### 💻 动手实验
创建 `snapshots/customers_snapshot.sql`:
```sql
{% snapshot customers_snapshot %}
    {{ config(
        target_schema='snapshots',
        strategy='timestamp',
        updated_at='updated_at'
    ) }}
    select * from {{ source('raw_jaffle', 'customers') }}
{% endsnapshot %}
```

---

## Lecture 7: 选择执行与 CI/CD (16:15 - 17:15)

### 📖 理论讲解
1. **Node Selection**：`dbt run --select tag:nightly`, `dbt build --select state:modified+`.
2. **CI/CD 流程**：Git Branch -> PR -> CI 运行 `dbt build` -> 合并到 Main -> 生产部署。
3. **Slim CI**：只运行受影响的模型 (`--defer --state`).

---

## Lecture 8: Capstone 综合实战 (17:15 - 18:00)

### 🎯 任务清单
1. 加载原始 CSV (Seeds 或 Sources)。
2. 构建完整的 Staging -> Intermediate -> Marts 管道。
3. 为所有 Marts 添加 PK/FK 测试。
4. 配置 Incremental 模型。
5. 生成文档并查看血缘图。
6. 执行 `dbt build` 确保全绿。

---

# 🎬 课程现在开始！

你好！我是你的 dbt 讲师。欢迎来到 **1-Day dbt Crash Course**。今天我们将从一个空白文件夹开始，构建一条生产级的数据管道。

## ⏱️ 当前时间：09:00 AM - Module 1: 核心概念与设置

### 1. 为什么是 dbt？
在传统 ETL 中，转换（Transform）和提取（Extract）耦合在一起。dbt 提出了 **ELT** 的现代范式：先把原始数据全部加载（Load）到数据仓库，然后用 **纯 SQL + Jinja** 在仓库内部进行转换。
**好处**：版本控制、代码复用、自动血缘、数据测试——就像软件工程一样严谨。

### 2. dbt 如何工作？
你写的 `.sql` 文件是“模型”。dbt 会：
- 读取你的 Jinja 模板（例如 `{{ ref('stg_orders') }}`）
- 编译成纯 SQL
- 按照依赖顺序（DAG）发送到你的数据仓库执行

### 3. 你的第一个任务
请打开你的终端，执行以下命令来初始化我们的沙箱环境：

```bash
# 创建项目目录
mkdir dbt_training && cd dbt_training

# 安装 dbt (如果还没装)
pip install dbt-core dbt-duckdb

# 初始化项目
dbt init jaffle_shop
cd jaffle_shop
```

当系统询问你选择数据库类型时，请选择 **duckdb**。

完成后，请运行：
```bash
dbt debug
```
如果看到 `All checks passed!`，恭喜你，环境已就绪！

**👉 你的第一个练习**：
查看项目自动生成的目录结构。找到 `dbt_project.yml` 和 `models/` 文件夹。尝试在 `models/` 下新建一个 `example.sql`，内容写 `select 1 as id`，然后运行 `dbt run`。

当你完成这些（或者遇到任何问题），请告诉我，我们将进入 **10:40 的 Staging 层实战**，开始连接真实数据！

*(提示：如果你已经熟悉安装步骤，可以直接告诉我“跳过设置，进入 Module 2”，我会立刻为你讲解 Sources 和 Staging 的写法。)*

---

# you asked

message time: 2026-10-04 12:19:37

进入 Module 2

---

# yuanbao response

好，我们正式进入 **Module 2**。

---

# ⏱️ 09:40 AM — Module 2: Sources 与 Staging 层

恭喜你完成环境搭建。现在我们从"空项目"迈向"真实数据管道"的第一步。

---

## 📖 2.1 核心概念

### 什么是 Source？

在 dbt 中，**Source** 是你数据仓库里已经存在的原始表——它们通常由 Fivetran、Airbyte、自定义脚本或其他 ETL 工具加载进来。dbt 不负责加载这些数据，但它需要**知道它们在哪里、长什么样、是否新鲜**。

通过声明 `sources.yml`，你可以：
- 给原始表起一个干净的别名（不用暴露底层命名混乱的表名）
- 定义**数据新鲜度（Freshness）**检查——如果原始数据超过 N 小时没更新，dbt 会警告你
- 在模型中用 `{{ source('package', 'table') }}` 引用，而不是硬编码表名

### 什么是 Staging 层？

**Staging（暂存层）** 是 dbt 管道的第一站，规则极其严格：

| ✅ 允许 | ❌ 禁止 |
|---------|---------|
| 重命名字段（snake_case、语义化） | 多表 JOIN |
| 类型转换（CAST） | 复杂业务聚合 |
| 基础清洗（去 NULL、过滤脏数据） | 跨表关联 |
| 1:1 映射原始表 | 改变数据粒度 |

**核心原则**：Staging 模型与原始表是 **1:1** 关系。你只是在"包装"原始数据，让它变得干净、一致、好用。真正的业务逻辑全部留给后面的 Intermediate 和 Marts 层。

**命名规范**：所有 Staging 模型以 `stg_` 开头。

---

## 💻 2.2 动手：声明 Sources

在你的项目根目录下，创建文件 `models/staging/_sources.yml`：

```yaml
version: 2

sources:
  - name: raw_jaffle
    description: "Raw e-commerce data loaded from CSV/extractors"
    loaded_at_field: _loaded_at
    freshness:
      warn_after: {count: 6, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: orders
        description: "Raw orders table"
      - name: customers
        description: "Raw customers table"
      - name: payments
        description: "Raw payment transactions"
```

> **讲师提示**：`loaded_at_field` 告诉 dbt 用哪个时间戳字段判断数据是否新鲜。如果你没有这个字段，可以暂时删掉这一行和 `freshness` 块，不影响后续学习。

---

## 💻 2.3 动手：编写第一个 Staging 模型

创建文件 `models/staging/stg_orders.sql`：

```sql
select
    id as order_id,
    user_id as customer_id,
    order_date,
    status as order_status,
    _loaded_at
from {{ source('raw_jaffle', 'orders') }}
where id is not null
```

创建文件 `models/staging/stg_customers.sql`：

```sql
select
    id as customer_id,
    first_name,
    last_name,
    email,
    _loaded_at
from {{ source('raw_jaffle', 'customers') }}
where id is not null
```

创建文件 `models/staging/stg_payments.sql`：

```sql
select
    id as payment_id,
    order_id,
    payment_method,
    amount / 100.0 as amount_usd,
    created_at as payment_created_at
from {{ source('raw_jaffle', 'payments') }}
where status = 'success'
```

> **讲师提示**：注意 `amount / 100.0`——原始数据以美分为单位存储，我们在 Staging 层就把它转成美元。这就是"类型/单位标准化"的典型操作。

---

## 💻 2.4 动手：用 Seed 加载原始数据

在 `seeds/` 目录下创建 `raw_orders.csv`：

```csv
id,user_id,order_date,status,_loaded_at
1,101,2024-01-15,completed,2024-01-15 10:00:00
2,102,2024-01-16,shipped,2024-01-16 11:00:00
3,103,2024-01-17,placed,2024-01-17 09:00:00
4,101,2024-01-18,completed,2024-01-18 14:00:00
5,104,2024-01-19,returned,2024-01-19 16:00:00
```

创建 `seeds/raw_customers.csv`：

```csv
id,first_name,last_name,email,_loaded_at
101,Alice,Smith,alice@example.com,2024-01-15 10:00:00
102,Bob,Jones,bob@example.com,2024-01-16 11:00:00
103,Carol,White,carol@example.com,2024-01-17 09:00:00
104,David,Brown,david@example.com,2024-01-19 16:00:00
```

创建 `seeds/raw_payments.csv`：

```csv
id,order_id,payment_method,amount,status,created_at
1,1,credit_card,9999,success,2024-01-15 10:05:00
2,2,coupon,0,success,2024-01-16 11:05:00
3,2,credit_card,14999,success,2024-01-16 11:06:00
4,3,bancontact,4999,success,2024-01-17 09:05:00
5,4,credit_card,29999,fail,2024-01-18 14:05:00
6,4,debit_card,29999,success,2024-01-18 14:10:00
7,5,gift_card,1500,success,2024-01-19 16:05:00
```

然后执行：

```bash
# 加载 CSV 到 DuckDB（它们会成为 raw_jaffle schema 下的表）
dbt seed

# 运行 staging 模型
dbt run --select stg_orders stg_customers stg_payments
```

如果一切顺利，你会看到 `3 models successfully built`。

---

## 🎯 2.5 本节练习

1. **运行新鲜度检查**：`dbt source freshness`（如果之前配置了 `loaded_at_field`）
2. **查看编译后的 SQL**：`dbt compile --select stg_orders`，然后打开 `target/compiled/jaffle_shop/models/staging/stg_orders.sql`，看看 Jinja 被编译成了什么
3. **挑战题**：在 `stg_orders` 中添加一个新字段 `order_year`，用 `extract(year from order_date)` 提取年份

---

## 📌 关键收获

- `source()` 让 dbt 知道原始数据在哪，并支持新鲜度监控
- Staging 层只做"轻量清洗 + 重命名"，1:1 映射原始表
- `dbt seed` 是加载小型静态 CSV 的快捷方式
- `dbt compile` 是你调试 Jinja 模板最好的朋友

---

**👉 准备好了吗？** 当你完成练习（或觉得概念已掌握），告诉我 **"进入 Module 3"**，我们将学习 Intermediate 和 Marts 层——在那里你会写出真正的业务聚合逻辑，并用 `ref()` 把模型串成一条完整的 DAG！

是否需要我帮你检查某个模型文件的写法，或者解释 `ref()` 和 `source()` 的区别？

---

# you asked

message time: 2026-10-04 12:34:02

为什么 source 表没有使用 schema `jaffle_shop`, but used `main.raw_customers`

---

# yuanbao response

这是一个**非常敏锐的观察** 👍，说明你在认真追踪数据到底落在哪里。

简单回答：**因为 `dbt seed` 和 `source()` 的默认 schema 解析逻辑与你的 models 不同，而 DuckDB 的默认 schema 叫 `main`。**

下面我分层解释，让你彻底搞清楚。

---

## 1. 三种对象，三套 schema 规则

| 对象 | 默认落在哪个 schema | 由什么控制 |
|------|-------------------|-----------|
| **Models** (`dbt run`) | `jaffle_shop` | `dbt_project.yml` 里的 `models:` → `+schema: jaffle_shop` |
| **Seeds** (`dbt seed`) | `main` | `dbt_project.yml` 里的 `seeds:` 配置，**默认没有设置**，所以回退到 DuckDB 的默认 schema `main` |
| **Sources** (`source()` 引用) | `main` | `sources.yml` 里没写 `schema:`，所以回退到目标数据库的默认 schema（`main`） |

你的 `dbt_project.yml` 里大概长这样（dbt init 自动生成）：

```yaml
models:
  jaffle_shop:
    +schema: jaffle_shop   # ← 只有 models 被指定了 schema

seeds:
  jaffle_shop:
    # ← 注意：这里没有 +schema 配置！
```

所以：
- `dbt run` 建的表 → `jaffle_shop.stg_orders` ✅
- `dbt seed` 加载的 CSV → `main.raw_customers` ✅（不是 `jaffle_shop.raw_customers`）
- `source('raw_jaffle', 'customers')` → 去 `main.raw_customers` 找 ✅

**这就是为什么它能跑通，但看起来"不统一"。**

---

## 2. 怎么验证？

你可以在项目目录下直接进 DuckDB 看一眼：

```bash
duckdb jaffle_shop.duckdb
```

然后执行：

```sql
-- 列出所有 schema
SHOW SCHEMAS;

-- 列出 main 下的表（你的原始数据）
SHOW TABLES FROM main;

-- 列出 jaffle_shop 下的表（你的模型）
SHOW TABLES FROM jaffle_shop;
```

你会看到类似：

```
main:           raw_orders, raw_customers, raw_payments
jaffle_shop:    stg_orders, stg_customers, stg_payments, ...
```

---

## 3. 想让它统一？有两种做法

### 方案 A：让 seeds 也进 `jaffle_shop`（或子 schema）

编辑 `dbt_project.yml`，加上 seeds 的 schema 配置：

```yaml
seeds:
  jaffle_shop:
    +schema: jaffle_shop    # 或者 +schema: raw_jaffle
```

然后重新 seed：

```bash
dbt seed --full-refresh
```

这样 `raw_customers` 就会落在 `jaffle_shop` 或 `raw_jaffle` schema 下。

### 方案 B（推荐）：在 `sources.yml` 里显式声明 schema

这是**更专业的做法**——明确告诉 dbt 你的原始数据在哪个 schema：

```yaml
# models/staging/_sources.yml
version: 2

sources:
  - name: raw_jaffle
    schema: main              # ← 显式指定！
    description: "Raw e-commerce data"
    tables:
      - name: orders
      - name: customers
      - name: payments
```

这样即使默认 schema 变了，source 引用也不会断。

---

## 4. 最佳实践建议

在实际项目中，**raw 数据和 transformed 数据通常放在不同 schema**，这是有意为之的隔离：

```
raw_jaffle    → 原始数据（seeds / Fivetran / Airbyte 写入）
staging       → stg_ 模型
intermediate  → int_ 模型
marts         → fct_ / dim_ 模型
```

对应配置：

```yaml
# dbt_project.yml
models:
  jaffle_shop:
    staging:
      +schema: staging
    intermediate:
      +schema: intermediate
    marts:
      +schema: marts

seeds:
  jaffle_shop:
    +schema: raw_jaffle
```

这样血缘清晰，权限也好管（BI 工具只给 `marts` 的读权限）。

---

## 5. 回到你的课程进度

**目前不需要改任何东西**——你的 pipeline 能跑通就是因为 `source()` 和 `dbt seed` 都默认指向 `main`，刚好对上了。

但既然你注意到了这一点，我建议你在 `sources.yml` 里**加上 `schema: main`**（或你实际看到的 schema 名），养成显式声明的习惯。

---

**👉 要继续 Module 3（Intermediate + Marts + `ref()`）吗？** 还是想先把 schema 配置调整好再往下走？告诉我你的选择。

---

# you asked

message time: 2026-10-04 12:54:20

analyse this log and fix

"""
23:52:47  [ERROR]: in model stg_orders (models\staging\stg_orders.sql)
23:52:47    Runtime Error in model stg_orders (models\staging\stg_orders.sql)
  Catalog Error: Table with name orders does not exist!
  Did you mean "raw_orders"?
  
  LINE 11: from "jaffle_shop"."main"."orders"
"""

---

# yuanbao response

这个错误**非常典型**，而且正好把我们上一轮讨论的 schema 问题和一个新坑串起来了。

---

## 🔍 根因分析

错误信息的关键行：

```
Catalog Error: Table with name orders does not exist!
Did you mean "raw_orders"?

LINE 11: from "jaffle_shop"."main"."orders"
```

**发生了什么：**

| 你以为 dbt 在查 | 实际数据库里的表名 |
|---|---|
| `orders` | `raw_orders` |
| `customers` | `raw_customers` |
| `payments` | `raw_payments` |

**原因链：**

1. 你在 `seeds/` 下放了 `raw_orders.csv`、`raw_customers.csv`、`raw_payments.csv`
2. `dbt seed` 加载时，**直接用文件名（去掉 `.csv`）作为表名** → 所以数据库里是 `raw_orders`，不是 `orders`
3. 你的 `sources.yml` 里声明的是：

```yaml
tables:
  - name: orders        # ← dbt 去找名为 "orders" 的表
  - name: customers
  - name: payments
```

4. dbt 拼接出的查询是 `from "jaffle_shop"."main"."orders"` → **表不存在！**

---

## ✅ 修复方案（二选一）

### 方案 A：改 `sources.yml` 匹配实际表名（推荐，最简单）

编辑 `models/staging/_sources.yml`，把 `name` 改成和 seed 生成的表名一致：

```yaml
version: 2

sources:
  - name: raw_jaffle
    schema: main          # 显式声明 schema（上一轮讨论过的）
    description: "Raw e-commerce data loaded from CSV seeds"
    tables:
      - name: raw_orders      # ← 改成 raw_orders
        description: "Raw orders table"
      - name: raw_customers   # ← 改成 raw_customers
        description: "Raw customers table"
      - name: raw_payments    # ← 改成 raw_payments
        description: "Raw payment transactions"
```

然后你的 SQL 模型里也要同步更新引用名（逻辑名变了）：

```sql
-- models/staging/stg_orders.sql
select ...
from {{ source('raw_jaffle', 'raw_orders') }}   -- ← 改成 raw_orders
```

```sql
-- stg_customers.sql → source('raw_jaffle', 'raw_customers')
-- stg_payments.sql  → source('raw_jaffle', 'raw_payments')
```

### 方案 B：用 `identifier` 映射（不改逻辑名）

如果你想保持代码里写 `source('raw_jaffle', 'orders')` 不动，可以在 `sources.yml` 里加 `identifier`：

```yaml
sources:
  - name: raw_jaffle
    schema: main
    tables:
      - name: orders
        identifier: raw_orders    # ← 告诉 dbt 物理表名叫 raw_orders
      - name: customers
        identifier: raw_customers
      - name: payments
        identifier: raw_payments
```

这样 SQL 里 **不用改**，dbt 会自动映射到正确的物理表。

---

## 🧹 修复后重新运行

```bash
# 先确认 seed 表确实在
dbt seed --full-refresh

# 重新编译检查
dbt compile --select stg_orders

# 运行
dbt run --select stg_orders
```

如果看到 `1 model successfully built`，就说明修好了。

---

## 💡 教训总结

| 坑 | 怎么避免 |
|---|---|
| seed 文件名 ≠ source 逻辑名 | 要么文件名直接叫 `orders.csv`，要么在 `sources.yml` 里用 `identifier` 映射 |
| 没指定 `schema` | 养成习惯：在 `sources.yml` 里**永远显式写 `schema:`** |
| 不知道表实际叫什么 | 用 `duckdb jaffle_shop.duckdb` → `SHOW TABLES FROM main;` 确认 |

---

**👉 修复完告诉我，我们继续 Module 3（Intermediate + Marts + `ref()`）！** 还是你想先把 schema 分层配置（raw / staging / marts）一起整理好再往下走？

---

# you asked

message time: 2026-10-04 13:21:58

this is my yaml set up, why staging schema is not `staging` but `main_staging`

"""

models:
  jaffle_shop:
    staging:
      +schema: staging
    intermediate:
      +schema: intermediate
    marts:
      +schema: marts
    # Config indicated by + and applies to all files under models/example/
    example:
      +materialized: view

"""

---

# yuanbao response

这个现象**完全符合 dbt 的默认命名规则**，不是你配错了，而是 dbt 在"帮你拼接"。

---

## 🔍 为什么会变成 `main_staging`？

dbt 有一个内置宏叫 `generate_schema_name`，它的**默认逻辑**是：

> **最终 schema = `target.schema` + `_` + 你在 `+schema` 里写的值**

你的 `profiles.yml` 里 target schema 是 `main`（DuckDB 的默认 schema），所以：

| 你的配置 | 实际生成的 schema |
|---------|-----------------|
| `+schema: staging` | `main_staging` |
| `+schema: marts` | `main_marts` |
| `+schema: intermediate` | `main_intermediate` |

你可以在 `target/compiled/` 里随便打开一个编译后的 SQL 文件，会看到类似：

```sql
create view "main_staging"."stg_orders" as ...
```

这就是证据。

---

## ✅ 三种修复方案

### 方案 1：自定义 `generate_schema_name` 宏（推荐 ✅）

这是**最干净、最专业**的做法。让 dbt 不再自动拼接前缀，直接用你写的值。

创建文件 `macros/generate_schema_name.sql`：

```sql
{% macro generate_schema_name(custom_schema_name, node) %}
    {% if custom_schema_name is none %}
        {{ target.schema }}
    {% else %}
        {{ custom_schema_name }}
    {% endif %}
{% endmacro %}
```

这样配置后：

| 你的配置 | 实际 schema |
|---------|------------|
| `+schema: staging` | `staging` ✅ |
| `+schema: marts` | `marts` ✅ |

> **原理**：覆盖了 dbt-core 的默认拼接行为，直接返回你指定的 schema 名。

---

### 方案 2：把 `profiles.yml` 的 target schema 改成项目名

编辑 `~/.dbt/profiles.yml`：

```yaml
jaffle_shop:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: 'jaffle_shop.duckdb'
      schema: jaffle_shop    # ← 改这里
      threads: 4
```

这样拼接出来是 `jaffle_shop_staging`——虽然不是纯 `staging`，但至少语义清晰。

然后你需要 **`dbt clean` + `dbt seed --full-refresh` + `dbt run`** 重新构建。

---

### 方案 3：不用 `+schema`，改用 `schema` 关键字（不推荐）

`+schema` 是给目录层级用的，有些版本的 dbt 行为不一致。不如方案 1 稳定。

---

## 🎯 我建议你这么做

**直接用方案 1**，因为：

1. 这是 dbt 社区的**标准做法**（dbt 官方文档也推荐自定义这个宏）
2. 你得到的 schema 名就是 `staging`、`marts`、`intermediate`——干净利落
3. 以后换到 Snowflake/BigQuery 也不会有意外拼接

创建完宏之后，验证一下：

```bash
dbt clean                    # 清掉旧的构建产物
dbt seed --full-refresh      # 重新加载 seeds
dbt run                      # 重新构建所有模型
```

然后进 DuckDB 确认：

```bash
duckdb jaffle_shop.duckdb
SHOW SCHEMAS;
```

你应该看到：

```
staging
intermediate
marts
main          (seeds 还在 main 里，除非你也配了 seeds 的 schema)
```

---

## 💡 顺带一提：seeds 的 schema 也需要单独配

如果你想让 seeds 也进一个独立 schema（比如 `raw`），在 `dbt_project.yml` 加：

```yaml
seeds:
  jaffle_shop:
    +schema: raw
```

然后 `dbt seed --full-refresh`，原始数据就进 `raw` schema 了，和 `staging` 彻底隔离。

---

**👉 修复完之后告诉我，我们继续 Module 3！** 还是你想先把整个 schema 分层（raw / staging / intermediate / marts）全部整理到位再往下走？

---

# you asked

message time: 2026-10-04 13:28:20

继续 Module 3

---

# yuanbao response

好，进入 **Module 3**！

---

# ⏱️ 11:00 AM — Module 3: Intermediate 与 Marts 层

恭喜你走到这里。Staging 层已经把原始数据"洗干净"了，现在我们来做**真正的业务建模**。

---

## 📖 3.1 为什么要继续分层？

如果只在 Staging 层就停住，你的 BI 工具会直接查 `stg_orders` + `stg_payments` + `stg_customers`——这意味着：

- 每个 Dashboard 都要重复写 JOIN 和聚合逻辑
- 改一个业务规则要改 N 个地方
- 没人知道"哪个指标是对的"

**解法：继续往上走。**

```
Staging (干净但仍是事务视角)
    ↓
Intermediate (业务拼接，复用中间层)
    ↓
Marts (面向消费的维度模型)
```

---

## 📖 3.2 Grain（粒度）——最重要的概念

**每个模型必须声明"一行代表什么"。**

| 模型 | Grain（粒度） |
|------|-------------|
| `stg_orders` | 一行 = 一笔订单的原始记录 |
| `int_order_payments` | 一行 = 一笔订单的一次支付 |
| `fct_orders` | 一行 = 一笔订单（汇总后） |
| `dim_customers` | 一行 = 一个客户 |

**如果粒度模糊，你的聚合就会重复或丢失。** 这是新手最容易踩的坑。

---

## 📖 3.3 三层职责速查

### Intermediate (`int_`)
- **多表 JOIN** 的地方
- 业务规则转换（状态映射、金额计算）
- 可以被多个 Marts 复用
- 不直接被 BI 工具查询

### Marts — Dimensions (`dim_`)
- 描述性实体：客户、产品、门店
- 一行一个实体
- 包含 SCD（缓慢变化维）逻辑

### Marts — Facts (`fct_`)
- 业务事件：订单、点击、支付
- 包含外键指向 Dimension
- 包含可度量的数值（金额、数量）
- **永远声明粒度**

---

## 💻 3.4 动手：创建 `dim_customers`

创建文件 `models/marts/dim_customers.sql`：

```sql
{{ config(materialized='table') }}

with customers as (
    select * from {{ ref('stg_customers') }}
),

orders as (
    select * from {{ ref('stg_orders') }}
),

-- 计算每个客户的生命周期指标
customer_orders as (
    select
        customer_id,
        count(order_id) as total_orders,
        min(order_date) as first_order_date,
        max(order_date) as most_recent_order_date,
        sum(
            case when order_status = 'completed' then 1 else 0 end
        ) as completed_orders_count
    from orders
    group by customer_id
)

select
    c.customer_id,
    c.first_name,
    c.last_name,
    c.email,
    c.first_name || ' ' || c.last_name as full_name,
    coalesce(co.total_orders, 0) as total_orders,
    co.first_order_date,
    co.most_recent_order_date,
    coalesce(co.completed_orders_count, 0) as completed_orders_count,
    case
        when co.total_orders is null then 'new'
        when co.total_orders = 1 then 'one_time'
        when co.total_orders <= 3 then 'repeat'
        else 'loyal'
    end as customer_tier
from customers c
left join customer_orders co on c.customer_id = co.customer_id
```

**注意 `{{ ref('stg_customers') }}`**——这是 dbt 的魔法：dbt 看到 `ref()`，就知道 `dim_customers` 依赖 `stg_customers`，自动排好执行顺序。你**永远不需要手写表名**。

---

## 💻 3.5 动手：创建 Intermediate 模型

创建文件 `models/intermediate/int_order_payments.sql`：

```sql
{{ config(materialized='view') }}

-- 将订单和支付拼接，供上游 fct_orders 复用
with orders as (
    select * from {{ ref('stg_orders') }}
),

payments as (
    select * from {{ ref('stg_payments') }}
),

order_payments as (
    select
        order_id,
        payment_method,
        amount_usd,
        payment_created_at
    from payments
),

-- 每个订单的支付汇总
order_payment_summary as (
    select
        order_id,
        count(*) as payment_count,
        sum(amount_usd) as total_payment_amount,
        array_agg(payment_method) as payment_methods_used
    from order_payments
    group by order_id
)

select
    o.order_id,
    o.customer_id,
    o.order_date,
    o.order_status,
    ops.payment_count,
    ops.total_payment_amount,
    ops.payment_methods_used
from orders o
left join order_payment_summary ops on o.order_id = ops.order_id
```

> **讲师提示**：这里用 `array_agg` 聚合支付方式（DuckDB 语法）。如果你用的是 Postgres 也是同样语法。Snowflake 用 `array_agg()`，BigQuery 用 `ARRAY_AGG()`。

---

## 💻 3.6 动手：创建 `fct_orders`

创建文件 `models/marts/fct_orders.sql`：

```sql
{{ config(materialized='table') }}

-- Grain: one row per order
-- Dependencies: int_order_payments

with order_payments as (
    select * from {{ ref('int_order_payments') }}
)

select
    order_id,
    customer_id,
    order_date,
    order_status,
    coalesce(total_payment_amount, 0) as total_revenue_usd,
    payment_count,
    payment_methods_used,
    case
        when total_payment_amount is null then true
        else false
    end as is_unpaid
from order_payments
```

**关键设计决策**：
- `fct_orders` **不直接 JOIN `stg_payments`**，而是从 `int_order_payments` 读取——这就是 Intermediate 层的价值：复用。
- `is_unpaid` 标记让 BI 层可以直接过滤未支付订单，不用每次重算。

---

## 💻 3.7 创建 Marts 层的属性文件

创建 `models/marts/properties.yml`：

```yaml
version: 2

models:
  - name: dim_customers
    description: "One row per customer. Enriched with lifetime order metrics."
    columns:
      - name: customer_id
        data_tests: [unique, not_null]
      - name: email
        data_tests: [not_null]
      - name: customer_tier
        data_tests:
          - accepted_values:
              values: ['new', 'one_time', 'repeat', 'loyal']

  - name: fct_orders
    description: "One row per order. Grain: order_id. Includes payment rollup."
    columns:
      - name: order_id
        data_tests: [unique, not_null]
      - name: customer_id
        data_tests:
          - not_null
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: order_status
        data_tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
              severity: warn
      - name: total_revenue_usd
        data_tests:
          - not_null
```

---

## 🎯 3.8 本节练习

### 练习 1：运行并验证 DAG

```bash
dbt run
dbt test
```

你应该看到所有模型成功构建，所有测试通过。如果有失败，仔细阅读错误信息——大概率是上游数据问题（比如某个 order 的 customer_id 在 customers 表里不存在）。

### 练习 2：查看血缘图

```bash
dbt docs generate
dbt docs serve
```

打开浏览器，你会看到一张清晰的 DAG 图：

```
raw_orders ──→ stg_orders ──┐
                             ├──→ int_order_payments ──→ fct_orders
raw_payments → stg_payments ─┘
                                                        
raw_customers → stg_customers ──→ dim_customers ←── (FK from fct_orders)
```

### 练习 3（挑战）：新增一个 `fct_payments` 模型

要求：
- Grain: 一行 = 一笔成功的支付
- 包含 `order_id`, `payment_method`, `amount_usd`, `payment_created_at`
- 添加 FK 测试指向 `fct_orders`
- 使用 `ref()` 引用上游 staging 模型

---

## 📌 关键收获

| 概念 | 记住这个 |
|------|---------|
| `ref()` | dbt 的命脉——自动解析依赖，永远别硬编码表名 |
| Grain | 每个模型开头注释写清楚"一行代表什么" |
| Intermediate | 复杂 JOIN 和聚合放这里，让 Marts 保持简洁 |
| `dbt run` | 按 DAG 拓扑顺序自动执行，你不用管先后 |
| `dbt test` | 数据质量的守门员——PK/FK/值域全靠它 |

---

## 🔧 排错锦囊

| 问题 | 解决 |
|------|------|
| `ref()` 找不到模型 | 检查模型文件名是否和 `ref('文件名去掉.sql')` 一致 |
| FK 测试失败 | 说明有孤儿记录（order 指向不存在的 customer），去上游查数据 |
| `array_agg` 报错 | DuckDB 需要 `group by` 配合；如果不需要聚合，删掉这列 |
| schema 还是 `main_staging` | 确认 `macros/generate_schema_name.sql` 已创建并 `dbt clean` |

---

**👉 准备好了吗？** 当你完成练习（或觉得概念已掌握），告诉我 **"进入 Module 4"**，我们将学习数据质量测试的深度用法、文档生成和物化策略优化！

是否需要我帮你 review `fct_payments` 的写法，或者解释某个概念？

---

# you asked

message time: 2026-10-04 14:19:53

进入 Module 4

---

# yuanbao response

# ⏱️ 13:00 — Module 4: 数据质量与测试（Data Quality & Testing）

欢迎回来！下午场的第一节课，也是 dbt 最让你"睡得着觉"的模块。

---

## 📖 4.1 为什么测试是 dbt 的灵魂？

没有测试的 dbt 项目只是一堆 SQL 文件。有了测试，dbt 才变成**数据可靠性引擎**。

> **核心理念**：数据问题应该在 Pipeline 运行时就被捕获，而不是等 BI 报表出错后被业务方发现。

dbt 提供三层测试体系：

| 层级 | 类型 | 写法 | 适用场景 |
|------|------|------|---------|
| L1 | Generic (Schema) Tests | YAML 声明 | PK/FK/值域/唯一性 |
| L2 | Singular Tests | 独立 `.sql` 文件 | 复杂业务规则、坏数据行返回 |
| L3 | Unit Tests | YAML + mock 数据 | 模型逻辑验证，不依赖真实数据 |

---

## 📖 4.2 Generic Tests（泛型测试）——你的第一道防线

### 四种内置测试

```yaml
# models/marts/properties.yml (我们已经写过一部分，现在补全)
version: 2

models:
  - name: fct_orders
    description: "One row per order."
    columns:
      - name: order_id
        data_tests:
          - unique
          - not_null
      - name: customer_id
        data_tests:
          - not_null
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: order_status
        data_tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
              severity: warn   # ← 警告但不阻断 pipeline
      - name: total_revenue_usd
        data_tests:
          - not_null
          - dbt_utils.at_least_one:   # 需要 dbt_utils 包
              config:
                severity: error
```

### 关键概念：`severity`

- `error`（默认）：测试失败 → `dbt build` 退出码非 0 → CI 失败
- `warn`：测试失败 → 打印警告 → Pipeline 继续执行

**什么时候用 `warn`？** 数据质量问题已知但不致命，或者上游脏数据你暂时无法修复。

### 添加 `dbt_utils` 包（可选但推荐）

创建 `packages.yml` 在项目根目录：

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1
```

然后运行：

```bash
dbt deps
```

这样你就能用 `dbt_utils.unique_combination_of_columns`、`dbt_utils.at_least_one` 等实用测试。

---

## 📖 4.3 Singular Tests（单测）——写 SQL 抓坏数据

当 YAML 测试不够表达你的业务规则时，写 Singular Test。

### 创建自定义测试

创建文件 `tests/assert_orders_have_positive_revenue.sql`：

```sql
-- 返回所有 total_revenue_usd <= 0 的订单
-- 如果有任何行返回，测试失败

select order_id, total_revenue_usd
from {{ ref('fct_orders') }}
where total_revenue_usd <= 0
  and is_unpaid = false   -- 排除未支付订单
```

创建文件 `tests/assert_no_duplicate_customers_by_email.sql`：

```sql
-- 检查是否有重复邮箱（同一邮箱注册多次）

select email, count(*) as cnt
from {{ ref('dim_customers') }}
group by email
having count(*) > 1
```

**命名约定**：`tests/` 目录下所有 `.sql` 文件都会被 dbt 当作测试执行。文件名以 `assert_` 开头是社区惯例。

运行：

```bash
dbt test --select test_assert_orders_have_positive_revenue
```

---

## 📖 4.4 Unit Tests（单元测试）——dbt 1.8+ 新特性

Unit Test 让你用 **mock 数据** 验证模型逻辑，完全不需要真实数据仓库。

### 为 `int_order_payments` 写单元测试

创建文件 `models/intermediate/unit_tests.yml`（或放在 `tests/unit/` 下）：

```yaml
unit_tests:
  - name: test_order_with_multiple_payments
    model: int_order_payments
    given:
      - input: ref('stg_orders')
        rows:
          - {order_id: 1, customer_id: 101, order_date: '2024-01-01', order_status: 'completed'}
          - {order_id: 2, customer_id: 102, order_date: '2024-01-02', order_status: 'placed'}
      - input: ref('stg_payments')
        rows:
          - {payment_id: 1, order_id: 1, payment_method: 'credit_card', amount_usd: 50.0}
          - {payment_id: 2, order_id: 1, payment_method: 'coupon', amount_usd: 10.0}
          - {payment_id: 3, order_id: 2, payment_method: 'debit_card', amount_usd: 30.0}
    expect:
      rows:
        - {order_id: 1, customer_id: 101, payment_count: 2, total_payment_amount: 60.0}
        - {order_id: 2, customer_id: 102, payment_count: 1, total_payment_amount: 30.0}
```

运行：

```bash
dbt test --select unit_test:test_order_with_multiple_payments
```

如果模型逻辑变了导致输出不匹配，测试立刻失败——**这就是数据工程的 TDD**。

---

## 📖 4.5 Source Testing & Freshness

除了模型测试，你还可以直接测试 **source** 数据质量。

编辑 `models/staging/_sources.yml`：

```yaml
version: 2

sources:
  - name: raw_jaffle
    schema: main
    freshness:
      warn_after: {count: 6, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: raw_orders
        data_tests:
          - unique:
              column_name: id
          - not_null:
              column_name: id
      - name: raw_customers
        data_tests:
          - unique:
              column_name: id
          - not_null:
              column_name: email
      - name: raw_payments
        data_tests:
          - not_null:
              column_name: order_id
```

运行：

```bash
dbt source freshness        # 检查数据新鲜度
dbt test --select source:*  # 运行所有 source 测试
```

---

## 📖 4.6 `dbt build` —— 一站式执行

之前我们分开跑 `dbt run` + `dbt test`，现在用 **`dbt build`** 一条命令搞定：

```bash
dbt build
```

`dbt build` 的行为：
1. 按 DAG 顺序执行模型
2. 每个模型构建完成后**立即运行其测试**
3. 如果某个模型的测试失败，**下游依赖自动跳过**（不会浪费计算）
4. 最后运行 standalone tests（singular + source tests）

这是生产环境的推荐命令。

---

## 💻 4.7 动手实验

### 实验 1：故意制造一个测试失败

编辑 `seeds/raw_orders.csv`，加一行：

```csv
6,999,2024-01-20,completed,2024-01-20 10:00:00
```

（customer_id = 999，但 `raw_customers.csv` 里没有这个 ID）

然后：

```bash
dbt seed --full-refresh
dbt build
```

你应该看到 FK 测试 `relationships` 失败，指出 `fct_orders` 里有一个 `customer_id = 999` 找不到对应客户。

**修复方法**：删除那行脏数据，重新 `dbt seed --full-refresh && dbt build`。

### 实验 2：添加 `severity: warn` 测试

在 `fct_orders` 的 properties.yml 里加：

```yaml
      - name: total_revenue_usd
        data_tests:
          - not_null
          - accepted_values:
              values: [0, 10.0, 20.0, 30.0, 40.0, 50.0, 60.0]  # 故意设一个窄范围
              severity: warn
```

运行 `dbt build`，你会看到警告但不阻断。

### 实验 3：写你的第一个 Singular Test

创建 `tests/assert_no_future_orders.sql`：

```sql
select order_id, order_date
from {{ ref('stg_orders') }}
where order_date > current_date
```

运行 `dbt test --select test_assert_no_future_orders`。

---

## 🎯 4.8 本节练习

1. **为 `dim_customers` 添加测试**：确保 `email` 列格式合法（提示：用 `dbt_utils.regex_match` 或写 singular test）
2. **为 `fct_orders` 添加 `dbt_utils.unique_combination_of_columns`** 测试，确保 `(order_id, order_date)` 组合唯一
3. **运行 `dbt build --select +fct_orders+`**：只构建 fct_orders 及其上下游，验证你的测试配置正确
4. **（挑战）写一个 singular test**：检查是否有客户在 `first_order_date` 之前就有支付记录

---

## 📌 关键收获

| 概念 | 记住这个 |
|------|---------|
| Generic Tests | YAML 声明，适合 PK/FK/值域，零 SQL 编写 |
| Singular Tests | 写 SQL 返回坏数据行，适合复杂业务规则 |
| Unit Tests | Mock 数据验证模型逻辑，CI 最快，不依赖真实数据 |
| `severity: warn` | 已知问题不阻断 pipeline，但仍需追踪 |
| `dbt build` | 生产环境唯一正确的执行方式 |
| Source Tests | 在数据入口就拦截脏数据 |

---

## 🔧 排错锦囊

| 问题 | 解决 |
|------|------|
| 测试失败但不知道哪行数据有问题 | 写 singular test 返回具体坏行 |
| `relationships` 测试报 FK 违规 | 去上游查孤儿记录：`select * from fct_orders where customer_id not in (select customer_id from dim_customers)` |
| `dbt test` 太慢 | 用 `--select` 只测改动的模型；或加 `store_failures: true` 配置把失败行存到表里方便分析 |
| `dbt_utils` 找不到 | 确认 `packages.yml` 存在且运行了 `dbt deps` |

---

**👉 准备好了吗？** 当你完成练习，告诉我 **"进入 Module 5"**，我们将学习文档生成、物化策略优化（view/table/incremental），以及性能调优技巧！

是否需要我帮你写一个具体的 singular test 或者解释某个测试失败的含义？

---

# you asked

message time: 2026-10-04 15:02:56

进入 Module 5

---

# yuanbao response

# ⏱️ 14:00 — Module 5: 文档、血缘与物化策略

下午好！现在进入今天课程中**最直接影响性能和可维护性**的模块。你的 Pipeline 已经能跑、有测试，但如果不懂物化策略，查询会慢如牛；如果不写文档，三个月后你自己都看不懂。

---

## 📖 5.1 物化策略（Materializations）——dbt 怎么把 SQL 变成仓库里的对象？

每次你运行 `dbt run`，dbt 需要决定：**把你的 SELECT 查询结果存成什么？**

dbt 提供四种内置策略：

| 策略 | 仓库里是什么 | 构建速度 | 查询速度 | 适用场景 |
|------|------------|---------|---------|---------|
| `view` | 视图（不存数据） | ⚡ 极快 | 🐢 每次重新计算 | Staging 层、轻量中间层 |
| `table` | 物理表（全量重建） | 🐢 慢（每次全删全插） | ⚡ 极快 | Marts 层、BI 直接查询的表 |
| `incremental` | 物理表（只插新增） | 🚀 只处理新数据 | ⚡ 极快 | 大事实表（千万行以上） |
| `ephemeral` | 不落地，编译为 CTE | ⚡ 极快 | 🐢 调用方变慢 | 一次性中间逻辑，不想在仓库里留痕迹 |

### 决策树

```
这个模型会被 BI 工具直接查吗？
  ├─ 是 → table（marts 层）
  └─ 否 → 它会被多个下游模型引用吗？
        ├─ 是 → view（staging 层）或 table（频繁引用的 intermediate）
        └─ 否 → ephemeral（单次使用的小 CTE）

数据量 > 100万行 且 只追加不改历史？
  → incremental
```

---

## 💻 5.2 动手：给各层配置物化策略

### 方式一：在 `dbt_project.yml` 中按目录统一配置（推荐）

编辑你的 `dbt_project.yml`，在 `models:` 块中加入：

```yaml
models:
  jaffle_shop:
    staging:
      +schema: staging
      +materialized: view          # ← staging 全部用 view
    intermediate:
      +schema: intermediate
      +materialized: view          # ← intermediate 默认 view，大表可单独改 table
    marts:
      +schema: marts
      +materialized: table         # ← marts 全部用 table
    example:
      +materialized: view
```

### 方式二：在单个模型文件顶部用 `{{ config() }}` 覆盖

```sql
-- models/marts/fct_orders.sql
{{ config(materialized='table') }}   -- ← 这个模型用 table，覆盖 dbt_project.yml 的默认值

select ...
```

**验证配置生效：**

```bash
dbt run --select stg_orders
# 编译后看 target/compiled/... 应该是 CREATE VIEW

dbt run --select fct_orders
# 编译后看 target/compiled/... 应该是 CREATE TABLE
```

---

## 📖 5.3 Incremental 模型——大数据的救命稻草

### 核心思想

全量 `table` 每次重建都要扫描所有历史数据。Incremental 只处理**新增或修改的行**，追加到已有表中。

### 改造 `fct_orders` 为 Incremental

编辑 `models/marts/fct_orders.sql`：

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}

-- Grain: one row per order

with order_payments as (
    select * from {{ ref('int_order_payments') }}
)

select
    order_id,
    customer_id,
    order_date,
    order_status,
    coalesce(total_payment_amount, 0) as total_revenue_usd,
    payment_count,
    payment_methods_used,
    case
        when total_payment_amount is null then true
        else false
    end as is_unpaid
from order_payments

{% if is_incremental() %}
  -- 增量模式：只处理上次运行之后新增或修改的订单
  where order_date >= (
      select coalesce(max(order_date), '1900-01-01')
      from {{ this }}
  )
{% endif %}
```

### 关键组件解析

| 组件 | 作用 |
|------|------|
| `materialized='incremental'` | 告诉 dbt 用增量策略 |
| `unique_key='order_id'` | 如果某行已存在，用这个 key 做 upsert/merge（DuckDB 用 `delete+insert` 或 `merge`） |
| `{% if is_incremental() %}` | 首次全量运行时为 False；后续增量运行时为 True |
| `{{ this }}` | 指向**当前模型在仓库中的表**，用来查 `max(order_date)` 作为水位线 |
| 水位线（watermark） | 上次处理到的最新日期，只拉取之后的数据 |

### 运行方式

```bash
# 首次运行：全量构建
dbt run --select fct_orders

# 后续运行：增量追加（加 --full-refresh 可强制全量重建）
dbt run --select fct_orders
```

> **⚠️ DuckDB 注意事项**：`dbt-duckdb` 的 incremental 策略在较新版本中使用 `merge`（需要 DuckDB ≥ 0.8）。如果你的版本较旧，它会退化为 `delete+insert`。行为一致，只是性能略有差异。

---

## 📖 5.4 Ephemeral——隐形的 CTE

有时候你有一个小逻辑，不想在仓库里建表也不想建视图，只想让 dbt 把它"内联"到下游模型的 SQL 里。

创建 `models/intermediate/int_status_mapping.sql`：

```sql
{{ config(materialized='ephemeral') }}

-- 状态映射表，只被一个地方用到，不需要落地

select *
from (
    values
        ('placed', 'Order received'),
        ('shipped', 'Order dispatched'),
        ('completed', 'Order delivered'),
        ('returned', 'Order returned')
) as t(status_code, status_label)
```

在 `fct_orders` 里引用它：

```sql
with status_map as (
    select * from {{ ref('int_status_mapping') }}
)
-- ... join 使用
```

编译后你会发现：dbt 直接把 `int_status_mapping` 的 SQL 内联成了 CTE，**仓库里根本不会有这个表**。

---

## 📖 5.5 文档与血缘（Docs & Lineage）

### 为什么要写文档？

- 新同事入职：看文档比看 SQL 快 10 倍
- 数据治理：合规审计需要字段定义
- 血缘图：排查"改这个字段会影响哪些报表"

### 完善你的 YAML 描述

编辑 `models/marts/properties.yml`，补全 description：

```yaml
version: 2

models:
  - name: dim_customers
    description: "Customer dimension table. One row per customer, enriched with lifetime metrics like total_orders and customer_tier."
    columns:
      - name: customer_id
        description: "Primary key. Surrogate key from raw customers table."
        data_tests: [unique, not_null]
      - name: email
        description: "Customer email address, used for login and marketing."
        data_tests: [not_null]
      - name: customer_tier
        description: "Computed segmentation: new (0 orders), one_time (1), repeat (2-3), loyal (4+)."
        data_tests:
          - accepted_values:
              values: ['new', 'one_time', 'repeat', 'loyal']
      - name: total_orders
        description: "Total number of orders placed by this customer across their lifetime."

  - name: fct_orders
    description: "Order fact table. Grain: one row per order. Includes payment rollup from raw payments."
    columns:
      - name: order_id
        description: "Primary key. Unique identifier for each order."
        data_tests: [unique, not_null]
      - name: customer_id
        description: "Foreign key to dim_customers."
        data_tests:
          - not_null
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: total_revenue_usd
        description: "Sum of all successful payment amounts for this order, in USD."
        data_tests: [not_null]
      - name: is_unpaid
        description: "Flag: true if no successful payment exists for this order."
```

### 为 Sources 添加描述

编辑 `models/staging/_sources.yml`：

```yaml
version: 2

sources:
  - name: raw_jaffle
    schema: main
    description: "Raw e-commerce data ingested from CSV seeds. This is the entry point of our pipeline."
    freshness:
      warn_after: {count: 6, period: hour}
      error_after: {count: 24, period: hour}
    tables:
      - name: raw_orders
        description: "Raw order events. One row per order creation."
      - name: raw_customers
        description: "Raw customer signups. One row per customer."
      - name: raw_payments
        description: "Raw payment transactions. Multiple rows per order possible."
```

---

## 📖 5.6 Exposures——告诉 dbt 谁在消费你的数据

创建 `models/marts/exposures.yml`：

```yaml
version: 2

exposures:
  - name: revenue_dashboard
    type: dashboard
    owner:
      name: "Data Team"
      email: "data@jaffle-shop.com"
    description: "Executive dashboard showing daily revenue, customer tiers, and order status breakdown."
    depends_on:
      - ref('fct_orders')
      - ref('dim_customers')

  - name: customer_ltv_report
    type: report
    owner:
      name: "Marketing Team"
      email: "marketing@jaffle-shop.com"
    description: "Monthly customer lifetime value report used for campaign planning."
    depends_on:
      - ref('dim_customers')
```

这样 dbt 血缘图里会显示：**哪些 BI 工具/报表依赖你的模型**。

---

## 💻 5.7 动手：生成并查看文档站点

```bash
# 生成文档（编译 catalog + manifest + 你的 YAML 描述）
dbt docs generate

# 启动本地服务器
dbt docs serve
```

打开浏览器访问 `http://localhost:8080`：

- **Lineage** 标签页：看到完整的 DAG 血缘图
- **Model** 标签页：搜索 `fct_orders`，看到字段描述、测试、依赖关系
- **Source** 标签页：看到原始数据的新鲜度状态
- **Exposure** 标签页：看到下游消费方

**试试点击节点之间的连线**——能看到字段级别的血缘（如果 dbt 版本支持）。

---

## 🎯 5.8 本节练习

### 练习 1：调整物化策略
- 确认 `stg_*` 模型用 `view`
- 确认 `fct_*` 和 `dim_*` 用 `table`
- 运行 `dbt run` 并验证编译后的 DDL 语句

### 练习 2：Incremental 验证
- 往 `seeds/raw_orders.csv` 加一行**新的**订单（用未来的日期，比如 `2024-02-01`）
- `dbt seed --full-refresh`
- 运行 `dbt run --select fct_orders`（不带 `--full-refresh`）
- 检查表里是否只新增了那一行，旧行没被重建

### 练习 3：文档完善
- 为 `int_order_payments` 添加 `properties.yml`，至少写 3 个列的 description
- 重新 `dbt docs generate && dbt docs serve`，确认新描述出现在页面上

### 练习 4（挑战）：添加 Exposure
- 创建一个 `exposures.yml`，声明一个"假想的" Tableau 仪表板
- 让它依赖 `fct_orders` 和 `dim_customers`
- 在文档站点的 Lineage 图里找到它

---

## 📌 关键收获

| 概念 | 记住这个 |
|------|---------|
| `view` | 默认选择，构建快查询慢，适合 staging |
| `table` | 查询快构建慢，适合 marts 层 |
| `incremental` | 大数据事实表必选，用 `unique_key` + `is_incremental()` + 水位线 |
| `ephemeral` | 一次性 CTE，不落地，适合小逻辑 |
| `dbt docs generate` | 每次改完 YAML 描述后重新生成 |
| Exposures | 让血缘图延伸到 BI 层，数据治理利器 |

---

## 🔧 排错锦囊

| 问题 | 解决 |
|------|------|
| Incremental 模型首次运行报 `relation already exists` | 删掉表重新跑，或加 `--full-refresh` |
| `is_incremental()` 里的 SQL 报 `this` 不存在 | 首次全量运行时 `this` 表还没建，`coalesce(max(...), '1900-01-01')` 是安全写法 |
| 文档站点打不开 | 确认端口 8080 没被占用；或 `dbt docs serve --port 8081` |
| 改了 YAML 但文档没更新 | `dbt docs generate` 必须重新执行 |
| `unique_key` 冲突导致重复行 | 确认你的 `unique_key` 真的是唯一的；DuckDB 的 incremental 用 `delete+insert` 策略时会先删再插 |

---

**👉 准备好了吗？** 当你完成练习，告诉我 **"进入 Module 6"**，我们将学习 Seeds、Macros 和 Snapshots——dbt 的"工具箱"模块，让你从"写 SQL"进化到"写可复用数据产品"！

是否需要我帮你检查 incremental 模型的水位线逻辑，或者解释 ephemeral 和 view 的性能取舍？

---

# you asked

message time: 2026-10-04 15:14:06

进入 Module 6

---

# yuanbao response

# ⏱️ 15:15 — Module 6: Seeds、Macros 与 Snapshots

下午最后一段高强度内容！这三个功能代表 dbt 从"写 SQL"到"构建可复用数据产品"的进化。

---

## 📖 6.1 Seeds 进阶——不只是加载原始数据

在 Module 2 里我们用 `dbt seed` 加载了 `raw_orders.csv` 等原始数据。但在**生产环境**，原始数据通常由 Fivetran/Airbyte/自定义脚本写入，而不是靠 seed。

**Seeds 的真正定位**：版本化管理的**小型静态查找表（Lookup Tables）**。

### 典型用例

| 场景 | 示例 |
|------|------|
| 国家/货币代码映射 | `US → United States → USD` |
| 业务分类映射 | `order_status` 码值 → 中文标签 |
| 数据修正补丁 | 手动维护的异常订单 ID 列表 |
| 环境变量/参数表 | 财政年度起止日期、税率表 |

### 动手：创建一个国家代码 Seed

创建 `seeds/country_codes.csv`：

```csv
country_code,country_name,currency_code,region
US,United States,USD,North America
GB,United Kingdom,GBP,Europe
CN,China,CNY,Asia Pacific
JP,Japan,JPY,Asia Pacific
DE,Germany,EUR,Europe
AU,Australia,AUD,Asia Pacific
```

运行：

```bash
dbt seed --select country_codes
```

验证：

```sql
-- 在 DuckDB 里查
select * from main.country_codes;  -- 或你的 seeds schema
```

### 在模型中引用 Seed

```sql
-- 在 dim_customers 里 JOIN country 信息（假设你有 country_code 字段）
select
    c.*,
    cc.country_name,
    cc.region
from {{ ref('dim_customers') }} c
left join {{ ref('country_codes') }} cc
    on c.country_code = cc.country_code
```

> **注意**：`ref()` 不仅可以引用 models 目录下的 SQL，也能引用 seeds 目录下的 CSV——dbt 自动处理依赖。

### Seed 配置

在 `dbt_project.yml` 里：

```yaml
seeds:
  jaffle_shop:
    +schema: raw              # seeds 进 raw schema
    country_codes:
      +column_types:          # 指定列类型（某些仓库需要）
        country_code: varchar(2)
```

---

## 📖 6.2 Macros——dbt 的"函数库"

Macro 是 dbt 的 **Jinja 函数**，让你把重复逻辑抽象成可复用组件。类似 SQL 里的 UDF，但更强大——它可以生成**整段 SQL 代码**。

### 什么时候用 Macro？

| ✅ 适合 | ❌ 不适合 |
|---------|---------|
| 生成重复的 CAST/COALESCE 模式 | 简单的一次性计算（直接写 SQL） |
| 跨模型通用的日期过滤逻辑 | 复杂业务逻辑（写在 model 里更清晰） |
| 动态 SQL 生成（pivot、union all） | 让 SQL 变得难以调试的过度抽象 |
| 封装仓库特定语法差异 | 团队新人不熟悉的"黑魔法" |

### 动手：创建你的第一个 Macro

创建 `macros/prefix_columns.sql`：

```sql
{% macro prefix_columns(column_list, prefix) %}
    {% for col in column_list %}
        {{ col }} as {{ prefix }}_{{ col }}{% if not loop.last %},{% endif %}
    {% endfor %}
{% endmacro %}
```

在模型中使用：

```sql
select
    order_id,
    {{ prefix_columns(['status', 'date', 'total'], 'order') }}
from {{ ref('stg_orders') }}
```

编译后变成：

```sql
select
    order_id,
    status as order_status,
    date as order_date,
    total as order_total
from "staging"."stg_orders"
```

### 动手：更实用的 Macro——安全除法

创建 `macros/safe_divide.sql`：

```sql
{% macro safe_divide(numerator, denominator, default=0) %}
    case
        when {{ denominator }} = 0 or {{ denominator }} is null
        then {{ default }}
        else {{ numerator }} * 1.0 / {{ denominator }}
    end
{% endmacro %}
```

使用：

```sql
select
    order_id,
    {{ safe_divide('total_revenue_usd', 'payment_count', 0) }} as avg_payment_per_transaction
from {{ ref('fct_orders') }}
```

### 动手：Surrogate Key 生成（经典 Macro 用例）

创建 `macros/surrogate_key.sql`：

```sql
{% macro surrogate_key(field_list) %}
    md5(
        {% for field in field_list %}
            coalesce(cast({{ field }} as varchar), '') || '-' ||
        {% endfor %}
        ''
    )
{% endmacro %}
```

使用：

```sql
select
    {{ surrogate_key(['customer_id', 'order_date']) }} as order_sk,
    *
from {{ ref('fct_orders') }}
```

> **💡 提示**：`dbt_utils.surrogate_key()` 已经做了这个，生产环境直接用包就行。自己写一遍是为了理解原理。

### ⚠️ 讲师警告：Macro 反模式

```sql
-- ❌ 不要这样写：一个 macro 生成整个模型 SQL
{% macro build_all_facts() %}
    create table fct_orders as select ...;
    create table fct_payments as select ...;
{% endmacro %}

-- ✅ 正确做法：macro 只封装小段可复用逻辑，每个 model 还是独立 SQL 文件
```

**黄金法则**：如果 Macro 让你看不懂最终生成的 SQL，就拆开写。

---

## 📖 6.3 Snapshots——缓慢变化维（SCD Type 2）

### 问题场景

`dim_customers` 里客户的地址变了。你直接 `UPDATE` 了那行——**历史地址信息永远丢失了**。

BI 团队想问："这个客户下单时的地址是什么？" 你答不上来。

### SCD Type 2 解决方案

Snapshots 为每行维护**有效时间范围**，保留完整历史：

| customer_id | email | address | dbt_valid_from | dbt_valid_to | is_current |
|-------------|-------|---------|----------------|--------------|------------|
| 101 | alice@old.com | 旧地址 | 2024-01-01 | 2024-03-15 | false |
| 101 | alice@new.com | 新地址 | 2024-03-15 | null | true |

### 两种策略

| 策略 | 原理 | 要求 |
|------|------|------|
| `timestamp` | 比较 `updated_at` 字段，有变化就新增版本 | 源表有可靠的更新时间戳 |
| `check` | 比较指定列的值，任何列变了就新增版本 | 指定 `check_cols` |

### 动手：创建 Snapshot

创建 `snapshots/customers_snapshot.sql`：

```sql
{% snapshot customers_snapshot %}

    {{
        config(
            target_schema='snapshots',
            strategy='timestamp',
            updated_at='_loaded_at',
            unique_key='customer_id'
        )
    }}

    select * from {{ source('raw_jaffle', 'raw_customers') }}

{% endsnapshot %}
```

运行：

```bash
dbt snapshot
```

验证结果：

```bash
duckdb jaffle_shop.duckdb
select customer_id, email, _loaded_at, dbt_valid_from, dbt_valid_to, dbt_is_current
from snapshots.customers_snapshot;
```

### 在模型中引用 Snapshot

```sql
-- 获取某个时间点的客户状态
select *
from {{ ref('customers_snapshot') }}
where dbt_valid_from <= '2024-02-01'
  and ('2024-02-01' < dbt_valid_to or dbt_valid_to is null)
```

或者更简单——用 `dbt_utils.current_snapshot()` 获取当前版本：

```sql
select * from {{ ref('customers_snapshot') }}
where dbt_is_current = true
```

### Snapshot 配置

在 `dbt_project.yml` 中：

```yaml
snapshots:
  jaffle_shop:
    +schema: snapshots
    +target_database: "{{ target.database }}"  # 可选：指定不同数据库
```

---

## 🎯 6.4 本节练习

### 练习 1：创建 Lookup Seed

1. 创建 `seeds/order_status_labels.csv`：

```csv
status_code,status_label,status_category
placed,Order Received,Active
shipped,Order Dispatched,Active
completed,Order Delivered,Completed
returned,Order Returned,Completed
```

2. `dbt seed` 加载
3. 在 `fct_orders` 里 JOIN 这个 seed，把 `order_status` 展开为 `status_label` 和 `status_category`

### 练习 2：写一个 Macro

写一个 `macros/format_phone.sql`：

```sql
-- 输入: 15551234567
-- 输出: +1 (555) 123-4567
-- 提示: 用 substring 或 regexp_replace
```

在 `dim_customers` 里使用它格式化一个假设的 `phone` 字段。

### 练习 3：Snapshot 实战

1. 为 `raw_orders` 创建 snapshot，用 `check` 策略监控 `status` 列变化
2. 运行 `dbt snapshot`
3. 手动修改 `raw_orders.csv` 里一个订单的 status
4. 重新运行 `dbt snapshot`
5. 查询 snapshot 表，确认出现了两条记录（旧版本 + 新版本）

### 练习 4（挑战）：Macro + Snapshot 组合

写一个 macro `get_current_version()`，接受一个 snapshot 模型名作为参数，自动生成 `where dbt_is_current = true` 的 CTE。然后在 `fct_orders` 里用这个 macro 引用 `customers_snapshot`。

---

## 📌 关键收获

| 功能 | 记住这个 |
|------|---------|
| Seeds | 版本化管理的静态查找表，不是 ETL 入口 |
| Macros | Jinja 函数，封装重复逻辑，但别过度抽象 |
| `ref()` on seed | 和引用 model 一样，`ref('country_codes')` |
| Snapshots | SCD Type 2 历史追踪，`timestamp` 或 `check` 策略 |
| `dbt snapshot` | 独立于 `dbt run` 的命令，需要单独调度 |
| Snapshot 表结构 | `dbt_valid_from`, `dbt_valid_to`, `dbt_is_current` 是标准字段 |

---

## 🔧 排错锦囊

| 问题 | 解决 |
|------|------|
| `dbt seed` 报 column type mismatch | 在 `dbt_project.yml` 里用 `+column_types` 指定 |
| Macro 编译后 SQL 语法错误 | 用 `dbt compile` 看编译结果；在 macro 里加 `{% do log("debug: " ~ var, info=True) %}` 调试 |
| Snapshot 没生成新版本 | 确认 `updated_at` 字段值确实变了；或 `check_cols` 里指定的列确实被修改了 |
| Snapshot 表越来越大 | 这是设计如此——SCD2 就是保留全量历史。如果不需要历史，用 `table` materialization 直接覆盖 |
| `dbt snapshot` 找不到 snapshot 文件 | 确认文件在 `snapshots/` 目录下，且文件名以 `.sql` 结尾 |

---

## 📅 今日进度总结

| 模块 | 状态 |
|------|------|
| Module 1: 核心概念与设置 | ✅ |
| Module 2: Sources 与 Staging | ✅ |
| Module 3: Intermediate 与 Marts | ✅ |
| Module 4: 数据质量与测试 | ✅ |
| Module 5: 文档、血缘与物化策略 | ✅ |
| Module 6: Seeds、Macros 与 Snapshots | ✅ 当前 |
| Module 7: 选择执行与 CI/CD | ⏳ 下一步 |
| Module 8: Capstone 综合实战 | ⏳ 最后 |

---

**👉 准备好了吗？** 当你完成练习，告诉我 **"进入 Module 7"**，我们将学习 dbt 的"生产级调度"——如何选择执行特定模型、如何配置 CI/CD 流水线、如何实现 Slim CI（只测试变更的模型），让你的 Pipeline 真正进入生产环境！

是否需要我帮你调试某个 macro 的 Jinja 语法，或者解释 snapshot 的 `check` vs `timestamp` 策略的取舍？

---

# you asked

message time: 2026-10-04 15:15:10

进入 Module 7

---

# yuanbao response

# ⏱️ 16:15 — Module 7: 选择执行与 CI/CD

欢迎来到今天倒数第二节课！你的 Pipeline 已经功能完整、有测试、有文档、有物化策略。现在要解决的是**生产环境的核心问题**：怎么高效运行？怎么防止坏代码合入？

---

## 📖 7.1 Node Selection——别每次都全量跑

随着项目增长，模型数量从几十变成几百。每次改一行 SQL 就 `dbt run` 全量构建？**浪费时间，浪费计算资源。**

dbt 的选择器（selector）让你精确控制**运行什么、测试什么**。

### 基础选择语法

```bash
# 只运行指定模型
dbt run --select fct_orders

# 多个模型
dbt run --select fct_orders dim_customers

# 用 tag 选择
dbt run --select tag:nightly

# 用路径选择
dbt run --select path:models/marts/
```

### 血缘感知选择（最强大）

| 语法 | 含义 |
|------|------|
| `--select +fct_orders` | `fct_orders` **及其所有上游依赖** |
| `--select fct_orders+` | `fct_orders` **及其所有下游消费者** |
| `--select +fct_orders+` | `fct_orders` **及其上下游全部** |
| `--select @fct_orders` | 同上（`@` 是 `+...+` 的简写） |
| `--select 1+fct_orders` | `fct_orders` + **只往上追溯 1 层** |

### 动手实验

```bash
# 改了 stg_orders，只运行它和所有依赖它的下游
dbt build --select stg_orders+

# 想验证 dim_customers 及其上游是否完整
dbt build --select +dim_customers

# 只测试 marts 层
dbt test --select path:models/marts/

# 只测试今天改过的模型（git 配合）
dbt test --select "git diff --name-only HEAD~1"
```

---

## 📖 7.2 `state:` 选择器——Slim CI 的核心

### 问题

PR 里你只改了 `fct_orders.sql`。CI 应该：
- ❌ 不重新构建没改过的 `dim_customers`（浪费时间）
- ✅ 构建 `fct_orders` 和所有**下游**模型
- ✅ 测试所有受影响的模型
- ✅ 但**对比数据**时要参照生产环境的 manifest

### `state:modified` 原理

`state:` 选择器需要两个 **manifest.json** 文件：
1. **生产环境的 manifest**（代表"上次成功的状态"）
2. **当前 PR 的 manifest**（代表"改了什么"）

dbt 对比两者，找出差异节点。

### 工作流程

```bash
# CI 环境中：
# 1. 下载生产环境的 manifest.json（从 S3/Artifacts 等）
# 2. 生成当前代码的 manifest
dbt parse  # 生成 target/manifest.json

# 3. 只运行/测试变更的模型 + 下游
dbt build --select state:modified+ --defer --state prod-manifest/
```

### `--defer` 是什么？

`--defer` 告诉 dbt：**对于我没有构建的模型，引用生产环境的版本**。

- 你改了 `fct_orders`
- `dim_customers` 没改 → dbt 不会重建它
- 但 `fct_orders` 需要 JOIN `dim_customers` → dbt 自动指向**生产环境的 `dim_customers`**

这样 CI 既快又准。

---

## 📖 7.3 Git Flow + CI/CD 基础

### 标准分支策略

```
main (生产)
  ↑
  │  PR (CI 自动运行 dbt build)
  │
feature/add-revenue-metric
```

### 本地开发流程

```bash
# 1. 从 main 拉新分支
git checkout main
git pull
git checkout -b feature/add-customer-tier

# 2. 写代码、测试
dbt run --select +dim_customers
dbt test --select +dim_customers

# 3. 提交
git add .
git commit -m "feat: add customer_tier to dim_customers"
git push origin feature/add-customer-tier

# 4. 在 GitHub/GitLab 上创建 PR
# CI 自动触发
```

### CI 做什么？

1. **安装依赖**：`pip install dbt-core dbt-duckdb && dbt deps`
2. **编译检查**：`dbt parse`（快速失败，语法错误立即报）
3. **运行构建+测试**：`dbt build --select state:modified+ --defer --state prod-artifacts/`（有 manifest 时）或 `dbt build`（首次/无 manifest 时全量）
4. **生成文档**：`dbt docs generate`
5. **上传产物**：manifest.json、docs 站点

---

## 💻 7.4 动手：创建 GitHub Actions CI 工作流

创建 `.github/workflows/dbt-ci.yml`：

```yaml
name: dbt CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  dbt-build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dbt
        run: |
          pip install dbt-core dbt-duckdb
          dbt deps

      - name: Debug connection
        run: dbt debug

      - name: Seed raw data
        run: dbt seed

      - name: Build and test (full on main push, slim on PR)
        if: github.event_name == 'push'
        run: dbt build

      - name: Build and test (slim CI on PR)
        if: github.event_name == 'pull_request'
        run: |
          # 下载生产 manifest（如果存在）
          # curl -o prod-manifest.json https://your-storage/manifest.json || true
          # if [ -f prod-manifest.json ]; then
          #   dbt build --select state:modified+ --defer --state .
          # else
          dbt build
          # fi

      - name: Generate docs
        run: dbt docs generate

      - name: Upload docs artifact
        uses: actions/upload-artifact@v3
        with:
          name: dbt-docs
          path: target/
```

### 本地模拟 CI（不用 GitHub 也能练）

```bash
# 模拟 PR 环境
dbt clean
dbt seed
dbt build --select +fct_orders   # 假装只改了 fct_orders
```

---

## 📖 7.5 生产部署（CD）

### 什么时候部署到生产？

| 策略 | 做法 |
|------|------|
| **Merge to main → 立即部署** | PR 合并后触发 `dbt build` 到生产仓库 |
| **定时调度** | 每天凌晨 2 点运行（配合 Airflow / Prefect / dbt Cloud） |
| **事件触发** | 上游数据到达后触发 |

### 生产运行的关键配置

`profiles.yml` 中至少三个 target：

```yaml
jaffle_shop:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: 'jaffle_shop_dev.duckdb'
      schema: main
    ci:
      type: duckdb
      path: 'jaffle_shop_ci.duckdb'
      schema: main
    prod:
      type: duckdb
      path: 'jaffle_shop_prod.duckdb'
      schema: main
```

生产运行：

```bash
dbt build --target prod
```

> **⚠️ 黄金法则**：永远不要在本地直接跑 `--target prod`。通过 CI/CD 自动部署。

---

## 📖 7.6 dbt Cloud 简介（可选）

如果你不想自己管 CI/CD 基础设施：

| 功能 | dbt Cloud |
|------|-----------|
| 调度 | 内置 cron 调度 |
| CI | 自动为每个 PR 创建临时环境 |
| 文档托管 | 自动生成并托管 docs 站点 |
| IDE | 浏览器内编写、编译、运行 |
| 价格 | 个人免费（1 个免费开发者 seat） |

对于个人学习和小团队，dbt Cloud 的免费层已经够用。

---

## 🎯 7.7 本节练习

### 练习 1：选择器实战

```bash
# 1. 列出所有可用的模型
dbt list

# 2. 只运行 staging 层
dbt run --select path:models/staging/

# 3. 运行 fct_orders 及其所有上游
dbt run --select +fct_orders

# 4. 测试所有 marts 层模型
dbt test --select path:models/marts/

# 5. 用 tag 选择（先给 fct_orders 加一个 tag）
# 在 fct_orders.sql 顶部加：{{ config(tags=['revenue']) }}
dbt run --select tag:revenue
```

### 练习 2：模拟 Slim CI

```bash
# 1. 保存当前 manifest 作为"生产版本"
cp target/manifest.json prod_manifest.json

# 2. 修改 fct_orders.sql（加一个字段或改一行）

# 3. 重新解析
dbt parse

# 4. 用 state 选择器（需要 dbt 1.5+）
dbt build --select state:modified+ --defer --state . --state-manifest prod_manifest.json
```

### 练习 3：创建 `.github/workflows/dbt-ci.yml`

按照上面的模板创建文件，然后：

```bash
# 初始化 git 仓库（如果还没做）
git init
git add .
git commit -m "Initial dbt project"

# 创建 GitHub 仓库并推送
gh repo create jaffle-shop-dbt --public
git remote add origin https://github.com/YOUR_USER/jaffle-shop-dbt.git
git push -u origin main
```

然后在 GitHub 上创建一个 PR，观察 Actions 是否自动运行。

---

## 📌 关键收获

| 概念 | 记住这个 |
|------|---------|
| `--select` | 精确控制运行范围，别每次全量 |
| `+model+` | 血缘感知选择，上下游一网打尽 |
| `state:modified+` | Slim CI 核心，只跑变更的模型+下游 |
| `--defer` | CI 中引用生产环境的未变更模型 |
| `dbt parse` | 快速语法检查，CI 第一步 |
| `dbt build` | 生产环境唯一正确的执行命令 |
| 分支策略 | feature branch → PR → CI → merge → prod deploy |
| 黄金法则 | 永远不在本地跑 `--target prod` |

---

## 🔧 排错锦囊

| 问题 | 解决 |
|------|------|
| `--select` 找不到模型 | 用 `dbt list` 查看所有节点的全名（格式：`model.jaffle_shop.fct_orders`） |
| `state:` 选择器报 "no nodes selected" | 确认 manifest.json 路径正确；确认确实有模型被修改 |
| `--defer` 报找不到生产表 | 确认生产环境已构建过；确认 `--state` 指向正确的 manifest 目录 |
| CI 中 `dbt seed` 很慢 | seeds 通常只在首次或数据更新时运行，CI 中可以缓存或跳过 |
| GitHub Actions 报 `dbt: command not found` | 确认 `pip install` 步骤成功；可能需要 `export PATH=$PATH:~/.local/bin` |

---

## 📅 最终进度

| 模块 | 状态 |
|------|------|
| Module 1: 核心概念与设置 | ✅ |
| Module 2: Sources 与 Staging | ✅ |
| Module 3: Intermediate 与 Marts | ✅ |
| Module 4: 数据质量与测试 | ✅ |
| Module 5: 文档、血缘与物化策略 | ✅ |
| Module 6: Seeds、Macros 与 Snapshots | ✅ |
| Module 7: 选择执行与 CI/CD | ✅ 当前 |
| Module 8: Capstone 综合实战 | ⏳ 最后一节！ |

---

**👉 准备好了吗？** 这是今天的**终极挑战**——**Module 8: Capstone 综合实战**。我会给你一个完整的业务场景，要求你从零搭建一条端到端的 dbt Pipeline，把所有学过的东西串起来。

告诉我 **"进入 Module 8"**，我们开始终极实战！

是否需要我先帮你复习某个选择器语法，或者解释 `--defer --state` 的更多细节？

---

# you asked

message time: 2026-10-04 15:15:51

进入 Module 8

---

# yuanbao response

# ⏱️ 17:15 — Module 8: Capstone 综合实战 🏆

恭喜你走到最后！这是今天的终极挑战。接下来你要**独立**（或参考我给的骨架）从零搭建一条完整的 dbt Pipeline，把 Module 1–7 的所有知识串起来。

---

## 🎯 业务场景

**Jaffle Shop** 是一家快速增长的在线餐厅，业务方需要以下分析能力：

| 业务需求 | 数据支持 |
|---------|---------|
| 每日收入追踪 | 订单金额 + 支付状态 |
| 客户分层与留存 | 客户订单历史 + 生命周期价值 |
| 订单履约监控 | 订单状态流转时间 |
| 支付渠道分析 | 各支付方式的成功率与金额 |
| 供应链洞察 | 菜品销量与库存预警 |

**你的任务**：构建支撑这些需求的数据管道。

---

## 📋 验收标准（Checklist）

完成以下每一项后打勾：

### 基础设施
- [ ] `dbt init` 初始化项目，连接 DuckDB
- [ ] `macros/generate_schema_name.sql` 自定义 schema 命名（去掉 `main_` 前缀）
- [ ] `dbt_project.yml` 配置分层 schema + 物化策略
- [ ] `packages.yml` 引入 `dbt_utils`（可选）

### 数据加载
- [ ] 创建至少 4 个 seed CSV：`raw_orders`, `raw_customers`, `raw_payments`, `raw_menu_items`
- [ ] `dbt seed` 成功加载

### Sources
- [ ] `models/staging/_sources.yml` 声明所有 source，含 `schema` 和 `freshness`
- [ ] `dbt source freshness` 通过

### Staging 层
- [ ] 每个 source 对应一个 `stg_` 模型
- [ ] 只做重命名、类型转换、基础清洗
- [ ] 物化策略：`view`

### Intermediate 层
- [ ] 至少 1 个 `int_` 模型，做多表 JOIN 或复杂聚合
- [ ] 物化策略：`view` 或 `table`

### Marts 层
- [ ] `fct_orders.sql`：订单事实表，含收入、支付汇总
  - 物化：`incremental`，`unique_key='order_id'`
  - 水位线：`is_incremental()` + `max(order_date)`
- [ ] `dim_customers.sql`：客户维度表，含生命周期指标
  - 物化：`table`
- [ ] `fct_payments.sql`：支付事实表，粒度 = 每笔支付
  - 物化：`table`
- [ ] `dim_menu_items.sql`（可选挑战）：菜品维度

### 数据测试
- [ ] `properties.yml` 为每个 mart 模型配置：
  - PK: `unique` + `not_null`
  - FK: `relationships` 指向父表
  - 值域: `accepted_values` 至少一个字段
- [ ] 至少 1 个 singular test（写 SQL 抓坏数据）
- [ ] 至少 1 个 source test

### 文档与血缘
- [ ] 每个模型和关键列有 `description`
- [ ] 创建 `exposures.yml` 声明至少 1 个下游 Dashboard
- [ ] `dbt docs generate && dbt docs serve` 成功
- [ ] 血缘图中能看到完整 DAG

### 执行与 CI
- [ ] `dbt build` 全绿（所有模型构建 + 所有测试通过）
- [ ] 使用 `--select` 选择器只运行部分模型验证
- [ ] 创建 `.github/workflows/dbt-ci.yml`

---

## 📁 参考项目结构

```
jaffle_shop/
├── dbt_project.yml
├── packages.yml
├── profiles.yml (在 ~/.dbt/ 下)
├── macros/
│   ├── generate_schema_name.sql
│   └── surrogate_key.sql
├── seeds/
│   ├── raw_orders.csv
│   ├── raw_customers.csv
│   ├── raw_payments.csv
│   ├── raw_menu_items.csv
│   └── country_codes.csv
├── models/
│   ├── staging/
│   │   ├── _sources.yml
│   │   ├── stg_orders.sql
│   │   ├── stg_customers.sql
│   │   ├── stg_payments.sql
│   │   └── stg_menu_items.sql
│   ├── intermediate/
│   │   └── int_order_payments.sql
│   └── marts/
│       ├── properties.yml
│       ├── fct_orders.sql
│       ├── fct_payments.sql
│       ├── dim_customers.sql
│       ├── dim_menu_items.sql
│       └── exposures.yml
├── snapshots/
│   └── customers_snapshot.sql
├── tests/
│   ├── assert_no_future_orders.sql
│   ├── assert_orders_have_positive_revenue.sql
│   └── assert_no_orphan_payments.sql
└── .github/
    └── workflows/
        └── dbt-ci.yml
```

---

## 💡 讲师给的"骨架代码"（卡住时可以参考）

### `fct_orders.sql` 参考实现

```sql
{{ config(
    materialized='incremental',
    unique_key='order_id'
) }}

-- Grain: one row per order
-- Source: int_order_payments + stg_orders

with order_data as (
    select * from {{ ref('int_order_payments') }}
)

select
    order_id,
    customer_id,
    order_date,
    order_status,
    coalesce(total_payment_amount, 0) as total_revenue_usd,
    payment_count,
    payment_methods_used,
    case
        when total_payment_amount is null then true
        else false
    end as is_unpaid,
    current_timestamp as dbt_loaded_at

from order_data

{% if is_incremental() %}
where order_date >= (
    select coalesce(max(order_date), '1900-01-01')
    from {{ this }}
)
{% endif %}
```

### `properties.yml` 参考片段

```yaml
version: 2

models:
  - name: fct_orders
    description: "Order fact table. One row per order. Includes payment rollup."
    columns:
      - name: order_id
        data_tests: [unique, not_null]
      - name: customer_id
        data_tests:
          - not_null
          - relationships:
              to: ref('dim_customers')
              field: customer_id
      - name: order_status
        data_tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned', 'cancelled']
              severity: warn
      - name: total_revenue_usd
        data_tests: [not_null]

  - name: dim_customers
    description: "Customer dimension with lifetime metrics."
    columns:
      - name: customer_id
        data_tests: [unique, not_null]
      - name: email
        data_tests: [not_null]

  - name: fct_payments
    description: "Payment fact table. One row per successful payment."
    columns:
      - name: payment_id
        data_tests: [unique, not_null]
      - name: order_id
        data_tests:
          - not_null
          - relationships:
              to: ref('fct_orders')
              field: order_id
```

### Singular Test 参考

`tests/assert_no_orphan_payments.sql`：

```sql
-- 所有支付必须指向存在的订单
select p.payment_id, p.order_id
from {{ ref('stg_payments') }} p
left join {{ ref('stg_orders') }} o on p.order_id = o.order_id
where o.order_id is null
```

---

## 🏁 最终验证命令

```bash
# 1. 全量构建 + 测试
dbt build

# 2. 检查新鲜度
dbt source freshness

# 3. 生成文档
dbt docs generate

# 4. 验证选择器
dbt run --select +fct_orders    # 只跑上游
dbt test --select tag:revenue  # 如果有 tag

# 5. 增量验证
# 往 CSV 加新行 → dbt seed --full-refresh → dbt run --select fct_orders
```

---

## 🎓 完成后的自我评估

问自己：

| 问题 | 能回答 = 掌握 |
|------|-------------|
| `ref()` 和 `source()` 的区别是什么？ | ✅ |
| 为什么 staging 层禁止多表 JOIN？ | ✅ |
| `incremental` 模型的三个必要组件是什么？ | ✅ |
| `dbt build` 和 `dbt run` + `dbt test` 的区别？ | ✅ |
| `severity: warn` 什么时候用？ | ✅ |
| `--defer --state` 解决了什么问题？ | ✅ |
| 什么时候用 Macro vs 直接写 SQL？ | ✅ |
| Snapshot 的 `timestamp` vs `check` 策略怎么选？ | ✅ |

---

## 🎉 恭喜你完成 1-Day dbt Crash Course！

从 Module 1 的 `dbt init` 到今天 Module 8 的完整 Pipeline，你已经掌握了：

```
✅ ELT 哲学与 dbt 架构
✅ 分层建模（Staging → Intermediate → Marts）
✅ ref() / source() / DAG 自动编排
✅ 数据质量测试（Generic / Singular / Unit）
✅ 物化策略（View / Table / Incremental / Ephemeral）
✅ 文档生成与血缘可视化
✅ Seeds、Macros、Snapshots
✅ Node Selection 与 CI/CD 基础
✅ 端到端生产级 Pipeline 搭建
```

**你现在的水平**：能够独立为一家中小公司搭建从原始数据到 BI 就绪的数据仓库转换层。

---

## 🚀 下一步学习建议

| 方向 | 推荐资源 |
|------|---------|
| 深入学习 dbt 官方文档 | [docs.getdbt.com](https://docs.getdbt.com) |
| dbt Labs 的示例项目 | `git clone https://github.com/dbt-labs/jaffle_shop` |
| 进阶：dbt Mesh（多项目） | dbt Docs → "Mesh" |
| 进阶：语义层（Metrics） | dbt Semantic Layer |
| 实战：接入真实仓库 | Snowflake/BigQuery 免费额度 |
| 社区 | [dbt Discourse](https://discourse.getdbt.com) |

---

**👉 你的 Capstone 完成了吗？** 把你的项目结构或遇到的错误贴出来，我帮你 review！

或者告诉我你想深入哪个方向，我们可以继续探索 dbt 的进阶世界 🌟

是否需要我帮你 review 某个具体模型的代码，或者解释某个概念在实际生产中的最佳实践？