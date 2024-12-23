This prototype deploys the following:
  a) minikube     - with a docker vm for cni;
  b) metallb      - loadbalancer for external minikube access;
  c) gateway      - kubernetes gateway api standard crds;
  d) istio        - Istio in ambient mode (no sidecars)
  e) cert-manager - certificate generator for tls;
  f) nginx        - Returns welcome to Nginx
  
  The istio kubernetes gateway, tlsroute with passthrough mode is deployed to the nginx server
  in the nginx namespace.
