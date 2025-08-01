<div align="center">
  <img src="assets/porter-logo.svg" alt="Porter Logo" width="150" height="150">
  
# Porter
  
*Zero‑copy analytics, delivered at Mach Arrow.*

[![Go Report Card](https://goreportcard.com/badge/github.com/TFMV/porter)](https://goreportcard.com/report/github.com/TFMV/porter)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Build and Test](https://github.com/TFMV/porter/actions/workflows/ci.yml/badge.svg)](https://github.com/TFMV/porter/actions/workflows/ci.yml)

</div>

> **Status: Experimental**  
> Porter is under **active development** and is stable for development and experimental use cases.

---

> **Seeking Testers & Contributors**  
> Porter is actively evolving, and I'm looking for developers willing to spend a few hours testing any use cases you might have in mind.  
> If you hit a bug, open an issue or send a PR. Otherwise, feel free to assign it to me.  
> Your feedback will shape the future of fast, simple, open SQL over Arrow Flight.
> 
> **Ready to contribute?** Check out our [Contributing Guidelines](CONTRIBUTING.md) for community standards, development setup, and technical details.

---

## What is Porter?

Porter is a **high-performance Flight SQL server** that makes Arrow-native analytics effortless. It provides a flexible abstraction layer for different database backends, supporting both **DuckDB** and **ClickHouse** with a unified Flight SQL interface. It bridges the gap between analytical databases and modern data infrastructure by speaking the Flight SQL protocol natively.

Think of it as **analytics with wings** – all the analytical power you love, now network-accessible with zero-copy Arrow streaming.

## Key Features

### Blazing Fast
- **Zero-copy Arrow streaming** for maximum throughput
- **Connection pooling** and resource optimization
- **In-memory caching** with LRU eviction
- **Concurrent query execution** with transaction support

### Developer Friendly
- **Single binary** deployment
- **Configuration via YAML** or environment variables  
- **Docker support** with optimized images
- **Comprehensive CLI** with intuitive commands
- **Extensive middleware** support for customization

### Standards Compliant
- **Apache Arrow Flight SQL** protocol
- **JDBC/ODBC compatibility** through Flight SQL drivers
- **SQL standard compliance** via database backends
- **Cross-platform** support (Linux, macOS, Windows)

### Built-in Benchmarking
- **TPC-H benchmark suite** with DuckDB's native TPC-H module
- **Multiple output formats** (table, JSON, Arrow)
- **Configurable scale factors** and iterations
- **Query plan analysis** and performance metrics
- **Zero setup** – no external data files required

## Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Flight SQL    │    │     Porter       │    │    Database     │
│    Clients      │◄──►│   Flight SQL     │◄──►│    Backends     │
│                 │    │     Server       │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
        │                        │                        │
        │                        │                        │
    ┌───▼────┐              ┌────▼────┐              ┌────▼────┐
    │ Arrow  │              │ Metrics │              │ Schema  │
    │Streamng│              │Promtheus│              │ Cache   │
    └────────┘              └─────────┘              └─────────┘
```

**Core Components:**
- **Flight SQL Handler** – Protocol implementation and query routing
- **Database Backends** – Flexible abstraction supporting DuckDB and ClickHouse
- **Connection Pool** – Efficient database connection management with backend-specific optimizations
- **Cache Layer** – Smart caching for schemas and query results
- **Metrics Collection** – Comprehensive observability and monitoring
- **Middleware Stack** – Authentication, logging, and custom extensions

## Quick Start

### Prerequisites
- Go 1.24+ (for building from source)
- DuckDB (bundled with Go bindings)
- ClickHouse server (optional, for ClickHouse backend)

### Installation

#### Option 1: Build from Source
```bash
git clone https://github.com/TFMV/porter.git
cd porter
make build
```

#### Option 2: Using Go Install
```bash
go install github.com/TFMV/porter/cmd/server@latest
```

### Running Porter

#### Basic Usage
```bash
# Start with in-memory DuckDB database
./porter serve

# Start with persistent DuckDB database
./porter serve --database ./my-data.db --address 0.0.0.0:32010

# Start with ClickHouse backend
./porter serve --database "clickhouse://localhost:9000/default"

# Start with configuration file
./porter serve --config ./config.yaml
```

#### Using Docker
```bash
# Build the image
make docker-build

# Run the container
make docker-run

# Or run directly
docker run -p 32010:32010 -p 9090:9090 porter:latest
```

### Configuration

Create a `config.yaml` file:

```yaml
server:
  address: "0.0.0.0:32010"
  max_connections: 100
  connection_timeout: "30s"
  query_timeout: "5m"
  max_message_size: 16777216  # 16MB

database:
  # DuckDB backend (default)
  dsn: "duckdb://./data.db"
  # ClickHouse backend
  # dsn: "clickhouse://localhost:9000/default?username=default&password="
  max_open_conns: 32
  max_idle_conns: 8
  conn_max_lifetime: "1h"
  health_check_period: "30s"

logging:
  level: "info"
  format: "json"

metrics:
  enabled: true
  address: ":9090"

tls:
  enabled: false
  cert_file: ""
  key_file: ""

auth:
  enabled: false
  type: "oauth2"
```

## Supported Database Backends

Porter provides a unified Flight SQL interface across multiple database backends:

### DuckDB Backend
- **Connection String**: `duckdb://path/to/database.db` or `duckdb://:memory:`
- **Features**: Full analytical SQL, columnar storage, embedded deployment
- **Use Cases**: Development, analytics workloads, embedded applications
- **Performance**: Excellent for OLAP queries, TPC-H benchmarks

### MotherDuck Backend
- **Connection String**: `duckdb://motherduck/<db>`
- **Token Source**: `--token` flag or `MOTHERDUCK_TOKEN` environment variable
- **Features**: Hosted DuckDB with cloud execution

### ClickHouse Backend
- **Connection String**: `clickhouse://host:port/database?username=user&password=pass`
- **Features**: Distributed analytics, real-time ingestion, horizontal scaling
- **Use Cases**: Large-scale analytics, time-series data, real-time dashboards
- **Performance**: Optimized for high-throughput analytical workloads

### Backend Selection
The backend is automatically detected from the DSN connection string:

```yaml
database:
  # DuckDB (embedded)
  dsn: "duckdb://./analytics.db"

  # MotherDuck (cloud)
  dsn: "duckdb://motherduck/my_db"

  # ClickHouse (distributed)
  dsn: "clickhouse://localhost:9000/analytics?username=default"
```

Both backends support:
- ✅ **Full Flight SQL protocol** compliance
- ✅ **Arrow-native streaming** for zero-copy data transfer
- ✅ **Connection pooling** and resource management
- ✅ **Transaction support** and prepared statements
- ✅ **Schema discovery** and metadata operations

## Connecting to Porter

### Python (using ADBC)
```python
import adbc_driver_flightsql.dbapi

# Connect to Porter
conn = adbc_driver_flightsql.dbapi.connect(
    "grpc://localhost:32010"
)

# Execute queries
cursor = conn.cursor()
cursor.execute("SELECT * FROM my_table LIMIT 10")
results = cursor.fetchall()
```

### Python (using PyArrow)
```python
from pyarrow import flight

# Connect to Porter Flight SQL server
client = flight.FlightClient("grpc://localhost:32010")

# Execute query and get results as Arrow table
info = client.get_flight_info(
    flight.FlightDescriptor.for_command(b"SELECT * FROM my_table")
)
table = client.do_get(info.endpoints[0].ticket).read_all()
```

### Python (using Polars)
```python
import adbc_driver_flightsql.dbapi
import pyarrow as pa
import polars as pl

# Connect and run a query
conn = adbc_driver_flightsql.dbapi.connect("grpc://localhost:32010")
cursor = conn.cursor()
cursor.execute("SELECT 1 AS id, 'example' AS label")

# Collect record batches and convert to Polars
batches = []
batch = cursor.fetch_record_batch()
while batch is not None:
    batches.append(batch)
    batch = cursor.fetch_record_batch()

arrow_table = pa.Table.from_batches(batches)
df = pl.from_arrow(arrow_table)
print(df)
cursor.close()
conn.close()
```

### Java (using Arrow Flight SQL JDBC)
```java
String url = "jdbc:arrow-flight-sql://localhost:32010";
Connection conn = DriverManager.getConnection(url);

Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM my_table");

while (rs.next()) {
    System.out.println(rs.getString(1));
}
```

## Benchmarking

Porter includes a comprehensive TPC-H benchmarking suite that uses DuckDB's built-in TPC-H module for reproducible performance testing.

### Quick Benchmark
```bash
# Run a single query with small dataset
./porter bench --query q1 --scale 0.01

# Run multiple queries with medium dataset  
./porter bench --query q1,q3,q5 --scale 0.1 --iterations 3

# Run all queries with full dataset
./porter bench --all --scale 1 --format json --output results.json
```

### Example Output
```
TPC-H Benchmark Results
=======================

Configuration:
  Scale Factor: 0.01
  Iterations: 1
  Total Time: 120.5ms

Environment:
  DuckDB Version: v1.3.1
  Platform: darwin/arm64

Results:
Query  Iter Time         Rows     Status
-----  ---- ----         ----     ------
q1     1    2.952ms      4        OK
q6     1    0.562ms      1        OK
q10    1    3.665ms      20       OK
```

### Benchmark Analysis
Use the included Python script to analyze results:

```bash
# Analyze single benchmark
python3 examples/benchmark_analysis.py results.json

# Compare two benchmark runs
python3 examples/benchmark_analysis.py --compare baseline.json current.json
```

For detailed benchmarking documentation, see [docs/benchmarking.md](docs/benchmarking.md).

### Microbenchmarks
Porter also ships a small suite of microbenchmarks under `bench/` for
quick performance checks. First generate a dataset:

```bash
go run ./bench/gen_data.go --rows=100000 --cols=5 --format=parquet --out=test.parquet
```

Run the benchmarks with:

```bash
make bench
```

Results are printed in standard Go benchmark format and can be formatted as
CSV or Markdown using the helpers in `bench/report.go`.

## Development

### Building
```bash
# Standard build
make build

# Optimized build (smaller binary)
make optimize

# Show binary size and dependencies
make size
```

### Testing
```bash
# Run all tests
make test

# Run with verbose output
go test -v ./...

# Run specific package tests
go test -v ./pkg/handlers
```

### Development Server
```bash
# Run with hot reload (requires air)
air

# Or run directly
make run
```

## Monitoring & Observability

### Metrics
Porter exposes Prometheus metrics on `:9090` by default:

- **Query Performance**: Execution time, throughput, error rates
- **Connection Pool**: Active connections, wait times, health status  
- **Cache Performance**: Hit/miss ratios, eviction rates
- **Resource Usage**: Memory allocation, GC statistics
- **Flight SQL Operations**: RPC call counts, streaming metrics

### Health Checks
```bash
# Check server health
curl http://localhost:9090/health

# Prometheus metrics
curl http://localhost:9090/metrics
```

### Logging
Structured logging with configurable levels:
```bash
# Set log level
export PORTER_LOG_LEVEL=debug

# JSON format (default)
export PORTER_LOG_FORMAT=json

# Human-readable format
export PORTER_LOG_FORMAT=console
```

## Advanced Configuration

### Environment Variables
All configuration options can be set via environment variables with the `PORTER_` prefix:

```bash
export PORTER_ADDRESS="0.0.0.0:32010"
export PORTER_DATABASE=":memory:"
export PORTER_LOG_LEVEL="debug"
export PORTER_METRICS_ENABLED="true"
export PORTER_MAX_CONNECTIONS="100"
export MOTHERDUCK_TOKEN=your_token
```

### Command Line Options
```bash
./porter serve --help

# Key options:
--address string              Server listen address (default "0.0.0.0:32010")
--database string             DuckDB database path (default ":memory:")
--config string               Configuration file path
--log-level string            Log level: debug, info, warn, error (default "info")
--max-connections int         Maximum concurrent connections (default 100)
--metrics                     Enable Prometheus metrics (default true)
--metrics-address string      Metrics server address (default ":9090")
--tls                         Enable TLS encryption
--auth                        Enable authentication
```

## Contributing

We welcome contributions! Porter is designed to be extensible and community-driven.

### Areas for Contribution
- **Performance optimizations** and benchmarking
- **Additional authentication providers** (JWT, LDAP, etc.)
- **Enhanced monitoring** and alerting capabilities  
- **Client libraries** and integrations
- **Documentation** and examples
- **Testing** and quality assurance

### Development Setup
```bash
# Clone the repository
git clone https://github.com/TFMV/porter.git
cd porter

# Install dependencies
go mod download

# Run tests
make test

# Build and run
make build
./porter serve
```

## Documentation

- **[Flight SQL Protocol](docs/flightsql-manifest.md)** - Protocol specification and implementation details
- **[Architecture Guide](docs/README.md)** - Deep dive into Porter's architecture
- **[API Reference](pkg/)** - Go package documentation
- **[Configuration Reference](config/)** - Complete configuration options

## Porter and the Future of Data Migration

With Porter's current capabilities:

- **Multi-backend support** (DuckDB and ClickHouse, with more planned)
- **Unified Flight SQL interface** across all backends
- **Arrow-native streaming** between different database systems

And as Porter gains:

- **Distributed query execution** (via DoExchange)
- **Additional backend support** (PostgreSQL, BigQuery, etc.)
- **Enhanced Flight SQL interoperability**

…it becomes not just a query engine, but a conduit — a streamlined pathway for moving structured data between systems without serialization overhead.

### Why This Matters

Most data migration today is:

- **Batch-oriented**
- **File-based** (CSV, Parquet)
- **Bloated** with ETL pipelines and format juggling

With Porter, you get:

- **Arrow in, Arrow out**: A columnar memory format travels intact
- **Query-based filtering**: Only move what matters
- **Streaming transport**: No intermediate staging or local disk
- **Backend-agnostic execution**: Read from DuckDB, write to ClickHouse, or vice versa

### Example Use Case

```bash
# Query DuckDB and stream results to ClickHouse (conceptual example)
# Currently: Use Porter as a unified Flight SQL gateway
porter serve --database "duckdb://source.db" &
porter serve --database "clickhouse://localhost:9000/target" --address :32011 &

# Future: Direct data migration pipeline
porter migrate --from "duckdb://source.db" --to "clickhouse://localhost:9000/target" \
  --query "SELECT * FROM users WHERE active = true" --target-table users_active
```

With distributed query + DoExchange, this grows into:

- **Live sync jobs** between heterogeneous databases
- **Selective replication** across cloud and edge
- **In-flight transformations** using SQL only

---

## Known Issues & Limitations

- **Experimental status**: APIs may change without notice
- **Limited authentication**: Only basic auth and OAuth2 currently supported
- **Single-node only**: No distributed query execution yet
- **Memory constraints**: Large result sets may cause memory pressure
- **Platform support**: Some features may be limited on Windows

## License

Released under the MIT License. See [LICENSE](LICENSE) for details.

---

