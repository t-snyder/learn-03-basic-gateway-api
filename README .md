<h1>Prototypes Using Kubernetes Gateway API, Istio Ambient Mode, and Cert-Manager</h1>

<h2>Prototype Overview</h2>
Kubernetes, abbreviated as K8s, is a powerful open-source platform designed to
automate the deployment, scaling, and management of containerized applications. 
Originally designed and developed by Google, it was open sourced to the Linux 
Foundation in June 2014. By July 2014 IBM, Redhat, Microsoft, and Docker had 
joined the Kubernetes community.

## Per Wikipedia, Kubernetes is one of the most widely deployed software
systems in the world. All major Cloud Providers (AWS, Azure, Google, Alibaba, IBM, Oracle, Digital
Ocean, etc. ) having Kubernetes based offerings. Currently the majority
of Cloud deployments are deployed using Kubernetes as the container
infrastructure. * CNCF Annual Survey 2022 (https://www.cncf.io/reports/cncf-annual-survey-2022/)**. CNCF.
January 31, 2023)

This adoption path and resulting experience has informed the ecosystem
as to what is working and what could work better, resulting in a rapidly
evolving and expanding platform.

This set of prototypes target 2 newer components of the Kubernetes
environment. The first is an evolutionary path from Ingress resources to
the Kubernetes Gateway API. The second is an evolution of the Istio
Service Mesh from sidecars to Ambient mode. As the prototypes become
more complex additional supporting deployments are included - Metallb
Load Balancer, Cert-Manager, and in a subsequent set of prototypes,
Hashicorp Vault.

## Prototype Purpose
The purpose of these prototypes is to provide a working deployment and
test environment for the new functionality (Kubernetes Gateway API and
Istio Ambient mode). The deployments are not production ready but
provide a basis for understanding how to use and deploy the
functionality.

## Evolution from Ingress to Kubernetes Gateway API
Per the Kubernetes Gateway API documentation - Gateway API is an
official Kubernetes project focused on L4 and L7 routing in Kubernetes.
This project represents the next generation of Kubernetes Ingress, Load
Balancing, and Service Mesh APIs. From the outset, it has been designed
to be generic, expressive, and role-oriented.
(https://gateway-api.sigs.k8s.io/)

The Gateway API is a relatively new API in the Kubernetes ecosystem that
provides a standardized way to manage ingress and egress traffic for
Kubernetes clusters. Kubernetes Gateway API (Gateway) is a set of APIs
implemented as Kubernetes Custom Resource Definitions that configure the
flow of traffic into and out of a Kubernetes cluster. It provides a
standardized way to define, configure, and manage gateways, which are
entry and exit points for traffic in a cluster.

The main drivers for the new Gateway API were to overcome the
limitations encountered with Ingress resources as the scale and
complexity of Kubernetes deployments grew. Originally the goal of
Ingress was to provide a centralized way to manage external access to
services running inside a Kubernetes cluster, typically via HTTP/HTTPS
traffic at the L7 layer. It successfully met this goal with simple
traffic management use cases.

However, ****as Kubernetes deployments grew in complexity, several
limitations of the ingress resource became apparent:****

- ****Limited protocol support:**** Ingress works only at Layer 7,
  specifically optimizing for HTTP and HTTPS traffic. Other L7 protocols
  (like gRPC) and non-L7 protocols (like TCP and UDP) must be handled
  using custom controller extensions rather than native Ingress
  capabilities.
- ****Lack of **Advanced Routing **:**** Complex routing and traffic
  management scenarios required non-standard annotations. These include
  use cases like A/B testing, canary roll-outs, distributed tracing
  which require vendor specific annotations.

****The Gateway API was designed to ****overcome these limitations and
provide a flexible and extensible framework by providing:****

1.  ****Multi-protocol support****: Gateway API supports multiple
    protocols, including HTTP(S), TCP, UDP, and gRPC. It provides
    support at both the L4 ( the Transport Layer - example protocols:
    TCP/UDP ) and L7( the Application Layer - example protocols: HTTP(s)
    / SIP ) layers.
2.  ****Decoupling****: Gateway API decouples the gateway configuration
    from the underlying implementation, allowing for more flexibility
    and choice.
3.  ****More configurability****: Gateway API provides a more
    comprehensive set of resources (Gateway, GatewayClass, Listener,
    Route, Filter) to manage complex traffic scenarios.
4.  ****Standardization****: Gateway API aims to standardize the way
    gateways are managed across different Kubernetes distributions and
    environments. This decouples the gateway configuration from the
    underlying implementation.
5.  **Complex Routing and Traffic Management: **Gateway API provides for
    complex traffic management scenarios such as A/B testing, canary
    roll-outs. It also enables route customization based on arbitrary
    header fields as well as paths and hosts.

**Istio Service Mesh**

Istio is one of the major open-source service mesh platforms that
provide a uniform way to manage and orchestrate microservices in a
distributed environment. Istio is generally deployed to support one or
more kubernetes clusters.

Per Wikipedia (Service Mesh)

An overview of Istio\'s key features and architectural components: (From
huggingface.io -- istio overview )

