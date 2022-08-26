---
id: upload-metrics-to-http-source
---

# Upload Metrics to an HTTP Source

After you have added an [HTTP Logs and Metrics Source](/docs/send-data/sources/hosted-collectors/http-logs-metrics-source) to a [Hosted Collector](../../../configure-hosted-collector.md) you can begin uploading data. You can upload both logs and metrics to the same HTTP source, however not in the same HTTP request. This document provides instructions on uploading metrics, if you are uploading logs see [Upload Logs to an HTTP Source](/docs/send-data/sources/hosted-collectors/http-logs-metrics-source).

:::note
Sumo Logic enforces limits on the volume of metrics and associated metadata you ingest. For more information, see Data Limits for Metrics. 
:::

## Metrics units requirements

Metrics you upload to an HTTP source must be in seconds or milliseconds. Metrics in microseconds are not supported.

## Content-Type headers for metrics

The table below lists the types of metrics you can upload to an HTTP Source and Content-Type header you must use when uploading metrics of each type. If you omit the Content-Type header, or set a value not listed below, the HTTP payload will be ingested as log data. 

You must specify the Content-Type header value exactly as shown in the table below. If a Content-Type header value contains any extra text other one of the Content-Type values below, Sumo ignores the extraneous text. (Some metric frameworks  append or prepend  extra text in Content-Type headers.)

So, Sumo will treat a `Content-Type` value of: 

```
application/vnd.sumologic.graphite; charset=utf8
```

as 

```
application/vnd.sumologic.graphite
```

Based on the Content-Type header specified, Sumo parses and interprets each line in the HTTP payload in the desired metric format.

| Format | Content-Type Header |
|--|--|
| [Graphite](http://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-plaintext-protocol) | `application/vnd.sumologic.graphite` |
| [Carbon 2.0](http://metrics20.org/implementations/) | `application/vnd.sumologic.carbon2` |
| [Prometheus](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md) | `application/vnd.sumologic.prometheus`<br/>Sumo won't ingest Prometheus comments or malformed Prometheus metrics. For more information, see [Prometheus metrics not accepted by Sumo](prometheus-metrics-not-accepted-by-sumo.md). |

:::note
For information about metric formats, see [Metric Formats](https://help.sumologic.com/Metrics/Introduction-to-Metrics/Metric-Formats).
:::

## Sending multiple metrics in a single payload

To minimize HTTP overhead, Sumo supports sending multiple metrics in a single HTTP payload. To send multiple metrics in a single HTTP payload, specify each metric in the same format, separated by newlines:

**Content-Type: application/vnd.sumologic.graphite**

```
prod.lb-1.cpu 87.2 1501753030
prod.lb-1.memory 32390 1501753030
prod.lb-1.disk 2.2 1501753030
prod.lb-1.cpu 84.6 1501753040
prod.lb-1.memory 32250 1501753040
...
```

**Content-Type: application/vnd.sumologic.carbon2**

```
cluster=prod node=lb-1 metric=cpu ip=2.2.3.4  team=infra 87.2 1501753030
cluster=prod node=lb-1 metric=memory ip=2.2.3.4  team=infra 32390 1501753030
cluster=prod node=lb-1 metric=disk ip=2.2.3.4  team=infra 2.2 1501753030
cluster=prod node=lb-1 metric=cpu ip=2.2.3.4  team=infra 84.6 1501753040
cluster=prod node=lb-1 metric=memory ip=2.2.3.4  team=infra 32250 1501753040
...
```

:::note
In the Carbon 2.0 format, you must put two spaces after the intrinsic tags.
:::

**Content-Type: application/vnd.sumologic.prometheus**

```
# HELP net_conntrack_dialer_conn_failed_total Total number of connections failed to dial by the dialer a given name.
# TYPE net_conntrack_dialer_conn_failed_total counter
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="refused"} 7 1530708469
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="resolution"} 5
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="timeout"} 0 1530708471
net_conntrack_dialer_conn_failed_total{dialer_name="default",reason="unknown"} 3
```

## Supported HTTP headers

The table below lists parameter you can specify in HTTP headers when sending data to an HTTP Source. The settings will apply to all messages in the request. For Source Name, Host, and Category, the header value will override any default value already specified for the Source and/or Collector.

:::note
Overridden metadata field values are not returned with [Search Autocomplete](../../../../search/get-started-with-search/search-basics/search-autocomplete.md).
:::

| Setting | Header Name | Header Value |
|--|--|--|
| Compressed data | `Content-Encoding` | gzip or deflate<br/>Required if you are uploading compressed data. | 
| Content Type (for Metrics) | Content-Type | `application/vnd.sumologic.graphite`<br/>`application/vnd.sumologic.carbon2`<br/>`application/vnd.sumologic.prometheus`<br/>Required if you are uploading metrics. |
| Custom Source Name | `X-Sumo-Name` | Desired source name.<br/>Useful if you want to override the source name configured for the source. | 
| Custom Source Host | `X-Sumo-Host` | Desired host name.<br/>Useful if you want to override the source host configured for the source. | 
| Custom Source Category | `X-Sumo-Category` | Desired source category.<br/>Useful if you want to override the source category configured for the source. | 
| Custom Metric Dimensions | `X-Sumo-Dimensions` | Comma-separated key=value list of dimensions to apply to every metric.<br/>For metrics only. Custom dimensions will allow you to query your metrics at a more granular level. | 
| Custom Metric Metadata | `X-Sumo-Metadata` | Comma-separated, key=value list of metadata to apply to every metric.<br/>For metrics only. Custom metadata  will allow you to query your metrics at a more granular level. |

## Data volume and metadata limits for metrics

For information about the limits Sumo Logic enforces on the metrics you ingest to Sumo Logic, including data volume limits and metadata-related limits, see Data Limits for Metrics.

## Command line examples using cURL

When using cURL to POST data from a file: 

 * Make sure to use the `-T` parameter to specify the file path, not `-d`.   The `-d` parameter causes new lines to be removed from the content, which will interfere with message boundary detection.
 * Make sure that each line in the file follows the format specified by the `Content-Type` header for the HTTP request.

**POST upload ([Graphite](http://metrics20.org/implementations/)-formatted metrics)**

```bash
curl -v -X POST -H 'Content-Type:application/vnd.sumologic.graphite' -T [local_file_name] https://collectors.sumologic.com/receiver/v1/http/[UniqueHTTPCollectorCode] 
```

**POST upload ([Carbon 2.0](http://graphite.readthedocs.io/en/latest/feeding-carbon.html#the-plaintext-protocol)-formatted metrics)**

```bash
curl -v -X POST -H 'Content-Type:application/vnd.sumologic.carbon2' -T [local_file_name] https://collectors.sumologic.com/receiver/v1/http/[UniqueHTTPCollectorCode]
```

**POST upload (gzip compressed [Graphite](http://metrics20.org/implementations/)-formatted metrics)** 

```bash
curl -v -X POST -H 'Content-Encoding:gzip' -H 'Content-Type:application/vnd.sumologic.graphite' -T [local_file_name.gz] https://collectors.sumologic.com/receiver/v1/http/[UniqueHTTPCollectorCode]
```

**POST upload ([Prometheus](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md)-formatted metrics)**

```bash
curl -v -X POST -H 'Content-Type:application/vnd.sumologic.prometheus' -T [local_file_name] http://collectors.sumologic.com/receiver/v1/http/[UniqueHTTPCollectorCode]
```