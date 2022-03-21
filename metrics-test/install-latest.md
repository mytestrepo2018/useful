# install

helm install -n metrics thanos bitnami/thanos \
  --values thanos-values.yaml


prometheus:
  serviceAccount:
    create: false
    name: thanos-s3

  thanos:
    create: true
    extraArgs:
      - --log.level=debug
    objectStorageConfig:
      key: thanos-storage-config.yaml
      name: thanos-storage-config


prom

      - args:
        - --web.console.templates=/etc/prometheus/consoles
        - --web.console.libraries=/etc/prometheus/console_libraries
        - --config.file=/etc/prometheus/config_out/prometheus.env.yaml
        - --storage.tsdb.path=/prometheus
        - --storage.tsdb.retention.time=10h
        - --web.enable-lifecycle
        - --web.external-url=http://prom-kube-prometheus-prometheus.metrics:9090/
        - --web.route-prefix=/
        - --web.config.file=/etc/prometheus/web_config/web-config.yaml
        - --log.level=debug
        - --storage.tsdb.wal-compression
        - --storage.tsdb.max-block-duration=300s
        - --storage.tsdb.min-block-duration=300s

kubectl create secret -n metrics generic thanos-objstore-config --from-file=thanos-storage-config

type: s3
config:
  bucket: 71879-thanos #S3 bucket name
  endpoint: s3.eu-west-2.amazonaws.com #S3 Regional endpoint

# versions
prom -> 2022-03-14 12:21:28.866865 +0000 UTC	deployed	kube-prometheus-6.6.11	0.54.1     
thanos -> 2022-03-14 12:35:22.069898 +0000 UTC	deployed	thanos-10.0.0         	0.25.0  


helm install prom --set prometheus.thanos.create=true --set prometheus.disableCompaction=true --namespace metrics bitnami/kube-prometheus -f values.yaml

helm install -n metrics thanos bitnami/thanos \
  --values thanos-values.yaml
