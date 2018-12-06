### Vagrant ENV:

Run `vagrant up`. This will automatically provision two VM with Kubernetes cluster installed.

```
k8s Environment Info:
Kubernetes version: 1.13 (or latest from repo)
CNI: Weave-Net
Default number of nodes: 2

root@k8s-m1:~# export KUBECONFIG=/etc/kubernetes/admin.conf
root@k8s-m1:~# kubectl get all --all-namespaces
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   pod/coredns-86c58d9df4-4tg5l                1/1     Running   0          36m
kube-system   pod/coredns-86c58d9df4-jjdjr                1/1     Running   0          36m
kube-system   pod/etcd-k8s-m1                             1/1     Running   0          52m
kube-system   pod/kube-apiserver-k8s-m1                   1/1     Running   0          52m
kube-system   pod/kube-controller-manager-k8s-m1          1/1     Running   3          52m
kube-system   pod/kube-proxy-cbw8d                        1/1     Running   0          52m
kube-system   pod/kube-proxy-cncbd                        1/1     Running   0          44m
kube-system   pod/kube-scheduler-k8s-m1                   1/1     Running   2          52m
kube-system   pod/kubernetes-dashboard-79ff88449c-pqgx8   1/1     Running   0          52m
kube-system   pod/weave-net-ktbvl                         2/2     Running   0          51m
kube-system   pod/weave-net-qzfcx                         2/2     Running   3          44m

NAMESPACE     NAME                           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
default       service/kubernetes             ClusterIP   10.96.0.1        <none>        443/TCP         52m
kube-system   service/kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   52m
kube-system   service/kubernetes-dashboard   ClusterIP   10.111.210.147   <none>        443/TCP         52m

NAMESPACE     NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
kube-system   daemonset.apps/kube-proxy   2         2         2       2            2           <none>          52m
kube-system   daemonset.apps/weave-net    2         2         2       2            2           <none>          51m

NAMESPACE     NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/coredns                2/2     2            2           52m
kube-system   deployment.apps/kubernetes-dashboard   1/1     1            1           52m

NAMESPACE     NAME                                              DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/coredns-86c58d9df4                2         2         2       52m
kube-system   replicaset.apps/kubernetes-dashboard-79ff88449c   1         1         1       52m


root@k8s-m1:~# kubectl get pods -o wide --sort-by="{.spec.nodeName}" --all-namespaces
NAMESPACE     NAME                                    READY   STATUS    RESTARTS   AGE   IP             NODE     NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-jjdjr                1/1     Running   0          38m   10.32.0.3      k8s-m1   <none>           <none>
kube-system   coredns-86c58d9df4-4tg5l                1/1     Running   0          38m   10.36.0.0      k8s-n1   <none>           <none>
kube-system   etcd-k8s-m1                             1/1     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-apiserver-k8s-m1                   1/1     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-controller-manager-k8s-m1          1/1     Running   3          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-proxy-cbw8d                        1/1     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-scheduler-k8s-m1                   1/1     Running   2          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   kube-proxy-cncbd                        1/1     Running   0          45m   192.16.35.10   k8s-n1   <none>           <none>
kube-system   kubernetes-dashboard-79ff88449c-pqgx8   1/1     Running   0          53m   10.32.0.2      k8s-m1   <none>           <none>
kube-system   weave-net-ktbvl                         2/2     Running   0          53m   192.16.35.11   k8s-m1   <none>           <none>
kube-system   weave-net-qzfcx                         2/2     Running   3          45m   192.16.35.10   k8s-n1   <none>           <none>
root@k8s-m1:~# 


```
You can edit Vagrantfile and hack/setup-vms.sh for your needs

Example:
```
Vagrantfile: 
  master = 1
  node = 2

$ diff setup-vms.sh setup-vms.sh.NEW 
11d10
< 
13,14c12,13
< 192.16.35.11 k8s-m1
< 
---
> 192.16.35.11 k8s-n2
> 192.16.35.12 k8s-m1
38c37
<   HOSTS="192.16.35.10 192.16.35.11"
---
>   HOSTS="192.16.35.10 192.16.35.11 192.16.35.12"
```

## Manual Installation

### Kubeadm Ansible Playbook

Build a Kubernetes cluster using Ansible with kubeadm. The goal is easily install a Kubernetes cluster on machines running:

  - Ubuntu 16.04
  - CentOS 7
  - Debian 9

System requirements:

  - Deployment environment must have Ansible `2.4.0+`
  - Master and nodes must have passwordless SSH access

