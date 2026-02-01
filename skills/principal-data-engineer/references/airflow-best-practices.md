# Airflow DAG Best Practices

This reference provides Apache Airflow DAG development conventions and anti-patterns for data engineering.

## Version & Core Principles

- Use Airflow 2.10+ features and syntax
- Ensure DAGs follow best practices and naming conventions
- Avoid top-level code execution or side effects
- Structure DAGs for clarity and maintainability
- Ensure DAGs are idempotent and can be retried without side effects

## Critical Anti-Patterns to Avoid

### ❌ Top-Level Code Execution

```python
# BAD: Code runs on scheduler import
import pandas as pd

df = pd.read_csv("data.csv")  # Executed on every scheduler loop!

@dag(...)
def my_dag():
    ...
```

```python
# GOOD: Code only runs during task execution
@dag(...)
def my_dag():
    @task
    def load_data():
        import pandas as pd
        df = pd.read_csv("data.csv")  # Only runs during task execution
        return df
```

### ❌ Non-Idempotent Operations

```python
# BAD: Re-running creates duplicates
@task
def load_data():
    df = get_data()
    df.to_sql("my_table", engine, if_exists="append")  # Duplicates on retry!
```

```python
# GOOD: Idempotent with proper handling
@task
def load_data():
    df = get_data()
    # Option 1: Replace entire partition
    df.to_sql("my_table", engine, if_exists="replace")

    # Option 2: Upsert with unique constraint
    upsert_records(df, table="my_table", unique_cols=["id"])
```

## DAG Structure Best Practices

### Use TaskFlow API (Airflow 2.0+)

```python
from airflow.decorators import dag, task
from datetime import datetime

@dag(
    dag_id='my_pipeline',
    start_date=datetime(2025, 1, 1),
    schedule='@daily',
    catchup=False,
    default_args={'owner': 'data-team', 'retries': 2},
    tags=['etl', 'production'],
    doc_md="""
    # My Pipeline

    **Purpose**: Extract, transform, and load customer data
    **Owner**: data-team
    **Schedule**: Daily at midnight UTC
    """
)
def my_pipeline():
    @task
    def extract():
        """Extract data from source."""
        return {"data": [1, 2, 3]}

    @task
    def transform(data: dict):
        """Transform and clean data."""
        return [x * 2 for x in data["data"]]

    @task
    def load(transformed: list):
        """Load data to warehouse."""
        print(f"Loaded {len(transformed)} records")

    # Define task dependencies
    load(transform(extract()))

my_pipeline()
```

### Meaningful Naming

```python
# GOOD: Descriptive DAG IDs
dag_id='customer_data_daily_etl'
dag_id='marketing_metrics_hourly_aggregation'
dag_id='fraud_detection_realtime_pipeline'

# BAD: Generic names
dag_id='pipeline'
dag_id='dag1'
dag_id='test'
```

### Documentation

```python
@dag(
    dag_id='revenue_reporting',
    doc_md="""
    ## Revenue Reporting Pipeline

    Aggregates daily revenue metrics from multiple sources.

    **Data Sources**:
    - Stripe API (payments)
    - Salesforce (opportunities)
    - Internal billing database

    **Output**: `analytics.revenue_daily` table

    **Dependencies**: Requires Stripe and Salesforce connections
    """,
)
def revenue_reporting():
    @task(doc_md="Fetch payments from Stripe API")
    def get_stripe_data():
        ...
```

## Configuration Management

### Use Connections & Variables

```python
from airflow.models import Variable
from airflow.hooks.base import BaseHook

@task
def extract_from_api():
    # Retrieve connection from Airflow
    conn = BaseHook.get_connection('my_api')
    api_key = conn.password
    base_url = conn.host

    # Use Variables for configuration
    batch_size = Variable.get("batch_size", default_var=1000)

    # Extract data...
```

### Environment-Specific Configuration

```python
@dag(...)
def my_dag():
    @task
    def process():
        env = Variable.get("environment", default_var="dev")

        if env == "prod":
            table = "prod.analytics.metrics"
        else:
            table = "dev.analytics.metrics"
```

## Dependency Management

### Task Dependencies

```python
# Explicit dependency chaining
@dag(...)
def pipeline():
    t1 = extract()
    t2 = transform(t1)
    t3 = validate(t2)
    t4 = load(t3)

    # Or using bit-shift operators (classic operators)
    # t1 >> t2 >> t3 >> t4
```

### Parallel Execution

```python
@dag(...)
def parallel_pipeline():
    @task
    def extract_source_a():
        ...

    @task
    def extract_source_b():
        ...

    @task
    def merge(data_a, data_b):
        ...

    # Both extracts run in parallel
    data_a = extract_source_a()
    data_b = extract_source_b()
    merge(data_a, data_b)
```

## Testing

### Unit Testing

