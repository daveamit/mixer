This documents is a mess and will need to be cleaned up. it currently is being used to track the different parts of metric gatehering and alerting in prometheus. once the queries have been fleshed out, creating alerts should be trivial.


## Default Metrics

By default, these are the metrics and labels captured by mixer for prometheus

| Metric name| Kind | Labels/tags |
| ---------- | ----------- | ----------- |
| istio_requestcount | COUNTER | `source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_request_duration  | DISTRIBUTION |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_request_size  | DISTRIBUTION |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_response_size  | DISTRIBUTION |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version`<br/>`response_code` |
| istio_tcp_bytes_sent | COUNTER |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version` |
| istio_tcp_bytes_received | COUNTER |`source_service`<br/>`source_version`<br/>`destination_service`<br/>`destination_version` |

## Queries
Here are a list of prometheus queries that get us closer to creating alerts. currently these might not all be correct or monitor the right things but they are examples to get us closer.

*  Comparing the current rate of 5xx errors to the previous minute for each service
  * (wouldnt this metric assume a steady acceptable level of 500 errors and monitor the increase in frequency?)
```
rate(istio_mixer_request_count{response_code=~"5.*"}[1m]) -  rate(istio_mixer_request_count{response_code=~"5.*"}[1m] offset 1m)
```

* Comparing the current rate of 5xx errors to the previous minute for all traffic

```
sum(rate(istio_mixer_request_count{response_code=~"5.*"}[1m])) -  sum(rate(istio_mixer_request_count{response_code=~"5.*"}[1m] offset 1m))
```

* Comparing the current rate of 4xx errors to the previous minute for each service
```
rate(istio_mixer_request_count{response_code=~"4.*"}[1m]) -  rate(istio_mixer_request_count{response_code=~"4.*"}[1m] offset 1m)
```

* Comparing the current rate of 4xx errors to the previous minute for all traffic
```
sum(rate(istio_mixer_request_count{response_code=~"4.*"}[1m])) -  sum(rate(istio_mixer_request_count{response_code=~"4.*"}[1m] offset 1m))
```

* Rate of requests per second per destination service
```
sum(irate(istio_mixer_request_count[1m])) by (destination_service)
```

* Long running 90th percentile request duration
* Summary: Select any HTTP endpoints that have a 90th percentile latency higher than 50ms (0.05s) but only for the dimensional combinations that receive more than one request per second. We use the histogram_quantile() function for the percentile calculation here. It calculates the 90th percentile latency for each sub-dimension. To filter the resulting bad latencies and retain only those that receive more than one request per second. histogram_quantile is only suitable for usage with a Histogram metric.

```
 histogram_quantile(0.9, rate(istio_mixer_request_duration_bucket[5m])) > 0.05
and
    rate(istio_mixer_request_duration_count[5m]) > 1
```

# Alerts

Once the above queries have been straightened out we should be able to create alerts from them. Below is few alerts that would be nice to have.

## Nice to have alerts
* Requests timed out
* Large response times
* Spike in total request rate?
* increase in 4xx errors (specific service?)
* increase in 5xx errors (specific service?)
* calls by path from istio-ingress