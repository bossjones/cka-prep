render:
	helm template --values kube-prometheus-stack-overrides.yaml monitoring-stack prometheus-community/kube-prometheus-stack -n monitoring  > prometheus_manifests.yaml

install:
	kubectl apply -f namespace.yaml
	helm upgrade --install --values kube-prometheus-stack-overrides.yaml monitoring-stack prometheus-community/kube-prometheus-stack -n monitoring

proxy:
	kubectl port-forward service/monitoring-stack-grafana 8081:80 -n monitoring

render-install: render install