```python
import pytest
from airflow.models import DagBag

def test_dag_loaded():
    """Test DAG loads without errors."""
    dagbag = DagBag(dag_folder="dags/", include_examples=False)
    assert len(dagbag.import_errors) == 0

def test_dag_structure():
    """Test DAG has expected tasks."""
    dagbag = DagBag(dag_folder="dags/")
    dag = dagbag.get_dag("my_pipeline")

    assert dag is not None
    assert len(dag.tasks) == 3
    assert "extract" in dag.task_ids
```

### Integration Testing

```python
@pytest.fixture
def dag_test_context():
    """Create test context with required connections."""
    from airflow.models import Connection
    conn = Connection(
        conn_id="test_api",
        conn_type="http",
        host="https://test.api.com"
    )
    return {"conn": conn}

def test_extract_task(dag_test_context):
    """Test extract task logic."""
    from my_dags.pipeline import extract
    result = extract()
    assert result is not None
```

## Data Quality Patterns

### Pre-conditions & Post-conditions

```python
@task
def validate_input(data):
    """Validate input data quality."""
    assert len(data) > 0, "Input data is empty"
    assert all("id" in row for row in data), "Missing required 'id' field"
    return data

@task
def transform(data):
    """Transform data."""
    # Transformation logic
    return transformed_data

@task
def validate_output(data):
    """Validate output data quality."""
    assert len(data) > 0, "Output data is empty"
    assert all("timestamp" in row for row in data), "Missing timestamp"
    return data

@dag(...)
def quality_pipeline():
    raw = extract()
    validated = validate_input(raw)
    transformed = transform(validated)
    final = validate_output(transformed)
    load(final)
```

### Using Soda for Data Quality

```python
@task
def run_soda_checks():
    """Run data quality checks with Soda."""
    from soda.scan import Scan

    scan = Scan()
    scan.set_data_source_name("my_warehouse")
    scan.add_sodacl_yaml_file("checks/my_table_checks.yml")
    scan.execute()

    if scan.has_check_fails():
        raise ValueError(f"Data quality checks failed: {scan.get_logs_text()}")
```

## Performance Optimization

### Partition Awareness

```python
@task
def process_partition(partition_date):
    """Process single partition for efficiency."""
    query = f"""
        SELECT * FROM raw_data
        WHERE date = '{partition_date}'
    """
    # Process only the partition needed
```

### Dynamic Task Mapping

```python
from datetime import datetime, timedelta

@dag(...)
def backfill_pipeline():
    @task
    def generate_dates():
        """Generate list of dates to process."""
        dates = []
        for i in range(30):
            date = (datetime.now() - timedelta(days=i)).strftime("%Y-%m-%d")
            dates.append(date)
        return dates

    @task
    def process_date(date):
        """Process single date partition."""
        # Process logic for date
        pass

    # Dynamic mapping: creates one task per date
    dates = generate_dates()
    process_date.expand(date=dates)
```

## Monitoring & Observability

### Logging Best Practices

```python
@task
def process_with_logging():
    """Process data with comprehensive logging."""
    import logging
    logger = logging.getLogger(__name__)

    logger.info("Starting data processing")

    try:
        data = fetch_data()
        logger.info(f"Fetched {len(data)} records")

        processed = transform(data)
        logger.info(f"Processed {len(processed)} records")

        return processed
    except Exception as e:
        logger.error(f"Processing failed: {str(e)}")
        raise
```

### SLAs and Alerting

```python
@dag(
    default_args={
        'sla': timedelta(hours=2),  # Alert if task takes > 2 hours
        'email': ['data-team@company.com'],
        'email_on_failure': True,
        'email_on_retry': False,
    }
)
def monitored_pipeline():
    ...
```

## Common Pitfalls

### ❌ Using Dynamic DAG Generation Without Care

```python
# BAD: Creates too many DAGs
for customer in get_all_customers():  # 1000+ customers!
    @dag(dag_id=f'customer_{customer}_pipeline')
    def customer_pipeline():
        ...
```

```python
# GOOD: Use dynamic task mapping instead
@dag(dag_id='all_customers_pipeline')
def customer_pipeline():
    @task
    def get_customers():
        return list_customers()

    @task
    def process_customer(customer):
        # Process single customer
        pass

    customers = get_customers()
    process_customer.expand(customer=customers)
```

### ❌ Accessing Metadata Database Directly

```python
# BAD: Direct database access
from airflow.settings import Session
session = Session()
# Query metadata tables directly
```

```python
# GOOD: Use Airflow's public API
from airflow.models import DagRun
dag_runs = DagRun.find(dag_id="my_dag", state="failed")
```

### ❌ Storing Sensitive Data in Code

```python
# BAD: Hardcoded credentials
API_KEY = "sk-1234567890abcdef"

# GOOD: Use Airflow connections
conn = BaseHook.get_connection('my_api')
api_key = conn.password
```

## References

- [Airflow Best Practices](https://airflow.apache.org/docs/apache-airflow/stable/best-practices.html)
- [TaskFlow API Documentation](https://airflow.apache.org/docs/apache-airflow/stable/tutorial/taskflow.html)
- [Astronomer Best Practices](https://www.astronomer.io/docs/learn/airflow-best-practices)