### Usage

Add the system information gathered above into a file called `hosts.ini`. For example:
```
[master]
192.16.35.12

[node]
192.16.35.[10:11]

[kube-cluster:children]
master
node
```

Before continuing, edit `group_vars/all.yml` to your specified configuration.

**Note:**  By default, `k8s-ansible-kubeadm` uses `eth1`. Your default interface may be `eth0`.

After going through the setup, run the `site.yaml` playbook:

```sh
$ ansible-playbook site.yaml
...
==> master1: TASK [addon : Create Kubernetes dashboard deployment] **************************
==> master1: changed: [192.16.35.12 -> 192.16.35.12]
==> master1:
==> master1: PLAY RECAP *********************************************************************
==> master1: 192.16.35.10               : ok=18   changed=14   unreachable=0    failed=0
==> master1: 192.16.35.11               : ok=18   changed=14   unreachable=0    failed=0
==> master1: 192.16.35.12               : ok=34   changed=29   unreachable=0    failed=0
```

Download the `admin.conf` from the master node:

```sh
$ scp k8s@k8s-master:/etc/kubernetes/admin.conf .
```

Verify cluster is fully running using kubectl:

```sh

$ export KUBECONFIG=~/admin.conf
$ kubectl get node
NAME      STATUS    AGE       VERSION
master1   Ready     22m       v1.6.3
node1     Ready     20m       v1.6.3
node2     Ready     20m       v1.6.3

$ kubectl get po -n kube-system
NAME                                    READY     STATUS    RESTARTS   AGE
etcd-master1                            1/1       Running   0          23m
...
```

### K8s addons install via ansible role

Example:
----------------

```yaml
- hosts: master
  gather_facts: yes
  become: yes
  roles:
    - { role: addons, tags: addons }

```