****

****Key Features:****

1.  ****Service Discovery****: Istio provides a service registry that
    allows services to register themselves and be discovered by other
    services.
2.  ****Traffic Management****: Istio enables traffic management
    features such as load balancing, circuit breaking, and request
    routing.
3.  ****Security****: Istio provides mutual TLS encryption,
    authentication, and authorization for services.
4.  ****Observability****: Istio collects metrics, logs, and tracing
    data for services, providing insights into performance and behavior.
5.  ****Policy Enforcement****: Istio allows administrators to define
    and enforce policies for services, such as rate limiting and quotas.
6.  ****Integration with Existing Tools****: Istio integrates with
    popular tools like Prometheus, Grafana, and Jaeger for monitoring
    and tracing.

****Architecture -- ****2 Main Components****:****

1.  ****Control Plane****: The control plane is responsible for managing
    the service mesh, including service discovery, traffic management,
    and policy enforcement.
2.  ****Data Plane****: The data plane is responsible for forwarding
    traffic between services and enforcing policies. The historical data
    plane was a sidecar. The newer recent data plane addition is ambient
    mode.

Overall, Istio provides a robust and flexible way to manage and
orchestrate microservices in a distributed environment, providing
features like traffic management, security, and observability.

Istio Ambient Mode

Istio Ambient Mode is a relatively new feature, and its capabilities and
limitations are subject to change as the project evolves but are fairly
stable with the 1.23 Istio release. In Ambient Mode, the Istio control
plane is not injected into each pod as a sidecar, unlike in the
traditional sidecar injection model. Instead, the control plane is
deployed as a separate entity, and the data plane is composed of
lightweight, ambient proxies that run as daemons on each node.

****

****Key characteristics:****

1.  ****No sidecar injection****: Unlike traditional Istio deployments,
    Ambient Mode does not require injecting the Istio control plane into
    each pod as a sidecar.
2.  ****Lightweight ambient proxies****: Ambient proxies are small,
    lightweight, and run as daemons on each node. They are responsible
    for intercepting and routing traffic.
3.  ****Centralized control plane****: The control plane is deployed
    separately and manages the ambient proxies.
4.  ****No per-pod resource overhead****: Since there is no sidecar,
    there is no additional resource overhead (e.g., CPU, memory) per
    pod.

****Benefits:****

1.  ****Improved performance****: Reduced overhead and latency due to
    the absence of sidecars.
2.  ****Simplified deployment****: Easier deployment and management, as
    the control plane is decoupled from the data plane.
3.  ****Better scalability****: Ambient Mode can handle a larger number
    of services and pods more efficiently.

****Use cases:****

1.  ****Large-scale deployments****: Ambient Mode is suitable for
    large-scale deployments with thousands of services and pods.
2.  ****Low-latency applications****: Applications requiring ultra-low
    latency might benefit from the reduced overhead of Ambient Mode.
3.  ****Simplified service mesh****: Ambient Mode can be a good choice
    for organizations looking for a simplified service mesh solution
    with minimal overhead.

The Prototypes

The development and runtime environment that these prototypes were
developed and tested in is listed in the following table. It should be
noted that the prototypes were developed in ascending complexity, so not
all software components were used within every prototype.

|                        |             |                                                             |
|------------------------|-------------|-------------------------------------------------------------|
| Software Component     | Version     | Description                                                 |
| Ubuntu                 | 20.04.6 LTS |                                                             |
| Minikube               | 1.34.0      | Addons Dashboard, Metallb                                   |
| Kubernetes             | 1.31.0      |                                                             |
| Docker                 | 27.2.0      |                                                             |
| Kubernetes Gateway API | 1.1.0       | Standard / Experimental                                     |
| Istio                  | 1.23.2      | Using Ambient Mode (no Sidecars)                            |
| Cert-Manager           | 1.15.3      | Certificate generation and lifecycle management             |
| OpenSSL                | 3.4.0       | Cert and Key generation for Kubernetes (Minikube) and Vault |

|                    |         |                               |
|--------------------|---------|-------------------------------|
| Software Component | Version | Description                   |
| BookInfo           |         | Istio Example app for Gateway |
| Apple App          |         | Http Echo App                 |

**Proto-01 -- Kubernetes Gateway API, Istio, Http transport**

This prototype deploys the following:

This prototype deploys the Istio sample app BookInfo using the
Kubernetes Gateway API. Istio is deployed in Ambient mode (no sidecars).
This essentially follows the steps used by the Istio Sample App project.
In addition, to test the use of multiple Gateway deployments within
different namespaces the Apple app (http echo) is deployed to the apple
namespace while the BookInfo app is deployed to the default namespace as
per the istio sample. Both gateways and httproutes use http.

