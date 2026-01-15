```instructions
---
description: "Apache Airflow DAG development conventions and best practices"
applyTo: "**/*airflow*.py, **/dags/**/*.py"
---

# Apache Airflow Conventions

## Airflow Version & Features

- Use Airflow 2.10+ features and syntax.
- Ensure that DAGs follow Airflow best practices and naming conventions.
- Ensure that DAGs do not use top-level code execution or side effects.
- Use Airflow's built-in operators and sensors where applicable.
- Structure DAGs for clarity and maintainability.
- Use Airflow's templating features for dynamic task parameters.
- Use Airflow's logging features to capture task logs.
- Ensure that DAGs are idempotent and can be retried without side effects.
- Use Airflow's built-in testing framework for unit tests of DAGs.

## DAG Structure Best Practices

- **Avoid top-level execution**: Keep all DAG logic inside the DAG context manager or callables
- **Use Airflow 2.0+ decorator syntax** where possible for clarity
- **Separate concerns**: Keep DAG definitions separate from task logic
- **Meaningful DAG IDs**: Use descriptive names reflecting purpose and frequency
- **Documentation**: Add descriptions to DAGs and tasks using `doc_md` or docstrings

## Testing DAGs

- Use Airflow's `pytest` integration for testing
- Mock external dependencies and API calls
- Test task logic independently before DAG integration
- Validate DAG structure and dependencies
- Test error handling and retries

```
