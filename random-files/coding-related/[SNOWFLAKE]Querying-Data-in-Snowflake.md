> From: https://gemini.google.com/app/141a8d2f323875ea?utm_source=app_launcher&utm_medium=owned&utm_campaign=base_all&hl=en_GB

# you asked

message time: 2026-03-06 17:16:25

Various ways to query data in snowfalke

---

# gemini response

Snowflake has evolved significantly from a traditional SQL data warehouse into a versatile "AI Data Cloud." In 2026, you can query data using everything from standard SQL to natural language and advanced programmatic APIs.

Here is a breakdown of the various ways to query data in Snowflake, categorized by the interface and language used.

-----

## 1\. SQL-Based Querying (Traditional)

SQL remains the backbone of Snowflake. You can execute SQL queries through several interfaces:

  * **Snowsight (Web UI):** The modern web interface where you use **Worksheets** to write and run SQL. It includes features like auto-complete, data visualization (charts), and query profiling.
  * **SnowSQL (CLI):** A command-line client for executing SQL and performing data lifecycle operations (like `PUT` and `COPY INTO`).
  * **Drivers & Connectors:** You can query Snowflake via standard drivers like **JDBC, ODBC, .NET,** and **Node.js**, or native connectors for **Python, Go,** and **PHP**.

-----

## 2\. Programmatic & Data Science Querying (Snowpark)

Snowpark allows you to write queries in non-SQL languages using a DataFrame API that is lazily evaluated—meaning the logic is pushed down and executed in the Snowflake engine.

  * **Python, Java, & Scala:** Using the **Snowpark API**, you can write code that looks like local DataFrame operations (similar to PySpark or Pandas) but runs entirely inside Snowflake.
  * **Snowflake Notebooks:** An interactive, cell-based environment within Snowsight. You can seamlessly switch between **SQL and Python** cells, even referencing a SQL result set as a Python variable in the next cell.
  * **Pandas on Snowflake:** You can run existing Pandas code at Snowflake scale. By using the `snowflake.snowpark.pandas` module, your Pandas commands are converted into SQL and executed on Snowflake’s distributed compute.

-----

## 3\. AI & Natural Language Querying (Cortex AI)

With the rise of Generative AI, Snowflake now allows "chatting" with your data.

  * **Cortex Analyst:** A service that converts **Natural Language** questions into SQL. For example, a business user can type *"What were the top 5 selling products last quarter?"* and Snowflake will generate and run the SQL automatically.
  * **Cortex Search:** Enables semantic (vector-based) search over both structured and unstructured data, allowing you to query documents or text fields based on meaning rather than just keywords.

-----

## 4\. Specialized Data Queries

Beyond standard tables, Snowflake supports querying data in unique states:

  * **Querying Staged Files:** You can query data directly sitting in cloud storage (S3, Azure Blob, GCS) or internal stages without loading it into a table first using the `@stage` syntax:
    ```sql
    SELECT $1, $2 FROM @my_stage/data_file.csv;
    ```
  * **Hybrid Tables (Unistore):** Optimized for transactional (OLTP) workloads. You query these just like regular tables, but they provide sub-second response times for single-row lookups and high-concurrency writes.
  * **Time Travel:** You can query data as it existed at a specific point in the past (up to 90 days) using the `AT` or `BEFORE` clause:
    ```sql
    SELECT * FROM my_table AT(TIMESTAMP => '2026-03-01 10:00:00'::timestamp);
    ```

-----

## Comparison Table: Which method to use?

| Method | Best For... | Language |
| :--- | :--- | :--- |
| **Snowsight** | Quick analysis & dashboards | SQL |
| **Snowpark** | Data Engineering & ML Pipelines | Python / Java / Scala |
| **Cortex Analyst** | Non-technical business users | Natural Language |
| **Notebooks** | Exploratory Data Analysis (EDA) | SQL & Python mix |
| **SnowSQL** | Automation & Batch Scripting | SQL |

Would you like me to provide a specific SQL example for a use case, or perhaps show you how to set up a Snowpark session in Python?

---

# you asked

message time: 2026-03-06 17:18:54

Various ways to query data using SQL in snowflake

---

# gemini response

In Snowflake, while there are many programmatic ways to interact with data, **SQL** remains the primary language. Because Snowflake is a cloud-native platform, it provides several specialized SQL extensions that go beyond standard ANSI SQL.

Here are the various ways you can query data using SQL in Snowflake:

---

## 1. Standard DQL (Data Query Language)
This is the most common method, using standard `SELECT` statements to retrieve data from tables and views.
* **Joins and Aggregations:** Full support for complex joins, window functions, and CTEs (Common Table Expressions).
* **Subqueries:** Support for correlated and non-correlated subqueries.

## 2. Querying Semi-Structured Data
One of Snowflake’s strongest features is the ability to query **JSON, AvRO, ORC, Parquet, or XML** using SQL without pre-loading them into relational columns.
* **Dot Notation:** Use a colon `:` to navigate through JSON hierarchies.
* **FLATTEN:** A table function used to explode nested arrays into individual rows.
* **Sample Syntax:** `SELECT src:customer_name, src:address.city FROM raw_customer_data;`



---

## 3. Querying Files in Stages (Data Lake Queries)
You can run SQL directly against files sitting in **Internal Stages** or **External Stages** (S3, Azure Blob, GCS) before you ever "load" them into a table.
* **Metadata Querying:** You can query the file's metadata (like filename or row number) alongside the content.
* **Example:** `SELECT $1, $2 FROM @my_s3_stage/folder/data.csv;`

---

