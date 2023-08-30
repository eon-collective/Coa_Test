## Table of Contents
1. [equal_rowcount](#equal_rowcount)
2. [test_name_2](#test_name_2) *(Placeholder for your next test)*
3. [test_name_3](#test_name_3) *(Placeholder for your next test)*
... *(continue as needed)*

---

### [equal_rowcount](#equal_rowcount)

####  Purpose

`equal_rowcount` helps ensure that two tables, `model` and `compare_model`, have the same number of rows. This test is useful during data validation tasks, like verifying data transformations or ensuring consistency between source and target tables.

#### 📝 Syntax

```jinja
{{ equal_rowcount('<model>', '<compare_model>', ['<group_by_column1>', '<group_by_column2>', ...]) }}
```

**Parameters:**
- `<model>`: The first table you wish to compare.
- `<compare_model>`: The second table you wish to compare.
- `['<group_by_column1>', '<group_by_column2>', ...]`: (Optional) Columns to group by.

#### 🚀 Usage Example

```jinja
{{ equal_rowcount('"SNOWFLAKE_SAMPLE_DATA"."TPCH_SF1"."CUSTOMER"', '"ANALYTICS"."COATEST"."STG_CUSTOMER"') }}
```

#### 📌 Notes

- Ensure table and column names are correctly spelled.
- The macro returns SQL that, when executed, gives the differences in row counts between the two tables.
- No rows returned means the two tables have matching row counts.

---

### [test_name_2](#test_name_2)

... *(Details about this test will go here)*

---

### [test_name_3](#test_name_3)

... *(Details about this test will go here)*

