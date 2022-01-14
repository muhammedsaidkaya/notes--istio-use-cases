# istio-use-cases

Installation istioctl-kiali
```
minikube start --vm=true
minikube addons enable ingress

curl -L https://istio.io/downloadIstio | sh -
cd istio-
export PATH=$PATH:$PWD/bin
istioctl version

istioctl install --set profile=demo -y
istioctl verify-install

istioctl analyze
kubectl label namespace default istio-injection=enabled

kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl get pods -A

echo -e "$(minikube ip)\tbookinfo.app" | sudo tee -a /etc/hosts

kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

export INGRESS_HOST=$(minikube ip)
export INGRESS_PORT=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.spec.ports[?(@.name=="http2")].nodePort}')
curl http://$INGRESS_HOST:$INGRESS_PORT/productpage

while sleep 0.01; do curl -sS 'http:&&'"$INGRESS_HOST"':'"$INGRESS_PORT"'/productpage'\ &> /dev/null ; done

kubectl apply -f samples/addon

curl -HHost:myapp.example.com "http://$INGRESS_HOST:$INGRESS_PORT/myapp"
```

