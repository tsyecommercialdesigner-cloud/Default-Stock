---
title: DaemonSet
created: 2026-08-07
source: 《Kubernetes 零基础实战》
tags: DaemonSet
---
---

# 忠实可靠的看门狗 DaemonSet

Deployment 代表了在线业务，能够管理多个 Pod 副本，让应用永远在线，还能够任意扩容、缩容，但并没有完全解决部署应用程序的所有难题。与简单的离线业务相比，在线业务的应用场景太复杂，Deployment 的功能特性只能满足部分场景的需求。

本节介绍另一类管理在线业务的 API 对象：DaemonSet，它会在 Kubernetes 集群的每个节点上运行一个 Pod，就好像 Linux 系统里的守护进程（daemon）。[^1]

[^1]: Linux 系统里比较出名的守护进程是 systemd，它是系统里的 1 号进程，管理其他所有的进程。  

# 为什么要有 DaemonSet

Deployment 能够创建任意多个 Pod 实例，并且维护这些 Pod 的正常运行，保证应用始终处于可用状态。但 Deployment 并不关心这些 Pod 在集群的哪些节点上运行，在它看来，Pod 的运行环境与功能无关，只要 Pod 的数量足够应用程序就应该正常工作。

这个假设对于大多数业务来说是没问题的，例如 Nginx、WordPress、MySQL，它们不需要知道集群、节点的细节信息，只要配置好环境变量和存储卷，在哪里运行都是一样的。

不过有一些业务比较特殊，它们不完全独立于系统运行，而是与主机存在绑定关系，必须依附于节点才能产生价值。例如下面这些业务：

* 网络应用，必须每个节点运行一个 Pod，否则节点无法加入 Kubernetes 网络；
* 监控应用，必须每个节点运行一个 Pod 用来监控节点的状态，实时上报信息；
* 日志应用，必须每个节点运行一个 Pod，才能够收集容器运行时产生的日志数据；
* 安全应用，必须每个节点运行一个 Pod 来执行安全审计、入侵检查、漏洞扫描等工作。

这些业务不适合用 Deployment 来部署，因为 Deployment 管理的 Pod 数量是固定的，而且可能会在集群里漂移，但实际的需求却是要在集群里的每个节点上运行 Pod，所以 Kubernetes 定义了新的 API 对象 DaemonSet。虽然 DaemonSet 在形式上和 Deployment 类似，都是管理 Pod，但它们的调度策略不同。DaemonSet 的目标是在集群的每个节点上运行且仅运行一个 Pod，就好像是为节点配置一只看门狗，忠实地守护着节点，这就是 DaemonSet 名字的由来。[^2]

[^2]: 网络插件 Flannel 就是一个 DaemonSet，它位于名字空间“kube-flannel”，可以使用“`kubectl get ds -n kube-flannel`”命令查看。  

# 用 YAML 描述 DaemonSet

DaemonSet 和 Deployment 都属于在线业务，所以都是“apps”组，使用命令“`kubectl api-resources`”可以看到它的简称是“`ds`”，YAML 文件头信息如下：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: xxx-ds
```

但 Kubernetes 不提供自动创建 DaemonSet YAML 模板的功能，即不能使用命令“`kubectl create`”直接创建 DaemonSet 对象，如果用“`kubectl explain`”逐个查看字段再写 YAML 实在是太麻烦。

不过，可以在 Kubernetes 官网上复制任意一份 DaemonSet 的 YAML 示例，去掉多余部分做成一份 YAML 模板文件：

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: redis-ds
  labels:
    app: redis-ds

spec:
  selector:
    matchLabels:
      name: redis-ds

  template:
    metadata:
      labels:
        name: redis-ds
    spec:
      containers:
      - image: redis:7-alpine
        name: redis
        ports:
        - containerPort: 6379
```

这个 DaemonSet 对象的名字是“redis-ds”，镜像选是“redis:7-alpine”，使用了流行的非关系数据库 Redis。

对比这个 YAML 文件和 Deployment 的 YAML 文件，就会发现：前面的“kind”“metadata”是每个对象独有的信息，自然不同；“spec”部分，DaemonSet 也有“selector”字段匹配“template”里 Pod 的“labels”标签，和 Deployment 对象几乎一模一样。

再仔细观察，DaemonSet 在“spec”里没有“replicas”字段，这是它与 Deployment 的一个关键不同，意味着它不会在集群里创建多个 Pod 副本，而是在每个节点上只创建一个 Pod 实例。

