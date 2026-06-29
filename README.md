# Day 23 — Track 2 — Observability Stack Lab Submission

**Student:** Thái Thị Yến Nhi

**Submission date:** 2026-06-29

**Repository:** `Lemin9802/Day23-Lab_2A202600783_Thai-Thi-Yen-Nhi` 


This repository is my completed Day 23 Track 2 Observability Stack Lab submission. The lab builds and validates an open-source observability stack for an AI inference service using FastAPI, Prometheus, Grafana, Loki, Jaeger, Alertmanager, and the OpenTelemetry Collector.

The goal of this lab was to prove that an AI service can be operated and debugged using production-style observability signals: metrics, dashboards, alerts, traces, logs, drift detection, and cross-day integration metrics.

---

## Verification result

The final local verification passed:

```text
Result: 12/12 checks passed
```

I also created a 22-point submission checklist at the bottom of [`submission/REFLECTION.md`](submission/REFLECTION.md). The checklist maps every core rubric checkpoint to its evidence file or screenshot.

---

## What I completed

### 1. FastAPI instrumentation

The inference API exposes Prometheus metrics, OpenTelemetry traces, and structured JSON logs.

Implemented metric families include:

* `inference_requests_total`
* `inference_latency_seconds_bucket`
* `inference_active_gauge`
* `inference_tokens_total`
* `inference_quality_score`

Evidence:

* [`submission/active-gauge-evidence.txt`](submission/active-gauge-evidence.txt)
* [`submission/screenshots/dashboard-overview.png`](submission/screenshots/dashboard-overview.png)

---

### 2. Prometheus and Grafana dashboards

Prometheus scrapes the FastAPI inference service and prior-day integration targets. Grafana loads the Day 23 dashboards automatically.

Completed dashboards:

| Dashboard             | Evidence                                                                                           |
| --------------------- | -------------------------------------------------------------------------------------------------- |
| AI Service Overview   | [`submission/screenshots/dashboard-overview.png`](submission/screenshots/dashboard-overview.png)   |
| SLO Burn Rate         | [`submission/screenshots/slo-burn-rate.png`](submission/screenshots/slo-burn-rate.png)             |
| Cost & Tokens         | [`submission/screenshots/cost-and-tokens.png`](submission/screenshots/cost-and-tokens.png)         |
| Cross-Day Integration | [`submission/screenshots/cross-day-dashboard.png`](submission/screenshots/cross-day-dashboard.png) |

The overview dashboard shows service-level metrics such as request rate, latency, error rate, GPU utilization, token throughput, and in-flight requests.

The SLO dashboard shows burn-rate behavior and active alert state.

The cost dashboard shows token throughput and a non-zero estimated hourly cost.

The cross-day dashboard shows Day19 Qdrant and Day20 llama.cpp metrics in six panels.

---

### 3. Alertmanager and Slack alerting

Alertmanager was configured to send Slack notifications. I validated both the firing and resolved states of the `ServiceDown` alert.

Evidence:

* [`submission/screenshots/alertmanager-firing.png`](submission/screenshots/alertmanager-firing.png)
* [`submission/screenshots/slack-firing.png`](submission/screenshots/slack-firing.png)
* [`submission/screenshots/slack-resolved.png`](submission/screenshots/slack-resolved.png)
* [`submission/alert-run.log`](submission/alert-run.log)
* [`submission/alertmanager-firing-api.json`](submission/alertmanager-firing-api.json)

No real Slack webhook is committed. The local webhook is kept in `.env`, and `.env` is ignored by Git.

---

### 4. Jaeger tracing and GenAI span attributes

The `/predict` request path is visible in Jaeger as one complete trace:

```text
predict
├── embed-text
├── vector-search
└── generate-tokens
```

The trace includes GenAI semantic attributes such as:

* `gen_ai.request.model`
* `gen_ai.usage.input_tokens`
* `gen_ai.usage.output_tokens`
* `gen_ai.response.finish_reason`

Evidence:

* [`submission/screenshots/jaeger-trace.png`](submission/screenshots/jaeger-trace.png)
* [`submission/good-jaeger-trace-api.json`](submission/good-jaeger-trace-api.json)
* [`submission/good-jaeger-trace-id.txt`](submission/good-jaeger-trace-id.txt)

---

### 5. Structured logs with trace correlation

The app emits structured JSON logs with a `trace_id`, so an inference log line can be correlated back to a Jaeger trace.

