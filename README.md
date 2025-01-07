# Istio Notes

- https://istio.io/latest/docs/setup/getting-started/
- https://istio.io/latest/docs/setup/install/istioctl/
- https://istio.io/latest/docs/examples/bookinfo/

## Istio Setup

```bash
$ istioctl version
Istio is not present in the cluster: no running Istio pods in namespace "istio-system"
client version: 1.24.2
```

```bash
$ istioctl install --set profile=demo
        |\
        | \
        |  \
        |   \
      /||    \
     / ||     \
    /  ||      \
   /   ||       \
  /    ||        \
 /     ||         \
/______||__________\
____________________
  \__       _____/
     \_____/

This will install the Istio 1.24.2 profile "demo" into the cluster. Proceed? (y/N) y
✔ Istio core installed ⛵️
✔ Istiod installed 🧠
✔ Egress gateways installed 🛫
✔ Ingress gateways installed 🛬
✔ Installation complete
```

```bash
$ kubectl get namespaces istio-system
NAME           STATUS   AGE
istio-system   Active   2m12s
```

```bash
$ kubectl get crd gateways.gateway.networking.k8s.io &> /dev/null || \
{ kubectl kustomize "github.com/kubernetes-sigs/gateway-api/config/crd?ref=v1.2.1" | kubectl apply -f -; }

customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io created
```

### Sample application

```bash
$ git clone https://github.com/istio/istio.git -b release-1.24
$ cd istio/
-```

```bash
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```

```bash
$ kubectl apply -f ./samples/bookinfo/platform/kube/bookinfo.yaml
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
```

```bash
$ kubectl get services
NAME            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
details         ClusterIP   10.100.14.39     <none>        9080/TCP       23s
kubernetes      ClusterIP   10.100.0.1       <none>        443/TCP        4d
nginx-service   NodePort    10.100.217.5     <none>        80:30212/TCP   4d
productpage     ClusterIP   10.100.225.30    <none>        9080/TCP       18s
ratings         ClusterIP   10.100.4.190     <none>        9080/TCP       22s
reviews         ClusterIP   10.100.108.136   <none>        9080/TCP       20s
```

```bash
$ kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
details-v1-79dfbd6fff-hckqh          2/2     Running   0          58s
productpage-v1-dffc47f64-p72v2       2/2     Running   0          53s
ratings-v1-65f797b499-s7nss          2/2     Running   0          57s
reviews-v1-5c4d6d447c-98cx5          2/2     Running   0          55s
reviews-v2-65cb66b45c-zq8jn          2/2     Running   0          55s
reviews-v3-f68f94645-tjjmx           2/2     Running   0          54s
```

```bash
$ kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
<title>Simple Bookstore App</title>
```

### External Access

```bash
$ kubectl apply -f ./samples/bookinfo/networking/bookinfo-gateway.yaml
gateway.networking.istio.io/bookinfo-gateway created
virtualservice.networking.istio.io/bookinfo created
```

```bash
$ kubectl get gateway.networking.istio.io
NAME               AGE
bookinfo-gateway   40s
```

```bash
$ export GATEWAY_URL=$(kubectl get svc istio-ingressgateway -n istio-system -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
```

```bash
$ curl -s "http://${GATEWAY_URL}/productpage" | grep -o "<title>.*</title>"

<title>Simple Bookstore App</title>
```

### Verification

```
$ kubectl port-forward svc/bookinfo-gateway-istio 8080:80
```

```bash
$ istioctl analyze
✔ No validation issues found when analyzing namespace: default.
```

### Optional Setup

> https://istio.io/latest/docs/setup/getting-started/#dashboard

```bash
$ kubectl apply -f ./samples/addons
serviceaccount/grafana created
configmap/grafana created
service/grafana created
deployment.apps/grafana created
configmap/istio-grafana-dashboards created
configmap/istio-services-grafana-dashboards created
deployment.apps/jaeger created
service/tracing created
service/zipkin created
service/jaeger-collector created
serviceaccount/kiali created
configmap/kiali created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
service/kiali created
deployment.apps/kiali created
serviceaccount/loki created
configmap/loki created
configmap/loki-runtime created
clusterrole.rbac.authorization.k8s.io/loki-clusterrole created
clusterrolebinding.rbac.authorization.k8s.io/loki-clusterrolebinding created
service/loki-memberlist created
service/loki-headless created
service/loki created
statefulset.apps/loki created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

```bash
$ kubectl rollout status deployment/kiali -n istio-system
Waiting for deployment "kiali" rollout to finish: 0 of 1 updated replicas are available...
deployment "kiali" successfully rolled out
```

```bash
$ istioctl version
client version: 1.24.2
control plane version: 1.24.2
data plane version: 1.24.2 (8 proxies)
```

```bash
$ istioctl analyze --all-namespaces --suppress "IST0102=Namespace *" --suppress "IST0118=Service *"
```

```bash
$ istioctl dashboard kiali
http://localhost:20001/kiali
```

![image](https://github.com/user-attachments/assets/2dafbfa8-3c02-40a4-9fc3-c08fbcf9365a)