### K8s addons manual install via Helm 
```
$ cd helm-prometheus-grafana

$ kubectl create clusterrolebinding default-sa-admin --user system:serviceaccount:kube-system:default  --clusterrole cluster-admin

$ helm init

$ helm install stable/prometheus --name prometheus -f prometheus.yml

Edit grafana.yml and config:
        lookup("file", "LOCAL_PATH to file/kubernetes_cluster_monitoring_prometheus.json")

$ helm install stable/grafana --name grafana -f ./grafana.yml

root@k8s-m1:~# kubectl get all --all-namespaces
NAMESPACE     NAME                                             READY   STATUS    RESTARTS   AGE
default       pod/grafana-65bcd6c887-t2qhk                     1/1     Running   0          111s
default       pod/prometheus-alertmanager-777d964c6b-mkbkj     2/2     Running   0          6m36s
default       pod/prometheus-kube-state-metrics-5c5bc7-z25n7   1/1     Running   0          6m36s
default       pod/prometheus-node-exporter-zf9qj               1/1     Running   0          6m36s
default       pod/prometheus-pushgateway-5f457bff66-m6272      1/1     Running   0          6m36s
default       pod/prometheus-server-9f8c98dbc-m4r2g            2/2     Running   0          6m36s
kube-system   pod/coredns-86c58d9df4-q8f5g                     1/1     Running   0          21m
kube-system   pod/coredns-86c58d9df4-ssllq                     1/1     Running   0          21m
kube-system   pod/etcd-k8s-m1                                  1/1     Running   0          23m
kube-system   pod/kube-apiserver-k8s-m1                        1/1     Running   0          23m
kube-system   pod/kube-controller-manager-k8s-m1               1/1     Running   0          23m
kube-system   pod/kube-proxy-988d9                             1/1     Running   0          23m
kube-system   pod/kube-proxy-r5c6c                             1/1     Running   0          22m
kube-system   pod/kube-scheduler-k8s-m1                        1/1     Running   0          23m
kube-system   pod/kubernetes-dashboard-79ff88449c-xzqc6        1/1     Running   0          23m
kube-system   pod/tiller-deploy-6f4dbc6d67-qs6qn               1/1     Running   0          9m14s
kube-system   pod/weave-net-4lcgq                              2/2     Running   0          22m
kube-system   pod/weave-net-pxhjh                              2/2     Running   0          23m

NAMESPACE     NAME                                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
default       service/grafana                         ClusterIP   10.108.233.101   <none>        80/TCP          111s
default       service/kubernetes                      ClusterIP   10.96.0.1        <none>        443/TCP         24m
default       service/prometheus-alertmanager         ClusterIP   10.102.138.183   <none>        80/TCP          6m37s
default       service/prometheus-kube-state-metrics   ClusterIP   None             <none>        80/TCP          6m37s
default       service/prometheus-node-exporter        ClusterIP   None             <none>        9100/TCP        6m37s
default       service/prometheus-pushgateway          ClusterIP   10.105.86.60     <none>        9091/TCP        6m37s
default       service/prometheus-server               ClusterIP   10.104.179.45    <none>        80/TCP          6m37s
kube-system   service/kube-dns                        ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   24m
kube-system   service/kubernetes-dashboard            ClusterIP   10.105.227.240   <none>        443/TCP         24m
kube-system   service/tiller-deploy                   ClusterIP   10.110.194.207   <none>        44134/TCP       9m14s

NAMESPACE     NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
default       daemonset.apps/prometheus-node-exporter   1         1         1       1            1           <none>          6m37s
kube-system   daemonset.apps/kube-proxy                 2         2         2       2            2           <none>          24m
kube-system   daemonset.apps/weave-net                  2         2         2       2            2           <none>          24m

NAMESPACE     NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/grafana                         1/1     1            1           111s
default       deployment.apps/prometheus-alertmanager         1/1     1            1           6m37s
default       deployment.apps/prometheus-kube-state-metrics   1/1     1            1           6m37s
default       deployment.apps/prometheus-pushgateway          1/1     1            1           6m37s
default       deployment.apps/prometheus-server               1/1     1            1           6m37s
kube-system   deployment.apps/coredns                         2/2     2            2           24m
kube-system   deployment.apps/kubernetes-dashboard            1/1     1            1           24m
kube-system   deployment.apps/tiller-deploy                   1/1     1            1           9m15s

NAMESPACE     NAME                                                   DESIRED   CURRENT   READY   AGE
default       replicaset.apps/grafana-65bcd6c887                     1         1         1       111s
default       replicaset.apps/prometheus-alertmanager-777d964c6b     1         1         1       6m37s
default       replicaset.apps/prometheus-kube-state-metrics-5c5bc7   1         1         1       6m37s
default       replicaset.apps/prometheus-pushgateway-5f457bff66      1         1         1       6m37s
default       replicaset.apps/prometheus-server-9f8c98dbc            1         1         1       6m37s
kube-system   replicaset.apps/coredns-86c58d9df4                     2         2         2       23m
kube-system   replicaset.apps/kubernetes-dashboard-79ff88449c        1         1         1       23m
kube-system   replicaset.apps/tiller-deploy-6f4dbc6d67               1         1         1       9m14s

$ helm list
NAME      	REVISION	UPDATED                 	STATUS  	CHART           	APP VERSION	NAMESPACE
grafana   	1       	Thu Dec  6 11:13:36 2018	DEPLOYED	grafana-1.17.3  	5.3.2      	default  
prometheus	1       	Thu Dec  6 11:08:49 2018	DEPLOYED	prometheus-7.3.4	2.4.3      	default 

$ kubectl get pods --namespace default -l "app=grafana" -o jsonpath="{.items[0].metadata.name}"
grafana-65bcd6c887-blvwz

$ kubectl --namespace default port-forward grafana-65bcd6c887-blvwz 3000

BROWSER: http://localhost:3000 --- admin:admin --- import json file if needed.

$ helm del --purge grafana
$ helm del --purge prometheus

```

