# Abstract

This document outlines a proposal for gathering metrics from Mobile Services running in a Kubernetes environment and visualizing them.

## Goals

### Primary Goals

* Provide a mechanism for Developers to gather, view & query metrics from supported Mobile Services.

### Secondary Goals

* Allow Developers to leverage the same mechanism for any other/non-mobile/custom services.

### Non Goals
* Provide an Analytics service
* Provide a Mobile SDK for Mobile Developers to store arbitrary metrics

## Terms

For a list of terms used in this document, see [Glossary](./glossary.md).

## Problem Description

During the Mobile Control Panel (MCP) proof of concept (Aug-Nov 2017), the `mcp-standalone` server had functionality added to gather metrics from Mobile Services. This functionality gathered metrics from 2 components:

* Keycloak - by polling the Events API
* Data Sync - by polling the `/stats` endpoint

The resulting metrics were stored in-memory in the `mcp-standalone` server. Endpoints were exposed to allow the MCP UI to request metrics and render them using Patternfly Widgets e.g. line graphs & counters. In order for the visualisations to be more useful and visually appealing, custom UI code was written for each service, so as to arrange the visualisations in a particular way. For example, queues and processing times were grouped for the Data Sync UI.

This approach surfaced a number of problems:

* Service specific logic required to be written and maintained in the 'mcp-standalone' server
* Service specific logic required to be written and maintained in the MCP UI
* Memory footprint of the `mcp-standalone` server was indeterminate

As the number of Mobile Services and supported other Services increases, the size and maintenance overhead of the server & UI would be very large.

Also, post proof of concept, it was decided the `mcp-standalone` server would not be continued as a concept, or any other 'required' server component specific to Mobile.

## Proposed Solution

### Metrics Service

No central metrics gathering or visualisations will be available for Mobile Services 'out of the box'. Instead, if a developer wants metrics, they can provision a Metrics Service from the Service Catalog.

The Metrics Service will consist of:

* Prometheus - for metrics gathering
* Grafana - for metrics visualisation

Prometheus & Grafana will each have a Persistent Volume for storing gathered metrics and visualisation settings respectively.

Grafana will be configured to have Proemtheus available as a 'Data Source' by default. Data Sources will be stored in a ConfigMap that gets mounted into the Grafana container. e.g.

https://github.com/aerogearcatalog/prometheus-apb/blob/ee86cfc67caaeaabcf7e7bc214c3a6dc6940c333/roles/provision-prometheus-apb/tasks/main.yml#L25-L26

```yaml
datasources:
- name: Prometheus
  type: prometheus
  access: proxy
  url: http://localhost:9090
  is_default: true
  version: 1
  editable: true
  json_data: {"timeInterval": "5s"}
```

### Metrics Discovery

Prometheus will be configured to 'auto-discover' any Services in the same Kubernetes namespace that expose a Prometheus style metrics endpoint. This will be done via an annoation on the `Service` resource.

For example, if a Service is exposing a HTTP metrics endpoint at `/metrics`, the annoation would be:

```yaml
org.aerogear.metrics/plain_endpoint: /metrics
```

If a Service is exposing a HTTPS metrics endpoint at `/rest/metrics`, the annoation would be:

```yaml
org.aerogear.metrics/ssl_endpoint: /rest/metrics
```

### Default Visualisations

Grafana will be made available as part of provisioning Metrics. Developers will be free to add their own Dashboards.

The 5.x stream of Grafana has the ability to load dashboards from disk and update the dashboards when the files on disk are also updated. This is possible using something called a provider. A provider is a configuration item that tells Grafana how and where to load dashboards. The idea is in the future Grafana may have different providers for loading dashboards from disk, source control, via an API, etc.

In the grafana `.ini` file, a provisioning path is configured:

```ini
provisioning = /etc/grafana/conf/provisioning
```

The `provisioning` directory contains two subdirectories:

* `datasources` - A directory with datasource definition files.
* `dashboards` - A directory with dashboard providers. (Not the actual dashboard definitions)

In the `dashboards` directory we will have a provider that looks like this:

```yaml
- name: 'default'
  org_id: 1
  folder: ''
  type: file
  options:
    path: /etc/grafana-dashboards
```

This file instructs Grafana to search for dashboard definitions in `/etc/grafana-dashboards`. Dashboard files can be inserted and modified at any time and the changes will be reflected in Grafana.

This approach allows us to load the datasources, dashboard providers and dashboard definitions using Config Maps. Dashboard definitions for individual mobile enabled services can be defined in their respective APBs. As part of the provision, the APB can append or update the dashboard in the dashboards Config Map.

One question arising from this approach is 'What if downstream users want to define their own dashboards and have them discovered?' Allowing users to pollute the Config Map that also stores the upstream service dashboards is not a good approach. To mitigate this, we could configure Grafana to scan another directory e.g. `user-dashboards`, allowing user dashboards to be stored and discovered in a separate Config Map.

### Auth

Both Prometheus and Grafana will be accessible to users who have an account in the OpenShift cluster where they are deployed. This will allow for easier use of those services via single sign on.

This can be achieved by using an [oauth proxy](https://github.com/openshift/oauth-proxy) in front of either service. This proxy will use an OAuth flow to authenticate the User against the OpenShift cluster. The namespace will have a ServiceAccount Oauth Client i.e. has the below annotations to allow OAuth redirects to the Grafana & Prometheus proxy routes.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: proxy
  annotations:
    serviceaccounts.openshift.io/oauth-redirectreference.grafana: '{"kind":"OAuthRedirectReference","apiVersion":"v1","reference":{"kind":"Route","name":"grafana"}}'
    serviceaccounts.openshift.io/oauth-redirectreference.prometheus: '{"kind":"OAuthRedirectReference","apiVersion":"v1","reference":{"kind":"Route","name":"prometheus"}}'
```

The Prometheus Data Source in Grafana will configured to bypass the OAuth Proxy. This can be achieved via an internal Service that proxies directly to the Prometheus API port. The exposed Route to Prometheus for end-user access will be use a Service that *does* go through the OAuth Proxy.
This approach is dependant on projects having isolated networks so that the Prometheus internal Service cannot be accessed from other projects.

### Scaling

The Grafana Deployment will be configured to allow scaling of replicas. Each Grafana replica will use a shared Read-Write-Many (RWX) Persistent Volume.

Prometheus will *not* be configured for scaling.

### Links

* Proof of concept Ansible Playbook Bundle for Prometheus & Grafana, including single sign on - https://github.com/aerogearcatalog/prometheus-apb
* Proof of concept demo - https://youtu.be/-37OPXXhrTw
* Grafana - http://docs.grafana.org/
* Prometheus - https://prometheus.io/docs/introduction/overview/
