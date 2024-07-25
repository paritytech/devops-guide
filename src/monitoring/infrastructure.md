## Infrastructure Tools

Prometheus, Loki, Alertmanager, and Grafana are powerful tools commonly used for monitoring and visualizing metrics and logs in a distributed system. Here, we focus on integrating these tools to effectively monitor Polkadot metrics across relay chains and parachains.

### Metrics

[Prometheus](https://prometheus.io/docs/introduction/overview/) is an open source solution that can be used to collect metrics from applications.
It collects metrics from configured targets endpoints at given intervals, evaluates rule expressions, displays the results, and can trigger alerts when specified conditions are observed.

#### Prometheus Configuration

Targets are a list of endpoints you want to scrape. The two main exporters we care about are for 1) polkadot/substrate and 2) the node-exporter. An example of scraping these on the IP address 10.100.0.0 would be:

```yaml
scrape_configs:
  - job_name: polkadot
    static_configs:
      - targets:
          - 10.100.0.0:9080    # promtail
          - 10.100.0.0:9100    # node exporter
          - 10.100.0.0:9615    # relaychain metrics
          - 10.100.0.0:9625    # parachain metrics
```

### Logs

[Loki](https://grafana.com/docs/loki/latest/) is an open source solution that can be used to aggregate logs from applications, allowing the operator to see errors, patterns and be able to search through the logs from all hosts very easily. An agent such as [Promtail](https://grafana.com/docs/loki/latest/send-data/promtail) or [Grafana Alloy]() is used to push logs to the Loki server.

Example promtail.yaml configuration to collect the logs and create a Promtail metrics that aggregates each log level:

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


### Alerts

[Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/) handles alerts sent by Prometheus client, and responsible for deduplicating, grouping and routing to the correct receiver such as email, PagerDuty, and other mechanisms via webhook receiver. 

A simple alert for block production being slow would look like:

```yaml
- alert: BlockProductionSlow
  annotations:
    message: 'Best block on instance {{ $labels.instance }} increases by
less than 1 per minute for more than 5 minutes.'
  expr: increase(substrate_block_height{status="best"}[1m]) < 1
  for: 5m
  labels:
    severity: warning
```

### Visualization

[Grafana](https://grafana.com/docs/grafana/latest/) is where you can define dashboards to show the time series information that Prometheus is collecting. You just need to ensure you add a datasource:

```yaml
datasources:
  - name: "Prometheus"
    type: prometheus
    access: proxy
    editable: false
    orgId: 1
    url: "http://prometheus.monitoring.svc.cluster.local"
    version: 1
    jsonData:
      timeInterval: 30s
  - name: Loki
    type: loki
    access: proxy 
    orgId: 1
    url: http://loki:3100
    basicAuth: false
    version: 1
    editable: true
```

### Polkadot-mixin
Polkadot-monitoring-mixin is a set of Polkadot monitoring dashboards, alerts and rules collected based on our experience operating Polkadot Relay Chain and Parachain nodes. You can find it in this [repo](https://github.com/paritytech/polkadot-monitoring-mixin).

## Docker Compose

You can install the following components Docker Compose if you are evaluating, or developing monitoring stack for Polkadot.
The configuration files associated with these installation instructions run the monitoring stack as a single binary.

```
version: "3.8"

networks:
  polkadot:

services:
  prometheus:
    container_name: prometheus
    image: prom/prometheus:v2.53.0
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - 9090:9090
    restart: unless-stopped
    configs:
      - source: prometheus_config
        target: /etc/prometheus/prometheus.yml
    networks:
      - polkadot

  loki:
    container_name: loki
    image: grafana/loki:3.1.0
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    networks:
      - polkadot

  promtail:
    container_name: promtail
    image: grafana/promtail:3.1.0
    command: -config.file=/etc/promtail/config.yml
    user: root # Required to read container logs
    ports:
      - 9080:9080
    volumes:
      - /var/lib/docker/containers:/var/lib/docker/containers
      - /var/run/docker.sock:/var/run/docker.sock
    configs:
      - source: promtail_config
        target: /etc/promtail/config.yml
    networks:
      - polkadot

  grafana:
    container_name: grafana
    image: grafana/grafana:latest
    environment:
      - GF_PATHS_PROVISIONING=/etc/grafana/provisioning
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    entrypoint:
      - sh
      - -euc
      - |
        mkdir -p /etc/grafana/provisioning/datasources
        cat <<EOF > /etc/grafana/provisioning/datasources/ds.yaml
        apiVersion: 1
        datasources:
        - name: Loki
          type: loki
          access: proxy 
          orgId: 1
          url: http://loki:3100
          basicAuth: false
          version: 1
          editable: true
        - name: Prometheus
          type: prometheus
          access: proxy 
          orgId: 1
          url: http://prometheus:9090
          basicAuth: false
          isDefault: true
          version: 1
          editable: true
        EOF
        /run.sh
    ports:
      - "3000:3000"
    networks:
      - polkadot

  polkadot_collator:
    container_name: polkadot_collator
    image: parity/polkadot-parachain:1.14.0
    command: >
      --tmp --prometheus-external --prometheus-port 9625 -- --tmp --prometheus-external --prometheus-port 9615
    ports:
      - "9615:9615"
      - "9625:9625"
    networks:
      - polkadot

configs:
  prometheus_config:
    content: |
      global:
        scrape_interval: 15s # By default, scrape targets every 15 seconds.
        evaluation_interval: 15s # Evaluate rules every 15 seconds.

      scrape_configs:
        - job_name: polkadot
          static_configs:
            - targets:
                - polkadot_collator:9615    # relaychain metrics
                - polkadot_collator:9625    # parachain metrics
  promtail_config:
    content: |
      server:
        http_listen_port: 9080
      
      positions:
        filename: /tmp/positions.yaml
      
      clients:
        - url: http://loki:3100/loki/api/v1/push

      scrape_configs:
      - job_name: containers
        docker_sd_configs:
          - host: unix:///var/run/docker.sock
            refresh_interval: 5s
            filters:
              - name: name
                values: [polkadot_collator]
        relabel_configs:
          - source_labels: ['__meta_docker_container_name']
            regex: '/(.*)'
            target_label: 'container'
```