```
cd helm-nginx-prometheus-grafana

helm install stable/nginx-ingress -f ./ingress-internal.yml 
helm install stable/nginx-ingress -f ./ingress-external.yml 
helm install stable/prometheus --name prometheus -f prometheus.yml

Edit grafana.yml and config:
        lookup("file", "LOCAL_PATH to file/kubernetes_cluster_monitoring_prometheus.json")

helm install stable/grafana --name grafana -f ./grafana.yml

In this case, there is no LoadBalancer integrated (unlike AWS or Google Cloud). With this default setup, you can only use NodePort (more info here: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport) or an Ingress Controller. With the Ingress Controller you can setup a domain name which maps to your pod (more information here: https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-controllers): pending


root@k8s-m1:~# kubectl get svc --all-namespaces|grep Load
default       dining-waterbuffalo-nginx-ingress-controller           LoadBalancer   10.104.235.136   <pending>     80:32568/TCP,443:31729/TCP   8m25s

root@k8s-m1:~# kubectl get all --all-namespaces
NAMESPACE     NAME                                                                  READY   STATUS    RESTARTS   AGE
default       pod/dining-waterbuffalo-nginx-ingress-controller-7b5b9dbb8b-nv7pw     1/1     Running   0          8m14s
default       pod/dining-waterbuffalo-nginx-ingress-default-backend-fffbc96cxxx4g   1/1     Running   0          8m14s
default       pod/grafana-65bcd6c887-2s86x                                          1/1     Running   0          4m58s
default       pod/prometheus-alertmanager-777d964c6b-87v6h                          2/2     Running   0          6m7s
default       pod/prometheus-kube-state-metrics-5c5bc7-z8wbk                        1/1     Running   0          6m6s
default       pod/prometheus-node-exporter-ftwfq                                    1/1     Running   0          6m7s
default       pod/prometheus-pushgateway-5f457bff66-pfv5j                           1/1     Running   0          6m6s
default       pod/prometheus-server-9f8c98dbc-cbpg7                                 2/2     Running   0          6m6s
default       pod/trendsetting-tapir-nginx-ingress-controller-8656b89c6c-r2l7k      1/1     Running   0          9m3s
default       pod/trendsetting-tapir-nginx-ingress-default-backend-588dbcb58rhk7w   1/1     Running   0          9m3s
kube-system   pod/coredns-86c58d9df4-q8f5g                                          1/1     Running   0          54m
kube-system   pod/coredns-86c58d9df4-ssllq                                          1/1     Running   0          54m
kube-system   pod/etcd-k8s-m1                                                       1/1     Running   0          55m
kube-system   pod/kube-apiserver-k8s-m1                                             1/1     Running   0          56m
kube-system   pod/kube-controller-manager-k8s-m1                                    1/1     Running   0          55m
kube-system   pod/kube-proxy-988d9                                                  1/1     Running   0          56m
kube-system   pod/kube-proxy-r5c6c                                                  1/1     Running   0          54m
kube-system   pod/kube-scheduler-k8s-m1                                             1/1     Running   0          56m
kube-system   pod/kubernetes-dashboard-79ff88449c-xzqc6                             1/1     Running   0          56m
kube-system   pod/tiller-deploy-6f4dbc6d67-qs6qn                                    1/1     Running   0          41m
kube-system   pod/weave-net-4lcgq                                                   2/2     Running   0          54m
kube-system   pod/weave-net-pxhjh                                                   2/2     Running   0          56m

NAMESPACE     NAME                                                           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
default       service/dining-waterbuffalo-nginx-ingress-controller           LoadBalancer   10.104.235.136   <pending>     80:32568/TCP,443:31729/TCP   8m14s
default       service/dining-waterbuffalo-nginx-ingress-controller-metrics   ClusterIP      10.105.13.175    <none>        9913/TCP                     8m14s
default       service/dining-waterbuffalo-nginx-ingress-controller-stats     ClusterIP      10.104.105.98    <none>        18080/TCP                    8m14s
default       service/dining-waterbuffalo-nginx-ingress-default-backend      ClusterIP      10.98.3.127      <none>        80/TCP                       8m14s
default       service/grafana                                                ClusterIP      10.103.199.143   <none>        80/TCP                       4m58s
default       service/kubernetes                                             ClusterIP      10.96.0.1        <none>        443/TCP                      56m
default       service/prometheus-alertmanager                                ClusterIP      10.99.251.67     <none>        80/TCP                       6m7s
default       service/prometheus-kube-state-metrics                          ClusterIP      None             <none>        80/TCP                       6m7s
default       service/prometheus-node-exporter                               ClusterIP      None             <none>        9100/TCP                     6m7s
default       service/prometheus-pushgateway                                 ClusterIP      10.102.149.197   <none>        9091/TCP                     6m7s
default       service/prometheus-server                                      ClusterIP      10.105.206.188   <none>        80/TCP                       6m7s
default       service/trendsetting-tapir-nginx-ingress-controller            NodePort       10.110.240.123   <none>        80:30080/TCP,443:30443/TCP   9m4s
default       service/trendsetting-tapir-nginx-ingress-controller-metrics    ClusterIP      10.101.191.223   <none>        9913/TCP                     9m4s
default       service/trendsetting-tapir-nginx-ingress-controller-stats      ClusterIP      10.110.142.20    <none>        18080/TCP                    9m4s
default       service/trendsetting-tapir-nginx-ingress-default-backend       ClusterIP      10.111.118.177   <none>        80/TCP                       9m3s
kube-system   service/kube-dns                                               ClusterIP      10.96.0.10       <none>        53/UDP,53/TCP                56m
kube-system   service/kubernetes-dashboard                                   ClusterIP      10.105.227.240   <none>        443/TCP                      56m
kube-system   service/tiller-deploy                                          ClusterIP      10.110.194.207   <none>        44134/TCP                    41m

NAMESPACE     NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
default       daemonset.apps/prometheus-node-exporter   1         1         1       1            1           <none>          6m7s
kube-system   daemonset.apps/kube-proxy                 2         2         2       2            2           <none>          56m
kube-system   daemonset.apps/weave-net                  2         2         2       2            2           <none>          56m

NAMESPACE     NAME                                                                READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/dining-waterbuffalo-nginx-ingress-controller        1/1     1            1           8m14s
default       deployment.apps/dining-waterbuffalo-nginx-ingress-default-backend   1/1     1            1           8m14s
default       deployment.apps/grafana                                             1/1     1            1           4m58s
default       deployment.apps/prometheus-alertmanager                             1/1     1            1           6m7s
default       deployment.apps/prometheus-kube-state-metrics                       1/1     1            1           6m7s
default       deployment.apps/prometheus-pushgateway                              1/1     1            1           6m7s
default       deployment.apps/prometheus-server                                   1/1     1            1           6m7s
default       deployment.apps/trendsetting-tapir-nginx-ingress-controller         1/1     1            1           9m3s
default       deployment.apps/trendsetting-tapir-nginx-ingress-default-backend    1/1     1            1           9m3s
kube-system   deployment.apps/coredns                                             2/2     2            2           56m
kube-system   deployment.apps/kubernetes-dashboard                                1/1     1            1           56m
kube-system   deployment.apps/tiller-deploy                                       1/1     1            1           41m

NAMESPACE     NAME                                                                          DESIRED   CURRENT   READY   AGE
default       replicaset.apps/dining-waterbuffalo-nginx-ingress-controller-7b5b9dbb8b       1         1         1       8m14s
default       replicaset.apps/dining-waterbuffalo-nginx-ingress-default-backend-fffbc96c4   1         1         1       8m14s
default       replicaset.apps/grafana-65bcd6c887                                            1         1         1       4m58s
default       replicaset.apps/prometheus-alertmanager-777d964c6b                            1         1         1       6m7s
default       replicaset.apps/prometheus-kube-state-metrics-5c5bc7                          1         1         1       6m7s
default       replicaset.apps/prometheus-pushgateway-5f457bff66                             1         1         1       6m7s
default       replicaset.apps/prometheus-server-9f8c98dbc                                   1         1         1       6m7s
default       replicaset.apps/trendsetting-tapir-nginx-ingress-controller-8656b89c6c        1         1         1       9m3s
default       replicaset.apps/trendsetting-tapir-nginx-ingress-default-backend-588dbcb588   1         1         1       9m3s
kube-system   replicaset.apps/coredns-86c58d9df4                                            2         2         2       56m
kube-system   replicaset.apps/kubernetes-dashboard-79ff88449c                               1         1         1       56m
kube-system   replicaset.apps/tiller-deploy-6f4dbc6d67                                      1         1         1       41m

$ helm list
NAME               	REVISION	UPDATED                 	STATUS  	CHART               	APP VERSION	NAMESPACE
dining-waterbuffalo	1       	Thu Dec  6 11:39:53 2018	DEPLOYED	nginx-ingress-0.29.2	0.20.0     	default  
grafana            	1       	Thu Dec  6 11:43:09 2018	DEPLOYED	grafana-1.17.3      	5.3.2      	default  
prometheus         	1       	Thu Dec  6 11:42:00 2018	DEPLOYED	prometheus-7.3.4    	2.4.3      	default  
trendsetting-tapir 	1       	Thu Dec  6 11:39:04 2018	DEPLOYED	nginx-ingress-0.29.2	0.20.0     	default  

$ kubectl get pods --namespace default -l "app=grafana" -o jsonpath="{.items[0].metadata.name}"
grafana-65bcd6c887-blvwz

$ kubectl --namespace default port-forward grafana-65bcd6c887-blvwz 3000

BROWSER: http://localhost:3000 --- admin:admin --- import json file if needed.

````


### Resetting the environment

Finally, reset all kubeadm installed state using `reset-site.yaml` playbook:

```sh
$ ansible-playbook reset-site.yaml
```
