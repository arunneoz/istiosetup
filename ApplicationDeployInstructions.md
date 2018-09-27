### Overview


### Clone from the following repository (ask asanthan@ to get access to the repo)

gcloud source repos clone https://source.developers.google.com/p/inbound-rune-zoro-poc/r/zoropoc [LOCAL_DIRECTORY]

cd [LOCAL_DIRECTORY]

### Deploy Zoro  App

We deploy our application directly using kubectl create and its regular YAML deployment file. We will inject Envoy containers into your application pods using istioctl:

```kubectl create namespace zorodev istio-injection=enabled```

```kubectl create -f backendservices/k8s/01_zorobackendmicroservices.yaml -n zorodev```

```kubectl create -f frontend/k8s/01_zorofrontendmicroservices.yaml -n zorodev```

Finally, confirm that the application has been deployed correctly by running the following commands:

Run the command:
```
kubectl get services -n zorodev
```
OUTPUT:
```
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
zoroproductsearch           ClusterIP   10.35.244.205   <none>        8080/TCP   1d
zoroproductsearchfrontend   ClusterIP   10.35.248.19    <none>        8090/TCP   1d
```

### Configure Istio

### Configure to enable the Ingress

```kubectl create -f frontend/k8s/02_zorofrontendvirtualservice.yaml -n zorodev```

```kubectl create -f frontend/k8s/03_zorofrontendsvcgateway.yaml -n zorodev```

### Configure to enable the Egress Traffic out to zorocloud

 ```kubectl create -f backendservices/k8s/istiofiles/01_egress_zorocloud_service_entry.yaml -n zorodev```

 ```kubectl create -f backendservices/k8s/istiofiles/02_egress_gateway.yaml -n zorodev```

 ```kubectl create -f backendservices/k8s/istiofiles/03_zorobackendsvcgateway.yaml -n zorodev```

 ### Testing the Services

 Now that it&#39;s deployed, let&#39;s see the Zoro application in action.

First you need to get the ingress IP and port, as follows:


 ```export GATEWAY_URL=$(kubectl -n istio-system get service istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}')```

 Check that the BookInfo app is running with curl:

Run the command:
```
curl -o /dev/null -s -w "%{http_code}\n" http://${GATEWAY_URL}/search
```
OUTPUT:
```
200
```

## View metrics and tracing <a name="viewing-metrics-and-tracing"/>

Istio-enabled applications can be configured to collect trace spans using, for instance, the popular [Jaeger](https://www.jaegertracing.io/docs/) distributed tracing system. Distributed tracing lets you see the flow of requests a user makes through your system, and Istio&#39;s model allows this regardless of what language/framework/platform you use to build your application.


Open your browser by clicking http://35.232.1.105/d/LJ_uJAvmk/istio-service-dashboard?refresh=10s&orgId=1&var-service=zoroproductsearchfrontend.zorodev.svc.cluster.local&var-srcns=All&var-srcwl=All&var-dstns=All&var-dstwl=All:
![Istio](media/grafana_dashboard.png)


Open your browser by clicking
http://35.239.71.115/search?end=1538080462449000&limit=20&lookback=3h&maxDuration&minDuration&operation=async%20outbound%7C9091%7C%7Cistio-policy.istio-system.svc.cluster.local%20egress&service=zoroproductsearchfrontend&start=1538069662449000
![Istio](media/tracing.png)
