# AWS LocalStack Development Environment

A docker-compose based local development environment for AWS services with Apache Airflow, Spark, Trino, Prometheus, and Grafana.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     docker-compose.yml                      │
├─────────────────────────────────────────────────────────────┤
│  LocalStack  │  Airflow  │  Spark  │  Trino  │ Prometheus  │
│  (S3, Lambda │  (Web,    │  (Master│  (Query │  (Metrics   │
│   Glue,      │  Sched,   │   Worker│  Engine)│   Store)    │
│   Athena)    │  Worker)  │         │         │             │
├─────────────────────────────────────────────────────────────┤
│         PostgreSQL          │         Redis                │
│       (Airflow DB)          │    (Celery Broker)            │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
                    Grafana (Dashboards)
```

## Services

| Service | Port | Description |
|---------|------|-------------|
| LocalStack | 4566 | AWS S3, Lambda, Glue, Athena, etc. |
| Airflow Webserver | 8080 | Airflow UI |
| Spark Master | 7077 | Spark cluster master |
| Spark UI | 8081 | Spark web interface |
| Trino | 8020 | SQL query engine |
| Prometheus | 9090 | Metrics collection |
| Grafana | 3000 | Visualization dashboards |

## Files Overview

### docker-compose.yml
The main orchestration file that defines all services.

**Benefits:**
- Single command to start all services (`docker compose up -d`)
- Health checks for reliable startup
- Volume persistence for data across restarts
- Environment variable based configuration
- Service dependencies handled automatically

**Limitations:**
- Resource intensive (multiple containers)
- Requires Docker with sufficient memory (8GB+ recommended)
- Some services may fail if LocalStack is slow to start

---

### .env
Environment configuration file for AWS credentials and LocalStack auth.

**Benefits:**
- Centralized configuration
- Keeps sensitive credentials out of docker-compose.yml
- Easy to modify without changing docker-compose.yml

**Limitations:**
- Contains a hardcoded auth token that should be rotated
- AWS credentials are test-only (not for production use)
- Must be sourced or docker-compose will fail

---

### prometheus/prometheus.yml
Prometheus configuration for metrics scraping.

**Benefits:**
- Scrapes LocalStack metrics (custom metrics endpoint)
- Includes Prometheus self-monitoring
- Easy to add new scrape targets

**Limitations:**
- Static configs only (no service discovery)
- No alerting configured
- 15s scrape interval may be too frequent for some use cases

---

### trino/catalog/hive-properties
Trino (formerly PrestoSQL) Hive connector configuration.

**Benefits:**
- Enables SQL queries over S3 data via LocalStack
- Uses AWS Glue as metastore (simulated by LocalStack)
- Configured for path-style access (S3 compatibility)

**Limitations:**
- Depends on LocalStack Glue being available
- Limited to Hive metastore (no Delta Lake, Iceberg)
- Test credentials hardcoded

---

### spark/conf/spark-defaults.conf
Spark configuration for S3A access to LocalStack S3.

**Benefits:**
- Enables Spark to read/write S3 data
- Uses S3A connector (better performance)
- SimpleAWSCredentialsProvider for test credentials
- Path-style access for LocalStack compatibility

**Limitations:**
- Credentials in config file (should use secrets in production)
- Only configured for S3 (not other AWS services)
- No HA/HA support for S3A

---

## Quick Start

```bash
# Start all services
mkdir -p airflow_dags airflow_logs
sudo chown -R 50000:0 airflow_logs airflow_dags
sudo chmod -R 775 airflow_logs airflow_dags

docker compose up -d

# Create Airflow admin user (one-time)
docker exec -it airflow-webserver airflow users create \
  --username admin \
  --firstname Admin \
  --lastname User \
  --role Admin \
  --email admin@example.com \
  --password admin

# Check service health
docker compose ps

# View logs
docker compose logs -f

# Stop all services
docker compose down


```

## Accessing Services

| Service | URL |
|---------|-----|
| Airflow | http://localhost:8080 |
| Spark UI | http://localhost:8081 |
| Trino | http://localhost:8082 |
| Prometheus | http://localhost:9090 |
| Grafana | http://localhost:3000 (admin/admin) |
| LocalStack | http://localhost:4566 |

## AWS CLI with LocalStack

```bash
# Configure AWS CLI
aws configure
# Access Key: test
# Secret Key: test
# Region: us-east-1

# Or use environment variables
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
export AWS_ENDPOINT_URL=http://localhost:4566

# List S3 buckets
aws s3 ls --endpoint-url=http://localhost:4566
```

## Stopping and Cleaning Up

```bash
# Stop services (preserves data)
docker compose down

# Stop and remove volumes (destroys all data)
docker compose down -v

# Remove all images
docker compose down --rmi all
```

## Version Information

- LocalStack: latest
- Apache Airflow: 2.10.3
- Apache Spark: 3.5.1
- Trino: latest
- PostgreSQL: 16
- Redis: 7
- Prometheus: v2.53.0
- Grafana: 11.0.0
