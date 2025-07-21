# Data Product Automation Demo - SQLMesh + dlt + OpenMetadata + ODCS

## Project Goal

Create a working demo that demonstrates automated integration between:
- **dlt**: for data ingestion into the landing zone
- **SQLMesh**: for transformations and lineage
- **ODCS**: for automatic data contract generation
- **OpenMetadata**: for cataloging and governance

## Target Architecture

```
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│   Source Data   │────▶│     dlt      │────▶│  Landing Zone   │
│  (CSV/JSON/API) │     │              │     │   (DuckDB)      │
└─────────────────┘     └──────────────┘     └────────┬────────┘
                                                       │
                                                       ▼
┌─────────────────┐     ┌──────────────┐     ┌─────────────────┐
│ OpenMetadata    │◀────│ ODCS Contract│◀────│    SQLMesh      │
│   (Catalog)     │     │  Generator   │     │ (Transformations)│
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
│   └── openmetadata.yaml
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
- `docker-compose.yml` with OpenMetadata and necessary services
- `requirements.txt` with all Python dependencies
- `Makefile` for utility commands
- `.env` for environment variables

### 2. Implement Sample dlt Pipeline

Create `src/pipelines/sample_pipeline.py`:
- Pipeline that reads from CSV/JSON in `data/sample/`
- Loads data into DuckDB landing zone
- Generates metadata for SQLMesh

### 3. Configure SQLMesh

In `sqlmesh_project/`:
- Create `config.yaml` with DuckDB connection
- Implement transformation models in `models/`
- Define audit rules in `audits/`

### 4. Implement ODCS Generator

Create `src/contracts/generator.py`:
- Extract metadata from SQLMesh models
- Generate ODCS v3.0.0 contracts
- Save contracts in JSON/YAML format

### 5. Integrate with OpenMetadata

In `src/integrations/openmetadata_client.py`:
- Connect to OpenMetadata APIs
- Ingest lineage from SQLMesh
- Upload ODCS contracts

### 6. Create Main Orchestrator

In `src/orchestrator/main.py`:
- Coordinate execution of all components
- Handle error handling and retry logic
- Provide REST API for manual triggers

### 7. Docker Compose Setup

Configure `docker-compose.yml` with:
- OpenMetadata (server + database)
- Python app container with all components
- Volume mounts for development
- Network configuration

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
3. **Run dlt pipeline**: `make run-pipeline`
4. **Generate contracts**: `make generate-contracts`
5. **View in OpenMetadata**: http://localhost:8585

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

# Pipeline execution
make pipeline    # Run dlt pipeline
make transform   # Run SQLMesh transformations
make contracts   # Generate ODCS contracts
make catalog     # Update OpenMetadata

# Utilities
make clean       # Clean all data
make reset       # Reset to initial state
make logs        # Show all logs
```

## Resources and References

- [SQLMesh Documentation](https://sqlmesh.readthedocs.io/)
- [dlt Documentation](https://dlthub.com/docs)
- [OpenMetadata APIs](https://docs.open-metadata.org/developers/apis)
- [ODCS Specification v3.0.0](https://github.com/bitol-io/open-data-contract-standard)

## Next Steps

1. Create GitHub repository
2. Implement base project structure
3. Develop first end-to-end pipeline
4. Iterate adding complexity progressively

Initial focus: **Get a simple flow working: CSV → dlt → SQLMesh → ODCS → OpenMetadata**