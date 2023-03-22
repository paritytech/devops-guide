# Prometheus

**“Prometheus, a Cloud Native Computing Foundation project, is a systems and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts when specified conditions are observed.”** - Prometheus Github Repository

Prometheus is the engine which drives our monitoring system. For our example we will mainly focus on two sections:

### Targets

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

### Rules

Now that you are gathering data from the host you can add rules that will trigger alerts. These are defined in the format:

```yaml
  - alert: BlockProductionSlow
    annotations:
      message: 'Best block on instance {{ $labels.instance }} increases by
      less than 1 per minute for more than 3 minutes.'
    expr: increase(polkadot_block_height{status="best"}[1m]) < 1
    for: 3m
    labels:
      severity: warning
```

Here’s an [example](https://github.com/ddorgan/substrate-alerting-rules/blob/main/alerting-rules.yaml) with some basic block production rules.
