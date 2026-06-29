# Day19 / Day20 Integration Evidence

Day23 Prometheus was configured to scrape prior-day lab services.

## Day19 Qdrant Vector Store

- Job: day19-qdrant
- Target: host.docker.internal:6333
- Status: up
- Evidence metrics:
  - collections_total = 1
  - collections_vector_total = 3

A demo Qdrant collection named `day23_integration_demo` was created with 3 vectors to show non-zero Day19 vector-store metrics in Day23 Prometheus.

## Day20 llama.cpp Model Server

- Job: day20-llamacpp
- Target: host.docker.internal:8080
- Status: up
- Evidence metrics:
  - llamacpp:prompt_tokens_total = 10
  - llamacpp:tokens_predicted_total = 16

A completion request was sent to the Day20 llama.cpp server, and Day23 Prometheus scraped non-zero token metrics from the llama.cpp `/metrics` endpoint.

## Evidence Files

- submission/integration-day19-day20-prometheus-evidence.txt
- submission/day20-llamacpp-completion-response.json
