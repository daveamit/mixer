# Metrics and Alerting


## Default Metrics

By default, these are the metrics captured by mixer for prometheus

| Metric name| Kind | Labels/tags |
| ---------- | ----------- | ----------- |
| istio_mixer_requestcount | COUNTER | `source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_mixer_request_duration  | DISTRIBUTION |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_mixer_request_size  | DISTRIBUTION |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_mixer_response_size  | DISTRIBUTION |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_mixer_tcp_bytes_sent | COUNTER |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version` |
| istio_mixer_tcp_bytes_received | COUNTER |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version` |

## Alerts

### Nice to have alerts

* Requests timed out
* Large response times
* Spike in total request rate?
* increase in 4xx errors (specific service?)
* increase in 5xx errors (specific service?)
* calls by path from istio-ingress