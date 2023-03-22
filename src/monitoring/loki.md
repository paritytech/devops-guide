# Loki

> "Loki is a horizontally scalable, highly available, multi-tenant log aggregation system inspired by Prometheus"

Loki is used to aggregate logs from blockchain nodes, allowing the operator to see errors, patterns and be able to search through the logs from all hosts very easily. An agent called promtail is used to push logs to the central Loki server.

Example promtail.yaml configuration to collect the logs of a node managed by systemd:

```yaml
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
      - source_labels: ["__journal__systemd_unit"]
        target_label: "unit"
      - source_labels: ["unit"]
        regex: "(.*.scope|user.*.service)"
        action: drop
      - source_labels: ["__journal__hostname"]
        target_label: "host"
      - action: replace
        source_labels:
          - __journal__cmdline
          - __journal__hostname
        separator: ";"
        regex: ".*--chain.*;(.*)"
        target_label: "node"
        replacement: $1
```

The above example also configures the following custom metrics derived from logs `promtail_chain_log_lines_total`, `promtail_chain_error_log_lines_total` and `promtail_chain_panic_log_lines_total` to be exposed on the promtail metrics endpoint <http://host:9080>.
