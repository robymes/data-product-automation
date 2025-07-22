## Next Steps

1. Create GitHub repository
2. Implement base project structure
3. Set up Kestra flows for orchestration
4. Develop first end-to-end pipeline
5. Add monitoring and alerting
6. Iterate adding complexity progressively

Initial focus: **Get a simple flow working: Kestra triggers → dlt → SQLMesh → ODCS → OpenMetadata**# Data Product Automation Demo - SQLMesh + dlt + OpenMetadata + ODCS + Kestra

## Project Goal

Create a working demo that demonstrates automated integration between:
- **dlt**: for data ingestion into the landing zone
- **SQLMesh**: for transformations and lineage
- **ODCS**: for automatic data contract generation
- **OpenMetadata**: for cataloging and governance
- **Kestra**: for orchestrating the entire data pipeline workflow

## Target Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Source Data   │────▶│     dlt      │────▶│  Landing Zone   │
│  (CSV/JSON/API) │     │              │     │   (DuckDB)      │
└─────────────────┘     └──────────────┘     └────────┬────────┘
         ▲                      ▲                      │
         │                      │                      ▼
         │              ┌───────┴──────┐     ┌─────────────────┐
         │              │    Kestra    │────▶│    SQLMesh      │
         │              │(Orchestrator)│     │ (Transformations)│
         │              └───────┬──────┘     └────────┬────────┘
         │                      │                      │
         │                      ▼                      ▼
┌─────────┴───────┐     ┌──────────────┐     ┌─────────────────┐
│ OpenMetadata    │◀────│ ODCS Contract│◀────│  ODCS Generator │
│   (Catalog)     │     │    Storage   │     │                 │
└─────────────────┘     └──────────────┘     └─────────────────┘
```

## Project Structure

```
data-product-automation/
├── docker-compose.yml
├── .env
├── README.md
├── claude.md
├── requirements.txt
├── Makefile
│
├── config/
│   ├── sqlmesh_config.yaml
│   ├── openmetadata.yaml
│   └── kestra_application.yaml
│
├── src/
│   ├── __init__.py
│   ├── orchestrator/
│   │   ├── __init__.py
│   │   └── main.py
│   │
│   ├── pipelines/
│   │   ├── __init__.py
│   │   └── sample_pipeline.py
│   │
│   ├── contracts/
│   │   ├── __init__.py
│   │   └── generator.py
│   │
│   └── integrations/
│       ├── __init__.py
│       ├── sqlmesh_client.py
│       └── openmetadata_client.py
│
├── kestra/
│   ├── flows/
│   │   ├── data_pipeline_orchestration.yml
│   │   └── monitoring_flow.yml
│   └── scripts/
│       ├── run_dlt.py
│       └── generate_contracts.py
│
├── sqlmesh_project/
│   ├── config.yaml
│   ├── models/
│   │   └── staging/
│   │       └── stg_customers.sql
│   └── audits/
│       └── data_quality.yaml
│
├── data/
│   └── sample/
│       ├── customers.csv
│       └── orders.json
│
└── tests/
    ├── __init__.py
    └── test_integration.py
```

## Implementation Steps

### 1. Initial Project Setup

Create the base project structure with:
- `docker-compose.yml` with Kestra, OpenMetadata, PostgreSQL, and necessary services
- `requirements.txt` with all Python dependencies
- `Makefile` for utility commands
- `.env` for environment variables

### 2. Configure Kestra Orchestrator

Set up Kestra as the main orchestrator:
- Create `kestra/flows/data_pipeline_orchestration.yml` for main workflow
- Configure Kestra to trigger dlt pipelines
- Set up monitoring and alerting flows
- Configure task dependencies and scheduling

### 3. Implement Sample dlt Pipeline

Create `src/pipelines/sample_pipeline.py`:
- Pipeline that reads from CSV/JSON in `data/sample/`
- Loads data into DuckDB landing zone
- Generates metadata for SQLMesh
- Wrapped in Kestra Python task

### 4. Configure SQLMesh

In `sqlmesh_project/`:
- Create `config.yaml` with DuckDB connection
- Implement transformation models in `models/`
- Define audit rules in `audits/`
- Configure Kestra task for SQLMesh execution

### 5. Implement ODCS Generator

Create `src/contracts/generator.py`:
- Extract metadata from SQLMesh models
- Generate ODCS v3.0.0 contracts
- Save contracts in JSON/YAML format
- Create Kestra task for contract generation

### 6. Integrate with OpenMetadata

In `src/integrations/openmetadata_client.py`:
- Connect to OpenMetadata APIs
- Ingest lineage from SQLMesh
- Upload ODCS contracts
- Configure Kestra webhook for metadata updates

### 7. Create Kestra Flows

Define orchestration in `kestra/flows/`:
- Main pipeline flow with sequential execution
- Error handling and retry logic
- Monitoring and alerting flows
- Schedule configuration (cron/event-based)

### 8. Docker Compose Setup

Configure `docker-compose.yml` with:
- Kestra (server + UI)
- PostgreSQL (for Kestra backend)
- OpenMetadata (server + database)
- Python app container with all components
- Volume mounts for development
- Network configuration
- Resource limits and health checks

## Main Dependencies

```txt
# Core dependencies
dlt[duckdb]>=0.4.0
sqlmesh>=0.50.0
duckdb>=0.9.0

