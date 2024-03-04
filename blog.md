![image](https://github.com/aahmedomar/coa-test/assets/132437758/a427d085-d7b5-4797-8e3b-761e85b4bd24)

## Introducing Re-usable dbt_utils genetic tests to Coalesce Users  

### Introduction

It is data engineer's critical responsibility to ensure data quality beacuse accurate, complete, and consistent data is the foundation for making sound business decisions.However, this can pose a significant challenge, particularly when working with large and complex datasets.

To address this, data engineers can implement genetic tests which use algorithms, to identify patterns and anomalies within data,resulting in  improved data quality, overall reliability and accuracy.
Genetic tests are useful in identifying a wide range of data quality issues, such as missing values, duplicate records, and inconsistent data formats.


**Eon-Collective** is a Data Modernization consulting company that helps clients cut costs, streamline operations, and make swift data-driven decisions. They are powered by ADEPT, their in-house technology & framework.

![image](https://github.com/aahmedomar/coa-test/assets/132437758/6698377b-8adf-427d-b2ba-82a6581ba246)  
**Coalesce** It's a data transformation  tool built for scale. Coalesce has some great features. It has a drag-and-drop system, making it easy for beginners. But if you like using SQL, you can still use it. Plus, Coalesce lets you track data from one column to another. Try it out with their free trial!

**Introducing the re-usable tests:**

Eon-Collective has built a set of reusable dbt_utils genetic tests for Coalesce users. These tests can be used to identify a wide range of data quality issues, and they are easy to use and configure.


Your one-stop shop? Right here - [Coalesce Reusable Tests](https://github.com/aahmedomar/coa-test)

We've listed a few, to give you a glimpse:

1. **equal_rowcount**: Ensures two tables have the same row count. Ideal for data validation tasks.
   
2. **row_count_delta**: Perfect when one table should always be a subset of another.

3. **equality**: Asserts the equality of two relations, optionally within specific columns.

... And so many more, each with its own syntax, parameters, and usage examples.

**How to use it:**

**1. Through the UI:**

   - Log into your Coalesce account.
   - Navigate to Build Settings > Macros.
   - Use the [GitHub repo](https://github.com/aahmedomar/coa-test) to find your desired test, read the description, and copy the source code.
   - Paste this code into the Coalesce UI under Macros.
   
   **To Apply Tests on Node Level:**
   
   - Navigate to the desired Node.
   - Click on the Testing toggle > New Test.
   - Follow the GitHub repo's syntax and examples to apply your tests.

**2. Programmatically:**

   - Clone your Coalesce repo.
   - Open with your favorite IDE and locate the `data.yml` file.
   - Under the `macros:` item, paste the source code from the [GitHub repo](https://github.com/aahmedomar/coa-test).
   - Commit and push your changes.
   
   **Update and Apply in Coalesce:**
   
   - In Coalesce, click the git icon and pull the latest changes.
   - Navigate to the Node you wish to test.
   - Click Testing > New Test.
   - Use the GitHub repo as a reference for syntax and examples when applying tests.

---

Your support means the world! If you find our reusable tests valuable, please do star our [GitHub Repo](https://github.com/aahmedomar/coa-test). Stay tuned for more innovations in reusable tests!
