---
id: endpoints
title: Endpoints
description: HTTP endpoints of the ConfigCat Proxy.
toc_max_heading_level: 4
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';
import CodeBlock from '@theme/CodeBlock';

The Proxy accepts HTTP requests on the following endpoints.

## CDN Proxy

The CDN proxy endpoint's purpose is to forward the underlying *config JSON* to other ConfigCat SDKs used by your application.  

<details open>
  <summary><span className="endpoint"><span className="http-method green">GET</span><span className="http-method gray">OPTIONS</span>/configuration-files/&#123;sdkId&#125;/&#123;config-json-file&#125;</span></summary>

This endpoint is mainly used by ConfigCat SDKs to retrieve all required data for feature flag evaluation. 

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.
- `config-json-file`: It's set by the ConfigCat SDK, it determines which *config JSON* schema must be used.  

**Responses**:
<ul className="responses">
  <li className="success"><span className="status">200</span>: The <code>config.json</code> file is downloaded successfully.</li>
  <li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
  <li className="success"><span className="status">304</span>: The <code>config.json</code> file isn't modified based on the <code>Etag</code> sent in the <code>If-None-Match</code> header.</li>
  <li className="error"><span className="status">400</span>: The <code>sdkId</code> is missing.</li>
  <li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>
</details>

### SDK Usage

In order to let a ConfigCat SDK use the Proxy, you have to set the SDK's `baseUrl` parameter to point to the Proxy's host.
Also, you have to pass the [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) as the SDK key.

So, let's assume you set up the Proxy with the following SDK option:

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml title="options.yml"
sdks:
  my_sdk:
    key: "<your-sdk-key>"
```

</TabItem>
<TabItem value="env-vars" label="Environment variables">

```shell
CONFIGCAT_SDKS={"my_sdk":"<your-sdk-key>"}
```

</TabItem>
</Tabs>

The SDK's initialization that works with the Proxy will look like this:

```js title="example.js"
import * as configcat from "configcat-js";

