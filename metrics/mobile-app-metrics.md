# Abstract

This document contains the proposal for gathering metrics on system events for running Mobile Apps and making them available to developers.

## Goals

### Primary Goals

- Provide mobile app developers a service that can receive overall runtime metrics (processor and memory data, authentication events, runtime versions, networking events, etc).

- Provide developers an interface to query and visualize the gathered data in order to make informed decisions towards deployment, deprecation of backend services and other actions that depend on the state of an application's installed base.

### Secondary Goals

- Provide mobile app developers with an easy to use SDK to connect easily and securely to the metrics service

### Non Goals

- Identify individual devices or users in metrics events

- Registering of user-defined metrics metrics

## Terms

For a list of terms used in this document, see [Glossary](./glossary.md).

## Problem Description

As mobile applications become more successful, system metrics data becomes invaluable for developers to observe the landscape of application installations and make sure further development and deployment of backend services will not affect users.

## Proposed Solution

A Metrics Service will be available from the Service Catalog, which can be provisioned by developers for use with their mobile applications.

The extra components outlined in this proposal will be added to the existing components outlined in the [Metrics Endpoints and Auto Discovery Proposal](./mobile-app-metrics-service.md).

The following diagram illustrates the components of the Metrics Service:

![service diagram](./app-service-diagram.jpg)

### Mobile App Client

A native, platform-specific SDK for targetting the Metrics Service will be available to developers.

Upon installation into a project and providing configuration for targetting the Metrics Service the SDK will automatically send general information about the general runtime for this application with a unique device ID, namely the `[SDK Version, clientID, timestamp]` tuple upon application initialization.

The SDK will also instrument calls at the network level in order to gather metrics around communication and bandwith consumption.

Other system resources utilization data can also be sent periodically.

### Mobile Metrics Service

A thin HTTP-based service will be present to receive metrics data from the mobile applications in an storage-agnostic way, then forwarding to the Storage service.

Compared to mobile applications interfacing directly with the Storage service, this presents the following tradeoff:

#### Pros

- Possibility to upgrade or change the Storage service independently of mobile client SDK version and implementation;

- Possible support for different Storage engines which can be selected based on application needs;

#### Cons

- Extra service deployment and scaling needs;

- Development and maintenance of the new service;

- Types of gathered metrics can become a subset of the underlying Storage's full range of types.

<!-- TODO: mention logstash, fluentd -->

### Storage

The Storage service will store metrics data forwarded from the Mobile Metrics Service. Its implementation can be supplied by the following projects:

#### Prometheus

##### Pros

- Shared service with backend metrics
- Requires an additional service for long-term metrics storage

##### Cons

- Data model is time-series exclusive
- Pull-based data gathering

#### ElasticSearch

##### Pros

- High advertised performance
- Highest variety of data types and modelling (supports time series data and non-time series data)
- Rich querying possibilities

##### Cons

- High minimum resource requirements (over evaluation-sized clusters)

#### MongoDB

##### Pros

- Supports time series data and non-time series data

##### Cons

- Support for time series data is not native thus needs modeling work
- Grafana/Kibana cannot talk to MongoDB without ugly workarounds
- We can't go with the approach Prometheus storing data in MongoDB since it is not supported

#### InfluxDB

##### Pros

- Long-term storage adapter for Prometheus
- HTTP-based API for push-based metrics gathering
- Flexible querying with direct comparison to SQL

##### Cons

- Focused on time-series data
- No clustering support in OSS offering
- Medium-sized minimum resource requirements

### Querying and Data Visualisation

The Querying and Visualization service will be supplied by Grafana, which can utilize the Storage service implementation directly as a Data Source.

User interface familiarity is an important factor for developers that use the overall metrics service, so addition of other visualisation tools like Kibana that would increase the learning curve of the solution should be avoided.

#### Authentication

As shared with the backend metrics service, Grafana's service will be behind OpenShift's [OAuth Proxy](https://github.com/openshift/oauth-proxy) that will pick up the permissions of the user's OpenShift cluster credentials.
