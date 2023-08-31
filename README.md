## Table of Contents
1. [equal_rowcount](#equal_rowcount)
2. [row_count_delta](#row_count_delta)
3. [equality](#equality)
4. [expression_is_true](#expression_is_true)
5. [recency](#recency)

---

### [equal_rowcount](#equal_rowcount)

####  Purpose

`equal_rowcount` helps ensure that two tables, `node` and `compare_node`, have the same number of rows. This test is useful during data validation tasks, like verifying data transformations or ensuring consistency between source and target tables.

#### Syntax

```jinja
{{ equal_rowcount('<node>', '<compare_node>', ['<group_by_column1>', '<group_by_column2>', ...]) }}
```

**Parameters:**
- `<node>`: The first table you wish to compare.
- `<compare_node>`: The second table you wish to compare.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### Usage Example

```jinja
{{ equal_rowcount('"SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER"', '"ANALYTICS"."COATEST"."STG_CUSTOMER"') }}
```
<details> 
<summary>🌏 Source</summary>

```sql
{% macro equal_rowcount(node, compare_node, group_by_columns=[]) %}

{% if group_by_columns %}
  {% set select_gb_cols = group_by_columns|join(', ') + ', ' %}
  {% set join_gb_cols = [] %}
  {% for c in group_by_columns %}
    {% set join_gb_cols = join_gb_cols + ["and a." + c + " = b." + c] %}
  {% endfor %}
  {% set join_gb_cols = join_gb_cols|join(' ') %}
  {% set groupby_gb_cols = 'group by ' + group_by_columns|join(',') %}
{% else %}
  {% set select_gb_cols = '' %}
  {% set join_gb_cols = '' %}
  {% set groupby_gb_cols = '' %}
{% endif %}

{# We must add a fake join key in case additional grouping variables are not provided #}
{% set group_by_columns = ['id_equal_rowcount'] + group_by_columns %}
{% set groupby_gb_cols = 'group by ' + group_by_columns|join(',') %}

with a as (
    select 
      {{ select_gb_cols }}
      1 as id_equal_rowcount,
      count(*) as count_a 
    from {{ node }}
    {{ groupby_gb_cols }}
),

b as (
    select 
      {{ select_gb_cols }}
      1 as id_equal_rowcount,
      count(*) as count_b 
    from {{ compare_node }}
    {{ groupby_gb_cols }}
),

final as (
    select
        {% for c in group_by_columns %}
          a.{{ c }} as {{ c }}_a,
          b.{{ c }} as {{ c }}_b,
        {% endfor %}
        count_a,
        count_b,
        abs(count_a - count_b) as diff_count
    from a
    full join b
    on
    a.id_equal_rowcount = b.id_equal_rowcount
    {{ join_gb_cols }}
)
select * from final

{% endmacro %}


```

</details>  

### [row_count_delta](#row_count_delta)

#### Purpose

The `row_count_delta` macro is used to determine if one table (`node`) has fewer rows than another table (`compare_node`). This test is useful in scenarios where you expect a specific table to always contain a subset of rows from another larger table. If the `node` has the same or more rows than the `compare_node`, the `row_count_delta` will be positive.

#### Syntax

```jinja
{{ row_count_delta('<node>', '<compare_node>', ['<group_by_column1>', '<group_by_column2>', ...]) }}
```

**Parameters:**
- `<node>`: The first table you wish to compare.
- `<compare_node>`: The second table you wish to compare.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### Usage Example

```jinja
{{ row_count_delta('"YOUR_SCHEMA"."YOUR_TABLE"', '"YOUR_SCHEMA"."YOUR_COMPARISON_TABLE"') }}
```
<details> 
<summary>🌏 Source</summary>

```sql
{% macro row_count_delta(node, compare_node, group_by_columns=[]) %}

{% if group_by_columns %}
  {% set select_gb_cols = group_by_columns|join(', ') + ', ' %}
  {% set join_gb_cols = [] %}
  {% for c in group_by_columns %}
    {% set join_gb_cols = join_gb_cols + ["and a." + c + " = b." + c] %}
  {% endfor %}
  {% set join_gb_cols = join_gb_cols|join(' ') %}
  {% set groupby_gb_cols = 'group by ' + group_by_columns|join(',') %}
{% else %}
  {% set select_gb_cols = '' %}
  {% set join_gb_cols = '' %}
  {% set groupby_gb_cols = '' %}
{% endif %}

{# We must add a fake join key in case additional grouping variables are not provided #}
{% set group_by_columns = ['id_test_fewer_rows_than'] + group_by_columns %}
{% set groupby_gb_cols = 'group by ' + group_by_columns|join(',') %}

WITH a AS (
    SELECT 
      {{ select_gb_cols }}
      1 AS id_test_fewer_rows_than,
      COUNT(*) AS count_our_model 
    FROM {{ node }}
    {{ groupby_gb_cols }}
),

b AS (
    SELECT 
      {{ select_gb_cols }}
      1 AS id_test_fewer_rows_than,
      COUNT(*) AS count_comparison_model 
    FROM {{ compare_node }}
    {{ groupby_gb_cols }}
),

counts AS (
    SELECT
        {% for c in group_by_columns %}
          a.{{ c }} AS {{ c }}_a,
          b.{{ c }} AS {{ c }}_b,
        {% endfor %}
        count_our_model,
        count_comparison_model
    FROM a
    FULL JOIN b ON a.id_test_fewer_rows_than = b.id_test_fewer_rows_than
    {{ join_gb_cols }}
),

final AS (
    SELECT *,
        CASE
            WHEN count_our_model > count_comparison_model THEN (count_our_model - count_comparison_model)
            WHEN count_our_model = count_comparison_model THEN 1
            ELSE 0
        END AS row_count_delta
    FROM counts
)

SELECT * FROM final;

{% endmacro %}


```

</details>  

### [equality](#equality)

#### Purpose

The `equality` macro asserts the equality of two relations. This test can be beneficial in scenarios where you expect two tables to have identical rows, potentially after a transformation. Optionally, you can specify a subset of columns to compare.

#### Syntax

```jinja
{{ equality('<node>', '<compare_node>', ['<compare_column1>', '<compare_column2>', ...]) }}
```

**Parameters:**
- `<node>`: The first table you wish to compare.
- `['<compare_column1>, '<compare_column2>', ...]`: The second table you wish to compare.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Specific columns to compare.

#### Usage Example

```jinja
{{ equality('"SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER"', '"ANALYTICS"."COATEST"."STG_CUSTOMER"', ['O_ORDERKEY', 'O_CUSTKEY']) }}
```
<details> 
<summary>🌏 Source</summary>

```sql
{% macro equality(node, compare_node, compare_columns=None) %}

{%- if not compare_columns -%}
    {%- set compare_columns = adapter.get_columns_in_relation(node) %}
{%- endif %}

{% set compare_cols_csv = compare_columns | join(', ') %}

WITH a AS (
    SELECT * FROM {{ node }}
),

b AS (
    SELECT * FROM {{ compare_node }}
),

a_minus_b AS (
    SELECT {{compare_cols_csv}} FROM a
    EXCEPT
    SELECT {{compare_cols_csv}} FROM b
),

b_minus_a AS (
    SELECT {{compare_cols_csv}} FROM b
    EXCEPT
    SELECT {{compare_cols_csv}} FROM a
),

unioned AS (
    SELECT 'a_minus_b' AS which_diff, a_minus_b.* FROM a_minus_b
    UNION ALL
    SELECT 'b_minus_a' AS which_diff, b_minus_a.* FROM b_minus_a
)

SELECT * FROM unioned

{% endmacro %}


```

</details>  

### [expression_is_true](#expression_is_true)

#### Purpose

The `expression_is_true` macro is designed to assert that a given SQL expression is true for all records in a table. It's an essential tool for validating data quality and ensuring column integrity. This can be useful in various scenarios, including:
- Verifying an outcome based on the application of basic algebraic operations between columns.
- Checking the length of a column.
- Verifying the truth value of a column.

#### Syntax

```jinja
{{ expression_is_true('<node>', '<expression>', '<column_name>') }}
```
**Parameters:**
- `<node>`:  The table you wish to evaluate.
- `<expression>`: The SQL expression you want to evaluate for truthiness.
- `['<column_name>']`: (Optional) The specific column to which you want to apply the expression. If omitted, the macro evaluates the expression for all records in the table.

#### Usage Example

```jinja
{{ expression_is_true('"ANALYTICS"."COATEST"."LINEITEM"', '>= 1', '"L_EXTENDEDPRICE"') }}
```
<details>
<summary>🌏 Source</summary>

```sql
{% macro expression_is_true(node, expression, column_name=None) %}

SELECT
    *
FROM {{ node }}
{% if column_name is none %}
WHERE NOT ({{ expression }})
{%- else %}
WHERE NOT ({{ column_name }} {{ expression }})
{%- endif %}

{% endmacro %}


```

</details> 

### [recency](#recency)

#### Purpose
The `recency` macro checks if the most recent date in a given column is within a specified interval. This can be useful to ensure data freshness or to validate that new records are being added as expected within a certain timeframe.

### Syntax
```jinja
{{ recency('<node>', '<field>', '<datepart>', <interval>, <ignore_time_component>, ['<group_by_column1>', '<group_by_column2>', ...]) }}
```
**Parameters:**
- `<node>`:  The table you wish to evaluate.
- `<field>`:  The timestamp or date column you want to check.
- `<datepart>`: The date part you want to use for the interval (e.g., 'day', 'month', 'year', etc.).
- - `<interval>`:  The interval value you want to subtract from the current date or timestamp.
- `<ignore_time_component>`:  If set to `True`, only the date part of the timestamp will be considered, otherwise the full timestamp is used.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### Usage Example

```jinja
{{ recency('"ANALYTICS"."COATEST"."STG_LINEITEM"', '"L_SHIPDATE"', 'day', 7, True) }}
```
This checks if the most recent L_SHIPDATE in the STG_LINEITEM table is within the last 7 days.  
<details>
<summary>🌏 Source</summary>

```sql
{% macro recency(node, field, datepart, interval, ignore_time_component=False, group_by_columns=[]) %}

{% set threshold = 'DATEADD(' ~ datepart ~ ', -' ~ interval ~ ', CURRENT_TIMESTAMP())' %}

{% if ignore_time_component %}
  {% set threshold = 'DATE(' ~ threshold ~ ')' %}
{% endif %}

{% if group_by_columns %}
  {% set select_gb_cols = group_by_columns|join(', ') + ', ' %}
  {% set groupby_gb_cols = 'GROUP BY ' + group_by_columns|join(', ') %}
{% else %}
  {% set select_gb_cols = '' %}
  {% set groupby_gb_cols = '' %}
{% endif %}

WITH recency AS (
    SELECT 
      {{ select_gb_cols }}
      {% if ignore_time_component %}
        CAST(MAX({{ field }}) AS DATE) AS most_recent
      {%- else %}
        MAX({{ field }}) AS most_recent
      {%- endif %}
    FROM {{ node }}
    {{ groupby_gb_cols }}
)

SELECT
    {{ select_gb_cols }}
    most_recent,
    {{ threshold }} AS threshold
FROM recency
WHERE most_recent < {{ threshold }}

{% endmacro %}


```

</details> 
