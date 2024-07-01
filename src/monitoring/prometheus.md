# Prometheus

> “Prometheus, a Cloud Native Computing Foundation project, is a systems and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts when specified conditions are observed.” - Prometheus GitHub repository.

Prometheus is the engine which drives our monitoring system. For our example we will mainly focus on two sections:

## Targets

Targets are a list of endpoints you want to scrape. The two main exporters we care about are for 1) polkadot/substrate and 2) the node-exporter. An example of scraping these on the IP address 1.1.1.1 would be:

```yaml
scrape_configs:
  - job_name: check blockchain node
    static_configs:
      - targets:
          - 1.1.1.1:9080 # promtail
          - 1.1.1.1:9100 # node exporter
          - 1.1.1.1:9615 # substrate node
```

