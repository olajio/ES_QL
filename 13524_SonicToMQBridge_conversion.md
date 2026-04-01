This watcher is nearly identical to the MQToSonicBridge variant — the only difference in the error query is `service.name: "SonicToMQBridge_Container"` instead of `"MQToSonicBridge_Container"`.

**13524_SonicToMQBridge Stage 1:**

```
FROM prod:filebeat-*
| WHERE @timestamp > NOW() - 5 minutes AND message LIKE "*MQRC_CONNECTION_BROKEN*" AND service.name == "SonicToMQBridge_Container" AND log.level == "ERROR" AND labels.client_code == "MDBOPROD"
| STATS message = TOP(message, 1, "desc"), client_code = TOP(labels.client_code, 1, "desc"), app_code = TOP(labels.app_code, 1, "desc") BY host.hostname
| LIMIT 10000
```

**13524_SonicToMQBridge Stage 2:**

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

Stage 2 and the message actions are identical to the MQToSonicBridge variant since they share the same event_type `"13524"`, metadata, and links. The only distinguishing piece is the `service.name` filter in Stage 1.
