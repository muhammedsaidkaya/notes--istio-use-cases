

# Traffic Management
* Gateways
* Virtual Services
* Destination Rules
* Subsets
* Timeouts
* Retries
* Circuit Breaking
* Fault Injection
* Request Routing
* A/B Testing

## Kubernetes Ingress
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress
spec:
  rules:
  - host: bookinfo.app
    http:
      paths:
      - path: /
        backend:
          serviceName: productpage
          servicePort: 8000
```

## Gateways
* default ingressgateway controllerına selector ile associate ediliyor.
* Kendi custom-ingress-gateway controllerımızı oluşturup ona da associa edilebilir.
* kubectl apply -f bookinfo-gateway.yaml
* kubectl get gateway
```
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: bookinfo-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "bookinfo.app"
```


## Virtual Services
* 
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name bookinfo
spec:
  hosts:
  - "bookinfo.app"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    - uri:
        prefix: /static
    - uri:
        exact: /login
    - uri:
        exact: /logout
    - uri:
        prefix: /api/v1/products
  route:
  - destination:
      host: productpage.default.svc.cluster.local
      port:
        number: 9080
```


```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews.default.svc.cluster.local
  http:
  - route:
    - destination:
        host: reviews.default.svc.cluster.local
        subset: v1
      weight: 99
    - destination:
        host: reviews.default.svc.cluster.local
        subset: v2
      weight: 1
```

## Destination Rules
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: reviews-destination
spec:
  host: reviews.default.svc.cluster.local
  trafficPolicy:
    tls:
      mode: MUTUAL
      clientCertificate: /myclientcert.pem
      privateKey: /client_private_key
    loadBalancer:
      simple: PASSTHROUGH #RANDOM / LEAST_CONN / ROUND_ROBIN
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

## DEMO
```
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml
vi virtualservice

apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  hosts:
  - reviews.default.svc.cluster.local
  http:
  - match:
    - headers:
        end-user:
          exact: kodekloud
    route:
    - destination:
        host: reviews.default.svc.cluster.local
        subset: v2
  - route:
    - destination:
        host: reviews.default.svc.cluster.local
        subset: v1
      weight: 75
    - destination:
        host: reviews.default.svc.cluster.local
        subset: v2
      weight: 25

kubectl apply -f virtualservice
```

## Fault Injection
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
      abort:
        percentage:
          value: 0.1
        httpStatus: 400
```

## Timeouts
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: bookinfo
spec:
  hosts:
  - "bookinfo.app"
  gateways:
  - bookinfo-gateway
  http:
  - match:
    - uri:
        exact: /productpage
    route:
    - destination:
        host: productpage
        port:
          number: 9080
    timeout: 3s
```

## Retries
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: my-service
spec:
  hosts:
  - my-service
  http:
  - route:
    - destination:
        host: my-service
        subset: v1
    retries:
      attempts: 3
      perTryTimeout: 2s
```

## Circuit Breaking
```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: productpage
spec:
  host: productpage
  subsets:
  - name: v1
    labels:
       version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 3
        http:
          http1MaxPendingRequests: 1
          maxRequestPerConnection: 1
```

```
# Concurrent client number is 2
h2load -n1000 -c2 'http://'"$INGRESS_HOST"':'"$INGRESS_PORT"'/productpage'
```