Evidence:

* [`submission/json-log-with-trace-id.txt`](submission/json-log-with-trace-id.txt)
* [`submission/REFLECTION.md`](submission/REFLECTION.md)

---

### 6. Drift detection

The drift detection task compares a reference dataset and a current dataset using PSI, KL, and KS-style statistics.

Detected drift:

* `prompt_length`
* `response_quality`

No meaningful drift:

* `embedding_norm`
* `response_length`

Evidence:

* [`04-drift-detection/reports/drift-summary.json`](04-drift-detection/reports/drift-summary.json)
* [`submission/drift-summary.json`](submission/drift-summary.json)
* [`submission/drift-summary.md`](submission/drift-summary.md)
* [`submission/drift-run.log`](submission/drift-run.log)
* [`submission/drift-run-with-evidently.log`](submission/drift-run-with-evidently.log)
* [`submission/evidently-html-report.html`](submission/evidently-html-report.html)
* [`submission/screenshots/evidently-html-report.png`](submission/screenshots/evidently-html-report.png)

---

### 7. Cross-day integration

I connected prior-day lab services into Day 23 Prometheus and displayed them in a Grafana dashboard.

Integrated sources:

| Source          | Metrics shown                              |
| --------------- | ------------------------------------------ |
| Day19 Qdrant    | target up, collection count, vector count  |
| Day20 llama.cpp | target up, prompt tokens, predicted tokens |

Evidence:

* [`submission/screenshots/cross-day-dashboard.png`](submission/screenshots/cross-day-dashboard.png)
* [`submission/integration-summary.md`](submission/integration-summary.md)
* [`submission/integration-day19-day20-prometheus-evidence.txt`](submission/integration-day19-day20-prometheus-evidence.txt)
* [`submission/cross-day-dashboard-api.json`](submission/cross-day-dashboard-api.json)
* [`submission/cross-day-dashboard-prometheus-evidence.txt`](submission/cross-day-dashboard-prometheus-evidence.txt)
* [`submission/day20-llamacpp-completion-response.json`](submission/day20-llamacpp-completion-response.json)

---

## Important fixes made during the lab

I made several implementation fixes to make the stack reliable and gradeable:

* Fixed the Grafana smoke test in the `Makefile`.
* Added a stable Grafana datasource UID so dashboard panels resolve Prometheus correctly.
* Fixed Alertmanager Slack configuration by using `api_url_file` instead of directly templating the Slack webhook.
* Fixed `/predict` trace parenting so Jaeger shows one complete trace with child spans.
* Added Day19 Qdrant and Day20 llama.cpp scrape targets to Prometheus.
* Added a Cross-Day Integration Grafana dashboard with six panels.
* Generated an Evidently HTML drift report and included a screenshot.
* Embedded a 22-point submission checklist at the bottom of `submission/REFLECTION.md`.

---

## How to run locally

Create a local `.env` file from the example:

```bash
cp .env.example .env
```

Edit `.env` locally if you want to test Slack alerting. Do not commit `.env`.

Start and verify the stack:

```bash
make setup
make up
make smoke
```

Generate inference load:

```bash
make load
```

Trigger alert fire and resolve:

```bash
make alert
```

Run drift detection:

```bash
make drift
```

Run the final verification gate:

```bash
make verify
```

Stop the stack:

```bash
make down
```

---

## Local services

| Service                         | URL                           |
| ------------------------------- | ----------------------------- |
| FastAPI app                     | http://localhost:8000         |
| Prometheus                      | http://localhost:9090         |
| Grafana                         | http://localhost:3000         |
| Alertmanager                    | http://localhost:9093         |
| Jaeger                          | http://localhost:16686        |
| Loki                            | http://localhost:3100         |
| OpenTelemetry Collector metrics | http://localhost:8888/metrics |

---

## Main evidence files

