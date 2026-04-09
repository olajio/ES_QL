# 52009 Watcher to Kibana Rule Conversion

## 52009 Stage 1

```
FROM prod:filebeat-*-mt-prd
| WHERE @timestamp > NOW() - 5 minutes AND (service.name == "Web Auth" OR service.name == "web-auth") AND log.level == "ERROR" AND message LIKE "*Failed to query Active Directory*" AND NOT message LIKE "*[USAGE]*"
| STATS message = TOP(message, 1, "desc"), cloud_account_name = TOP(cloud.account.name, 1, "desc"), kubernetes_labels_team = TOP(kubernetes.labels.team, 1, "desc"), service_name = TOP(service.name, 1, "desc") BY host.hostname
| LIMIT 10000
```

## 52009 Stage 2

```
FROM kibana_threshold_alerts
| WHERE @timestamp > NOW() - 5 minutes AND event_type == "sdpmt_52009"
| RENAME host.hostname AS hostname.keyword
| LOOKUP JOIN kibana_sdpmt_amdb ON hostname.keyword
| MV_EXPAND group
| WHERE group == "sdpmt_52009" AND alert_status == "enabled"
| KEEP hostname.keyword, event_type, message, cloud_account_name, kubernetes_labels_team, service_name
| LIMIT 10000
```

## 52009 Message Action 1

```json
{
  "message": "{{context.hits.0._source.message}}",
  "host": {
    "hostname": "{{context.hits.0._source.host.hostname}}"
  },
  "cloud_account_name": "{{context.hits.0._source.cloud_account_name}}",
  "kubernetes_labels_team": "{{context.hits.0._source.kubernetes_labels_team}}",
  "service_name": "{{context.hits.0._source.service_name}}",
  "event_type": "sdpmt_52009",
  "@timestamp": "{{context.date}}"
}
```

## 52009 Message Action 2

```json
{
  "target": "{{context.hits.0._source.hostname.keyword}}",
  "event_type": "sdpmt_52009",
  "event_uuid": "mt-prd_sdpmt_52009_{{context.hits.0._source.hostname.keyword}}",
  "alarm_name": "52009 - Failed to query Active Directory",
  "trace": {
    "id": "{{alert.uuid}}"
  },
  "tags_to_exclude": "NA",
  "cloud": {
    "region": "NA",
    "account": {
      "name": "{{context.hits.0._source.cloud_account_name}}"
    }
  },
  "labels": {
    "client_code": "NA"
  },
  "@timestamp": "{{date}}",
  "dashboard_link": "https://287d86a4b1184182b340bd5074cdfd7e.us-east-1.aws.found.io:9243/s/information-technology/app/r/s/5LiUA",
  "amdb_link": "https://hedgeservcorp.sharepoint.com/sites/globaltechnology/amdb/sitepages/52009.aspx",
  "service": {
    "type": "kibana_alerts",
    "name": "log_alerts"
  },
  "alarm_tags": {
    "hs:app:maintenance-window": "true",
    "hs:std:app-name": "NA",
    "hs:app:amdb": "sdpmt_52009",
    "hs:app:sdp-priority": "2 - High",
    "hs:std:svc-operator": "NA",
    "hs:std:svc-software-owner": "{{context.hits.0._source.kubernetes_labels_team}}",
    "hs:app:monitored": "true",
    "hs:std:app-code": "NA",
    "hs:app:ticket-group": "Monitoring and Analytics - Testing"
  },
  "alarm_reason": "{{context.hits.0._source.message}}"
}
```
