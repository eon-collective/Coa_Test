## Coalesce Reusable Tests
1. [equal_rowcount](#equal_rowcount)
2. [row_count_delta](#row_count_delta)
3. [equality](#equality)
4. [expression_is_true](#expression_is_true)
5. [recency](#recency)
6. [at_least_one](#at_least_one)
7. [not_constant](#not_constant)
8. [not_empty_string](#not_empty_string)
9. [cardinality_equality](#cardinality_equality)
10. [not_null_proportion](#not_null_proportion)
11. [not_accepted_values](#not_accepted_values)
12. [relationships_where](#relationships_where)
13. [recency](#recency)
14. [recency](#recency)

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
- `<interval>`:  The interval value you want to subtract from the current date or timestamp.
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

### [at_least_one](#at_least_one)

#### Purpose
The `at_least_one` macro checks if a given column in a table contains at least one non-null value. This test can be useful to ensure that essential columns in your dataset are populated.

#### Syntax
```jinja
{{ at_least_one('<node>', '<column_name>', ['<group_by_column1>', '<group_by_column2>', ...]) }}
```
**Parameters:**
- `<node>`:  The table you wish to evaluate.
- `<column_name>`: The column you want to check for non-null values.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### Usage Example

```jinja
{{ at_least_one('"ANALYTICS"."COATEST"."STG_SUPPLIER"', '"S_ACCTBAL"') }}
```
This checks if there's at least one non-null `S_ACCTBAL` in the `STG_SUPPLIER` table.  

<details>
<summary>🌏 Source</summary>

```sql
{% macro at_least_one(node, column_name, group_by_columns=[]) %}

{% if group_by_columns %}
{% set select_gb_cols = group_by_columns|join(', ') + ', ' %}
{% set groupby_gb_cols = 'GROUP BY ' + group_by_columns|join(', ') %}
{% else %}
{% set select_gb_cols = '' %}
{% set groupby_gb_cols = '' %}
{% endif %}

SELECT
    {{ select_gb_cols }}
    COUNT({{ column_name }}) AS count_of_values
FROM (
    SELECT
    {{ select_gb_cols }}
    {{ column_name }}
    FROM {{ node }}
    WHERE {{ column_name }} IS NOT NULL
    LIMIT 1
) AS pruned_rows
{{ groupby_gb_cols }}
HAVING COUNT({{ column_name }}) = 0

{% endmacro %}


```
</details> 

### [not_constant](#not_constant)

#### Purpose
The `not_constant` macro checks if a given column in a table does **not** have the same value across all rows. This test is useful to ensure that data diversity exists in columns where constant values would be problematic.

### Syntax
```jinja
{{ not_constant('<node>', '<column_name>', ['<group_by_column1>', '<group_by_column2>', ...]) }}
```
**Parameters:**
- `<node>`:  The table you wish to evaluate.
- `<column_name>`:  The column you want to check for non-constant values.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### Usage Example

```jinja
{{ not_constant('"ANALYTICS"."COATEST"."STG_CUSTOMER"', '"C_CUSTKEY"') }}
```
This checks if the `C_CUSTKEY` in the `STG_CUSTOMER` table does not have the same value across all rows.  

<details>
<summary>🌏 Source</summary>

```sql
{% macro not_constant(node, column_name, group_by_columns=[]) %}

{% if group_by_columns %}
{% set select_gb_cols = group_by_columns|join(', ') + ', ' %}
{% set groupby_gb_cols = 'GROUP BY ' + group_by_columns|join(', ') %}
{% else %}
{% set select_gb_cols = '' %}
{% set groupby_gb_cols = '' %}
{% endif %}

SELECT
    {{ select_gb_cols }}
    COUNT(DISTINCT {{ column_name }}) AS distinct_count_of_values
FROM {{ node }}
{{ groupby_gb_cols }}
HAVING COUNT(DISTINCT {{ column_name }}) = 1

{% endmacro %}


```
</details> 

### [not_empty_string](#not_empty_string)

#### Purpose
The `not_empty_string` macro checks if a given column in a table contains any empty string values. This test is useful to ensure that important columns don't contain blank values which might lead to erroneous analyses.

#### Syntax
```jinja
{{ not_empty_string('<node>', '<column_name>', trim_whitespace=True) }}
```
**Parameters:**
- `<node>`:  The table you wish to evaluate.
- `<column_name>`:  The column you want to check for empty string values.
- `trim_whitespace`: (Optional) If set to True, the macro will trim leading and trailing whitespaces from the values before checking for emptiness. Default is `True`.

#### Usage Example

```jinja
{{ not_empty_string('"ANALYTICS"."COATEST"."STG_PARTSUPP"', '"PS_COMMENT"') }}
```
This checks if the `PS_COMMENT` in the `STG_PARTSUPP` table contains any rows with empty string values.

<details>
<summary>🌏 Source</summary>

```sql
{% macro not_empty_string(node, column_name, trim_whitespace=True) %}

WITH all_values AS (
    SELECT 
        {% if trim_whitespace %}
            TRIM({{ column_name }}) AS {{ column_name }}
        {% else %}
            {{ column_name }}
        {% endif %}
    FROM {{ node }}
),

errors AS (
    SELECT * 
    FROM all_values
    WHERE {{ column_name }} = ''
)

SELECT * FROM errors

{% endmacro %}


```
</details> 

### [cardinality_equality](#cardinality_equality)

#### Purpose
This macro is designed to assert that values in a given column (`column_name`) from one table (`node`) have the same cardinality (i.e., the same number of unique values) as values from a different column (`field`) in another table (`to_node`). It's especially handy in scenarios where two columns, potentially from different tables, are expected to have matching sets of unique values.

#### Syntax
```jinja
{{ cardinality_equality('<node>', '<column_name>', '<to_node>', '<field>') }}
```

**Parameters:**
- `<node>`:  The initial table you want to evaluate.
- `<column_name>`:  The column in the initial table you wish to assess.
- `<to_node>`: he secondary table you're comparing against.
- `<field>`: The column in the secondary table you're comparing with.
  
#### Usage Example

```jinja
{{ cardinality_equality('"ANALYTICS"."COATEST"."STG_CUSTOMER"', '"C_CUSTKEY"', '"ANALYTICS"."COATEST"."STG_ORDERS"', '"O_ORDERKEY"') }}
```
This example checks if the unique values of `"C_CUSTKEY"` from the `"STG_CUSTOMER"` table have the same cardinality as the unique values of `"O_ORDERKEY"` from the `"STG_ORDERS"` table.

<details>
<summary>🌏 Source</summary>

```sql
{% macro cardinality_equality(node, column_name, to_node, field) %}

WITH table_a AS (
    SELECT
        {{ column_name }},
        COUNT(*) AS num_rows
    FROM {{ node }}
    GROUP BY {{ column_name }}
),

table_b AS (
    SELECT
        {{ field }},
        COUNT(*) AS num_rows
    FROM {{ to_node }}
    GROUP BY {{ field }}
),

except_a AS (
    SELECT *
    FROM table_a
    EXCEPT
    SELECT *
    FROM table_b
),

except_b AS (
    SELECT *
    FROM table_b
    EXCEPT
    SELECT *
    FROM table_a
),

unioned AS (
    SELECT *
    FROM except_a
    UNION ALL
    SELECT *
    FROM except_b
)

SELECT *
FROM unioned

{% endmacro %}


```
</details> 


### [not_null_proportion](#not_null_proportion)

#### **Purpose**

The `test_not_null_proportion` macro asserts that the proportion of non-null values in a specified column is within a given range, denoted by the parameters `at_least` and `at_most`. This test is useful for situations where you expect a certain fraction of records in a column to contain non-null values. For example, ensuring that at least 80% of email addresses in a dataset are populated, but not more than 100%.

#### **Syntax**

```jinja
{{ test_not_null_proportion('<node>', '<column_name>', at_least=<lower_bound>, at_most=<upper_bound>, group_by_columns=['<group_by_column1>', '<group_by_column2>', ...]) }}
```

**Parameters:**
- `<node>`:  The initial table you want to evaluate.
- `<column_name>`:   The column you want to evaluate for the proportion of non-null values.
- `<at_least>`: (Optional) The lower bound for the proportion of non-null values. Default is 0.
- `<at_most>`: (Optional) The upper bound for the proportion of non-null values. Default is 1.0.
- `<group_by_columns>`:  (Optional) Columns to group by if evaluating the non-null proportion for subsets of the data.
  
#### Usage Example

```jinja
{{ not_null_proportion('"ANALYTICS"."COATEST"."STG_LINEITEM"', '"L_SHIPDATE"', at_least=0.8, at_most=1.0) }}
```
This example checks if the proportion of non-null values in the L_SHIPDATE column from the STG_LINEITEM table is between 0.8 (or 80%) and 1.0 (or 100%).

<details>
<summary>🌏 Source</summary>
  
```sql
{% macro not_null_proportion(node, column_name, at_least=0, at_most=1.0, group_by_columns=[]) %}

{% if group_by_columns|length() > 0 %}
{% set select_gb_cols = group_by_columns|join(' ,') + ', ' %}
{% set groupby_gb_cols = 'group by ' + group_by_columns|join(',') %}
{% endif %}

with validation as (
select
    {{select_gb_cols}}
    sum(case when {{ column_name }} is null then 0 else 1 end) / cast(count(*) as numeric) as not_null_proportion
from {{ node }}
{{groupby_gb_cols}}
),
validation_errors as (
select
    {{select_gb_cols}}
    not_null_proportion
from validation
where not_null_proportion < {{ at_least }} or not_null_proportion > {{ at_most }}
)
select
*
from validation_errors

{% endmacro %}


```
</details> 

### [not_accepted_values](#not_accepted_values)

#### Purpose

The `not_accepted_values` macro asserts that there are no rows in a specified column of a table that match a list of provided values. This is particularly useful in data validation tasks where certain values are deemed unacceptable or incorrect.

#### Syntax

```jinja
{{ not_accepted_values('<node>', '<column_name>', ['<value1>', '<value2>', ...], <quote=True/False>) }}
```

📘 Parameters:
- `<node>`: The table you wish to evaluate.
- `<column_name>`: The column name to check for unacceptable values.
- `['<value1>', '<value2>', ...]`: List of values you want to ensure are not present in the column.
- `<quote>`: (Optional) A boolean indicating whether the values in the list should be treated as strings (quoted). Default is `True`.

#### 🚀 Usage Example

```jinja
{{ not_accepted_values('"ANALYTICS"."COATEST"."STG_LINEITEM"', '"L_SHIPMODE"', ['EXPRESS', 'AIR'], True) }}
```

<details>
<summary>🌏 Source</summary>

```sql
{% macro not_accepted_values(node, column_name, values, quote=True) %}

with all_values as (
    select distinct
        {{ column_name }} as value_field
    from {{ node }}
),

validation_errors as (
    select
        value_field
    from all_values
    where value_field in (
        {% for value in values -%}
            {% if quote -%}
            '{{ value }}'
            {%- else -%}
            {{ value }}
            {%- endif -%}
            {%- if not loop.last -%},{%- endif %}
        {%- endfor %}
    )
)

select *
from validation_errors

{% endmacro %}
```

</details>

### [relationships_where](#relationships_where)

#### Purpose

The `relationships_where` macro asserts referential integrity between two relations, It includes an added benefit: the ability to filter out some rows from the test using predicates. This is particularly useful when needing to exclude records such as test entities or rows that may have been created recently, which could temporarily skew results due to ETL limitations or other factors.

#### Syntax

```jinja
{{ relationships_where('<model>', '<column_name>', '<to>', '<field>', '<from_condition>', '<to_condition>') }}
```
📘 Parameters:
- `<node>`: The source table you wish to evaluate.
- `<column_name>`: The column in the source table to test.
- `<to>`: The target table you wish to evaluate against.
- `<field>`: The column in the target table to test.
- `<from_condition>`: The SQL condition to filter rows in the source table.
- `<to_condition>`: The SQL condition to filter rows in the target table.

#### 🚀 Usage Example

```jinja
{{ relationships_where('"ANALYTICS"."COATEST"."STG_ORDERS"', '"O_ORDERKEY"', '"ANALYTICS"."COATEST"."STG_LINEITEM"', '"L_ORDERKEY"', 'O_ORDERSTATUS != \'F\'', 'L_SHIPDATE > current_timestamp() - interval \'1 day\'') }}
```

<details>
<summary>🌏 Source</summary>

```sql
{% macro relationships_where(node, column_name, to, field, from_condition="1=1", to_condition="1=1") %}

{# Define the left table with conditions applied #}
with left_table as (
select
    {{column_name}} as id
from {{node}}
where {{column_name}} is not null
    and {{from_condition}}
),

{# Define the right (referenced) table with conditions applied #}
right_table as (
select
    {{field}} as id
from {{to}}
where {{field}} is not null
    and {{to_condition}}
),

{# Check for exceptions in the relationship between the tables #}
exceptions as (
select
    left_table.id,
    right_table.id as right_id
from left_table
left join right_table
        on left_table.id = right_table.id
where right_table.id is null
)

select * from exceptions

{% endmacro %}
```

</details>
