# setup minio
``` bash
sudo docker run --restart always --net=host -p 9000:9000 -p 9001:9001 -v /disks/1:/data --name minio -d quay.io/minio/minio server /data --console-address ":9001"
```

!> **check** it is runnning ```nc -w 5 -v <ipaddr> 9001```

?> **Tip** use recent version of minio the gui is improved, say something after 2021/8. On older versions replace above with ```--address```


# install k3d
``` bash
mkdir /tmp/data
k3d cluster create lokicluster -v /tmp/data:/tmp/data@server[0] -p "8081:80@loadbalancer"
kubectl config use-context k3d-lokicluster
```

# install loki stack

!> **warning** compactor is used here so do not run more than one replica at a time

``` bash
helm upgrade --install loki grafana/loki-stack  --set grafana.enabled=true,prometheus.enabled=true,prometheus.alertmanager.persistentVolume.enabled=false,prometheus.server.persistentVolume.enabled=false,loki.persistence.enabled=false -f values.yaml

```

?> **Tip** using local ephemeral storage but can enable persistent stores
``` bash
  loki.persistence.enabled=true,
  loki.persistence.storageClassName=local-path,
  loki.persistence.size=10Gi
```

?> **Help** a rendered version of the yaml can be found in the [git repo](https://github.com/mytestrepo2018/useful)

view the secret used by loki for the loaded config, it is base64 encoded

# configuring promtail for basic use against 'pod' sd targets plus dropping logs in a namespace

?> note requires a service for the pod to be discovered and scraped
  **need to watch out for static configs in the yaml**

remove all pods in the namespaces **secret** and **nosee**
``` yaml
    scrape_configs:
    - job_name: kubernetes-pods-name
      pipeline_stages:
        - docker: {}
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - action: drop
        regex: 'secret|nosee'
        source_labels:
        - __meta_kubernetes_namespace
```

# for grafana ui
``` bash
kubectl get secret loki-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

```

# to access grafana via ingress (and k3d lbalancer)
``` yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: grafana
  annotations:
    ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: loki-grafana
            port:
              number: 80
```

``` bash
kubectl create -f ingress.yaml
```

?> Grafana UI is now on the loadbalance port exposed by k3d (in the above case) this is ```http://localhost:8081```



# values for loki to read/write minio s3 store
``` yaml
loki:
  image:
    tag: 2.3.0  
  enabled: true
  config:
    ingester:
      lifecycler:
        address: 127.0.0.1
        ring:
          kvstore:
            store: inmemory
          replication_factor: 1
        final_sleep: 0s
      chunk_idle_period: 5m
      chunk_retain_period: 30s

    schema_config:
      configs:
      - from: 2020-05-15
        store: boltdb-shipper
        object_store: s3
        schema: v11
        index:
          prefix: index_
          period: 24h

    storage_config:
      boltdb_shipper:
        active_index_directory: /data/loki/boltdb-shipper-active
        cache_location: /data/loki/boltdb-shipper-cache
        cache_ttl: 24h         # Can be increased for faster performance over longer query periods, uses more disk space
        shared_store: s3

      aws:
        #s3: s3://user:password@192.168.2.222:9000/lokitesting
        s3forcepathstyle: true
        bucketnames: lokitesting
        endpoint: 192.168.2.222:9000
        insecure: true
        access_key_id: ${ACCESS_KEY_ID}
        secret_access_key: ${SECRET_ACCESS_KEY}
        #access_key_id: user
        #secret_access_key: password

    limits_config:
      enforce_metric_name: false
      reject_old_samples: true
      reject_old_samples_max_age: 168h

    compactor:
      working_directory: /data/compactor
      shared_store: s3
      compaction_interval: 5m
```


# run a docker promtail on a system and retrieve journal from systemd
``` bash
docker run --rm --name promtail -ti --net=host \
-v /var/log/journal/:/var/log/journal/ \
-v /run/log/journal/:/run/log/journal/ \
-v /etc/machine-id:/etc/machine-id \
-v /home/david/gits/arch1/:/config \
grafana/promtail:latest -config.file=/config/promtail-local-config.yaml
```

# using secrets for access to s3 bucket
``` yaml
          args:
            - "-config.expand-env=true"
            - "-config.file=/etc/loki/loki.yaml"
            - "-log.level=warn"
            #- "-print-config-stderr=true"
          env:
          - name: ACCESS_KEY_ID
            valueFrom:
              secretKeyRef:
                name: s3-creds
                key: access_key_id
          - name: SECRET_ACCESS_KEY
            valueFrom:
              secretKeyRef:
                name: s3-creds
                key: secret_access_key
```

``` bash
kubectl create secret generic s3-creds \
  --from-literal=access_key_id=user \
  --from-literal=secret_access_key=password
```
