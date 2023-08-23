render:
	helm template --values grafana-overrides.yaml grafana grafana/grafana -n grafana > grafana_manifests.yaml

install-minio:
	kubectl -n grafana apply -f minio.yaml

install: install-minio
	helm upgrade --install --values grafana-overrides.yaml grafana grafana/grafana -n grafana

render-install: render install

uninstall:
	helm -n grafana uninstall grafana


proxy-grafana:
	kubectl port-forward service/monitoring-stack-grafana 8081:80 -n monitoring
