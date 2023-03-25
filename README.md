# Istio Notes

Notes for Istio [Getting Started](https://istio.io/latest/docs/setup/getting-started/).

### Istio Setup

```bash
$ istioctl version
no running Istio pods in "istio-system"
1.17.1
```

```bash
$ istioctl install --set profile=demo -y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Egress gateways installed
✔ Installation complete
Making this installation the default for injection and validation.

Thank you for installing Istio 1.17.  Please take a few minutes to tell us about your install/upgrade experience!
```

```bash
$ kubectl get namespaces
NAME              STATUS   AGE
default           Active   5h58m
istio-system      Active   2m56s
kube-node-lease   Active   5h58m
kube-public       Active   5h58m
kube-system       Active   5h58m
```

```bash
$ kubectl label namespace default istio-injection=enabled
namespace/default labeled
```

### Sample application

```bash
$ git clone https://github.com/istio/istio.git
$ cd istio
$ git checkout release-1.17
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
details         ClusterIP   10.100.173.242   <none>        9080/TCP       40s
kubernetes      ClusterIP   10.100.0.1       <none>        443/TCP        6h3m
productpage     ClusterIP   10.100.186.130   <none>        9080/TCP       35s
ratings         ClusterIP   10.100.84.223    <none>        9080/TCP       39s
reviews         ClusterIP   10.100.167.125   <none>        9080/TCP       37s
```

```bash
$ kubectl get pods
NAME                                READY   STATUS    RESTARTS   AGE
details-v1-5ffd6b64f7-nw7b6         2/2     Running   0          55s
productpage-v1-979d4d9fc-dbbl9      2/2     Running   0          50s
ratings-v1-5f9699cfdf-lw4xj         2/2     Running   0          54s
reviews-v1-569db879f5-54ggt         2/2     Running   0          52s
reviews-v2-65c4dc6fdc-z4zfx         2/2     Running   0          52s
reviews-v3-c9c4fb987-9vk9n          2/2     Running   0          51s
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

### Verification

```bash
$ istioctl analyze
✔ No validation issues found when analyzing namespace: default.
```

### Optional Setup

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
clusterrole.rbac.authorization.k8s.io/kiali-viewer created
clusterrole.rbac.authorization.k8s.io/kiali created
clusterrolebinding.rbac.authorization.k8s.io/kiali created
role.rbac.authorization.k8s.io/kiali-controlplane created
rolebinding.rbac.authorization.k8s.io/kiali-controlplane created
service/kiali created
deployment.apps/kiali created
serviceaccount/prometheus created
configmap/prometheus created
clusterrole.rbac.authorization.k8s.io/prometheus created
clusterrolebinding.rbac.authorization.k8s.io/prometheus created
service/prometheus created
deployment.apps/prometheus created
```

```bash
$ kubectl rollout status deployment/kiali -n istio-system
deployment "kiali" successfully rolled out
```

```bash
$ istioctl dashboard kiali
http://localhost:20001/kiali
```
