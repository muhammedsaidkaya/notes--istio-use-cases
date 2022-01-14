
# Security
* Authentication
* Authorization
* Mutual TLS - Encryption
* Certificate Managemen

# Peer Authentication
* It is used for service-to-service authentication

## Checking MTLS exists or not
```
istioctl experimental authz check $(kubectl get pods -n tutorial | grep customer | aws '{ print $1}' | head -1) -a
```

## Namespace-wide policy
```
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-peer-policy"
  namespace: "book-info"
spec:
  mtls:
    mode: STRICT # PERMISSIVE ...
```

## Mesh-wide policy
```
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-peer-policy"
  namespace: "istio-system"
spec:
  mtls:
    mode: STRICT
```

## Workload-wide policy
```
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-peer-policy"
  namespace: "book-info"
spec:
  selector:
    matchLabels:
      app: reviews
  mtls:
    mode: STRICT
```

## Port-level policy
```
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: "example-peer-policy"
  namespace: "book-info"
spec:
  selector:
    matchLabels:
      app: reviews
  portLevelMtls:
    80:
      mode: DISABLE
```

# Authorization
* CUSTOM
* DENY
* ALLOW
* AUDIT
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: authdenypolicy
  namespace: bookinfo
spec:
  action: DENY
  rules:
  - from:
    - source:
        namespaces: ["bar"]
    to:
    - operation:
        methods: ["POST"] 
```

## Allow-nothing
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-nothing
  namespace: default
spec: {}
```

### productpage
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "productpage-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: productpage
  action: ALLOW
  rules:
  - to:
    - operation:
        methods: ["GET"]
```

### productpage -> details
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "details-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: details
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
```

### productpage -> reviews
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "reviews-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: reviews
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-productpage"]
    to:
    - operation:
        methods: ["GET"]
```

### reviews -> ratings
```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "ratings-viewer"
  namespace: default
spec:
  selector:
    matchLabels:
      app: ratings
  action: ALLOW
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/default/sa/bookinfo-reviews"]
    to:
    - operation:
        methods: ["GET"]
```

## Certificate Management
```
kubectl create ns istio-system
kubectl create secret generic cacerts -n istio-system \
--from-file=ca-cert.pem \
--from-file=ca-key.pem \
--from-file=root-cert.pem \
--from-file=cert-chain.pem
istioctl install --set profile=demo
```