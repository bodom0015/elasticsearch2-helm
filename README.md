# elasticsearch2-helm
A simple Helm chart proof-of-concept for Elasticsearch2.

WARNING: This version of Elasticsearch is extermely out-of-date.

Please consider using a newer version of Elasticsearch along with the official Helm chart, located here: https://github.com/elastic/helm-charts/tree/master/elasticsearch

Previous versions of the Helm chart can also be found here: https://github.com/helm/charts/tree/master/stable/elasticsearch

## Approach
Uses a Kubernetes [headless service](https://kubernetes.io/docs/concepts/services-networking/service/#headless-services) to discover additional nodes. This service needs some special configuration to tell Kubernetes not to allocate an IP for it and to create and publish endpoints even if things are not in a ready state. This is necessary since ES2 needs to discover during startup, at which point one or more containers may not yet be ready.

Inspired by the approach recommended by the now-defunct [elasticsearch-cloud-kubernetes](https://github.com/fabric8io/elasticsearch-cloud-kubernetes#kubernetes-cloud-plugin-for-elasticsearch) by Fabric8IO.

## How to Test

### The Setup

1. Install minikube binary and run `minikube start`
2. Install helm client and run `helm init` (this will run Tiller on your minikube cluster)
3. Clone this repository:`git clone https://github.com/bodom0015/elasticsearch2-helm && cd elasticsearch2-helm`
4. Modify parameters as desired: `vi values.yaml`
5. Deploy the Helm chart: `helm upgrade --install es2 .`

By default, 1 replica will be deployed and will become the master:
```bash
$ helm upgrade --install es2 .
Release "es2" does not exist. Installing it now.
NAME:   es2
LAST DEPLOYED: Tue Oct  1 22:40:17 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                  READY  STATUS             RESTARTS  AGE
es2-elasticsearch2-0  0/1    ContainerCreating  0         1s

==> v1/ConfigMap
NAME                DATA  AGE
es2-elasticsearch2  1     1s

==> v1/Service
NAME                          TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)   AGE
es2-elasticsearch2-api        ClusterIP  10.105.115.234  <none>       9200/TCP  1s
es2-elasticsearch2-discovery  ClusterIP  None            <none>       9300/TCP  1s

==> v1beta2/StatefulSet
NAME                DESIRED  CURRENT  AGE
es2-elasticsearch2  1        1        1s


$ kubectl logs -f es2-elasticsearch2-0
[2019-10-02 03:40:29,517][INFO ][node                     ] [Joe Fixit] version[2.4.6], pid[1], build[5376dca/2017-07-18T12:17:44Z]
[2019-10-02 03:40:29,521][INFO ][node                     ] [Joe Fixit] initializing ...
[2019-10-02 03:40:35,412][INFO ][plugins                  ] [Joe Fixit] modules [reindex, lang-expression, lang-groovy], plugins [], sites []
[2019-10-02 03:40:36,174][INFO ][env                      ] [Joe Fixit] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda1)]], net usable_space [14.1gb], net total_space [16.9gb], spins? [possibly], types [ext4]
[2019-10-02 03:40:36,175][INFO ][env                      ] [Joe Fixit] heap size [1007.3mb], compressed ordinary object pointers [true]
[2019-10-02 03:40:59,813][INFO ][node                     ] [Joe Fixit] initialized
[2019-10-02 03:40:59,813][INFO ][node                     ] [Joe Fixit] starting ...
[2019-10-02 03:41:00,750][INFO ][transport                ] [Joe Fixit] publish_address {172.17.0.5:9300}, bound_addresses {127.0.0.1:9300}, {172.17.0.5:9300}
[2019-10-02 03:41:00,917][INFO ][discovery                ] [Joe Fixit] es2-elasticsearch2/v5OsITW3RFGElU4yVTxSAA
[2019-10-02 03:41:04,443][INFO ][cluster.service          ] [Joe Fixit] new_master {Joe Fixit}{v5OsITW3RFGElU4yVTxSAA}{172.17.0.5}{172.17.0.5:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
[2019-10-02 03:41:04,545][INFO ][http                     ] [Joe Fixit] publish_address {172.17.0.5:9200}, bound_addresses {127.0.0.1:9200}, {172.17.0.5:9200}
[2019-10-02 03:41:04,546][INFO ][node                     ] [Joe Fixit] started
[2019-10-02 03:41:04,725][INFO ][gateway                  ] [Joe Fixit] recovered [0] indices into cluster_state
```

### The Scale-Up
Scale up to 2 replicas by running the following:
```bash
kubectl scale statefulset es2-elasticsearch2 --replicas=2 --current-replicas=1
```

This will create a second replica pod that will automatically join the cluster once it comes online:
```bash
$ kubectl get pods
NAME                   READY   STATUS    RESTARTS   AGE
es2-elasticsearch2-0   1/1     Running   0          13m
es2-elasticsearch2-1   1/1     Running   0          12m

$ kubectl logs -f es2-elasticsearch2-1
[2019-10-02 03:41:50,625][INFO ][node                     ] [Right-Winger] version[2.4.6], pid[1], build[5376dca/2017-07-18T12:17:44Z]
[2019-10-02 03:41:50,628][INFO ][node                     ] [Right-Winger] initializing ...
[2019-10-02 03:41:54,408][INFO ][plugins                  ] [Right-Winger] modules [reindex, lang-expression, lang-groovy], plugins [], sites []
[2019-10-02 03:41:54,855][INFO ][env                      ] [Right-Winger] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda1)]], net usable_space [14.1gb], net total_space [16.9gb], spins? [possibly], types [ext4]
[2019-10-02 03:41:54,862][INFO ][env                      ] [Right-Winger] heap size [1007.3mb], compressed ordinary object pointers [true]
[2019-10-02 03:42:12,612][INFO ][node                     ] [Right-Winger] initialized
[2019-10-02 03:42:12,612][INFO ][node                     ] [Right-Winger] starting ...
[2019-10-02 03:42:13,218][INFO ][transport                ] [Right-Winger] publish_address {172.17.0.6:9300}, bound_addresses {127.0.0.1:9300}, {172.17.0.6:9300}
[2019-10-02 03:42:13,239][INFO ][discovery                ] [Right-Winger] es2-elasticsearch2/RxiYhWrOTpyUG-M1GWRnkQ
[2019-10-02 03:42:17,019][INFO ][cluster.service          ] [Right-Winger] detected_master {Joe Fixit}{v5OsITW3RFGElU4yVTxSAA}{172.17.0.5}{172.17.0.5:9300}, added {{Joe Fixit}{v5OsITW3RFGElU4yVTxSAA}{172.17.0.5}{172.17.0.5:9300},}, reason: zen-disco-receive(from master [{Joe Fixit}{v5OsITW3RFGElU4yVTxSAA}{172.17.0.5}{172.17.0.5:9300}])
[2019-10-02 03:42:17,349][INFO ][http                     ] [Right-Winger] publish_address {172.17.0.6:9200}, bound_addresses {127.0.0.1:9200}, {172.17.0.6:9200}
[2019-10-02 03:42:17,351][INFO ][node                     ] [Right-Winger] started
```

And we can see in our original (master) logs that the new pod has successfully joined the cluster:
```bash
$ kubectl logs -f es2-elasticsearch2-0
[2019-10-02 03:40:29,517][INFO ][node                     ] [Joe Fixit] version[2.4.6], pid[1], build[5376dca/2017-07-18T12:17:44Z]
[2019-10-02 03:40:29,521][INFO ][node                     ] [Joe Fixit] initializing ...
[2019-10-02 03:40:35,412][INFO ][plugins                  ] [Joe Fixit] modules [reindex, lang-expression, lang-groovy], plugins [], sites []
[2019-10-02 03:40:36,174][INFO ][env                      ] [Joe Fixit] using [1] data paths, mounts [[/usr/share/elasticsearch/data (/dev/sda1)]], net usable_space [14.1gb], net total_space [16.9gb], spins? [possibly], types [ext4]
[2019-10-02 03:40:36,175][INFO ][env                      ] [Joe Fixit] heap size [1007.3mb], compressed ordinary object pointers [true]
[2019-10-02 03:40:59,813][INFO ][node                     ] [Joe Fixit] initialized
[2019-10-02 03:40:59,813][INFO ][node                     ] [Joe Fixit] starting ...
[2019-10-02 03:41:00,750][INFO ][transport                ] [Joe Fixit] publish_address {172.17.0.5:9300}, bound_addresses {127.0.0.1:9300}, {172.17.0.5:9300}
[2019-10-02 03:41:00,917][INFO ][discovery                ] [Joe Fixit] es2-elasticsearch2/v5OsITW3RFGElU4yVTxSAA
[2019-10-02 03:41:04,443][INFO ][cluster.service          ] [Joe Fixit] new_master {Joe Fixit}{v5OsITW3RFGElU4yVTxSAA}{172.17.0.5}{172.17.0.5:9300}, reason: zen-disco-join(elected_as_master, [0] joins received)
[2019-10-02 03:41:04,545][INFO ][http                     ] [Joe Fixit] publish_address {172.17.0.5:9200}, bound_addresses {127.0.0.1:9200}, {172.17.0.5:9200}
[2019-10-02 03:41:04,546][INFO ][node                     ] [Joe Fixit] started
[2019-10-02 03:41:04,725][INFO ][gateway                  ] [Joe Fixit] recovered [0] indices into cluster_state
[2019-10-02 03:42:16,933][INFO ][cluster.service          ] [Joe Fixit] added {{Right-Winger}{RxiYhWrOTpyUG-M1GWRnkQ}{172.17.0.6}{172.17.0.6:9300},}, reason: zen-disco-join(join from node[{Right-Winger}{RxiYhWrOTpyUG-M1GWRnkQ}{172.17.0.6}{172.17.0.6:9300}])
```

### Monitoring
ElasticHQ is also deployed alongside your cluster to monitor and administer it.

The UI can be accessed by navigating your browser to the hostname configured as `ingress.host` in your `values.yaml`.

You will be prompted by ElasticHQ for the hostname of your cluster, for which you should use the following format:
```
http://es2-elasticsearch2-api:9200
```

NOTE: If you used a different `--name` besides `es2` for your Helm release, that name will need to be substituted in for `es2` in the above formatted string.

## TODOs

* Pod anti-affinity
* Abstract ConfigMap contents to `values.yaml`
* Document configuration options
* Test sharding / additional replicas
