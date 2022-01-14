

# Improving Security with Istio
* https://www.youtube.com/watch?v=E0h1rS2D86k
* 16.29 Egress Blocking
* 18.44 Allow Egress Hosts
* 22.27 Authorization Policy between Microservices - RBAC
* 25.50 mTLS Enable - Encryption between Microservices
* 32.40 Authentication & Authorization - JWT

## Egress Blocking
```
kubectl get configmap istio-basic -n istio-system -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | kubectl replace -n istio-system -f -
```

## Allow Egress Host
```
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: worldclockapi-egress-rule
spec:
  hosts:
  - worldclockapi.com
  ports:
  - name: http-80
    number: 80
    protocol: http
```

## JWT - Request Authentication
```
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: customerjwt
  namespace: tutorial
spec:
  jwtRules:
  - issuer: testing@secure.istio.io
    jwksUri: "https:/gist.githubusercontent.com/.../jwks.json

```

```
TOKEN=$(curl https:/gist.githubusercontent.com/lordofthejars/f590c80b8d83ea1244febb2c73954739/raw/21ec0ba0184725444d99018761cf0cd0ece35971/token.role.jwt -s)
curl -H "Authorization: Bearer $TOKEN" http://istio-ingressgateway-istio-system.apps.openshift.sotogcp.com/customer
```

```
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: "require-jwt"
  namespace: tutorial
spec:
  selector:
    matchLabels:
      app: customer
  action: ALLOW
  rules:
  - from:
    - source:
        requestPrincipals: ["testing@secure.istio.io/testing@secure.istio.io"]
    when:
    - key: request.auth.claims[role]
      values: ["customer"]
```