也就是说，DaemonSet 仅仅是在 Pod 的部署调度策略上和 Deployment 不同，所以某种程度上也可以把 DaemonSet 看作 Deployment 的一个特例。

了解到 DaemonSet 与 Deployment 的区别就可以用变通的方法来创建 DaemonSet 的 YAML 模板文件了，只需要用“`kubectl create`”先创建一个 Deployment 对象，然后把“kind”改成“DaemonSet”，再删除“`spec.replicas`”就行了，如下是一个例子：

```bash
export out="--dry-run=client -o yaml"

# change "kind" to DaemonSet
kubectl create deploy redis-ds --image=redis:7-alpine $out \
  | sed 's/Deployment/DaemonSet/g' \
  | sed '/replicas/d'
```

# 用 kubectl 操作 DaemonSet

在 kubeadm 集群里执行命令“`kubectl apply`”，把 YAML 文件发送给 Kubernetes，就可以创建 DaemonSet 对象，再执行命令“`kubectl get`”查看对象的状态：

```text
[K8S ~]$kubectl apply -f ds.yml
daemonset.apps/redis-ds created

[K8S ~]$kubectl get pod -o wide
NAME            READY   STATUS    IP         NODE
redis-ds-d6wjj   1/1     Running   10.10.1.9  k8s-worker
```

虽然没有指定 DaemonSet 里运行的 Pod 数量，但它会自动查找集群里的节点，并在节点里创建 Pod。因为 kubeadm 实验环境里有一个控制面节点和一个数据面节点，而控制面节点默认不运行应用程序，所以 DaemonSet 只生成了一个 Pod，运行在了数据面节点上。

按照 DaemonSet 的设计，应该在每个节点上都运行一个 Pod 实例，但控制面节点却被排除在外了。显然，DaemonSet 没有尽到“看门”的职责，其设计与 Kubernetes 集群的工作机制发生了冲突。

# 污点和容忍度

为了应对 Pod 在某些节点上的调度和驱逐问题，Kubernetes 定义了两个概念：污点（taint）和容忍度（toleration）。[^3]

[^3]: 与污点、容忍度相关的另一个概念是节点亲和性（nodeAffinity），作用是更偏好选择哪些节点，用法略复杂。  

污点是 Kubernetes 节点的一个属性，它的作用是给节点添加标签，但为了不与已有的“labels”字段混淆，所以使用了“Taints”字段。和污点相对的，就是 Pod 的容忍度，顾名思义，就是 Pod 能否容忍污点。

在集群里，污点和容忍度配合使用，Pod 会根据自己对污点的容忍度来选择合适的节点，决定可能被调度到哪些节点上。

Kubernetes 在创建集群的时候会自动给节点设置一些污点，方便 Pod 的调度和部署。使用“`kubectl describe node`”命令能够查看控制面节点和数据面节点的状态：

```text
[K8S ~]$kubectl describe node k8s-master
Name:               k8s-master
Roles:              control-plane
...
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
...

[K8S ~]$kubectl describe node k8s-worker

Name:               worker
Roles:              <none>
...
Taints:             <none>
...
```

可以看到，控制面节点默认有一个污点“`node-role.kubernetes.io/control-plane`”，它的效果是“`NoSchedule`”，也就是说这个污点会拒绝 Pod 调度到本节点上运行，而数据面节点的“Taints”字段则是空的。[^4]

[^4]:在 Kubernetes 1.24 之前，污点“`node-role.kubernetes.io/control-plane`”的名字是“`node-role.kubernetes.io/master`”。  

这正是控制面节点和数据面节点在 Pod 调度策略上的区别，通常 Pod 不能容忍任何污点，所以无法调度到加上了污点属性的控制面节点上。

明白了污点和容忍度的概念，我们就知道该如何让 DaemonSet 在控制面节点（或者其他节点）上运行了，方法有以下两种。

第一种方法是去除控制面节点上的污点。操作节点的污点属性需要使用命令“`kubectl taint`”，并指定节点名、污点名和污点效果，如果是去掉污点要额外加上一个“-”（减号）。

例如去掉控制面节点的“NoSchedule”效果，命令如下：

```bash
kubectl taint node k8s-master \
  node-role.kubernetes.io/control-plane:NoSchedule-
```

因为 DaemonSet 一直在监控集群节点的状态，命令执行后控制面节点已经没有了污点，所以它立刻就会发现变化并在控制面节点上创建一个守护 Pod。

