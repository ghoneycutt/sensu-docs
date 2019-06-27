---
title: "Get started with the Sensu API"
linkTitle: "API Quickstart"
description: "The Sensu backend API provides access to Sensu workflow configurations and monitoring event data. Read this guide for an overview of the Sensu Go API, including URL format, data format, versioning, and more."
weight: 5
version: "5.10"
product: "Sensu Go"
menu:
  sensu-go-5.10:
    parent: api
---

The Sensu API accepts HTTP requests at the URL defined by the Sensu backend ___ attribute (default: ).
[Resource management endpoints]() are available at `/api/core/v2/namespaces/{namespace}/{resource type}`.
In this example, we'll use the events API to see the latest monitoring events within the `default` namespace.

**1. Getting your access token**

Before you start, set up your access token using your Sensu username and password.
Here's an example that sets a `SENSU_TOKEN` variable for the [default admin user][] using the [curl][] and [jq]{} utilities.

{{< highlight shell >}}
export SENSU_USER=admin && SENSU_PASS=P@ssw0rd!

export SENSU_TOKEN=`curl -XGET -u "$SENSU_USER:$SENSU_PASS" -s http://localhost:8080/auth | jq -r ".access_token"`
{{< /highlight >}}

For more information about API authentication, see the [API overview][].

**2. Fetch the latest monitoring events**

Use the events API to see the latest Sensu monitoring events.

{{< highlight shell >}}
curl -H "Authorization: Bearer $SENSU_TOKEN" \
http://127.0.0.1:8080/api/core/v2/namespaces/default/events
{{< /highlight >}}

You should see a 200 response and a JSON array of events.

{{< highlight shell >}}
HTTP/1.1 200 OK
[
  {
    "timestamp": 1542667666,
    "entity": {
      "entity_class": "agent",
      "system": {
        "hostname": "webserver01",
        "...": "...",
        "arch": "amd64"
      },
      "subscriptions": [
        "testing",
        "entity:webserver01"
      ],
      "metadata": {
        "name": "check-nginx",
        "namespace": "default",
        "labels": null,
        "annotations": null
      }
    },
    "check": {
      "check_hooks": null,
      "duration": 2.033888684,
      "command": "http_check.sh http://localhost:80",
      "handlers": [
        "slack"
      ],
      "high_flap_threshold": 0,
      "interval": 20,
      "low_flap_threshold": 0,
      "publish": true,
      "runtime_assets": [],
      "subscriptions": [
        "testing"
      ],
      "proxy_entity_name": "",
      "check_hooks": null,
      "stdin": false,
      "ttl": 0,
      "timeout": 0,
      "duration": 0.010849143,
      "output": "",
      "state": "failing",
      "status": 1,
      "total_state_change": 0,
      "last_ok": 0,
      "occurrences": 1,
      "occurrences_watermark": 1,
      "output_metric_format": "",
      "output_metric_handlers": [],
      "env_vars": null,
      "metadata": {
        "name": "check-nginx",
        "namespace": "default",
        "labels": null,
        "annotations": null
      }
    }
  }
]
{{< /highlight >}}

To learn about resource objects and attributes, see the references docs.

**4. Refreshing your token**

{{< highlight shell >}}
export SENSU_TOKEN=`curl -XGET -u "$SENSU_USER:$SENSU_PASS" -s http://localhost:8080/auth | jq -r ".access_token"`
{{< /highlight >}}

### Resource APIs

- [Assets API] ([spec](../../reference/))
- [Checks API] ([spec](../../reference/))
- [Entities API] ([spec](../../reference/))
- [Events API] ([spec](../../reference/))
- [Filters API] ([spec](../../reference/))
- [Handlers API] ([spec](../../reference/))
- [Hooks API] ([spec](../../reference/))
- [Roles and bindings API] ([spec](../../reference/))
- [Silences API] ([spec](../../reference/))

### Admin APIs

- Auth
- Auth providers
- Clustr roles and bindings
- cluster
- Health
- license
- metrics
- Tessen
- Users
- Version

### Agent API

For the Sensu agent API, see the [agent reference][4].
