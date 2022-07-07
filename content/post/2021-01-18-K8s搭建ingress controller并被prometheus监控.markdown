---
layout:     post
title:      K8s搭建prometheus+grafana+ingress-controller
subtitle:   实现prometheus+grafana监控nginx-ingress-controller
date:       2021-01-18
author:     Bingo
header-img: img/k8s.png
catalog: true
tags:
    - k8s
    - prometheus
    - grafana
    - nginx
---

> 最近有个小伙伴在搭建k8s监控nginx-ingress-controller时发现监控不到，接下来就从0到1-搭建k8s，并配置prometheus监控ingress-controller。

# 1. 申请一个AKS集群
 😉 
 
 ---

# 2. 在本地实现对AKS的控制
### a. 安装kubectl
### b. 连接到aks(可以直接点击aks概述中的连接获取命令)
- 使用azure cli获取aks的配置信息（比如获取<yourself_cluster_name>的k8s配置）
```
az account set --subscription <yourself_subscriptionId>
az aks get-credentials --resource-group <yourself_resource_group> --name <yourself_cluster_name>
```
- 检查本地kubectl配置项
```
kubectl config get-contexts
```
- 切换到你的aks中(比如切换到namespace为**************的aks中)
```
kubectl config use-context <yourself_cluster_name>
```
### c. 验证是否连接成功
- 获取命名空间
```
kubectl get namespaces
```

- 获取所有部署项
```
kubectl get deployments --all-namespaces=true
```
- 获取所有服务
```
kubectl get svc --all-namespaces=true
```

---

# 3. K8s Dashboard
![k8s-dashboard](https://pic2.zhimg.com/80/v2-006e3352e5471a0c9c885a84f6184814_1440w.png)
### a. 部署dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
```

### b. 在本地访问dashboard
- 启动本地代理
```
kubectl proxy
```
然后再浏览器中直接访问连接即可：http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
- Token查找方法

进入aks主页 → 配置 → 机密 → namespace选择kubernetes-dashboard → kubernetes-dashboard-token-****

---

# 4. Prometheus & Grafana

### a. 安装prometheus&grafana
- 克隆kube-prometheus代码，这里要注意服务器的k8s版本，选择对应版本的代码，对应关系见项目readme
```
git clone -b release-0.6 git@github.com:prometheus-operator/kube-prometheus.git
cd kube-prometheus
kubectl create -f manifests/setup
```
- 等上面的应用部署完成后再执行下面这个命令!
```
kubectl create -f manifests/
```
### b. 访问prometheus
公网IP：修改service/grafana的type为LoadBanlancer，稍等片刻会出现对应的external ip地址
```
...
spec:
  ...
  type: LoadBalancer
  ...
...
```
![prometheus-dashboard](https://pic2.zhimg.com/80/v2-fece38580065c4a7ef7e526b418a6e3d_1440w.png)
### c. 访问grafana
免密登录：修改deployments/grafana中的yaml
```
...
spec:
  ...
  template:
    ...
    containers:
      - name: grafana
        ...
        env:
          - name: GF_AUTH_PROXY_ENABLED
            value: 'true'
          - name: GF_AUTH_ANONYMOUS_ENABLED
            value: 'true'
          - name: GF_AUTH_ANONYMOUS_ORG_ROLE
            value: Admin
        ...
...
```
- 公网IP：修改service/grafana的type为LoadBanlancer，稍等片刻会出现对应的external ip地址
```
...
spec:
  ...
  type: LoadBalancer
  ...
...
```
![grafana-dashboard](https://pic4.zhimg.com/80/v2-51d0d1d1cf21e3c8304cefc5fe6e24d6_1440w.png)

---

# 5. Ingress Controller
### a. 使用helm3安装nginx-ingress-controller
- 添加helm源
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
```
- 更新helm源
```
helm repo update
```
- 获取ingress-nginx参数(非必须)
```
helm show values ingress-nginx/ingress-nginx
```
- 安装ingress-nginx
```
helm install nginx-ingress ingress-nginx/ingress-nginx \
      --namespace monitoring \
      --set controller.metrics.enabled=true \
      --set controller.metrics.serviceMonitor.enabled=true \
      --set defaultBackend.enabled=true
```
### b. Grafana中添加nginx ingress controller面板
Grafana → Create Import → Input dashboard id: 9614 → Load → chose Prometheus data source (default) → Import

![ingress-controller](https://pic1.zhimg.com/80/v2-f7817951379ddb0188bc9fbb7f480ef2_1440w.png)

---

# Q&A
- 出现Back-off restarting failed container
```
...
spec:
  containers:
  - name: ingress-nginx
    command: [ "/bin/bash", "-ce", "tail -f /dev/null" ]
    ...
...
```
- pod日志出现：Invalid IngressClass (Spec.Controller) value "nginx.org/ingress-controller". Should be "k8s.io/ingress-nginx"
```
kubectl delete ingressclass nginx
```
- 查看helm可配置的参数
```
helm show values ingress-nginx/ingress-nginx
```
- 访问dashboard
```
kubectl proxy
```
Brower open url: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

- 新部署的nginx-ingress-controller,prometheus监控不到，grafana中的controller没有新的pod？

检查自定义资源中，ServiceMonitor中对应的spec: selector是否与新的ingress controller一致

---

# Preference
- https://docs.microsoft.com/en-us/azure/aks/ingress-internal-ip
- https://docs.openshift.com/container-platform/4.4/rest_api/monitoring_apis/servicemonitor-monitoring-coreos-com-v1.html
- https://prometheus.io/docs/prometheus/latest/configuration/configuration/#metric_relabel_configs
