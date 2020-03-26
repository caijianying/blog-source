---
title: Window下Docker Desktop搭建 Kubernetes
tags:
  - 教程
originContent: ''
categories:
  - docker
toc: false
date: 2019-09-29 09:31:05
---


# Window下Docker Desktop搭建 Kubernetes

## 前言

本节主要讲解如何启用Kubernetes，以及如何搭建Kubernetes Dashboard。如果排除掉网络原因，本文没有任何意思，因为众所周知的原因，谷歌资源被墙，所以才存在搭建问题，这也就是写本文的原因。

因为不了解Kubernetes能做什么，所以才想着先搭建一个环境，玩一玩，看看这个到底能做什么。

## 准备

Docker Desktop 版本：2.1.0.1

支持Kubernetes版本：v1.14.3

查看这个版本很重要，具体查看About Docker Desktop菜单即可知道支持哪个版本的k8s。

### 首先安装Docker Desktop

安装Docker Desktop步骤略....

安装好Docker Desktop先别启用k8s。

### 其次拉取镜像

先把需要的镜像拉取下来，可以写个**docker-k8s-images.bat**，放入以下内容：

```bat
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.14.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.14.3 k8s.gcr.io/kube-proxy:v1.14.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.14.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.14.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.14.3 k8s.gcr.io/kube-scheduler:v1.14.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.14.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.14.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.14.3 k8s.gcr.io/kube-controller-manager:v1.14.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.14.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.14.3
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.14.3 k8s.gcr.io/kube-apiserver:v1.14.3
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.14.3

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1 k8s.gcr.io/pause:3.1
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.1

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.3.10

docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:1.3.1


docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1 k8s.gcr.io/kubernetes-dashboard-amd64:v1.10.1
docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64:v1.10.1
```

其中**kubernetes-dashboard-amd64**为Kubernetes Dashboard，不是必须镜像以外，其他都是k8s必须的镜像。

### 最后启动Kubernetes

在Kubernetes菜单选项里，勾选所有的选项。然后执行`kubectl get pods --namespace kube-system`查看k8s相关容器是否启动。当启动必须的7个容器以后，再查看Docker Desktop左下角Kubernetes状态即为绿色。

```
C:\Users\lab>kubectl get pods --namespace kube-system
NAME                                     READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-4w2ht                  1/1     Running   1          17m
coredns-fb8b8dccf-b5vdv                  1/1     Running   1          17m
etcd-docker-desktop                      1/1     Running   0          16m
kube-apiserver-docker-desktop            1/1     Running   0          16m
kube-controller-manager-docker-desktop   1/1     Running   0          16m
kube-proxy-7w9lw                         1/1     Running   0          17m
kube-scheduler-docker-desktop            1/1     Running   0          16m
```

![](http://blogstatic.aibibang.com/docker%20desktop.png)

## 搭建Kubernetes Dashboard

### 步骤1

部署Dashboard ,执行以下命令：

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml`

注意不同Dashboard选择不同的版本配置文件，这里的地址可以在[kubernetes/dashboard/releases](https://github.com/kubernetes/dashboard/releases)获取不同版本文件。

### 步骤2 Creating sample user

新建`dashboard-adminuser.yaml`文件，填写如下内容：

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system

```



### 步骤3  Bearer Token

步骤2完成，执行`kubectl proxy`既可以访问Dashboard,但是需要登录。执行如下命令：

`kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')`

生成如下结果：

```
Name:         admin-user-token-6gl6l
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name=admin-user
              kubernetes.io/service-account.uid=b16afba9-dfec-11e7-bbb9-901b0e532516

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTZnbDZsIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJiMTZhZmJhOS1kZmVjLTExZTctYmJiOS05MDFiMGU1MzI1MTYiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06YWRtaW4tdXNlciJ9.M70CU3lbu3PP4OjhFms8PVL5pQKj-jj4RNSLA4YmQfTXpPUuxqXjiTf094_Rzr0fgN_IVX6gC4fiNUL5ynx9KU-lkPfk0HnX8scxfJNzypL039mpGt0bbe1IXKSIRaq_9VW59Xz-yBUhycYcKPO9RM2Qa1Ax29nqNVko4vLn1_1wPqJ6XSq3GYI8anTzV8Fku4jasUwjrws6Cn6_sPEGmL54sq5R4Z5afUtv-mItTmqZZdxnkRqcJLlg2Y8WbCPogErbsaCDJoABQ7ppaqHetwfM_0yMun6ABOQbIwwl8pspJhpplKwyo700OSpvTT9zlBsu-b35lzXGBRHzv5g_RA
```

现在访问：

```
http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/
```

复制以上生成的token,填入token,即可显示如下页面：

![](http://blogstatic.aibibang.com/k8s.png)

至此k8s部署成功！Enjoy!

## 参考

1. [如何成功启动 Docker 自带的 Kubernetes？](https://www.jianshu.com/p/e5c056baa8ab)
2. [kubernetes/dashboard](https://github.com/kubernetes/dashboard)
3. [Creating-sample-user](https://github.com/kubernetes/dashboard/wiki/Creating-sample-user)