# OpenMetadata SDK
openmetadata-ingestion>=1.3.0

# Utils
pydantic>=2.0
pyyaml>=6.0
httpx>=0.25.0
typer>=0.9.0
rich>=13.0

# Development
pytest>=7.0
black>=23.0
ruff>=0.1.0
```

## Key Configurations

### SQLMesh config.yaml
```yaml
gateways:
  local:
    connection:
      type: duckdb
      database: data/warehouse.db
      
model_defaults:
  dialect: duckdb
  start: 2024-01-01
```

### OpenMetadata Connection
```python
# Use OpenMetadata in development mode
OPENMETADATA_CONFIG = {
    "host": "http://localhost:8585/api",
    "auth_provider": "openmetadata",
    "jwt_token": "eyJ..."  # Generated from UI
}
```

## Demo Workflow

1. **Start infrastructure**: `docker-compose up -d`
2. **Initialize SQLMesh**: `make init-sqlmesh`
3. **Access Kestra UI**: http://localhost:8080
4. **Deploy Kestra flows**: `make deploy-flows`
5. **Trigger pipeline**: Via Kestra UI or `make trigger-pipeline`
6. **Monitor execution**: Watch real-time in Kestra UI
7. **View in OpenMetadata**: http://localhost:8585

## Kestra Flow Example

Here's a sample Kestra flow for orchestrating the data pipeline:

```yaml
id: data_product_pipeline
namespace: demo
description: Automated Data Product Pipeline

inputs:
  - id: source_name
    type: STRING
    defaults: customers
    
triggers:
  - id: daily_schedule
    type: io.kestra.plugin.core.trigger.Schedule
    cron: "0 9 * * *"

tasks:
  - id: ingest_data
    type: io.kestra.plugin.scripts.python.Script
    runner: DOCKER
    docker:
      image: python:3.11-slim
    inputFiles:
      - run_dlt.py
    script: |
      python run_dlt.py --source {{inputs.source_name}}

  - id: transform_data
    type: io.kestra.plugin.sqlmesh.cli.SQLMeshCLI
    commands:
      - sqlmesh plan --auto-apply
      
  - id: generate_contracts
    type: io.kestra.plugin.scripts.python.Script
    runner: DOCKER
    docker:
      image: python:3.11-slim
    script: |
      python /app/src/contracts/generator.py
      
  - id: update_catalog
    type: io.kestra.plugin.core.http.Request
    uri: http://openmetadata:8585/api/v1/lineage
    method: POST
    headers:
      Authorization: "Bearer {{secrets.openmetadata_token}}"
    body: "{{outputs.generate_contracts.files['contract.json']}}"

errors:
  - id: alert_on_failure
    type: io.kestra.plugin.notifications.webhook.WebhookExecution
    url: "{{secrets.slack_webhook}}"
    method: POST
    body:
      text: "Pipeline failed for {{inputs.source_name}}"
```

## Features to Implement

### MVP (Phase 1)
- [ ] Basic dlt pipeline with CSV source
- [ ] 2-3 example SQLMesh models
- [ ] Basic automatic ODCS generation
- [ ] Lineage ingestion in OpenMetadata

### Enhancements (Phase 2)
- [ ] Multiple data sources (API, Database)
- [ ] Complex SQLMesh transformations
- [ ] Data quality rules in ODCS
- [ ] Automated scheduling with Airflow

### Production Ready (Phase 3)
- [ ] PostgreSQL instead of DuckDB
- [ ] Authentication/Authorization
- [ ] Monitoring and alerting
- [ ] CI/CD pipeline

## Technical Notes

### DuckDB vs PostgreSQL
- Start with DuckDB for simplicity
- Structure code for easy switch to PostgreSQL
- Use connection abstraction in SQLMesh and dlt

### Testing Strategy
- Unit tests for each component
- End-to-end integration tests
- Mock OpenMetadata for offline testing

### Development Tips
- Use volume mounts for hot reload
- Enable debug logging for troubleshooting
- Test incrementally each step

## Useful Commands

```bash
# Development
make dev         # Start in development mode
make test        # Run all tests
make lint        # Run linters

