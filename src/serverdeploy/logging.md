Logging
===========


### Local Logging

By default output from your systemd service will go to local syslog. This will mean it ends up in a place like `/var/log/syslog` or `/var/log/messages`

You can also view these using the `journalctl` command. To tail the current output of the polkadot process run:

```
journalctl -u polkadot -f
```

It is also possible to remove old logs (say older than two days ago) using a command:

```
journalctl -u polkadot --vacuum-time=2d
```

Or to retain only the past 1G of data run:

```
journalctl --vacuum-size=1G
```


## Remote Logging

In a setup with many hosts you will want to aggregate the logs at a central point, some common options include Loki or Elasticsearch. 

### Loki

To log to a remote loki instance you need to install the `promtail` package. An example configuration file to send logs to a remote host:

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
  - url: http://myloki.mycompany.com/loki/api/v1/push
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
```

### Elasticsearch

To log to a remote elasticsearch cluster you need to install the `logstash` package. An example configuration would look like:


```yaml
nput {
     journald {
       path      => "/var/log/journal"
       seekto => "tail"
       thisboot => true
       filter    => {
           "_SYSTEMD_UNIT" => "polkadot.service"
       }
       type => "systemd"
     }
}


filter {
date {
     match => ["timestamp", "YYYY-mm-dd HH:MM:ss.SSS"]
     target => "@timestamp"
  }
  mutate {
    rename => [ "MESSAGE", "message" ]
    remove_field => [ "cursor", "PRIORITY", "SYSLOG_FACILITY", "SYSLOG_IDENTIFIER", "_BOOT_ID", "_CAP_EFFECTIVE", "_CMDLINE", "_COMM", "_EXE", "_GID", "_HOSTNAME", "_MACHINE_ID", "_PID", "_SELINUX_CONTEXT", "_STREAM_ID", "_SYSTEMD_CGROUP", "_SYSTEMD_INVOCATION_ID", "_SYSTEMD_SLICE", "_SYSTEMD_UNIT", "_TRANSPORT", "_UID" ]
  }
  if ([message] =~ ".*TRACE .*") { drop{ } }
  grok {
     match => { "message" => "%{NOTSPACE:thread} %{LOGLEVEL:log-level} %{NOTSPACE:namespace} %{GREEDYDATA:message}" }
   }
}

output {
   elasticsearch {
     hosts => ["https://myelasticsearch.mycompany.com:9243"]
     user => "username"
     password => "password"
     index => "logstash-polkadot-%{+YYYY.MM.dd}"
   }
}
```

## Logging Options

Logging output from any substrate based changes has many log levels and targets.  All targets are set to `info` logging by default. You can adjust individual log levels using the `--log (-l short)` option, for example `-l afg=trace,sync=debug` or globally with `-ldebug`.

Levels:
- error
- warn
- info
- debug
- trace

Targets:
- afg
- aura
- babe
- beefy
- db
- gossip
- header
- peerset
- pow
- rpc
- runtime
- runtime::contracts
- sc_offchain
- slots
- state-db
- state_tracing
- sub-libp2p
- sync
- telemetry
- tracing
- trie
- txpool
