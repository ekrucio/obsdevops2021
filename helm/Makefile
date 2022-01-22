CURRENT_K8S_CTX :=$(shell  command kubectl config current-context)
MNKB_CTX =minikube
ENVIRONMENT = environment/local
check-k8s-ctx:
	@echo "Current kubernetes context is "$(CURRENT_K8S_CTX)
	@echo "Do you wish to continue? [y/N] " && read ans && [ $${ans:-N} = y ]
ifeq ($(CURRENT_K8S_CTX),$(MNKB_CTX))
	@echo "OK. Continuing with the command"
else
	@echo "This is most probably production environment. Are you sure? [y/N] " && read ans && [ $${ans:-N} = y ]
	@echo "OK. Continuing with the command"
endif

setup-monitoring: ## Setup the environment: adding Helm repos, creating needed K8S namespaces and enabling minikube addons.
	@helm repo add stable https://kubernetes.github.io/ingress-nginx
	@helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
	@helm repo add grafana https://grafana.github.io/helm-charts
	@helm repo add elastic https://helm.elastic.co

	@helm repo update
	@kubectl create namespace ingress-nginx
	@kubectl create namespace monitoring
	@minikube addons enable metrics-server
	@minikube addons enable default-storageclass
	@minikube addons enable storage-provisioner


deploy-monitoring: ## Deploys local monitoring dependencies
	make check-k8s-ctx
	@echo "Deploying Nginx Ingress Controller ..."
	make deploy-minikube-nginx-ingress
	@kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
	@echo "Deploying Prometheus ..."
	make deploy-prometheus
	@echo "Deploying Elasticserach ..."
	make deploy-elastic
	@echo "Waiting 1 minute..." && sleep 60;
	@echo "Deploying Kibana ..."
	make deploy-kibana
	@echo "Deploying Logstash ..."
	make deploy-logstash
	@echo "Waiting 30 seconds..." && sleep 30;
	@echo "Deploying Filebeat ..."
	make deploy-filebeat
	@echo "Deploying Grafana ..."
	make deploy-grafana
	@echo "Deployment finished."


deploy-minikube-nginx-ingress: ## Enables and configures the Minikube ingress. The deployment is scaled to 0 then to 1 again, because sometimes there are 2 replicas that are being created, which breaks ingress.
	@minikube addons enable ingress
	@kubectl patch configmap tcp-services -p "`cat environment/local/nginx-configmap-patch.yaml`" -n ingress-nginx
	@kubectl patch deployment.apps/ingress-nginx-controller -p "`cat environment/local/nginx-deployment-patch.yaml`" -n ingress-nginx
	@kubectl scale deployment.apps/ingress-nginx-controller --replicas=0 -n ingress-nginx
	@kubectl scale deployment.apps/ingress-nginx-controller --replicas=1 -n ingress-nginx
	@kubectl rollout status deployment.apps/ingress-nginx-controller -n ingress-nginx

deploy-grafana:
		@helm upgrade --install grafana --namespace=monitoring --values $(ENVIRONMENT)/grafana.values.yaml grafana/grafana
		@echo "Grafana admin password is : "
		@kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

deploy-prometheus:
		@helm upgrade --install prometheus-cluster --namespace=monitoring --values $(ENVIRONMENT)/prometheus.values.yaml  prometheus-community/prometheus

deploy-elastic:
		@helm upgrade --install elasticsearch --namespace=monitoring --values $(ENVIRONMENT)/elasticsearch.values.yaml --set imageTag=7.16.0 elastic/elasticsearch

deploy-kibana:
		@helm upgrade --install kibana --namespace=monitoring --values $(ENVIRONMENT)/kibana.values.yaml --set imageTag=7.7.1 elastic/kibana

deploy-filebeat:
		@helm upgrade --install filebeat --namespace=monitoring --set imageTag=7.16.0 elastic/filebeat --values $(ENVIRONMENT)/filebeat.values.yaml

deploy-logstash:
		@helm upgrade --install logstash --namespace=monitoring --set imageTag=7.16.0 elastic/logstash --values $(ENVIRONMENT)/logstash.values.yaml


destroy-monitoring:
	make check-k8s-ctx
	@echo "Uninstalling monitoring ..."
	@helm uninstall prometheus-cluster -n monitoring
	@helm uninstall grafana -n monitoring
	@helm uninstall elastic -n monitoring
	@helm uninstall filebeat -n monitoring
	@helm uninstall kibana -n monitoring
	@helm uninstall logstash -n monitoring
	@echo "Uninstalling artefacts completed."