| Evidence                       | Location                                                                         |
| ------------------------------ | -------------------------------------------------------------------------------- |
| Main reflection and checklist  | [`submission/REFLECTION.md`](submission/REFLECTION.md)                           |
| Screenshots                    | [`submission/screenshots/`](submission/screenshots/)                             |
| Setup report                   | [`00-setup/setup-report.json`](00-setup/setup-report.json)                       |
| Final verify log               | [`submission/make-verify.log`](submission/make-verify.log)                       |
| Drift summary                  | [`submission/drift-summary.json`](submission/drift-summary.json)                 |
| Evidently HTML report          | [`submission/evidently-html-report.html`](submission/evidently-html-report.html) |
| Cross-day integration summary  | [`submission/integration-summary.md`](submission/integration-summary.md)         |
| Good Jaeger trace API evidence | [`submission/good-jaeger-trace-api.json`](submission/good-jaeger-trace-api.json) |
| JSON log with trace ID         | [`submission/json-log-with-trace-id.txt`](submission/json-log-with-trace-id.txt) |

---

## Repository structure

```text
.
├── 00-setup/
│   ├── README.md
│   ├── setup-report.json
│   └── verify-docker.py
│
├── 01-instrument-fastapi/
│   └── app/
│       ├── Dockerfile
│       ├── main.py
│       ├── metrics.py
│       └── requirements.txt
│
├── 02-prometheus-grafana/
│   ├── alertmanager/
│   │   └── alertmanager.yml
│   ├── grafana/
│   │   ├── dashboards/
│   │   │   ├── cross-day-integration.json
│   │   │   ├── cost-and-tokens.json
│   │   │   ├── overview.json
│   │   │   └── slo-burn-rate.json
│   │   └── provisioning/
│   │       ├── dashboards/
│   │       │   └── dashboards.yml
│   │       └── datasources/
│   │           └── datasources.yml
│   └── prometheus/
│       ├── prometheus.yml
│       └── rules/
│           └── slo-burn-rate.yml
│
├── 03-tracing-and-logs/
│   ├── loki/
│   │   └── loki-config.yaml
│   └── otel-collector/
│       └── otel-config.yaml
│
├── 04-drift-detection/
│   ├── README.md
│   ├── data/
│   │   ├── current.parquet
│   │   └── reference.parquet
│   ├── reports/
│   │   └── drift-summary.json
│   └── scripts/
│       └── drift_detect.py
│
├── 05-integration/
│   ├── monitor-day16-cloud.py
│   ├── monitor-day17-pipelines.py
│   ├── monitor-day18-lakehouse.py
│   ├── monitor-day19-qdrant.py
│   ├── monitor-day20-llama-cpp.py
│   └── monitor-day22-alignment.py
│
├── BONUS-agentops/
├── BONUS-ebpf-profiling/
├── BONUS-llm-native-obs/
│
├── scripts/
│   ├── lint-dashboards.py
│   ├── trigger-alert.sh
│   └── verify.py
│
├── submission/
│   ├── REFLECTION.md
│   ├── active-gauge-evidence.txt
│   ├── alert-run.log
│   ├── alertmanager-firing-api.json
│   ├── cross-day-dashboard-api.json
│   ├── cross-day-dashboard-prometheus-evidence.txt
│   ├── current-span-validation.json
│   ├── day20-llamacpp-completion-response.json
│   ├── drift-run.log
│   ├── drift-run-with-evidently.log
│   ├── drift-summary.json
│   ├── drift-summary.md
│   ├── evidently-html-report.html
│   ├── good-jaeger-trace-api.json
│   ├── good-jaeger-trace-id.txt
│   ├── integration-day19-day20-prometheus-evidence.txt
│   ├── integration-summary.md
│   ├── json-log-with-trace-id.txt
│   ├── make-verify.log
│   └── screenshots/
│       ├── alertmanager-firing.png
│       ├── cost-and-tokens.png
│       ├── cross-day-dashboard.png
│       ├── dashboard-overview.png
│       ├── evidently-html-report.png
│       ├── jaeger-trace.png
│       ├── slack-firing.png
│       ├── slack-resolved.png
│       └── slo-burn-rate.png
│
├── .env.example
├── .gitignore
├── docker-compose.yml
├── HARDWARE-GUIDE.md
├── Makefile
├── README.md
├── requirements-evidently.txt
├── requirements.txt
├── rubric.md
└── VIBE-CODING.md
```

---

## Bonus status

No bonus work is submitted in this version. This submission focuses on completing the full core rubric with clean evidence.

---

## Notes for grading

The fastest grading path is:

1. Open [`submission/REFLECTION.md`](submission/REFLECTION.md).
2. Review the embedded images and links.
3. Scroll to the final section: **Submission checklist**.
4. Run:

```bash
make verify
```

Expected result:

```text
Result: 12/12 checks passed
```
