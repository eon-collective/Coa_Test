## Table of Contents
1. [equal_rowcount](#equal_rowcount)
2. [test_name_2](#test_name_2) *(Placeholder for your next test)*
3. [test_name_3](#test_name_3) *(Placeholder for your next test)*
... *(continue as needed)*

---

### [equal_rowcount](#equal_rowcount)

####  Purpose

`equal_rowcount` helps ensure that two tables, `node` and `compare_node`, have the same number of rows. This test is useful during data validation tasks, like verifying data transformations or ensuring consistency between source and target tables.

#### 📝 Syntax

```jinja
{{ equal_rowcount('<node>', '<compare_node>', ['<group_by_column1>', '<group_by_column2>', ...]) }}
```

**Parameters:**
- `<node>`: The first table you wish to compare.
- `<compare_node>`: The second table you wish to compare.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### 🚀 Usage Example

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

### [test_name_2](#test_name_2)

... *(Details about this test will go here)*

---

### [test_name_3](#test_name_3)

... *(Details about this test will go here)*

