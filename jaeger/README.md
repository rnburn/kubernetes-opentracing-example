This example shows how to set up a traced service in kubernetes using minikube and Jaeger.

First start an nginx ingress controller
```
# mandatory steps
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/namespace.yaml \
    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/default-backend.yaml \
    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/configmap.yaml \
    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/tcp-services-configmap.yaml \
    | kubectl apply -f -

curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/udp-services-configmap.yaml \
    | kubectl apply -f -

# start ingress-controller
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/without-rbac.yaml \
    | kubectl apply -f -
```

Expose the ingress
```
curl https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml \
    | kubectl apply -f -
```
and save the port
```
INGRESS_PORT=$(kubectl get service ingress-nginx --namespace=ingress-nginx -o jsonpath='{.spec.ports[0].nodePort}')
```

Set up a jaeger collector in kubernetes
```
kubectl create -f https://raw.githubusercontent.com/jaegertracing/jaeger-kubernetes/master/all-in-one/jaeger-all-in-one-template.yml
kubectl expose deployment jaeger-deployment --name jaeger-frontend --type=NodePort --target-port=16686
```
save the port to later access
```
JAEGER_QUERY_PORT=$(kubectl get service jaeger-query -o jsonpath='{.spec.ports[0].nodePort}')
```

Start a dockerized version of Jaeger's hotrod example:
```
curl https://raw.githubusercontent.com/rnburn/kubernetes-opentracing-example/jaeger/hotrod.yaml \
    | kubectl apply -f -
```
Turn on OpenTracing in the ingress-controller
```
curl https://raw.githubusercontent.com/rnburn/kubernetes-opentracing-example/jaeger/opentracing-zipkin.yaml \
  | kubectl replace -f -
```

```
echo "Visit: $(minikube ip):$INGRESS_PORT"
echo "View traces at $(minikube ip):$JAEGER_QUERY_PORT"
```