但这种方法修改的是节点的状态，影响面比较大，可能会导致很多 Pod 被调度到这个节点上运行，所以第二种方法是保留节点的污点，为需要的 Pod 添加容忍度，只允许某些 Pod 被调度到个别节点上，实现精准调度。具体方法是，为 Pod 添加字段“`tolerations`”，让它能够容忍某些污点，可以被调度到有某些污点的节点上。

“`tolerations`”是一个数组，可以包含多个被容忍的污点，需要写清楚污点的名字、效果。比较特别的是要通过“operator”字段指定如何匹配污点，一般情况下使用“Exists”，即存在这个名字和效果的污点。

如果想让 DaemonSet 里的 Pod 能够在控制面节点上运行，就要设置这样的“`tolerations`”，容忍节点的“`node-role.kubernetes.io/control-plane:NoSchedule`”污点：

```yaml
tolerations:
- key: node-role.kubernetes.io/control-plane
  effect: NoSchedule
  operator: Exists
```

可以先运行“`kubectl taint`”命令为控制面节点添加污点：

```bash
kubectl taint node k8s-master \
  node-role.kubernetes.io/control-plane:NoSchedule
```

再运行命令“`kubectl apply -f ds.yml`”重新部署设置了容忍度的 DaemonSet，就会看到 DaemonSet 仍然有两个 Pod，分别运行在控制面节点和数据面节点上，与第一种方法的效果相同。

需要特别说明的是，容忍度并不是 DaemonSet 独有的概念，而是 Pod 的属性，所以我们也可以在 Job、CronJob、Deployment 对象中为 Pod 加上“`tolerations`”，从而更灵活地调度应用。

理解了污点和容忍度的工作原理之后，读者可自行阅读 Kubernetes 官方文档，了解污点的各种效果。

# 静态 Pod

DaemonSet 是在 Kubernetes 里运行节点专属 Pod 的常用方式，但不是唯一方式，Kubernetes 还支持静态 Pod 的应用部署手段。

静态 Pod 非常特殊，它不受 Kubernetes 系统的管控，不与 apiserver、scheduler 通信，所以是静态的。[^5]

[^5]: 虽然静态 Pod 在 Kubernetes 里已经存在了很长时间，但因为它在系统之外，不方便管理，所以将来有被废弃的可能。

但既然它是 Pod，也必然会运行在容器运行时上，也会有 YAML 文件来描述它，而唯一能够管理它的 Kubernetes 组件也只能是在每个节点上运行的 kubelet。

静态 Pod 的 YAML 文件默认放在节点的“`/etc/kubernetes/manifests`”目录下，这个目录是 Kubernetes 的专用目录。

下面就是 kubeadm 集群控制面节点的目录：

```text
[K8S-CP ~]$ls /etc/kubernetes/manifests
-rw------- 1 root root 2.4K etcd.yaml
-rw------- 1 root root 3.9K kube-apiserver.yaml
-rw------- 1 root root 3.4K kube-controller-manager.yaml
-rw------- 1 root root 1.5K kube-scheduler.yaml
```

可以看到，Kubernetes 的 4 个核心组件 apiserver、etcd、scheduler 和 controller-manager 都是以静态 Pod 的形式存在的，这也是为什么它们能够先于 Kubernetes 集群启动的原因。

如果有某些特殊需求无法通过 DaemonSet 满足，则可以考虑使用静态 Pod，编写一个 YAML 文件放到“`/etc/kubernetes/manifests`”目录下，节点的 kubelet 会定期检查目录里的文件，发现变化就会调用容器运行时创建或者删除静态 Pod。

---

# 小结

本节介绍了在 Kubernetes 中部署应用程序的另一种方式：DaemonSet。它与 Deployment 类似，其区别在于 Pod 的调度策略，DaemonSet 适用于在系统里运行节点的守护进程。

本节的内容要点如下：

* **DaemonSet 的目标是为集群里的每个节点部署唯一的 Pod**，常用于监控、日志等业务；
* **DaemonSet 的 YAML 描述与 Deployment 非常接近**，只是没有“`replicas`”字段；
* **污点和容忍度是与 DaemonSet 相关的两个重要概念**，分别从属于节点和 Pod，共同决定了 Pod 的调度策略；
* **静态 Pod 也可以实现和 DaemonSet 同样的效果**，但它不受 Kubernetes 系统控制，必须在节点上手动部署，应当慎用。
