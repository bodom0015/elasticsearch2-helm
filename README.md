# elasticsearch2-helm
A simple Helm chart proof-of-concept for Elasticsearch2.

WARNING: This version of Elasticsearch is extermely out-of-date.

Please consider using a newer version of Elasticsearch along with the official Helm chart, located here: https://github.com/elastic/helm-charts/tree/master/elasticsearch

Previous versions of the Helm chart can also be found here: https://github.com/helm/charts/tree/master/stable/elasticsearch

## Approach
Uses a Kubernetes [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to discover additional nodes.

Inspired by the approach recommended by the now-defunct [elasticsearch-cloud-kubernetes](https://github.com/fabric8io/elasticsearch-cloud-kubernetes#kubernetes-cloud-plugin-for-elasticsearch) by Fabric8IO.

## TODOs

* Script to generate secrets (are there secrets?)
* Test sharding / scaling-up additional replicas
* Abstract ConfigMap contents to `values.yaml`