# Kestra operations
make kestra-ui   # Open Kestra UI in browser
make deploy-flows # Deploy flows to Kestra
make trigger-flow FLOW=data_product_pipeline # Trigger specific flow

# Pipeline execution
make pipeline    # Run dlt pipeline
make transform   # Run SQLMesh transformations
make contracts   # Generate ODCS contracts
make catalog     # Update OpenMetadata

# Monitoring
make logs SERVICE=kestra  # Show logs for specific service
make health      # Check all services health
make metrics     # View Prometheus metrics

# Utilities
make clean       # Clean all data
make reset       # Reset to initial state
make backup      # Backup all databases
```

## Resources and References

- [Kestra Documentation](https://kestra.io/docs)
- [SQLMesh Documentation](https://sqlmesh.readthedocs.io/)
- [dlt Documentation](https://dlthub.com/docs)
- [OpenMetadata APIs](https://docs.open-metadata.org/developers/apis)
- [ODCS Specification v3.0.0](https://github.com/bitol-io/open-data-contract-standard)
- [Docker Compose Best Practices](https://docs.docker.com/compose/production/)
- [Podman Compose Guide](https://github.com/containers/podman-compose)

## Complete Docker Compose Example

```yaml
version: '3.9'

# Shared configurations
x-common-variables: &common-variables
  TZ: UTC
  LOG_LEVEL: INFO

x-healthcheck-defaults: &healthcheck-defaults
  interval: 30s
  timeout: 10s
  retries: 5
  start_period: 40s

# Networks
networks:
  data-platform:
    driver: bridge
    name: data-platform-network

# Volumes
volumes:
  postgres-data:
    driver: local
  kestra-data:
    driver: local
  openmetadata-data:
    driver: local
  duckdb-data:
    driver: local

services:
  # PostgreSQL for Kestra
  postgres:
    image: postgres:15-alpine
    container_name: postgres
    environment:
      POSTGRES_DB: kestra
      POSTGRES_USER: kestra
      POSTGRES_PASSWORD: k3str4
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - data-platform
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD-SHELL", "pg_isready -d kestra -U kestra"]
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M

  # Kestra Orchestrator
  kestra:
    image: kestra/kestra:latest
    container_name: kestra
    user: "root"
    command: server standalone --worker-thread=128
    environment:
      <<: *common-variables
      KESTRA_CONFIGURATION: |
        datasources:
          postgres:
            url: jdbc:postgresql://postgres:5432/kestra
            driverClassName: org.postgresql.Driver
            username: kestra
            password: k3str4
        kestra:
          server:
            basicAuth:
              enabled: false
              username: "admin@localhost.dev"
              password: kestra
          repository:
            type: postgres
          storage:
            type: local
            local:
              basePath: "/app/storage"
          queue:
            type: postgres
          tasks:
            tmpDir:
              path: /tmp/kestra-wd
    volumes:
      - kestra-data:/app/storage
      - /var/run/docker.sock:/var/run/docker.sock
      - /tmp/kestra-wd:/tmp/kestra-wd
      - ./kestra/flows:/app/flows
      - ./kestra/scripts:/app/scripts
    ports:
      - "8080:8080"
      - "8081:8081"
    networks:
      - data-platform
    depends_on:
      postgres:
        condition: service_healthy
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '2.0'
        reservations:
          memory: 1G
          cpus: '1.0'

  # OpenMetadata
  openmetadata:
    image: openmetadata/server:latest
    container_name: openmetadata
    environment:
      <<: *common-variables
      OPENMETADATA_CONFIG: |
        # OpenMetadata configuration
    volumes:
      - openmetadata-data:/opt/openmetadata
    ports:
      - "8585:8585"
    networks:
      - data-platform
    healthcheck:
      <<: *healthcheck-defaults
      test: ["CMD", "curl", "-f", "http://localhost:8585/healthcheck"]
    deploy:
      resources:
        limits:
          memory: 2G
        reservations:
          memory: 1G

  # Python App Container (for dlt, SQLMesh, ODCS)
  data-processor:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: data-processor
    environment:
      <<: *common-variables
      DUCKDB_PATH: /data/warehouse.db
      SQLMESH_CONFIG: /app/config/sqlmesh_config.yaml
    volumes:
      - ./src:/app/src
      - ./sqlmesh_project:/app/sqlmesh_project
      - ./data:/data
      - duckdb-data:/data/db
    networks:
      - data-platform
    command: ["python", "-m", "src.orchestrator.main"]
    deploy:
      resources:
        limits:
          memory: 1G
        reservations:
          memory: 512M
