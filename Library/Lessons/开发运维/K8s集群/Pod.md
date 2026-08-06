---
title: Pod
created: 2026-08-06
source: 《Kubernetes 零基础实战》
tags:
  - Pod
  - Kubernetes_Pod
---
---

# Pod 的核心概念

Pod 这个词原意是“豌豆荚”，后来又延伸出“舱室”、“太空舱”等含义[^1]，形象地说 Pod 就是包含了很多组件、成员的一种结构。如果把容器比作是小小的豌豆，那么 Pod 就相当于是把豌豆组织在一起的那层“豌豆荚”。在 Pod 的 YAML 定义里，`spec.containers` 字段其实是一个数组，里面允许定义多个容器[^2]。

[^1]: 在科幻电影里“Pod”常用采称呼飞船的分离舱，而 Apple 的音乐播放器 iPod、HomePod 从属于 Mac，与飞船和分离舱的关系有点类似，所以被命名为“Pod”。
[^2]: Pod 内部有一个名为 `infra` 的“隐藏”容器，它实际上代表了 Pod，维护着 Pod 内多容器共享的主机名、网络和存储。`infra` 容器的镜像叫 `pause`，非常小，只有不到 500 KB。

Pod 是为了解决多应用联合运行的问题而设计的，它可以确保多个应用联合运行，同时不破坏容器的隔离环境。Pod 就像是在容器外建立的“收纳舱”，让多个容器既保持相对独立，又能够小范围共享网络、存储等资源，而且永远是“绑定在一起”的状态。

Pod 是对容器的“打包”，里面的容器是一个整体，总是能够一起调度、一起运行，绝不会出现分离的情况，而且 Pod 属于 Kubernetes，可以在不触碰底层容器运行时的情况下任意定制修改。所以有了 Pod 这个抽象概念，Kubernetes 在集群级别上管理应用就会更方便。

Kubernetes 让 Pod 去编排处理容器，然后把 Pod 作为应用调度部署的最小单位，Pod 也因此成了 Kubernetes 世界里的“原子”（当然这个“原子”内部是有结构的，不是铁板一块），基于 Pod 就可以构建出更多更复杂的业务形态。

以 Pod 为中心的 Kubernetes 资源对象关系：

- 从 Pod 开始，扩展出了 Kubernetes 里的一些重要 API 对象，例如配置信息 `ConfigMap`、离线作业 `Job`、多实例部署 `Deployment` 等，分别对应现实中的各种运维需求，比较全面地描述了 Kubernetes 的资源对象。
- 所有的 Kubernetes 资源都直接或者间接地依附在 Pod 之上，Kubernetes 的所有功能都必须通过 Pod 来实现，所以 Pod 成了 Kubernetes 的核心对象。

---

# 用 YAML 描述 Pod

Pod 非常重要，理解了 Pod 的概念，Kubernetes 学习之旅就成功了一半。

因为始终可以用“`kubectl explain`”来查看任意字段的详细说明，所以本节只简要介绍编写 YAML 时 Pod 里的一些常用字段。

Pod 是 API 对象，它也必然具有 `apiVersion`、`kind`、`metadata` 和 `spec` 4 个基本组成部分。

`apiVersion` 和 `kind` 这两个字段很简单，对于 Pod 来说分别是固定值“`v1`”和“`Pod`”，而 `metadata` 里通常应该有 `name` 和 `labels` 这两个字段。

在使用 Docker 创建容器的时候，可以不给容器起名字，但在 Kubernetes 里，Pod 必须有一个名字，这也是 Kubernetes 里所有资源对象的一个约定[^3]。

[^3]: 本书通常会为 Pod 的名字统一加上“pod”后缀，这样可以和其他类型的资源区分开。

`name` 只是一个基本标识，信息有限，而 `labels` 字段可以添加任意数量的键值对，给 Pod 添加归类的标签，结合 `name` 更容易识别和管理。

例如，可以根据运行环境，使用标签 `env=dev/test/prod`，或者根据所在的数据中心，使用标签 `region: north/south`，还可以根据应用在系统中的层次，使用标签 `tier=front/middle/back`，等等[^4]。

[^4]: `metadata` 中的标签必须符合域名规范（FQDN），不能随意写。

下面这段 YAML 代码描述了一个简单的 Pod，名字是 `busy-pod`，再附加一些标签：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: busy-pod
  labels:
    owner: chrono
    env: demo
    region: north
    tier: back
