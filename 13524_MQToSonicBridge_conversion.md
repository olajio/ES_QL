**13524_MQToSonicBridge Stage 1:**

```
FROM prod:filebeat-*
| WHERE @timestamp > NOW() - 5 minutes AND message LIKE "*MQRC_CONNECTION_BROKEN*" AND service.name == "MQToSonicBridge_Container" AND log.level == "ERROR" AND labels.client_code == "MDBOPROD"
| STATS message = TOP(message, 1, "desc"), client_code = TOP(labels.client_code, 1, "desc"), app_code = TOP(labels.app_code, 1, "desc") BY host.hostname
| LIMIT 10000
```

**13524_MQToSonicBridge Stage 2:**

```
FROM kibana_threshold_alerts
| WHERE @timestamp > NOW() - 5 minutes AND event_type == "13524"
| RENAME host.hostname AS hostname.keyword
| LOOKUP JOIN kibana_sdp_amdb ON hostname.keyword
| MV_EXPAND group
| WHERE group == "13524" AND alert_status == "enabled"
| KEEP hostname.keyword, event_type, labels.app_code, labels.client_code, message
| LIMIT 10000
```

And the two message actions:

**13524_MQToSonicBridge Message Action 1 & 2** (identical):

```json
{
  "target": "{{context.hits.0._source.hostname.keyword}}",
  "event_type": "13524",
  "event_uuid": "on-prem_13524_{{context.hits.0._source.hostname.keyword}}",
  "alarm_name": "13524 - MQRC_CONNECTION_BROKEN",
  "trace": {
    "id": "{{alert.uuid}}"
  },
  "tags_to_exclude": "NA",
  "cloud": {
    "region": "NA",
    "account": {
      "name": "NA",
      "id": "NA"
    }
  },
  "labels": {
    "client_code": "{{context.hits.0._source.labels.client_code}}"
  },
  "@timestamp": "{{date}}",
  "dashboard_link": "https://287d86a4b1184182b340bd5074cdfd7e.us-east-1.aws.found.io:9243/s/information-technology/app/r/s/ViUKb",
  "amdb_link": "https://hedgeservcorp.sharepoint.com/sites/globaltechnology/amdb/sitepages/13524.aspx?web=1",
  "service": {
    "type": "kibana_alerts",
    "name": "log_alerts"
  },
  "alarm_tags": {
    "hs:app:maintenance-window": "true",
    "hs:std:app-name": "NA",
    "hs:app:amdb": "13524",
    "hs:app:sdp-priority": "2 - High",
    "hs:std:svc-operator": "NA",
    "hs:std:svc-software-owner": "NA",
    "hs:app:monitored": "true",
    "hs:std:app-code": "{{context.hits.0._source.labels.app_code}}",
    "hs:app:ticket-group": "Monitoring and Analytics - Testing"
  },
  "alarm_reason": "{{context.hits.0._source.message}}"
}
```

One thing to note — this watcher checks `open_tickets` with `"status": "Onhold"` instead of `"status": "open"` like 13513 and 13523 did. That won't affect the ES|QL stages themselves, but it's worth keeping in mind if the Kibana rule's ticketing logic needs to account for on-hold tickets rather than open ones.