## 4. Time Travel & Fail-safe Queries
Snowflake allows you to query the state of a table at a specific point in the past using the `AT` or `BEFORE` clauses.
* **Timestamp:** Query data as it was at a specific clock time.
* **Offset:** Query data from "10 minutes ago."
* **Statement ID:** Query data as it looked before a specific (perhaps accidental) SQL command was run.
* **Example:** `SELECT * FROM employees BEFORE(statement => '01af23...');`

---

## 5. Directory Tables
If you have a stage full of unstructured data (like PDFs, images, or audio files), you can query a **Directory Table**. This is a specialized view that uses SQL to catalog the files, their sizes, and their "File URLs" for processing by AI models or external applications.

---

## 6. Dynamic Tables and Streams
Instead of a static query, you can use SQL to define data movement:
* **Streams:** A "Change Data Capture" (CDC) query that only shows you rows that have changed (inserted, updated, or deleted) since you last looked.
* **Dynamic Tables:** You provide a `SELECT` statement, and Snowflake automatically manages the refresh process to keep the results up to date.

---

## 7. Metadata & Account Usage Queries
Snowflake stores all its own administrative data in a shared database called `SNOWFLAKE`. You can use SQL to query:
* **ACCOUNT_USAGE:** To see who is using the most credits or which queries are failing.
* **INFORMATION_SCHEMA:** To get lists of tables, columns, and permissions.

---

### Summary Table: SQL Query Types

| Query Type | SQL Feature/Keyword | Best Use Case |
| :--- | :--- | :--- |
| **Relational** | `SELECT`, `JOIN`, `WHERE` | Standard reporting and analytics. |
| **Semi-Structured** | `FLATTEN`, `:`, `PARSE_JSON` | Analyzing JSON logs or IoT data. |
| **Historical** | `AT`, `BEFORE` | Recovering deleted data or auditing changes. |
| **External** | `FROM @stage_name` | Previewing data before ingestion. |
| **Change Tracking** | `CHANGES`, `STREAM` | Building incremental ETL pipelines. |

---

Would you like to see a specific SQL example for **Time Travel** or a complex **JSON flattening** query?

---

# you asked

message time: 2026-03-06 17:21:10

Various ways to query data using variables in snowflake

---

# gemini response

Using variables in Snowflake SQL is a game changer for making scripts reusable, readable, and dynamic. Depending on whether you are working in a standard worksheet, a script, or a programmatic environment, the syntax changes slightly.

Here are the four primary ways to query data using variables in Snowflake.

-----

## 1\. Using SQL Variables (`SET`)

This is the most common method for ad-hoc analysis in Snowsight or SnowSQL. You define a variable in one statement and reference it in another using the `$` prefix.

  * **How to do it:**
    ```sql
    -- Set the variable
    SET my_date = '2026-03-05';
    SET department = 'Sales';

    -- Use the variable in a query
    SELECT * FROM employees 
    WHERE hire_date > $my_date 
    AND dept_name = $department;
    ```
  * **Best for:** Quick filters in SQL worksheets or scripts where you need to update a value in multiple places at once.

-----

## 2\. Using Session Variables in Snowpark (Python/Java)

If you are querying Snowflake via Python (Snowpark), you don't use the `SET` command. Instead, you use **parameterized queries** or **f-strings** (though parameterized is safer).

  * **How to do it:**
    ```python
    # Define your variable in Python
    target_city = 'New York'

    # Query using the variable
    df = session.sql("SELECT * FROM weather WHERE city = ?", params=[target_city])
    df.show()
    ```
  * **Best for:** Application development and data science pipelines where variables come from user input or external APIs.

-----

## 3\. Snowflake Scripting (Blocks)

When writing complex logic (Stored Procedures or anonymous blocks), you use `DECLARE` and `BEGIN...END`. These variables are local to that specific block of code.

  * **How to do it:**
    ```sql
    EXECUTE IMMEDIATE $$
    DECLARE
        min_salary NUMBER DEFAULT 50000;
    BEGIN
        -- Reference the variable using the colon (:) prefix inside the block
        LET rs := (SELECT * FROM jobs WHERE salary > :min_salary);
        RETURN TABLE(rs);
    END;
    $$;
    ```
  * **Best for:** Heavy-duty automation, loops, and conditional logic (`IF/THEN`) within Snowflake.

-----

## 4\. Identifier Variables

Sometimes you don't want to filter a *value*; you want to change the **Table Name** or **Column Name** itself. Standard variables won't work here unless you use the `IDENTIFIER()` function.

  * **How to do it:**
    ```sql
    SET table_name = 'RETAIL_SALES_2026';

    -- This will fail: SELECT * FROM $table_name;
    -- This will work:
    SELECT * FROM IDENTIFIER($table_name) LIMIT 10;
    ```
  * **Best for:** Dynamic SQL where you need to switch between different tables (e.g., `sales_jan`, `sales_feb`) programmatically.

-----

## Summary Table

| Method | Syntax | Scope | Best Use Case |
| :--- | :--- | :--- | :--- |
| **SQL SET** | `$var_name` | Session | Manual Worksheet analysis. |
| **Scripting** | `:var_name` | Block | Stored Procedures / Automation. |
| **Snowpark** | `?` or `:1` | App | Python/Data Science integration. |
| **Identifier** | `IDENTIFIER($var)` | Session | Dynamic Table/Column names. |

-----

### Pro-Tip: Viewing your variables

If you lose track of what variables you have active in your current session, you can run:
`SHOW VARIABLES;`

Would you like to see how to use these variables inside a **Stored Procedure** or a **Dynamic Table** definition?