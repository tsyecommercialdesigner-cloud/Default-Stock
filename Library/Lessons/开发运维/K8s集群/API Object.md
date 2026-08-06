---
title: API Object
created: 2026-08-05
source: Cherry Studio
tags:
  - YAML
  - Kubernetes_API_Object
source_url: 实体书
---
---
# 什么是 API 对象

YAML 语言只相当于“语法”，要与 Kubernetes 对话，还必须有足够的“词汇”来表达“语义”，能够让 Kubernetes 明白我们的意思。

作为一个集群操作系统，Kubernetes 归纳总结了 Google 多年的经验，在理论层面抽象出了很多概念，用来描述系统的管理运维工作。

`apiserver` 是 Kubernetes 系统的唯一管理入口，外部用户和内部组件必须和它通信，而 `apiserver` 采用了 HTTP 的 URL 资源理念，API 风格用的是 RESTful 的 GET、POST、DELETE 等，所以，Kubernetes 中的资源很自然地就被称为 API 对象。

使用

 ```bash
 kubectl api-resources
 ``` 

可以查看当前 Kubernetes 版本支持的所有对象：

```text
[K8S ~]$ kubectl api-resources
NAME                      SHORTNAMES   APIVERSION   KIND
configmaps                cm           v1           ConfigMap
endpoints                 ep           v1           Endpoints
namespaces                ns           v1           Namespace
nodes                     no           v1           Node
persistentvolumeclaims    pvc          v1           PersistentVolumeClaim
persistentvolumes         pv           v1           PersistentVolume
pods                      po           v1           Pod
secrets                                v1           Secret
services                  svc          v1           Service
daemonsets                ds           apps/v1      DaemonSet
...
```

第一列 NAME 是对象的名字，例如 configmaps、pods、services 等；第二列 SHORTNAMES 则是对象名字的缩写，在使用 kubectl 命令的时候可以减少键盘输入，例如 pods 可以简写为 po、services 可以简写为 svc；第三列 APIVERSION 代表对象的版本；第四列 KIND 代表对象的类型。

如果使用 kubectl 命令时加上参数 `--v=9`，就会显示详细的命令行执行过程，清楚地看到发出的 HTTP 请求，例如：

```bash
kubectl get pod --v=9
```

会返回：

```text
[K8S ~]$ kubectl get pod --v=9
Config loaded from file:  /home/chrono/.kube/config
curl -v -XGET 'https://192.168.26.210:6443/api/v1/namespaces/default/pods?limit=500'
HTTP Trace: Dial to tcp:192.168.26.210:6443 succeed
GET https://192.168.26.210:6443/api/v1/namespaces/default/pods?limit=500
Response Headers:
Audit-Id: c8bc3847-c87e-47e4-ac34-379726369d71
Cache-Control: no-cache, private
Content-Type: application/json
X-Kubernetes-Pf-Flowschema-Uid: ebb14040-ff2c-4fb5-8ef0-fc9f06c2b782
X-Kubernetes-Pf-Prioritylevel-Uid: 73afbd1e-2c6b-4837-ae71-b3bed880e9ba
Content-Length: 2957
```

可以看到，kubectl 客户端等价于调用 curl，向 6443 端口发送了 HTTP GET 请求，URL 是 `/api/v1/namespaces/default/pods`。

Kubernetes 1.27.3 版本有 50 多种 API 对象，全面描述了集群的节点、应用、配置、服务、账号等信息，apiserver 会把它们存储在数据库 etcd 里，kubelet、scheduler、controller-manager 等组件通过 apiserver 来访问它们，就在 API 对象这个抽象层次实现了对整个集群的管理。

---
# YAML 描述 API 对象

下面使用 YAML 语言在 Kubernetes 里描述并创建 API 对象。

之前验证 Kubernetes 时启动了一个 Nginx 应用，执行的命令是 `kubectl run`，和 Docker 一样是命令行的方式：

```bash
kubectl run ngx --image=nginx:alpine
```

现在可以把它改写成声明式的 YAML，描述清楚 Nginx 应用的属性，也就是目标状态，由 Kubernetes 决定如何拉取镜像并运行：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod
  labels:
    env: demo
    owner: chrono

spec:
  containers:
    - image: nginx:alpine
      name: ngx
      ports:
        - containerPort: 80
