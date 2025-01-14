---
id: monitoring
title: Monitoring
description: Monitoring options of the ConfigCat Proxy.
toc_max_heading_level: 4
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';

This section will go through the monitoring options of the ConfigCat Proxy.

## Status Endpoint

The Proxy provides status information (health check) about its components on the following endpoint:

<details open>
  <summary><span className="endpoint"><span className="http-method green">GET</span><span className="http-method gray">OPTIONS</span>/status</span></summary>

The Proxy regularly checks whether the underlying SDKs can communicate with their configured source and with the cache. This endpoint returns the actual state of these checks.

If everything is operational, each `status` node shows the value `healthy`. If an SDK could not connect to its source, it'll put an error to its `records` collection. 
If a component's last two records are errors, its `status` will switch to `degraded`. 
If a component becomes operational again it'll put an `[ok]` to the `records` and will switch to `healthy` again. 

The root `status` is `healthy` if all of the SDKs are `healthy`. If any of the SDKs become `degraded`, the root will also switch to `degraded`.

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The status returned successfully.</li>
<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
</ul>

**Example Response**:
```json
{
  "status": "healthy",
  "sdks": {
    "my_sdk": {
      "key": "****************************************hwTYg",
      "mode": "online",
      "source": {
        "type": "remote",
        "status": "healthy",
        "records": [
          "Mon, 29 May 2023 16:36:40 UTC: [ok] config fetched"
        ]
      }
    },
    "another_sdk": {
      "key": "****************************************ovVnQ",
      "mode": "offline",
      "source": {
        "type": "cache",
        "status": "healthy",
        "records": [
          "Mon, 29 May 2023 16:36:40 UTC: [ok] reload from cache succeeded",
          "Mon, 29 May 2023 16:36:45 UTC: [ok] config from cache not modified"
        ]
      }
    }
  },
  "cache": {
    "status": "healthy",
    "records": [
      "Mon, 29 May 2023 16:36:40 UTC: [ok] cache read succeeded",
      "Mon, 29 May 2023 16:36:40 UTC: [ok] cache write succeeded",
      "Mon, 29 May 2023 16:36:40 UTC: [ok] cache read succeeded",
      "Mon, 29 May 2023 16:36:45 UTC: [ok] cache read succeeded"
    ]
  }
}
```

</details>

## Prometheus Metrics

You can set up the Proxy to export metrics about its internal state in Prometheus format. These metrics are served via the `/metrics` endpoint on a specific port, so you can separate it from the public HTTP communication. The default port is `8051`.

The following metrics are exported:

<table className="proxy-arg-table">
<thead><tr><th>Name</th><th>Type</th><th>Description</th></tr></thead>
<tbody>
<tr>
<td>

`configcat_http_request_duration_seconds`

</td>
<td>
Histogram
</td>
<td>
Histogram of Proxy HTTP response time in seconds.<br/><br/>
Tags:

- `route`: The request's URL path.
- `method`: The request's HTTP method.
- `status`: The response's HTTP status.

</td>
</tr>

<tr>
<td>

`configcat_sdk_http_request_duration_seconds`

</td>
<td>
Histogram
</td>
<td>
Histogram of ConfigCat CDN HTTP response time in seconds.<br/><br/>
Tags:

- `sdk`: The SDK's identifier that initiated the request.
- `route`: The request's URL path.
- `status`: The response's HTTP status.

</td>
</tr>

<tr>
<td>

`configcat_stream_connections`

</td>
<td>
Gauge
</td>
<td>
Number of active client connections per stream.<br/><br/>
Tags:

- `sdk`: The SDK's identifier that handles the connection.
- `type`: `sse` or `grpc`.
- `flag`: The feature flag's key.

</td>
</tr>
</tbody>
</table>

:::info
The Proxy also exports metrics about the Go environment, e.g., `go_goroutines` or `go_memstats_alloc_bytes`, and process-related stats, e.g., `process_cpu_seconds_total`.
:::

To integrate with Prometheus, put the following scrape config—that points to the Proxy—into your Prometheus configuration:

```yaml
scrape_configs:
  - job_name: configcat_proxy
    metrics_path: /metrics
    static_configs:
      - targets:
          - <proxy-host>:8051
```

### Available Options

The following metrics related options are available:

<table className="proxy-arg-table">
<thead><tr><th>Option</th><th>Default</th><th>Description</th></tr></thead>
<tbody>
<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
metrics:
  enabled: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_METRICS_ENABLED=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables Prometheus metrics.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
metrics:
  port: 8051
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_METRICS_PORT=8051
```

</TabItem>
</Tabs>

</td>
<td><code>8051</code></td>
<td>The port used for serving metrics.</td>
</tr>

</tbody>
</table>