# 13525 Watcher to Kibana Rule Conversion

## 13525 Stage 1

```
FROM prod:filebeat-*
| WHERE @timestamp > NOW() - 5 minutes AND message LIKE "*completed_children = [ExecutedReportDetails.create_from_executed_report(children_reports[i]) for i in parent_children[qr.executed_report_id] if children_reports[i].finished]*" AND service.name == "SharedServices_Container"
| STATS message = TOP(message, 1, "desc"), client_code = TOP(labels.client_code, 1, "desc"), app_code = TOP(labels.app_code, 1, "desc") BY host.hostname
| LIMIT 10000
```

## 13525 Stage 2

```
FROM kibana_threshold_alerts
| WHERE @timestamp > NOW() - 5 minutes AND event_type == "13525"
| RENAME host.hostname AS hostname.keyword
| LOOKUP JOIN kibana_sdp_amdb ON hostname.keyword
| MV_EXPAND group
| WHERE group == "13525" AND alert_status == "enabled"
| KEEP hostname.keyword, event_type, labels.app_code, labels.client_code, message
| LIMIT 10000
```

## 13525 Message Action 1

```json
{
  "message": "{{context.hits.0._source.message}}",
  "host": {
    "hostname": "{{context.hits.0._source.host.hostname}}"
  },
  "labels": {
    "client_code": "{{context.hits.0._source.labels.client_code}}",
    "app_code": "{{context.hits.0._source.labels.app_code}}"
  },
  "event_type": "13525",
  "@timestamp": "{{context.date}}"
}```

## 13525 Message Action 2

```json
{
  "target": "{{context.hits.0._source.hostname.keyword}}",
  "event_type": "13525",
  "event_uuid": "on-prem_13525_{{context.hits.0._source.hostname.keyword}}",
  "alarm_name": "13525 - Deleting Multi Report Subs from Datastream",
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
  "dashboard_link": "https://287d86a4b1184182b340bd5074cdfd7e.us-east-1.aws.found.io:9243/s/information-technology/app/r/s/i1snO",
  "amdb_link": "https://hedgeservcorp.sharepoint.com/sites/GlobalTechnology/AMDB/SitePages/13525.aspx",
  "service": {
    "type": "kibana_alerts",
    "name": "log_alerts"
  },
  "alarm_tags": {
    "hs:app:maintenance-window": "true",
    "hs:std:app-name": "NA",
    "hs:app:amdb": "13525",
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
