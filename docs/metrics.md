# Notes for metrics and charts

## chart versions 
bitnami versions of charts
``` bash
helm repo add bitnami https://charts.bitnami.com/bitnami   
```
```
chart                   app version
kube-prometheus-6.6.11	0.54.1     
thanos-10.0.0         	0.25.0    
```

### note
Run minio server for s3
``` bash
docker run --network=host --name=mc -it --entrypoint=/bin/sh minio/mc
mc alias set minio http://192.168.2.205:9000 user password
```
The kube-prometheus runs prometheus operator which means config is stored in secrets and generated automagically
this can sometimes cause confusion (if hacking config). My suggestion is to scale the operator to 0 after install.
I am also running a test client (called test_client) for prometheus scraping as an additional scrape for test purposes.

## install
``` bash
kubectl delete secret -n metrics thanos-objstore-config
kubectl create secret -n metrics generic thanos-objstore-config --from-file=thanos-storage-config
```

## install with sidecar
``` bash
helm install prom --set prometheus.thanos.create=true --set prometheus.disableCompaction=true --namespace metrics bitnami/kube-prometheus -f sidecar-values.yaml
helm install -n metrics thanos bitnami/thanos --values thanos-values.yaml
```

## install with remoteWrite
``` bash
helm install prom --set prometheus.disableCompaction=true --namespace metrics bitnami/kube-prometheus -f remote-write-values.yaml
helm install -n metrics thanos bitnami/thanos --values thanos-values.yaml
```

## post install edit config if required
``` bash
kubectl scale -n metrics deploy prom-kube-prometheus-operator --replicas=0
kubectl port-forward svc/prom-kube-prometheus-prometheus -n metrics 9090
```
or for thanos ```thanos-query-frontend```

## prom queries
```
group by(container,pod) (kube_pod_container_status_running)
group by (job) (scrape_samples_scraped)
```
