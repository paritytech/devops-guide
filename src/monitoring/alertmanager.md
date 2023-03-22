# Alertmanager

The final part of the monitoring stack will be how and where to alert if something does go wrong. For example, you may want to send instant messages for warning alerts, but page somebody 24/7 for critical alerts.

Alertmanager has many modules for email / many IM formats with almost any output being supported. You can also use 3rd party providers like PagerDuty or VictorOps and do very detailed management of oncall queues, schedules etc…

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
