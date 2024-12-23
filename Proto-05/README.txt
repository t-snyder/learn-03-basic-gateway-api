This prototype deploys the following:
  a) minikube     - with a docker vm for cni;
  b) metallb      - loadbalancer for external minikube access;
  c) gateway      - kubernetes gateway api standard crds;
  d) istio        - Istio in ambient mode (no sidecars)
  e) cert-manager - certificate generator for tls;
  f) mango        - An http echo app which on success returns - "juicy mango"
  
  The mango gateway and httproute is deployed to the mango namespace.