var configCatClient = configcat.getClient(
  // highlight-next-line
  "my_sdk", // SDK identifier as SDK key
  configcat.PollingMode.AutoPoll,
  // highlight-next-line
  { baseUrl: "http(s)://localhost:8050" } // Proxy URL
);
```

### Available Options

The following CDN Proxy related options are available:

<table className="proxy-arg-table">
<thead><tr><th>Option</th><th>Default</th><th>Description</th></tr></thead>
<tbody>
<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  cdn_proxy:
    enabled: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_CDN_PROXY_ENABLED=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the CDN proxy endpoint, which can be used by ConfigCat SDKs in your applications.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  cdn_proxy:
    allow_cors: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_CDN_PROXY_ALLOW_CORS=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the sending of CORS headers. It can be used to restrict access to specific domains. The default allowed origin is set to <code>*</code>, but you can override it with the <code>CONFIGCAT_HTTP_CDN_PROXY_HEADERS</code> option.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  cdn_proxy:
    headers:
      Access-Control-Allow-Origin: "https://yourdomain.com"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_CDN_PROXY_HEADERS='{"Access-Control-Allow-Origin":"https://yourdomain.com"}'
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Additional headers that must be sent back on each CDN proxy endpoint response.</td>
</tr>

</tbody>
</table>

## API

The API endpoints are for server side feature flag evaluation.

<details>
  <summary><span className="endpoint"><span className="http-method blue">POST</span><span className="http-method gray">OPTIONS</span>/api/&#123;sdkId&#125;/eval</span></summary>

This endpoint evaluates a single feature flag identified by a `key` with the given [user object](/advanced/user-object). 

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  

**Request body**:
```json
{
  "key": "<feature-flag-key>",
  "user": {
    "Identifier": "<user-id>",
    "Email": "<user-email>",
    "Country": "<user-country>",
    // any other attribute
  }
}
```

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The feature flag evaluation finished successfully.<br/>
<div className="response-body">Response body:</div>

```json
{
  "value": <evaluated-value>,
  "variationId": "<variation-id>"
}
```

</li>
<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code> or the <code>key</code> from the request body is missing.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

</details>

<details>
  <summary><span className="endpoint"><span className="http-method blue">POST</span><span className="http-method gray">OPTIONS</span>/api/&#123;sdkId&#125;/eval-all</span></summary>

This endpoint evaluates all feature flags with the given [user object](/advanced/user-object). 

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  

**Request body**:
```json
{
  "user": {
    "Identifier": "<user-id>",
    "Email": "<user-email>",
    "Country": "<user-country>",
    // any other attribute
  }
}
```

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The evaluation of all feature flags finished successfully.<br/>
<div className="response-body">Response body:</div>

```json
{
  "feature-flag-key-1": {
    "value": <evaluated-value>,
    "variationId": "<variation-id>"
  },
  "feature-flag-key-2": {
    "value": <evaluated-value>,
    "variationId": "<variation-id>"
  }
}
```

</li>
<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code> is missing.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

</details>

<details>
  <summary><span className="endpoint"><span className="http-method blue">POST</span><span className="http-method gray">OPTIONS</span>/api/&#123;sdkId&#125;/refresh</span></summary>

This endpoint commands the underlying SDK to download the latest available *config JSON*. 

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The refresh was successful.</li>
<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code> is missing.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

</details>

<details>
  <summary><span className="endpoint"><span className="http-method green">GET</span><span className="http-method gray">OPTIONS</span>/api/&#123;sdkId&#125;/keys</span></summary>

This endpoint returns all feature flag keys belonging to the given [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key). 

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The keys are returned successfully.<br/>
<div className="response-body">Response body:</div>

```json
{
  "keys": [
    "feature-flag-key-1",
    "feature-flag-key-1"
  ]
}
```

</li>
<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code> is missing.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

</details>

### Available Options

The following API related options are available:

<table className="proxy-arg-table">
<thead><tr><th>Option</th><th>Default</th><th>Description</th></tr></thead>
<tbody>
<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  api:
    enabled: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_API_ENABLED=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the API endpoints, which can be used for server side feature flag evaluation.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  api:
    allow_cors: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_API_ALLOW_CORS=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the sending of CORS headers. It can be used to restrict access to specific domains. The default allowed origin is set to <code>*</code>, but you can override it with the <code>CONFIGCAT_HTTP_API_HEADERS</code> option.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  api:
    headers:
      Access-Control-Allow-Origin: "https://yourdomain.com"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_API_HEADERS='{"Access-Control-Allow-Origin":"https://yourdomain.com"}'
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Additional headers that must be sent back on each API endpoint response.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  api:
    auth_headers:
      X-API-KEY: "<auth-value>"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_API_AUTH_HEADERS='{"X-API-KEY":"<auth-value>"}'
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Additional headers that must be on each request sent to the API endpoints. If the request doesn't include the specified header, or the values are not matching, the Proxy will respond with a <code>401</code> HTTP status code.</td>
</tr>

</tbody>
</table>

## SSE

The SSE endpoint allows you to subscribe for feature flag value changes through <a target="blank" href="https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events">Server-Sent Events</a> connections.

<details>
  <summary><span className="endpoint"><span className="http-method green">GET</span><span className="http-method gray">OPTIONS</span>/sse/&#123;sdkId&#125;/eval/&#123;data&#125;</span></summary> 

This endpoint subscribes to a single flag's changes. Whenever the watched flag's value changes, the Proxy sends the new value to each connected client.

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  
- `data`: The `base64` encoded input data for feature flag evaluation that must contain the feature flag's key and a [user object](/advanced/user-object).

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The SSE connection established successfully.</li>
<div className="response-body">Response body:</div>

```json
{
  "value": <evaluated-value>,
  "variationId": "<variation-id>"
}
```

<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code>, <code>data</code>, or the <code>key</code> attribute of <code>data</code> is missing.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

**Example**:
```js title="example.js"
const rawData = {
  key: "<feature-flag-key>",
  user: {
    Identifier: "<user-id>",
    Email: "<user-email>",
    Country: "<user-country>",
    // any other attribute
  }
}

const data = btoa(JSON.stringify(rawData))
const evtSource = new EventSource("http(s)://localhost:8050/sse/my_sdk/eval/" + data);
evtSource.onmessage = (event) => {
  console.log(event.data); // {"value":<evaluated-value>,"variationId":"<variation-id>"}
};
```

</details>

<details>
  <summary><span className="endpoint"><span className="http-method green">GET</span><span className="http-method gray">OPTIONS</span>/sse/&#123;sdkId&#125;/eval-all/&#123;data&#125;</span></summary> 

This endpoint subscribes to all feature flags' changes behind the given [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key). When any of the watched flags' value change, the Proxy sends its new value to each connected client.

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  
- `data`: **Optional**. The `base64` encoded input data for feature flag evaluation that contains a [user object](/advanced/user-object).

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The SSE connection established successfully.</li>
<div className="response-body">Response body:</div>

```json
{
  "feature-flag-key-1": {
    "value": <evaluated-value>,
    "variationId": "<variation-id>"
  },
  "feature-flag-key-2": {
    "value": <evaluated-value>,
    "variationId": "<variation-id>"
  }
}
```

<li className="success"><span className="status">204</span>: In response to an <code>OPTIONS</code> request.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code> is missing.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

**Example**:
```js title="example.js"
const rawData = {
  user: {
    Identifier: "<user-id>",
    Email: "<user-email>",
    Country: "<user-country>",
    // any other attribute
  }
}

