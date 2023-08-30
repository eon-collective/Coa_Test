## Table of Contents
1. [equal_rowcount](#equal_rowcount)
2. [row_count_delta](#row_count_delta)
3. [test_name_3](#test_name_3) *(Placeholder for your next test)*
... *(continue as needed)*

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

### [test_name_3](#test_name_3)

... *(Details about this test will go here)*

