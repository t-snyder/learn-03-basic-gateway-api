# Prototypes Using Kubernetes Gateway API, Istio Ambient Mode, and Cert-Manager

## Prototypes Purpose
The purpose of these prototypes is to provide a working deployment and
test environment for the new functionality (Kubernetes Gateway API and
Istio Ambient mode). The deployments are not production ready but
provide a basis for understanding how to use and deploy the
functionality.

## The Prototypes Environment

The development and runtime environment that these prototypes were
developed and tested in is listed in the following table. It should be
noted that the prototypes were developed in ascending complexity, so not
all software components were used within every prototype.

| Software Component     | Version     | Description                                                 |
|------------------------|-------------|-------------------------------------------------------------|
| Ubuntu                 | 20.04.6 LTS |                                                             |
| Minikube               | 1.34.0      | Addons Dashboard, Metallb                                   |
| Kubernetes             | 1.31.0      |                                                             |
| Docker                 | 27.2.0      |                                                             |
| Kubernetes Gateway API | 1.1.0       | Standard / Experimental                                     |
| Istio                  | 1.23.2      | Using Ambient Mode (no Sidecars)                            |
| Cert-Manager           | 1.15.3      | Certificate generation and lifecycle management             |
| OpenSSL                | 3.4.0       | Cert and Key generation for Kubernetes (Minikube) and Vault |

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| BookInfo           |         | Istio Example app for Gateway |
| Apple App          |         | Http Echo App                 |
| Nginx Server       |         | Nginx Web Server              |
| Mango App          |         | Http Echo Server              |

## Additional Supporting Documentation
  1. Istio setup    - <https://istio.io/latest/docs/ambient/getting-started/>
  2. Istio Minikube - <https://fabianlee.org/2024/09/29/minikube-istio-ambient-mode-on-minikube/>
  3. Istio Gateway API - https://istio.io/latest/docs/tasks/traffic-management/ingress/gateway-api/
  4. Kubernetes Gateway API - https://gateway-api.sigs.k8s.io/guides/
  5. Istio Gateway Passthru - https://istio.io/latest/docs/tasks/traffic-management/ingress/ingress-sni-passthrough/

  
## The Prototypes
**Note :**<br>
The commands within the shell files below are meant to be copy pasted (one or a few lines at a time) into a terminal, and not run as an automated bash script.

### Proto-01 -- Kubernetes Gateway API, Istio, Http transport

This prototype deploys the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| BookInfo           |         | Istio Example app for Gateway |
| Apple App          |         | Http Echo App                 |

This prototype deploys the Istio sample app BookInfo using the
Kubernetes Gateway API. Istio is deployed in Ambient mode (no sidecars).
This essentially follows the steps used by the Istio Sample App project.
In addition, to test the use of multiple Gateway deployments within
different namespaces the Apple app (http echo) is deployed to the apple
namespace while the BookInfo app is deployed to the default namespace as
per the istio sample. Both gateways and httproutes use http.

For external minikube access the deployments use ClusterIP service
types, and port forwarding.


### Proto-02 -- Kubernetes Gateway API, Istio, Http transport

This prototype deploys the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| BookInfo           |         | Istio Example app for Gateway |
| Apple App          |         | Http Echo App                 |

Proto-02 is essentially the same as Proto-01 with the exception that the
BookInfo deployment is moved from the default namespace into a book
namespace. Thus supporting separate Gateways each within their own
respective namespace.

### Proto-03 -- Kubernetes Gateway API, Istio, OpenSSL Certs, Https terminated

This prototype deploys or uses the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| Metallb            |         | Load Balancer                 |
| OpenSSL            |         | TLS Certificate Generation    |
| Mango App          |         | Http Echo App                 |

The major enhancement within the prototype is to move to https transport
within the Gateway configuration. As this prototype uses the HttpRoute the tls
is terminated at the gateway. However, as Istio provides the Gateway, mutual TLS is used
from the Gateway to the service endpoint.


### Proto-04 -- Kubernetes Gateway API, Istio, Cert-Manager, Https Terminated

This prototype deploys or uses the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| Cert-Manager       |         | Certificate Lifecycle Manager |
| Metallb            |         | Load Balancer                 |
| OpenSSL            |         | TLS Certificate Generation    |
| Mango App          |         | Http Echo App                 |

The major enhancement within this prototype is to include and configure
Cert-Manager for Issuer and Certificate generation. Using Cert-Manager within the
Gateway API environment requires installing in the following order:
  1. Kubernetes Gateway API CRDs
  2. Istio in Ambient Mode
  3. Cert-Manager Gateway API CRDs
  4. Cert Manager installed with enableGatewayAPI=true
  
The prototype uses self signed certificates generated via Cert-Manager. As this prototype uses the HttpRoute the tls
is terminated at the gateway. However istio then uses mutual TLS from the Gateway to the
service endpoint.


### Proto-05 -- Kubernetes Gateway API, Istio, Cert-Manager, Https Passthru

This prototype deploys or uses the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| Cert-Manager       |         | Certificate Lifecycle Manager |
| Metallb            |         | Load Balancer                 |
| Nginx Server       |         | Nginx Web Server              |

This prototype uses the basics provided within Proto-04 (Gateway,
Cert-Manager, Istio, etc.) and then configures the TLS transport to
passthru to the destination Pod endpoint (ie. Nginx). The passthru uses the
Gateway TLSRoute, which is still in an Alpha2 experimental Gateway component. 
The Nginx components within the prototype are deployed to the default namespace.


### Proto-06-- Kubernetes Gateway API, Istio, Cert-Manager, Https Passthru

This prototype deploys or uses the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| Cert-Manager       |         | Certificate Lifecycle Manager |
| Metallb            |         | Load Balancer                 |
| Nginx Server       |         | Nginx Web Server              |

This prototype uses the same basics provided within Proto-05 (Gateway,
Cert-Manager, Istio, TLS Passthru, etc.) The only difference is that
additional Nginx components are deployed to the nginx namespace. The
passthru uses the Gateway TLSRoute, which is still in the Alpha2
experimental component of the Gateway.

Note - The Kubernetes Gateway API CRDs, Istio and Cert-Manager deployments are slightly
different as the Experimental version of the Gateway API must be deployed.


### Proto-07-- Kubernetes Gateway API, Istio, Cert-Manager, Https Passthru, Multiple Gateways

This prototype deploys or uses the following:

| Software Component | Version | Description                   |
|--------------------|---------|-------------------------------|
| Cert-Manager       |         | Certificate Lifecycle Manager |
| Metallb            |         | Load Balancer                 |
| Nginx Server       |         | Nginx Web Server              |
| Mango App          |         | Http echo server              |

#### Purpose

To provide a functioning prototype which contains different apps /
services within different namespaces and uses the multiple Gateways to configure
different types of transport, tls termination and/or passthru.

#### Description

This prototype uses the same basics provided within Proto-06, Gateway,
Cert-Manager, Istio, Metallb,TLS Passthru, etc.) The difference is that
the Mango app is also deployed in its own namespace along with the Nginx
app in the nginx namespace. Mango used the HttpRoute which terminates
the TLS at the Gateway and Nginx uses TLSRoute for passthru to the Nginx
app. Each of the namespaces is set to ambient mode so mtls is used for
all internal traffic after the gateway.
