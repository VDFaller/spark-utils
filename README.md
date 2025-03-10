This [dbt](https://github.com/dbt-labs/dbt) package contains macros 
that:
- can be (re)used across dbt projects running on Spark
- define Spark-specific implementations of [dispatched macros](https://docs.getdbt.com/reference/dbt-jinja-functions/dispatch) from other packages

## Installation Instructions

Check [dbt Hub](https://hub.getdbt.com) for the latest installation 
instructions, or [read the docs](https://docs.getdbt.com/docs/package-management) 
for more information on installing packages.

----

## Compatibility

This package provides "shims" for:
- [dbt_utils](https://github.com/dbt-labs/dbt-utils), except for:
    - `dbt_utils.groupby`
    - `dbt_utils.recency`
    - `dbt_utils.any_value`
    - `dbt_utils.listagg`
    - `dbt_utils.pivot` with apostrophe(s) in the `values` 
- [snowplow](https://github.com/dbt-labs/snowplow) (tested on Databricks only)

In order to use these "shims," you should set a `dispatch` config in your root project (on dbt v0.20.0 and newer). For example, with this project setting, dbt will first search for macro implementations inside the `spark_utils` package when resolving macros from the `dbt_utils` namespace:
```
dispatch:
  - macro_namespace: dbt_utils
    search_order: ['spark_utils', 'dbt_utils']
```

### Note to maintainers of other packages

The spark-utils package may be able to provide compatibility for your package, especially if your package leverages dbt-utils macros for cross-database compatibility. This package _does not_ need to be specified as a dependency of your package in `packages.yml`. Instead, you should encourage anyone using your package on Apache Spark / Databricks to:
- Install `spark_utils` alongside your package
- Add a `dispatch` config in their root project, like the one above

----

## Useful macros: maintenance

_Caveat: These are not tested in CI, or guaranteed to work on all platforms._

Each of these macros accepts a regex pattern, finds tables with names matching the pattern, and will loop over those tables to perform a maintenance operation:

- `spark_optimize_delta_tables`: Runs `optimize` for all matched Delta tables
- `spark_vacuum_delta_tables`: Runs `vacuum` for all matched Delta tables
- `spark_analyze_tables`: Compute statistics for all matched tables

----

### Contributing

We welcome contributions to this repo! To contribute a new feature or a fix, 
please open a Pull Request with 1) your changes and 2) updated documentation for 
the `README.md` file.

## Testing

The macros are tested with [`pytest`](https://docs.pytest.org) and
[`pytest-dbt-core`](https://pypi.org/project/pytest-dbt-core/). For example,
the [`create_tables` macro is tested](./tests/test_macros.py) by:

1. Create a test table (test setup):
   ``` python
   spark_session.sql(f"CREATE TABLE {table_name} (id int) USING parquet")
   ```
2. Call the macro generator:
   ``` python
   tables = macro_generator()
   ```
3. Assert test condition:
   ``` python
   assert simple_table in tables
   ```
4. Delete the test table (test cleanup):
   ``` python
   spark_session.sql(f"DROP TABLE IF EXISTS {table_name}")
   ```

A macro is fetched using the 
[`macro_generator`](https://pytest-dbt-core.readthedocs.io/en/latest/dbt_spark.html#usage) 
fixture and providing the macro name trough 
[indirect parameterization](https://docs.pytest.org/en/7.1.x/example/parametrize.html?highlight=indirect#indirect-parametrization):

``` python
@pytest.mark.parametrize(
    "macro_generator", ["macro.spark_utils.get_tables"], indirect=True
)
def test_create_table(macro_generator: MacroGenerator) -> None:
```

----

### Getting started with dbt + Spark

- [What is dbt](https://docs.getdbt.com/docs/introduction)?
- [Installation](https://github.com/dbt-labs/dbt-spark)
- Join the #spark channel in [dbt Slack](http://slack.getdbt.com/)


## Code of Conduct

Everyone interacting in the dbt project's codebases, issue trackers, chat rooms, 
and mailing lists is expected to follow the 
[PyPA Code of Conduct](https://www.pypa.io/en/latest/code-of-conduct/).
