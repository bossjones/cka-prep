render:
	helm template --values loki-distributed-overrides.yaml loki grafana/loki-distributed -n monitoring > loki_manifests.yaml

install-minio:
	kubectl -n monitoring apply -f minio.yaml

install: install-minio
	helm upgrade --install --values loki-distributed-overrides.yaml loki grafana/loki-distributed -n monitoring

render-install: render install

uninstall:
	helm -n monitoring uninstall loki


proxy-grafana:
	kubectl port-forward service/monitoring-stack-grafana 8081:80 -n monitoring
