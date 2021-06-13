# Signoz K8s Helm Operator

Note:- Make sure to update this operator  whenever you have Signoz helm charts updated. 

This doc explains how to create helm operator for Signoz's helm chart. 


- Initiate Signoz Operator. (Make sure you installed operator-sdk, helm and kubectl on your system.)

```
$ mkdir signoz-k8s-helm-operator
$ cd signoz-k8s-helm-operator
$ operator-sdk init --plugins helm --domain signoz.io --group monitor --version v1 --kind Signoz
```

- Clone Signoz Repo to another directory

```
$ git clone https://github.com/SigNoz/signoz.git && cd signoz
$ helm dependency update deploy/kubernetes/platform
```

- Copy Charts to Operator Charts dir and decompress file

```
$ cp -r signoz/deploy/kubernetes/platform/charts signoz-k8s-helm-operator/helm-charts/signoz/
$ cp -r signoz/deploy/kubernetes/platform/signoz-charts signoz-k8s-helm-operator/helm-charts/signoz/ 
$ cd signoz-k8s-helm-operator/helm-charts/signoz/charts/
$ tar -xvf *.tgz #if this bulk extract doesn't work in your terminal, do it for each file at a time. 
$ rm -rf *.tgz
```

- Create `requirements.yaml` file for dependencies, inside signoz-k8s-helm-operator/helm-charts/signoz/ . Below snippet is just a sample. 

```
dependencies:
- name: kafka
  version: 12.0.0
  repository: https://charts.bitnami.com/bitnami
- name: zookeeper
  version: 2.1.4
  repository: https://charts.helm.sh/incubator
- name: druid
  version: 0.2.18
  repository: https://charts.helm.sh/incubator
- name: flattener-processor
  repository: "file://./signoz-charts/flattener-processor"
  version: 0.2.0
- name: query-service
  repository: "file://./signoz-charts/query-service"
  version: 0.3.1
- name: frontend
  repository: "file://./signoz-charts/frontend"
  version: 0.3.1
```

- Edit `values.yaml` , inside signoz-k8s-helm-operator/helm-charts/signoz/ 

```
$ rm -rf values.yaml
$ touch values.yaml
$ vim values.yaml
```

- Edit `config/samples/monitor_v1_signoz.yaml`

```
$ rm -rf config/samples/monitor_v1_signoz.yaml
$ touch config/samples/monitor_v1_signoz.yaml
$ vim config/samples/monitor_v1_signoz.yaml
```

- Opertor build and Push to registry

```
$ export IMG=dther/signoz-operator:v0.07
$ make docker-build docker-push IMG=$IMG
```

- Install CRD on cluster and deploy operator on Cluster

```
$ make install 
$ make deploy IMG=$IMG
```

- Install Signoz on Cluster

```
$ kubectl create -f config/samples/monitor_v1_signoz.yaml
```