const data = btoa(JSON.stringify(rawData))
const evtSource = new EventSource("http(s)://localhost:8050/sse/my_sdk/eval-all/" + data);
evtSource.onmessage = (event) => {
  console.log(event.data); // {"feature-flag-key":{"value":<evaluated-value>,"variationId":"<variation-id>"}}
};
```

</details>

### Available Options

The following SSE related options are available:

<table className="proxy-arg-table">
<thead><tr><th>Option</th><th>Default</th><th>Description</th></tr></thead>
<tbody>
<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  sse:
    enabled: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_SSE_ENABLED=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the SSE endpoint, which can be used for streaming feature flag value changes.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  sse:
    allow_cors: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_SSE_ALLOW_CORS=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the sending of CORS headers. It can be used to restrict access to specific domains. The default allowed origin is set to <code>*</code>, but you can override it with the <code>CONFIGCAT_HTTP_SSE_HEADERS</code> option.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  sse:
    headers:
      Access-Control-Allow-Origin: "https://yourdomain.com"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_SSE_HEADERS='{"Access-Control-Allow-Origin":"https://yourdomain.com"}'
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Additional headers that must be sent back on each <a href="#sse">SSE endpoint</a> response.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  sse:
    log:
      level: "<error|warn|info|debug>"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_SSE_LOG_LEVEL="<error|warn|info|debug>"
```

</TabItem>
</Tabs>

</td>
<td><code>warn</code></td>
<td>The verbosity of the SSE related logs.<br />Possible values: <code>error</code>, <code>warn</code>, <code>info</code> or <code>debug</code>.</td>
</tr>

</tbody>
</table>

## Webhook

Through the webhook endpoint, you can notify the Proxy about the availability of new feature flag evaluation data. Also, with the appropriate [SDK options](/advanced/proxy/proxy-overview/#additional-sdk-options), the Proxy can [validate the signature](/advanced/notifications-webhooks/#verifying-webhook-requests) of each incoming webhook request.

<details open>
  <summary><span className="endpoint"><span className="http-method green">GET</span><span className="http-method blue">POST</span>/hook/&#123;sdkId&#125;</span></summary>

Notifies the Proxy that the SDK with the given [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) must refresh its *config JSON* to the latest version. 

**Route parameters**:
- `sdkId`: The [SDK identifier](/advanced/proxy/proxy-overview/#sdk-identifier--sdk-key) that uniquely identifies an SDK within the Proxy.  

**Responses**:
<ul className="responses">
<li className="success"><span className="status">200</span>: The Proxy accepted the notification.</li>
<li className="error"><span className="status">400</span>: The <code>sdkId</code> is missing or the <a href="/advanced/notifications-webhooks/#verifying-webhook-requests">webhook signature validation</a> failed.</li>
<li className="error"><span className="status">404</span>: The <code>sdkId</code> is pointing to a non-existent SDK.</li>
</ul>

</details>

### ConfigCat Dashboard

You can set up webhooks to invoke the Proxy on the <a target="blank" href="https://app.configcat.com/product/webhooks">Webhooks page</a> of the ConfigCat Dashboard.

<img className="bordered zoomable" src="/docs/assets/proxy/webhook.png" alt="Webhook" />

### Available Options

The following webhook related options are available:

<table className="proxy-arg-table">
<thead><tr><th>Option</th><th>Default</th><th>Description</th></tr></thead>
<tbody>
<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  webhook:
    enabled: <true|false>
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_WEBHOOK_ENABLED=<true|false>
```

</TabItem>
</Tabs>

</td>
<td><code>true</code></td>
<td>Enables or disables the Webhook endpoint, which can be used for notifying the Proxy about the availability of new feature flag evaluation data.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  webhook:
    auth:
      user: "<auth-user>"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_WEBHOOK_AUTH_USER="<auth-user>"
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Basic authentication user. The basic authentication webhook header can be set on the <a target="blank" href="https://app.configcat.com/product/webhooks">Webhooks page</a> of the ConfigCat Dashboard.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  webhook:
    auth:
      password: "<auth-pass>"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_WEBHOOK_AUTH_PASSWORD="<auth-pass>"
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Basic authentication password. The basic authentication webhook header can be set on the <a target="blank" href="https://app.configcat.com/product/webhooks">Webhooks page</a> of the ConfigCat Dashboard.</td>
</tr>

<tr>
<td>

<Tabs groupId="yaml-env">
<TabItem value="yaml" label="YAML" default>

```yaml
http:
  webhook:
    auth_headers:
      X-API-KEY: "<auth-value>"
```

</TabItem>
<TabItem value="env-vars" label="Environment variable">

```shell
CONFIGCAT_HTTP_WEBHOOK_AUTH_HEADERS='{"X-API-KEY":"<auth-value>"}'
```

</TabItem>
</Tabs>

</td>
<td>-</td>
<td>Additional headers that ConfigCat must send with each request to the Webhook endpoint. Webhook headers can be set on the <a target="blank" href="https://app.configcat.com/product/webhooks">Webhooks page</a> of the ConfigCat Dashboard.</td>
</tr>

</tbody>
</table>
