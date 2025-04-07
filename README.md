# Observability Setup Documentation

## Introduction to Observability (O11y)

Observability is the ability to understand the internal state of a system by analyzing the data it produces, including logs, metrics, and traces.

**Monitoring(Metrics)**: involves tracking system metrics like CPU usage, memory usage, and network performance. Provides alerts based on predefined thresholds and conditions

Monitoring tells us what is happening.


**Logging(Logs)**: involves the collection of log data from various components of a system.

Logging explains why it is happening.


**Tracing(Traces)**: involves tracking the flow of a request or transaction as it moves through different services and components within a system.

Tracing shows how it is happening. 

 In our setup, we're using a combination of tools to achieve this:

* **VictoriaMetrics:** For metrics collection and storage.
* **Loki:** For log aggregation and querying.
* **Tempo:** For distributed tracing.
* **OpenTelemetry (OTel):** For standardizing and collecting telemetry data.
* **Grafana:** For visualization and analysis of all this data.

## VictoriaMetrics and its Data Flow

VictoriaMetrics is our go-to solution for metrics. 

**Data Flow:**

* **Application Metrics:** Our applications generate metrics  like CPU usage, memory usage, etc.
* **Node exporter to VM Agent :** Node exporter collect these metrics and send to vm agent .
* **VM Agent to VM Insert:** The `vm agent` sends these metrics to the `vm insert` component.
    * `vm insert` is the entry point for ingesting metrics into VictoriaMetrics.
* **VM Storage:** `vm insert` then writes the metrics to `vm storage`, where they're stored for long-term retention.
* **VM Select:** When we need to query these metrics (e.g., for Grafana dashboards), `vm select` retrieves the data from `vm storage`.
* **VM Alert:** `vm alert` constantly checks the metrics against defined rules. If a rule is triggered (e.g., high error rate), it sends out alerts.
* **OTel Collector Metrics:** The `otel collector` also generates its own metrics about its performance.
    * These metrics are sent to `vm insert` via Prometheus remote write.
* **Grafana:** We use Grafana to visualize the metrics from `vm select`, creating dashboards to monitor our applications and systems.

## Loki and its Data Flow

Loki is our log aggregation system. It's designed to be cost-effective and easy to operate.

**Data Flow:**

* **Promtail Collection:** `promtail` agents run on our application pods, collecting log files.
    * `Promtail` adds labels to the logs (e.g., pod name, namespace) for easy querying.
* **Promtail to Loki:** `promtail` sends the logs to Loki.
    * In our setup, we have a `loki standalone` instance, meaning all Loki components run in a single process.
* **Loki Storage:** Loki stores the log data in chunks.
    * `loki chunks cache` helps speed up retrieval.
* **Loki Querying:** When we query logs (e.g., in Grafana), Loki retrieves the relevant data.
    * `loki results cache` speeds up query performance.
* **Grafana:** We use Grafana to visualize and query the logs from Loki.

## Tempo

Tempo is our distributed tracing backend. It helps us understand the flow of requests through our applications.

**Data Flow:**

* **Application Traces:** Our applications are instrumented with OpenTelemetry to generate trace data.
* **OTel Collector:** The `otel collector` receives and processes these traces.
* **Tempo Standalone:** The `otel collector` sends the traces to `tempo standalone`, where they are stored.
* **Grafana:** We use Grafana to visualize and analyze the traces from Tempo.

## OpenTelemetry (OTel)

OpenTelemetry is a crucial part of our setup. It standardizes how we collect and export telemetry data.

**Key Components:**

* **OTel Collector:** A central hub for receiving, processing, and exporting telemetry data.
    * It receives traces and metrics from our applications.
    * It sends traces to Tempo and metrics to VictoriaMetrics.
    * It also generates its own metrics for monitoring its health.
* **OTel Webhook:** Used for receiving data via webhooks .
* **OTel Metrics:** The collector's internal metrics, sent to VictoriaMetrics.

## How It All Fits Together:

* Our applications generate logs, metrics, and traces.
* `promtail` collects logs and sends them to Loki.
* The OTel Collector collects traces and metrics, sending them to Tempo and VictoriaMetrics, respectively.
* VictoriaMetrics stores and queries metrics.
* Loki stores and queries logs.
* Tempo stores and queries traces.
* Grafana provides a unified view of all this data, allowing us to monitor and troubleshoot our applications effectively.

This setup provides a comprehensive observability solution, enabling us to understand the health and performance of our applications in detail.