For external minikube access the deployments use ClusterIP service
types, and port forwarding.

More information:

Istio setup - <https://istio.io/latest/docs/ambient/getting-started/>

Proto-02 -- Kubernetes Gateway API, Istio, Http transport

This prototype deploys the following:

a) minikube  - with a docker vm for cni;

b) gateway  - kubernetes gateway api standard crds;

c) istio  - Istio in ambient mode (no sidecars)

d) bookinfo  - Istio sample app

e) apple  - An http echo app

Proto-02 is essentially the same as Proto-01 with the exception that the
BookInfo deployment is moved from the default namespace into a book
namespace. Thus supporting separate Gateways each within their own
respective namespace.

Proto-03 -- Kubernetes Gateway API, Istio, OpenSSL Certs, Https
transport

This prototype deploys or uses the following:

a) minikube  - with a docker vm for cni;

b) metallb  - load balancer

c) gateway  - kubernetes gateway api standard crds;

d) istio  - Istio in ambient mode (no sidecars)

e) OpenSSL  - Certificate generation

f) Mango  - An http echo app which on success returns - \"juicy mango\"

The major enhancement within the prototype is to move to https transport
and Gateway configuration. As this prototype uses the HttpRoute the tls
is terminated at the gateway.

**Proto-04 -- Kubernetes Gateway API, Istio, Cert-Manager, Https
Terminated**

This prototype deploys or uses the following:

a) minikube  - with a docker vm for cni;

b) metallb  - load balancer for external minikube access

c) gateway  - kubernetes gateway api standard crds;

d) istio  - Istio in ambient mode (no sidecars)

e} cert-manger - certificate generation and management

f) OpenSSL  - Certificate generation

g) Mango  - An http echo app which on success returns - \"juicy mango\"

The major enhancement within this prototype is to include and configure
Cert-Manager for Issuer and Certificate generation. The prototype uses
self signed certificates. As this prototype uses the HttpRoute the tls
is terminated at the gateway.

**Proto-05 -- Kubernetes Gateway API, Istio, Cert-Manager, Https
Passthru**

This prototype deploys or uses the following:

a) minikube  - with a docker vm for cni;

b) metallb  - load balancer for external minikube access

c) gateway  - kubernetes gateway api standard crds;

d) istio  - Istio in ambient mode (no sidecars)

e} cert-manger - certificate generation and management

f) OpenSSL  - Certificate generation

g) Nginx  - Nginx server which on success returns the welcome page

This prototype uses the basics provided within Proto-04 (Gateway,
Cert-Manager, Istio, etc.) and then configures the TLS transport to
passthru to the destination Pod (ie. Nginx). The passthru uses the
Gateway TLSRoute, which is still in the Alpha2 experimental component of
the Gateway. The Nginx components within the prototype are deployed to
the default namespace.

**Proto-06-- Kubernetes Gateway API, Istio, Cert-Manager, Https
Passthru**

This prototype deploys or uses the following:

a) minikube  - with a docker vm for cni;

b) metallb  - load balancer for external minikube access

c) gateway  - kubernetes gateway api standard crds;

d) istio  - Istio in ambient mode (no sidecars)

e} cert-manger - certificate generation and management

f) OpenSSL  - Certificate generation

g) Nginx  - Nginx server which on success returns the welcome page

This prototype uses the same basics provided within Proto-05 (Gateway,
Cert-Manager, Istio, TLS Passthru, etc.) The only difference is that
additional Nginx components are deployed to the nginx namespace. The
passthru uses the Gateway TLSRoute, which is still in the Alpha2
experimental component of the Gateway.

**Proto-07-- Kubernetes Gateway API, Istio, Cert-Manager, Https
Passthru, Multiple Gateways**

This prototype deploys or uses the following:

a) minikube  - with a docker vm for cni;

b) metallb  - load balancer for external minikube access

c) gateway  - kubernetes gateway api standard crds;

d) istio  - Istio in ambient mode (no sidecars)

e} cert-manger - certificate generation and management

f) OpenSSL  - Certificate generation

g) Nginx  - Nginx server which on success returns the welcome page

h) Mango  - An http echo app which on success returns - \"juicy mango\"

Purpose

To provide a functioning prototype which contains different apps /
services within different namespaces and uses the Gateway to configure
different types of transport and tls termination and/or passthru.

Description

This prototype uses the same basics provided within Proto-06, Gateway,
Cert-Manager, Istio, Metallb,TLS Passthru, etc.) The difference is that
the Mango app is also deployed in its own namespace along with the Nginx
app in the nginx namespace. Mango used the HttpRoute which terminates
the TLS at the Gateway and Nginx uses TLSRoute for passthru to the Nginx
app. Each of the namespaces is set to ambient mode so mtls is used for
all internal traffic after the gateway.
