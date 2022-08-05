Example Monitoring Stack
=========================

An example of a very common open source monitoring stack.

## Prometheus

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


## Grafana

Grafana is where you can define dashboards to show the time series information that prometheus is collecting. You just need to ensure you add a datasource:

```yaml
  datasources:
      - name: "prometheus-prod"
        type: prometheus
        access: proxy
        editable: false
        orgId: 1
        url: "http://prometheus-prod.monitoring.svc.cluster.local"
        version: 1
        jsonData:
          timeInterval: 30s

```

You can find [existing](https://grafana.com/grafana/dashboards/?search=polkadot) polkadot dashboards at grafana.com -

 ![grafana example](images/lb0gOx.png)

## Alertmanager

The final part of the monitoring stack will be how and where to alert if something does go wrong. For example you may want to send instant messages for warning alerts, but page somebody 24/7 for critical alerts.

Alertmanager has many modules for email / many IM formats with almost any output being supported. You can also use 3rd party providers like pagerduty or victorops and do very detailed management of oncall queues, schedules etc…

A simple alert for block production being slow would look like:

```yaml
        - alert: BlockProductionSlow
          annotations:
            message: 'Best block on instance {{ $labels.instance }} increases by
        less than 1 per minute for more than 3 minutes.'
          expr: increase(substrate_block_height{status="best"}[1m]) < 1
          for: 3m
          labels:
            severity: warning
```

This will then be sent to the outbound path for the severity warning.

## Loki

**“Loki is a horizontally scalable, highly available, multi-tenant log aggregation system inspired by Prometheus”**

Loki is used to aggregate logs from blockchain nodes, allowing the operator to see errors, patterns and be able to search through the logs from all hosts very easily. An agent called promtail is used to push logs to the central loki server.

Example promtail.yaml configuration to collect the logs of a node managed by systemd:

```
# promtail server config
server:
  http_listen_port: 9080
  grpc_listen_port: 0
  log_level: info
positions:
  filename: /var/lib/promtail/positions.yaml

# loki servers
clients:
  - url: http://loki.example.com/loki/api/v1/push
    backoff_config:
      min_period: 1m
      max_period: 1h
      max_retries: 10000

scrape_configs:
  - job_name: journald
    journal:
      max_age: 1m
      path: /var/log/journal
      labels:
        job: journald
    pipeline_stages:
      - match:
          selector: '{job="journald"}'
          stages:
          - multiline:
              firstline: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}'
              max_lines: 2500
          - regex:
              expression: '(?P<date>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3})\s+(?P<level>(TRACE|DEBUG|INFO|WARN|ERROR))\s+(?P<worker>([^\s]+))\s+(?P<target>[\w-]+):?:?(?P<subtarget>[\w-]+)?:[\s]?(?P<chaintype>\[[\w-]+\]+)?[\s]?(?P<message>.+)'
          - labels:
              level:
              target:
              subtarget:
          - metrics:
              log_lines_total:
                type: Counter
                Description: "Total Number of Chain Logs"
                prefix: "promtail_chain_"
                max_idle_duration: 24h
                config:
                  match_all: true
                  action: inc
      - match:
          selector: '{job="journald", level="ERROR"}'
          stages:
          - multiline:
              firstline: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}'
              max_lines: 2500
          - metrics:
              log_lines_total:
                type: Counter
                Description: "Total Number of Chain Error Logs"
                prefix: "promtail_chain_error_"
                max_idle_duration: 24h
                config:
                  match_all: true
                  action: inc
      - match:
          selector: '{job="journald", level=~".+"} |~ "(?i)(panic)"'
          stages:
          - multiline:
              firstline: '^\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}\.\d{3}'
              max_lines: 2500
          - metrics:
              log_lines_total:
                type: Counter
                Description: "Total Number of Chain Panic Logs"
                prefix: "promtail_chain_panic_"
                max_idle_duration: 24h
                config:
                  match_all: true
                  action: inc
    relabel_configs:
      - source_labels: ['__journal__systemd_unit']
        target_label: 'unit'
      - source_labels: ['unit']
        regex: '(.*.scope|user.*.service)'
        action: drop
      - source_labels: ['__journal__hostname']
        target_label: 'host'
      - action: replace
        source_labels:
        - __journal__cmdline
        - __journal__hostname
        separator: ";"
        regex: '.*--chain.*;(.*)'
        target_label: 'node'
        replacement: $1
```

The above example also configures the following custom metrics derived from logs `promtail_chain_log_lines_total`, `promtail_chain_error_log_lines_total` and  `promtail_chain_panic_log_lines_total` to be exposed on the promtail metrics endpoint (http://host:9080).