```

`metadata` 一般包含 `name` 和 `labels` 两个字段就够了，而 `spec` 由于需要管理、维护 Pod 这个 Kubernetes 的基本调度单元，里面有非常多的关键信息，本节只介绍最重要的 `containers` 字段，其他字段（如 `hostname`、“`restartPolicy`”等）读者可以自行学习。

`containers` 字段是一个数组，里面的每个元素是一个 container 对象，也就是容器。

和 Pod 一样，container 对象也必须用 `name` 表示名字，然后还要用一个 `image` 字段来表示它的镜像，这两个字段是必须的，否则 Kubernetes 会报告数据验证错误。

container 对象的其他字段基本上都可以和第 2 章里的容器技术对应，比较容易理解，以下是部分字段：

- `ports`：列出容器对外暴露的端口，和 Docker 的 `-p` 参数类似。
- `imagePullPolicy`：指定镜像的拉取策略，可以是 `Always`、`Never` 或 `IfNotPresent`。一般默认 `IfNotPresent`，表示只有本地不存在时才会远程拉取镜像，以减少网络消耗。
- `env`：定义 Pod 的环境变量，和 Dockerfile 里的 `ENV` 指令类似，但它是容器运行时指定的，更加灵活、可配置。
- `command`：定义容器启动时要执行的命令，相当于 Dockerfile 里的 `ENTRYPOINT`。
- `args`：它是 command 运行时参数，相当于 Dockerfile 里的 `CMD`[^5]。

[^5]: 特别注意：`command`、`args` 这两条命令和 Docker 里的 ENTRYPOINT、CMD 含义不完全相同。

下面编写 `busy-pod` 的 `spec` 部分，添加 `env`、“`command`”、“`args`”等字段：

```yaml
spec:
  containers:
    - image: busybox:latest
      name: busy
      imagePullPolicy: IfNotPresent
      env:
        - name: os
          value: "ubuntu"
        - name: debug
          value: "on"
      command:
        - /bin/echo
      args:
        - "$(os), $(debug)"
```

这个 YAML 为 Pod 指定使用镜像 `busybox:latest`，拉取策略是 `IfNotPresent`，并定义了 `os` 和 `debug` 两个环境变量，启动命令是 `/bin/echo`，参数里输出刚才定义的环境变量。

对比这份 YAML 文件和 Docker 命令就可以看出，YAML 在 `spec.containers` 字段里用声明式的方式把容器的运行状态描述得非常清晰，比 `docker run` 这样的命令行要整洁得多，对人、对计算机都非常友好。

---

# 用 kubectl 操作 Pod

有了描述 Pod 的 YAML 文件，本节介绍用来操作 Pod 的 kubectl 命令。

`kubectl apply` 和 `kubectl delete` 已经介绍过，它们可以使用 `-f` 参数指定 YAML 文件创建或者删除 Pod，例如：

```bash
kubectl apply  -f busy-pod.yml
kubectl delete -f busy-pod.yml
```

因为在 YAML 里定义了 `name` 字段，所以也可以直接指定 Pod 的名字来删除：

```bash
kubectl delete pod busy-pod
```

和 Docker 不一样，Kubernetes 的 Pod 不会在前台运行，只能在后台（相当于默认使用了参数 `-d`），所以不能直接看到输出信息。命令 `kubectl logs` 可以显示 Pod 的标准输出流信息，执行如下命令后显示的是 Pod 预设的两个环境变量的值：

```bash
[K8S ~]$ kubectl logs busy-pod
ubuntu, on
```

使用命令 `kubectl get pod` 可以查看 Pod 列表和运行状态：

```text
[K8S ~]$ kubectl get pod
NAME       READY   STATUS              RESTARTS   AGE
busy-pod   0/1     CrashLoopBackOff    5          <invalid> ago
```

可以发现这个 Pod 运行不正常，状态是 `CrashLoopBackOff`，继续执行命令 `kubectl describe` 可以检查详细状态（这在调试排错时很有用）：

```text
[K8S ~]$ kubectl describe pod busy-pod
Name:         busy-pod
Namespace:    default
...
Events:
  Type     Reason     Message
  ----     ------     -------
  Normal   Scheduled  Successfully assigned default/busy-pod to k8s-worker
  Normal   Pulling    Pulling image "busybox:latest"
  Normal   Pulled     Successfully pulled image "busybox:latest"
  Normal   Created    Created container busy
  Normal   Started    Started container busy
  Warning  BackOff    Back-off restarting failed container busy in pod