```

## Docker Compose Optimization Best Practices

### Resource Management

**Memory and CPU Limits**
```yaml
services:
  kestra:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '1.0'
          memory: 1G
```

**Shared Memory for PostgreSQL**
```yaml
services:
  postgres:
    shm_size: 1g  # Important for performance
    command: >
      postgres
      -c shared_buffers=256MB
      -c max_connections=200
      -c effective_cache_size=1GB
```

### Health Checks and Dependencies

**Comprehensive Health Checks**
```yaml
services:
  openmetadata:
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8585/healthcheck"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 40s
    depends_on:
      postgres:
        condition: service_healthy
      elasticsearch:
        condition: service_healthy
```

### Networking Best Practices

**Custom Bridge Network**
```yaml
networks:
  data-platform:
    driver: bridge
    ipam:
      config:
        - subnet: 172.20.0.0/16
    driver_opts:
      com.docker.network.bridge.name: data_platform_br
```

**Service Discovery**
```yaml
services:
  kestra:
    networks:
      - data-platform
    environment:
      KESTRA_CONFIGURATION: |
        datasources:
          postgres:
            url: jdbc:postgresql://postgres:5432/kestra  # Use service name
```

### Volume Optimization

**Named Volumes with Drivers**
```yaml
volumes:
  postgres-data:
    driver: local
    driver_opts:
      type: none
      o: bind
      device: ./data/postgres  # Bind to specific host directory
      
  kestra-storage:
    driver: local
    driver_opts:
      type: tmpfs  # For temporary data - RAM-based
      device: tmpfs
      o: size=1g,mode=1777
```

### Logging Configuration

**Centralized Logging**
```yaml
x-logging: &default-logging
  driver: json-file
  options:
    max-size: "10m"
    max-file: "3"
    labels: "service"

services:
  kestra:
    logging: *default-logging
```

### Environment Variables Management

**Using .env Files**
```yaml
services:
  openmetadata:
    env_file:
      - .env.shared
      - .env.openmetadata
    environment:
      - OVERRIDE_VAR=value  # Overrides .env file
```

### Build Optimization

**Multi-stage Builds with Cache**
```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      cache_from:
        - registry.example.com/app:cache
      args:
        BUILDKIT_INLINE_CACHE: 1
```

### Container Restart Policies

**Smart Restart Configuration**
```yaml
services:
  kestra:
    restart: unless-stopped  # Restarts unless manually stopped
    
  postgres:
    restart: always  # Critical service - always restart
    
  monitoring:
    restart: on-failure:3  # Retry 3 times on failure
```

### Security Best Practices

**Read-only Containers**
```yaml
services:
  app:
    read_only: true
    tmpfs:
      - /tmp
      - /var/run
    security_opt:
      - no-new-privileges:true
```

**User Namespace Remapping**
```yaml
services:
  kestra:
    user: "1000:1000"  # Non-root user
    userns_mode: "host"
```

### Performance Tuning

**Connection Pooling**
```yaml
services:
  app:
    environment:
      - DB_POOL_SIZE=20
      - DB_POOL_TIMEOUT=30
      - DB_POOL_RECYCLE=3600
```

**Optimized Compose File Structure**
```yaml
version: '3.9'

x-common-variables: &common-variables
  TZ: UTC
  LOG_LEVEL: INFO

x-healthcheck-defaults: &healthcheck-defaults
  interval: 30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

### Monitoring and Observability

**Prometheus Metrics**
```yaml
services:
  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
```

### Backup Strategy

**Automated Backups**
```yaml
services:
  backup:
    image: postgres:15-alpine
    volumes:
      - ./backups:/backups
      - postgres-data:/data:ro
    command: >
      sh -c "while true; do
        PGPASSWORD=$POSTGRES_PASSWORD pg_dump -h postgres -U $POSTGRES_USER -d $POSTGRES_DB > /backups/backup_$(date +%Y%m%d_%H%M%S).sql;
        find /backups -name 'backup_*.sql' -mtime +7 -delete;
        sleep 86400;
      done"
```

### Development vs Production

**Override Files**
```bash
# Development
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

**Production Overrides (docker-compose.prod.yml)**
```yaml
services:
  kestra:
    restart: always
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
        failure_action: rollback
```