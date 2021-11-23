---
title: rancher集群修改ingress-controller为Traefik
date: 2021/04/20
updated: 2021/04/20
comments: true
tags: 
- Rancher
- Ingress
- Traefik
categories:
- Rancher
- Ingress
---

[TOC]

# 背景说明
在一些情况下，Rancher安装部署的集群为nginx-ingress-cntroller，我们需要更换为Traefik用作Kubernetes集群的Ingress控制器，本文将rancher已经部署的集群从nginx-ingress-controller切换为Traefik-controller；

# 准备工作
## 关闭Rancher创建集群的nginx-controller
* 注意：关闭自带的nginx-controller后在部署好Traefik之前，已经存在的ingress访问将受到影响

### Rancher Custom Cluster关闭nginx-controller
通过Rancher UI以自定义方式的方式创建的集群关闭方式如下：
 
 1、编辑自定义集群，找到高级集群选项，在Nginx-Ingress选项中选择禁用，然后保存集群，随后会触发集群自动更新；
 
 ![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E6%96%87%E7%AB%A0%E5%9B%BE%E7%89%87/rancher%E9%9B%86%E7%BE%A4%E4%BF%AE%E6%94%B9ingress-controller%E4%B8%BATraefik/clipboard_20210420_022445.png)
 
 ![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E6%96%87%E7%AB%A0%E5%9B%BE%E7%89%87/rancher%E9%9B%86%E7%BE%A4%E4%BF%AE%E6%94%B9ingress-controller%E4%B8%BATraefik/clipboard_20210420_022449.png)
 
 2、等待更新完毕后，执行kubectl命令检查集群中是否还有nginx-ingress命名空间和nginx-ingress工作负载,没有ginx-ingress命名空间和nginx-ingress工作负载为正确关闭，此时集群内所有ingress都无法正常访问；
 
 ![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E6%96%87%E7%AB%A0%E5%9B%BE%E7%89%87/rancher%E9%9B%86%E7%BE%A4%E4%BF%AE%E6%94%B9ingress-controller%E4%B8%BATraefik/clipboard_20210420_022453.png)
 
 ### RKE Cluster关闭nginx-controller
通过RKE方式部署的集群关闭方式如下：
1、编辑集群cluster.yml文件，配置以下参数：
```
ingress:
  provider: none ##设置`provider: none`禁用 ingress 控制器
```
2、保存文件后执行rke up命令更新集群：
```
rke up --config cluster.yml
```
3、集群更新完成后，执行kubectl命令检查集群中是否还有nginx-ingress命名空间和nginx-ingress工作负载,没有ginx-ingress命名空间和nginx-ingress工作负载为正确关闭，此时集群内所有ingress都无法正常访问；

# 部署Traefik
[参考链接](https://doc.traefik.io/traefik/v1.7/user-guide/kubernetes/)

## 创建集群RBAC
```
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
rules:
  - apiGroups:
      - ""
    resources:
      - services
      - endpoints
      - secrets
    verbs:
      - get
      - list
      - watch
  - apiGroups:
      - extensions
    resources:
      - ingresses
    verbs:
      - get
      - list
      - watch
  - apiGroups:
    - extensions
    resources:
    - ingresses/status
    verbs:
    - update
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: traefik-ingress-controller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: traefik-ingress-controller
subjects:
- kind: ServiceAccount
  name: traefik-ingress-controller
  namespace: kube-system
```
或者使用kubectl命令直接创建
```
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v1.7/examples/k8s/traefik-rbac.yaml

```

## 使用DaemonSet部署Traefik
```
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
---
kind: DaemonSet
apiVersion: apps/v1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  selector:
    matchLabels:
      k8s-app: traefik-ingress-lb
      name: traefik-ingress-lb
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      containers:
      - image: traefik:v1.7
        name: traefik-ingress-lb
        ports:
        - name: http
          containerPort: 80
          hostPort: 80
        - name: admin
          containerPort: 8080
          hostPort: 8080
        securityContext:
          capabilities:
            drop:
            - ALL
            add:
            - NET_BIND_SERVICE
        args:
        - --api
        - --kubernetes
        - --logLevel=INFO
---
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      port: 80
      name: web
    - protocol: TCP
      port: 8080
      name: admin
```
或者使用kubectl命令直接创建
```
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v1.7/examples/k8s/traefik-ds.yaml
```

 ![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E6%96%87%E7%AB%A0%E5%9B%BE%E7%89%87/rancher%E9%9B%86%E7%BE%A4%E4%BF%AE%E6%94%B9ingress-controller%E4%B8%BATraefik/clipboard_20210420_022456.png)

## 部署后的检查
1、部署完成后通过kubectl命令检查Pod是否正确部署
```
kubectl --namespace=kube-system get pods
```
2、在Rancher UI中可以到System项目中检查kube-system命名空间，UI上应该显示traefik-ingress-controller工作负载；

 ![avatar](https://zknow-1256858200.cos.ap-guangzhou.myqcloud.com/%E6%96%87%E7%AB%A0%E5%9B%BE%E7%89%87/rancher%E9%9B%86%E7%BE%A4%E4%BF%AE%E6%94%B9ingress-controller%E4%B8%BATraefik/clipboard_20210420_022458.png)

# 检查是否部署成功，并切换为Traefik-controller
在Rancher 2.4.15-ent版本中，当traefik-ingress-controller部署完成后，已有的Ingress就自动可以恢复访问，新建的ingress规则也自动使用了traefik-ingress-controller。
* 直接访问ingress域名可以检查ingress是否正常；
* 通过访问traefik的8080端口可以访问traefik UI查看已经存在的ingress host；

# 说明
* 切换完成traefik-ingress-controller后Rancher UI上新建的Ingress会处于Initializing状态，这个是因为获取不到status.loadBalancer.ingress,这个是正常现象，ingress本身是可以正常访问的,如果想解决这个状态，可以通过升级traefik-ingress-controller添加以下参数解决
```
--kubernetes.ingressendpoint.hostname=www.domain.com
```


# 参考链接

* ingressendpoint说明：https://doc.traefik.io/traefik/providers/kubernetes-ingress/#ingressendpoint
* traefik官方文档：https://doc.traefik.io/traefik/
* traefik安装官方文档：https://doc.traefik.io/traefik/getting-started/install-traefik/
* Rancher RKE cluster.yml示例：https://docs.rancher.cn/docs/rke/example-yamls/_index/
* Rancher Ingress配置选项：https://docs.rancher.cn/docs/rke/config-options/add-ons/ingress-controllers/_index/#%E7%A6%81%E7%94%A8%E9%BB%98%E8%AE%A4-ingress-controller