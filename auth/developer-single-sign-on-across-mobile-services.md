# Abstract

This document outlines a proposal for Single Sign On across Mobile Services & Mobile Enabled Services provisioned on OpenShift. This proposal is *not* intended as a feature for end users of Mobile Apps. It *is* intended as a development time feature for Mobile developers.

## Terms

* Mobile Services - Services that provide mobile specific functionality for Mobile Apps at development time or runtime e.g. Push Notifications, Data Sync, CI/CD
* Mobile Enabled Services - Services that provide functionality for Mobile solutions at development time or runtime, but functionality is not specific to Mobile e.g. Keycloak, Metrics.

## Problem Description

As a developer builds up their Project by provisioning different services, they will likely need to setup or configure ascpects of those services. For example, if a developer provisions the Push Notifications service, they will need to configure Push credentials for Mobile Apps. Some services may also provide additional info or dasboards. Accessing these dashboards & performing setup or configuration will likely require signing into those services via some auth mechanism.

Each service will have 1 or more of the following:

* it's own auth mechanism
* ability to configure an external auth mechanism
* no auth mechanism

This could cause an inconsistent and poor user experience for the following reasons:

* different auth flow for each service
* different auth credentials for each service, managed independently

Additionaly, a service that has no auth mechanism, e.g. Prometheus, will need to be secured in some manner if exposed externally.

## Proposed Solution

### OpenShift as the Single Sign On Provider

All Services will be running on OpenShift, or 'ordered' from the OpenShift Catalog. OpenShift is the only constant between the Developer and the Services. The Developer will need to be signed in to OpenShift. Leveraging this fact and having OpenShift be the single auth provider for all Services will address the main points in the problem description.

The flow for signing in to each Service will be the same i.e. an OAuth flow where the user signs in & authorises the Service with OpenShift. No additional credentials will be required.

For cases where no auth mechanism is provided with a service, a reverse proxy approach that uses OpenShift OAuth, as shown in Option 3 below, can be used.

### Provider Integration Options

Choosing how to integrate a Service with OpenShift OAuth will depend on what the Service is capable of already, and how extensible it is. The below options show practical examples of how this integration was done. There will likely be tweaks and other options required depending on the service. These options serve as references for how it could be done.

#### Option 1: Configure OAuth in the Service

If a Service has the ability to an OAuth provider, use it. For example, in Keycloak it is possible to use 'Identity Brokering' to defer to OpenShift Auth for who can administer that Keycloak instance. See this [Keycloak Identity Brokering Blog Post](https://developers.redhat.com/blog/2017/12/06/keycloak-identity-brokering-openshift/) for details.

However, with Keycloak this method is limited to only automatically setting up the initial permissions of *all* users who can successfully authenticate. There is no permissions mapping between OpenShift roles & Keycloak administration roles. Additionally, *any* OpenShift user will be able to sign in to Keycloak. Manual configuration is required to only allow certain users access and limit their roles.

#### Option 2: Use a plugin for OAuth

If a Service allows plugins, it may be possible to use a plugin for auth. For example, the [Jenkins OpenShift Login Plugin](https://github.com/openshift/jenkins-openshift-login-plugin#openshift-login) allows OpenShift users to login to Jenkins via an OAuth flow. The plugin syncs and maps the users permissions in OpenShift to Jenkins permissions. The plugin can be configured to only authorise users who have read or edit access to the same namespace as where Jenkins is deployed.

#### Option 3: Add a reverse proxy

This solution can be used in the following situations:

* The Service has no authentication e.g. Prometheus
* The Service has authentiation, but supports a 'http reverse proxy' e.g. [Grafana](http://docs.grafana.org/installation/configuration/#auth-proxy)

An example of both these approaches was progressed in https://github.com/aerogearcatalog/prometheus-apb/blob/51cdd00f1c562537c72cbeb298a0c3fb00d321b8/roles/provision-prometheus-apb/tasks/main.yml
This example used [OAuth Proxy](https://github.com/openshift/oauth-proxy). The proxy presents the User with an OAuth flow for OpenShift users to login. The proxy can be configured to protect certain (or all) endpoints in the upstream service. It can also be configured to do a SubjectAccessReview (SAR) to ensure the user has certain permissions e.g. can a user 'view' 'Services' in the current namespace.