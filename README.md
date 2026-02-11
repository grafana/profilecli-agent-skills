# pyroscope-skills

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) plugin that queries a remote [Pyroscope](https://github.com/grafana/pyroscope) server to analyze profiling data and correlate hot functions with source code in your repository.

## Installation

```
/plugin marketplace add https://github.com/grafana/pyroscope-skills
/plugin install profilecli-plugin@pyroscope-skills
```

### Prerequisites

- **`profilecli`** on your PATH — the skill will prompt you to download it from [Pyroscope releases](https://github.com/grafana/pyroscope/releases/latest) if not found
- **`pprof`** on your PATH, or `go tool pprof` as a fallback

## Setup

The skill requires a connection to a Pyroscope-compatible server.

### Local Pyroscope

For a local instance, only `PROFILECLI_URL` is needed (defaults to `http://localhost:4040`):

```bash
export PROFILECLI_URL="http://localhost:4040"
```

### Grafana Cloud

Set the following environment variables:

```bash
# Data source proxy URL
export PROFILECLI_URL="https://my-grafana.example.com/api/datasources/proxy/uid/<datasource-uid>"

# Service account token (format: glsa_...)
export PROFILECLI_TOKEN="glsa_your_service_account_token"

# Optional: tenant ID for multi-tenant setups
export PROFILECLI_TENANT_ID="your-tenant-id"
```

Create a service account token at **Grafana > Administration > Service Accounts > Add token** with the `Viewer` role.

## Usage

The `profilecli-insights` skill lets you ask free-form questions about service performance. Answers are backed by real profiling data and correlated with your source code.

```
/profilecli-insights Why is the distributor slow?
/profilecli-insights What's using the most memory in the ingester?
/profilecli-insights Show me CPU hot paths for the query-frontend
```

## How it works

1. **Discovers services** — queries available services and profile types from your Pyroscope instance, correlating them with the current repository via `service_repository` labels
2. **Fetches profiles** — queries CPU, memory, goroutine, mutex, and block profiles using `profilecli` and analyzes them with `pprof`
3. **Maps to source code** — strips module prefixes from pprof file paths to locate the corresponding files in your checkout, and checks git ref alignment to warn if line numbers may be stale
4. **Delivers a report** — produces a structured analysis with a summary, ranked hot functions table, source code snippets, and prioritized optimization recommendations

The skill also highlights significant runtime functions such as `runtime.mallocgc` (allocation pressure), `runtime.gcDrain` (GC pressure), and `runtime.futex` (lock contention).

## License

AGPL-3.0
