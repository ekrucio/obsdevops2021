# Teleric Academy DevOps 2021

This is a repository for the final project of the course.

## Description

The main focus of the project is [Observability](https://cloud.google.com/architecture/devops/devops-measurement-monitoring-and-observability). For the purposes of this project, all deployements are running in local minikube cluster with the following configuration:
```bash
minikube start --driver=virtualbox --kubernetes-version=v1.21.2 --vm=true --cpus=4 --memory=8g
```
It consists of several services:
* Nginx Ingress Controller
* Prometheus server
* Grafana
* Filebeat
* Logstash
* Elasticsearch
* Kibana

### Service breakdown
- Nginx's role is solely to expose Grafana and Kibana to outside the cluster. 
- Prometheus is responsible for collecting and storing the metrics exposed by the other services part of this deployment.
- Grafana is a visualization tool, currently utilising Prometheus as a datasource to aggregate and visualise metrics scraped by it. Also used for alerting.
- Filebeat is utilised to collect the application logs and enrich them with Kubernetes metadata from the other services and it ships them to Logstash.
- Logstash receives the collected logs from Filebeat and ships them to Elasticsearch, plus it controls the indices that they should belong to.
- Currently Elasticsearch is responsible for storing the logs collected from the application logs.
- Kibana is a visualization tool for Elasticsearch. It makes searching, aggregating and visualising data in Elasticsearch really easy.

## Installation

You can install all of the services one by one or all of them at once using the make commands defined in the helm holder:

```bash
make setup-monitoring ## Setup the environment: adding Helm repos, creating needed K8S namespaces and enabling minikube addons.
make deploy-monitoring ## Deploys local monitoring dependencies
make destroy-monitoring ## Uninstall all artifacts

make deploy-minikube-nginx-ingress ## Enables and configures the Minikube ingress. The deployment is scaled to 0 then to 1 again, because sometimes there are 2 replicas that are being created, which breaks ingress.
make deploy-grafana
make deploy-prometheus
make deploy-elastic
make deploy-kibana
make deploy-filebeat
make deploy-logstash
```

To correctly expose the nginx ingress we should find out the current minikube ip, by running the following command:
```bash
minikube ip
```
and add the returned ip as an entry in our local hosts file pointing to *ingress.local*
```bash
192.168.59.103   ingress.local
```

## Usage
After all artefacts are deployed and stable (stabilising the services written in java takes some time /Elasticsearch and Logstash/). To access the Grafana dashboard head to *ingress.local/grafana* and login as *admin* with the password received by the last step of the **make deploy-monitoring** or run the following command:
```bash
kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```
To access Kibana head to *ingress.local* (no authentication required). Currently there are only two indices set up:
* obs-filebeat-* : all of the application logs except those from Elasticsearch instance
* obs-es-filebeat-* : application logs from Elasticsearch

## K8s resource maps


#TODO Add resouce maps imgur -> links with preview
#TODO Add Grafana and Kibana screenshots

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