```

借助 已介绍的 YAML 语言知识应该能够明白，这个 Nginx 应用是一个 Pod，要使用 `nginx:alpine` 镜像创建一个容器，开放端口 80，其他部分是 Kubernetes 对 API 对象强制的格式要求。

因为 API 对象采用的是标准 HTTP，为了方便理解可以借鉴 HTTP 的报文格式，把 API 对象的描述分成“header”和“body”两部分。

“header”部分包含的是 API 对象的基本信息，包括 3 个字段：

> - `apiVersion`
> - `kind` 
> -  `metadata`

`apiVersion` 表示操作这种资源的 API 版本号，例如 v1、v1alpha1、v1beta1 等。由于 Kubernetes 的迭代速度很快，不同版本创建的对象有所差异，为了区分这些版本需要使用 `apiVersion` 字段[^3]。

[^3]: Kubernetes 的 API 版本命名有明确规范，正式版本（Generally Available，GA）是“v+数字”，如 v1；测试性质、不稳定版本的版本号带有“alpha”，如 v1alpha1；比较稳定、即将发布版本的版本号带有“beta”，如 v1beta1。

`kind` 表示资源对象的类型，例如 Pod、Node、Job、Service 等。

`metadata` 表示资源的一些元信息，也就是用来标记对象，方便 Kubernetes 管理的一些信息。

在上面的 YAML 示例里有两个元信息，一个是 `name`，定义 Pod 的名字是 `ngx-pod`；另一个是 `labels`，可以理解为便于查找 Pod 的标签，分别是 `env` 和 `owner`。

`apiVersion`、`kind` 和 `metadata` 被 `kubectl` 用来生成发给 `apiserver` 的 HTTP 请求（可以用 `--v=9` 参数在请求的 URL 里看到它们）。

和 HTTP 一样，“header”部分包含的 `apiVersion`、`kind` 和 `metadata` 3 个字段是任意对象都必须有的，而“body”部分则与特定对象相关，每种对象有不同的规格定义，在 YAML 里表现为 `spec`（即 specification）字段，表示对象的期望状态。

上面例子的 Pod 里，`spec` 里是一个 `containers` 数组，数组中的每个元素是一个对象，指定了名字、镜像、端口等信息。

综合这些字段看，这份 YAML 文档完整地描述了一个类型是 Pod 的 API 对象，要求使用 v1 版本的 API 接口管理，其他更具体的名称、标签、状态等细节都记录在 `metadata` 和 `spec` 字段等里。

使用 `kubectl apply` 或 `kubectl delete`，再加上参数 `-f`，就可以使用这个 YAML 文件创建或者删除对象：

```bash
kubectl apply  -f ngx-pod.yml
kubectl delete -f ngx-pod.yml
```

Kubernetes 收到这份“声明式”的数据，再根据 HTTP 请求里的 POST/DELETE 等方法，就会自动操作这个资源对象，至于对象在哪个节点上、怎么创建、怎么删除完全不用外部用户关心。

---
# 编写 YAML 的技巧

Kubernetes 里有很多 API 对象，如何知道该用什么 `apiVersion`、什么 `kind`？在 `metadata`、`spec` 里又该写哪些字段呢？YAML 看起来简单，写起来却比较麻烦，缩进对齐很容易搞错，有没有什么可靠的方法呢？

这些问题的权威答案无疑是 Kubernetes 的官方文档，在这里可以找到 API 对象的所有字段。不过官方文档内容太多，下面会介绍 3 个简单实用的技巧。

**技巧 1**：使用 `kubectl api-resources` 命令，会显示资源对象的 API 版本和类型，例如 Pod 的版本是 `v1`，Ingress 的版本是 `networking.k8s.io/v1`，照着它写就不会错。

**技巧 2**：使用 `kubectl explain` 命令，相当于 Kubernetes 自带的 API 文档，可以给出对象字段的详细说明。例如想要查看 Pod 里的字段该怎么写，可以执行如下命令[^4]：

[^4]: 因为 Kubernetes 的开发语言是 Go，所以 API 对象字段用的是 Go 的语法规范，例如字段命名遵循“Camel Case”，类型是“boolean”“string”“[]Object”等。

```bash
kubectl explain pod
kubectl explain pod.metadata
kubectl explain pod.spec
kubectl explain pod.spec.containers
```

使用这两个技巧编写 YAML 就基本上没有难度了。

为了更方便、简单，我们还可以让 kubectl 生成一份文档模板，免去我们打字和对齐格式的工作。

**技巧 3**：使用 kubectl 的两个特殊参数 `--dry-run=client` 和 `-o yaml`，前者表示空运行，后者表示生成 YAML 格式，组合使用会让 kubectl 不会有实际的创建动作，只生成 YAML 文件。

例如，想要生成一个 Pod 的 YAML 样板示例，在 `kubectl run` 后面加上这两个参数：

```bash
kubectl run ngx --image=nginx:alpine --dry-run=client -o yaml
```

就会生成一个正确的 YAML 文件：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: ngx
  name: ngx
spec:
  containers:
    - image: nginx:alpine
      name: ngx
      resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

接下来要做的是查阅对象的说明文档，通过添加或者删除字段来定制这个 YAML 文件了。

可以再进化一下，把这两个参数定义为 Shell 变量（变量可以是任意名字，例如 `$do` / `$go`，这里用的是 `$out`），使用起来更方便，例如：

```bash
export out="--dry-run=client -o yaml"
kubectl run ngx --image=nginx:alpine $out
```

除了一些特殊情况，后续不再使用 `kubectl run` 这样的命令直接创建 Pod，而是会编写 YAML 文件，以声明式方法来描述对象，再用 `kubectl apply` 发布 YAML，让 Kubernetes 自动创建对象。

---
# 小结

Kubernetes 采用 YAML 作为工作语言，这是它有别于其他系统的一大特色，声明式的语言能够更准确、更清晰地描述系统状态，避免引入繁琐的操作步骤扰乱系统，与 Kubernetes 高度自动化的内部结构相得益彰，而且纯文本形式的 YAML 也很容易版本化，适合持续集成/持续交付（CI/CD）。

本节的内容要点如下：

- Kubernetes 把集群里的一切资源都定义为 API 对象，通过 RESTful 接口来管理。描述 API 对象需要使用 YAML 语言，必需的字段是 `apiVersion`、`kind` 和 `metadata`。
- 运行命令 `kubectl api-resources` 可以查看对象的 API 版本和类型，运行命令 `kubectl explain` 可以查看对象字段的说明文档。
- 运行命令 `kubectl apply` / `kubectl delete` 发送 HTTP 请求，管理 API 对象。
- 使用参数 `--dry-run=client -o yaml` 可以生成 YAML 模板，简化 YAML 文件的编写工作。

---

