# 13527 Watcher to Kibana Rule Conversion

> **Note:** This watcher differs structurally from the others (13513, 13523, 13524, 13525).
> It aggregates by `labels.client_code` (via `client.keyword` in the AMDB) rather than
> `host.hostname`, uses a composite aggregation across client/host/service, and excludes
> `labels.datacenter: CS51`. The Stage 1 query reflects this different grouping pattern.

## 13527 Stage 1

```
FROM prod:filebeat-*
| WHERE @timestamp > NOW() - 5 minutes AND message LIKE "*[DBNETLIB]SQL Server does not exist or access denied*" AND NOT labels.datacenter == "CS51"
| STATS message = TOP(message, 1, "desc"), client_code = TOP(labels.client_code, 1, "desc"), app_code = TOP(labels.app_code, 1, "desc") BY host.hostname, labels.client_code, service.name
| LIMIT 10000
```

## 13527 Stage 2

```
FROM kibana_threshold_alerts
| WHERE @timestamp > NOW() - 5 minutes AND event_type == "13527"
| RENAME labels.client_code AS client.keyword
| LOOKUP JOIN kibana_sdp_amdb ON client.keyword
| MV_EXPAND group
| WHERE group == "13527"
| KEEP client.keyword, event_type, labels.app_code, labels.client_code, message, host.hostname, service.name
| LIMIT 10000
```

> **Stage 2 differences from other conversions:**
> - The AMDB lookup joins on `client.keyword` instead of `hostname.keyword`, matching
>   the watcher's use of `client.keyword` in `sdp_amdb`.
> - The watcher's AMDB query does not filter on `alert_status == "enabled"` — it only
>   filters on `group`. This is preserved above. If your environment requires the
>   `alert_status` filter, add `AND alert_status == "enabled"` to the WHERE clause.
> - Additional fields (`host.hostname`, `service.name`) are kept to support the
>   watcher's host/service pair reporting in ticket descriptions.

## 13527 Message Action 1

```json
{
  "target": "{{context.hits.0._source.client.keyword}}",
  "event_type": "13527",
  "event_uuid": "on-prem_13527_{{context.hits.0._source.client.keyword}}",
  "alarm_name": "13527 - SQL Server Does not Exist or Access Denied",
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
  "dashboard_link": "https://287d86a4b1184182b340bd5074cdfd7e.us-east-1.aws.found.io:9243/s/information-technology/app/r/s/8TRQw",
  "amdb_link": "https://hedgeservcorp.sharepoint.com/sites/GlobalTechnology/AMDB/SitePages/13527.aspx",
  "service": {
    "type": "kibana_alerts",
    "name": "log_alerts"
  },
  "alarm_tags": {
    "hs:app:maintenance-window": "true",
    "hs:std:app-name": "NA",
    "hs:app:amdb": "13527",
    "hs:app:sdp-priority": "1 - Critical",
    "hs:std:svc-operator": "NA",
    "hs:std:svc-software-owner": "NA",
    "hs:app:monitored": "true",
    "hs:std:app-code": "{{context.hits.0._source.labels.app_code}}",
    "hs:app:ticket-group": "Monitoring and Analytics - Testing"
  },
  "alarm_reason": "{{context.hits.0._source.message}}"
}
```

## 13527 Message Action 2

```json
{
  "target": "{{context.hits.0._source.client.keyword}}",
  "event_type": "13527",
  "event_uuid": "on-prem_13527_{{context.hits.0._source.client.keyword}}",
  "alarm_name": "13527 - SQL Server Does not Exist or Access Denied",
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
  "dashboard_link": "https://287d86a4b1184182b340bd5074cdfd7e.us-east-1.aws.found.io:9243/s/information-technology/app/r/s/8TRQw",
  "amdb_link": "https://hedgeservcorp.sharepoint.com/sites/GlobalTechnology/AMDB/SitePages/13527.aspx",
  "service": {
    "type": "kibana_alerts",
    "name": "log_alerts"
  },
  "alarm_tags": {
    "hs:app:maintenance-window": "true",
    "hs:std:app-name": "NA",
    "hs:app:amdb": "13527",
    "hs:app:sdp-priority": "1 - Critical",
    "hs:std:svc-operator": "NA",
    "hs:std:svc-software-owner": "NA",
    "hs:app:monitored": "true",
    "hs:std:app-code": "{{context.hits.0._source.labels.app_code}}",
    "hs:app:ticket-group": "Monitoring and Analytics - Testing"
  },
  "alarm_reason": "{{context.hits.0._source.message}}"
}
```