```

通常需要关注的是 `Events` 部分，它显示的是 Pod 运行过程中的一些关键节点事件。对于这个 `busy-pod`，因为它只执行了一条 `echo` 命令就退出了，而 Kubernetes 默认会重启 Pod，所以会进入一个反复循环“停止-启动”的错误状态[^6]。

[^6]: 对于确实不需要重启的 Pod，可以配置字段 `restartPolicy: Never`。

因为 Kubernetes 里运行的应用大部分是不会主动退出的服务，所以我们可以把 `busy-pod` 删除，用创建的 `ngx-pod.yml`，启动一个 Nginx 服务。这才是大多数 Pod 的工作方式。

启动之后，再用 `kubectl get pod` 查看 Pod 的状态，可以发现它已经是 `Running` 状态[^7]：

[^7]: `kubectl get pod` 的 `READY` 列显示的是 Pod 内部的容器状态，格式是 `x/y`，表示 Pod 里总共定义了 $y$ 个容器，其中 $x$ 个的状态是 ready。

```bash
[K8S ~]$ kubectl apply -f ngx-pod.yml
pod/ngx-pod created

[K8S ~]$ kubectl get pod
NAME      READY   STATUS    RESTARTS   AGE
ngx-pod   1/1     Running   0          6s
```

运行命令 `kubectl logs` 能够正常输出 Nginx 的运行日志：

```text
[K8S ~]$ kubectl logs ngx-pod
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/XX/XX 11:28:46 [notice] 1#1: using the "epoll" event method
2023/XX/XX 11:28:46 [notice] 1#1: nginx/1.25.1
2023/XX/XX 11:28:46 [notice] 1#1: built by gcc 12.2.1 20220924
2023/XX/XX 11:28:46 [notice] 1#1: OS: Linux 5.15.0-78-generic
2023/XX/XX 11:28:46 [notice] 1#1: getrlimit(RLIMIT_NOFILE)
2023/XX/XX 11:28:46 [notice] 1#1: start worker processes
```

kubectl 提供了与 Docker 类似的 `cp` 和 `exec` 命令，`kubectl cp` 命令可以把本地文件拷贝进 Pod，`kubectl exec` 命令是进入 Pod 内部执行 Shell 命令[^8]。

[^8]: 准确地说，`kubectl cp` 和 `kubectl exec` 操作的是 Pod 里的容器，需要用 `-c` 参数指定容器名，不过因为大多数 Pod 里只有一个容器，所以通常可以省略。

例如有一个 `a.txt` 文件，可以使用 `kubectl cp` 命令把这个文件拷贝到 Pod 的 `/tmp` 目录里：

```bash
echo 'aaa' > a.txt
kubectl cp a.txt ngx-pod:/tmp
```

不过 `kubectl exec` 的命令格式与 Docker 有所区别，需要在 Pod 后面加上 `--`，把 kubectl 的命令与 Shell 命令分隔开，使用的时候需要注意：

```bash
kubectl exec -it ngx-pod -- sh
```

```bash
uname -r
```

```bash
ls /tmp
```

```bash
nginx -v
```

输出如下：

```plain
[K8S ~]$ kubectl exec -it ngx-pod -- sh

/ # uname -r
5.15.0-78-generic
/ # ls /tmp
a.txt
/ # nginx -v
nginx version: nginx/1.25.1
```

---

# 小结

本节介绍了 Kubernetes 里核心且基本的概念 Pod，使用 YAML 来定制 Pod，以及使用 kubectl 命令来创建、删除、查看、调试 Pod。

Pod 屏蔽了容器技术的底层细节，同时又具备足够的控制管理能力，比起容器的“细粒度”、虚拟机的“粗粒度”，Pod 可以说是“中粒度”，灵活又轻便，非常适合在云计算领域作为应用调度的基本单元，反而成了 Kubernetes 世界里构建一切业务的“原子”。

本节的内容要点如下：

- 现实中有很多应用需要多个进程密切协作才能完成任务，仅使用容器很难描述这种协作关系，所以就出现了 Pod，它“打包”了一个或多个容器，保证里面的进程能够被整体调度；
- Pod 是 Kubernetes 管理应用的最小单位，其他所有概念都是从 Pod 衍生出来的；
- Pod 也应该使用 YAML 描述，关键字段是 `spec.containers`，列出名字、镜像、端口等要素，定义内部的容器运行状态；
- 操作 Pod 的很多命令与 Docker 类似，如 `kubectl run`、`kubectl cp`、`kubectl exec` 等，这些命令的用法有些小差异，使用的时候需要注意。

虽然 Pod 是 Kubernetes 的核心概念，但事实上在 Kubernetes 里通常不会直接创建 Pod，因为它只是对容器做了简单的包装，离复杂的业务需求还有些距离，需要通过 Job、CronJob、Deployment 等其他对象增添更多的功能才能投入生产环境使用。

---








