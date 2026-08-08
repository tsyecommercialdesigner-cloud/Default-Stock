---
title: Service 常用操作
created: 2026-08-05
tags:
  - Kubernetes
  - Kubernetes_Service
  - Service
---
---

# Service

## 创建 `svc.yml`

### 使用命令创建

为 ngx-dep 对象生成 Service 对象，命令如下：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 --target-port=80 $out
```

如果 Service 对象的映射端口和目标端口相同，例如都是 80，那么可以省略“--target-port”参数：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 $out
```

生成的 Service YAML 模板文件如下：

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ngx-svc

spec:
  selector:
    app: ngx-dep

  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

其中，`type` 字段的值是默认的 `ClusterIP` ：

```yaml
apiVersion: v1
kind: Service
...
spec:
  ...
  type: ClusterIP
```

如果在`kubectl expose`命令加上参数 `--type=NodePort`，或者在  Service 的 YAML 文件里添加字段 `type: NodePort`，那么 Service 不仅会对后端的 Pod 做负载均衡，还会在集群中的每个节点上创建一个独立的端口来对外提供服务：

```bash
export out="--dry-run=client -o yaml"
kubectl expose deploy ngx-dep --port=80 --target-port=80 --type=NodePort $out
```

可选字段包括：

- `--type=ClusterIP`
- `--type=ExternalName`
- `--type=LoadBalancer`
- `--type=NodePort`

其中“`ExternalName`”和“`LoadBalancer`”一般由云服务商提供。



### 从模板创建

#### 本地

定义 `svc-local.yml`：

```yaml
# 2026 version
apiVersion: v1
kind: Service
metadata:
  name: svc-local
spec:
  type: LoadBalancer
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    project: qsk-book
```


#### 云端

定义 `svc-cloud.yml`：

```yaml
# 2022 version
apiVersion: v1
kind: Service
metadata:
  name: cloud-lb
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    project: qsk-book
```

---

## 操作 Service 对象

### 创建 Service 对象

以指定文件的方式将 `svc.yml` 发送给 Kubernetes API 服务器：

```bash
kubectl apply -f svc-local.yml
```

云端则改成：

```bash
kubectl apply -f svc-cloud.yml
```

验证 Service 部署结果：

```bash
kubectl get svc
```

---

### 删除 Service 对象

删除指定的 Service ：

```bash
kubectl delete svc ${svcname}
```

---

# Ingress

## 创建 `ing.yml`

执行命令 `kubectl create` ：

```bash
export out="--dry-run=client -o yaml"
kubectl create ing ngx-ing \
    --rule="ngx.test/=ngx-svc:80" --class=ngx-ink $out
```

附加参数说明：

- `--class`，指定 Ingress 从属的 Ingress Class 对象；
- `--rule`，指定路由规则，基本形式是“URI=Service”，也就是说访问 HTTP 路径就转发到对应的 Service 对象，再由 Service 对象转发给后端的 Pod。

Ingress 的 YAML 文件如下：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ngx-ing

spec:
  ingressClassName: ngx-ink
  rules:
  - host: ngx.test
    http:
      paths:
      - backend:
          service:
            name: ngx-svc
            port:
              number: 80
        path: /
        pathType: Exact
```

>[!warnning] 注意
Ingress 的后端端口应填写 Service 的 `port`，而非 `nodePort`。


---

# Ingress Class

## 定义 `spec.controller`

Ingress Class 本身没有什么实际的功能，只是起到联系 Ingress 和 Ingress Controller 的作用。

Ingress Class 的定义方式为：

> 在 `spec` 字段里的 `controller` 字段指定要使用哪个 Ingress Controller 。

`spec` 字段如下：

```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: ngx-ink

spec:
  controller: nginx.org/ingress-controller    # Nginx Ingress Controller
```

常用的 Ingress Controller 有：

- Nginx 开发的 Nginx Ingress Controller（静态的配置文件）
- Kong 开发的 Kong Ingress Controller（完全动态的路由变更）

具体名称如下。

Nginx Ingress Controller：

```plain
nginx.org/ingress-controller
```

Kong Ingress Controller：

```plain
ingress-controllers.konghq.com/kong
```

## 创建 Ingress 、Ingress Class 对象

通常把 Ingress Class 与 Ingress 的定义合并为一个 YAML 文件，然后通过 `kubectl apply` 命令一次性创建 Ingress 和 Ingress Class 两个对象：

```bash
kubectl apply -f ing-inx.yml
```

---

# Ingress Controller

Ingress Controller 是应用程序，同时支持 Deployment 和 DaemonSet 两种部署方式。

## Nginx Ingress Controller

### 拉取仓库

在 GitHub 上找到 Nginx Ingress Controller 项目：

```bash
git clone https://github.com/nginx/kubernetes-ingress.git
```

Nginx Ingress Controller 对象包含的多个 YAML 放在“deployments/common”、“deploy”、“deployments/rbac”里，需要执行以下“kubectl apply”命令：

```bash
kubectl apply -f deploy/crds.yaml
```

```bash
kubectl apply -f deployments/common/ns-and-sa.yaml
kubectl apply -f deployments/common/ingress-class.yaml
kubectl apply -f deployments/common/nginx-config.yaml
```

```bash
kubectl apply -f deployments/rbac/rbac.yaml
```

这些 YAML 为 Ingress Controller 创建了独立的名字空间“nginx-ingress”、相应的账号和权限（访问 apiserver 获取 Service、Endpoint 信息），以及 ConfigMap 和 Secret，用来配置 HTTP/HTTPS 服务。

### 创建 Ingress Controller 对象

部署 Ingress Controller 不需要完全从头编写 Deployment 的 YAML 文件，因为 Nginx 提供了示例 YAML，只需要在创建之前做好如下修改以适配自己的应用：

- “metadata”字段的 name 要改成应用的名字，如 `ngx-kic-dep`；
- “spec.selector”和“template.metadata.labels”字段也要修改为应用的名字，如 `ngx-kic-dep`；
- 可以改用 `containers.image` 的 alpine 版本，加快下载速度，如 `nginx/nginx-ingress:3.2-alpine`；
- “args”字段要加上`-ingress-class=ngx-ink`，也就是 Ingress Class 的名字，这是让 Ingress Controller 处理 Ingress 的关键。

修改之后，Nginx Ingress Controller 的 YAML 文件如下：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ngx-kic-dep
  namespace: nginx-ingress

spec:
  replicas: 1
  selector:
    matchLabels:
      app: ngx-kic-dep

  template:
    metadata:
      labels:
        app: ngx-kic-dep
    ...
    spec:
      containers:
      - image: nginx/nginx-ingress:3.2-alpine
        ...
        args:
        - -ingress-class=ngx-ink
```

确认 Ingress Controller 的 YAML 修改完毕，可以用“kubectl apply”创建对象：

```bash
kubectl apply -f deploy.yml
```

Nginx Ingress Controller 默认位于名字空间“nginx-ingress”中，查看其状态需要用“-n”参数显式指定（否则只能查看“default”名字空间里的 Pod 的状态）：

```bash
kubectl get deploy -n nginx-ingress
```

```bash
kubectl get pod -n nginx-ingress
```

### 为 Ingress Controller Pod 定义 Service

把本地的 8080 端口映射到 Ingress Controller Pod 的 80 端口：

```bash
kubectl port-forward -n nginx-ingress \
    $(podname) 8080:80 &
```

在 curl 测试请求的时候需要注意，因为 Ingress 的路由规则是 HTTP 协议，不能直接用 IP 地址的方式访问，必须用域名，有三种方式可以实现。

#### 方式一：手工添加域名解析

修改“/etc/hosts”，手工添加对测试域名“ngx.test”的解析；

#### 方式二：使用 `--resolve` 参数

使用“--resolve”参数，指定 curl 对域名的解析规则，例如把“ngx.test”强制解析到“127.0.0.1”（即“kubectl port-forward”转发的本地地址）：

```bash
curl --resolve ngx.test:8080:127.0.0.1 http://ngx.test:8080
```

#### 方式三：使用 `Host` 头字段

使用 HTTP 的“Host”头字段，明确指定测试域名“ngx.test”：

```bash
curl 127.1:8080 -H 'Host: ngx.test'
```

---

## Kong Ingress Controller

### 拉取仓库

在 GitHub 上找到 Kong Ingress Controller 项目：

```bash
git clone https://github.com/Kong/kubernetes-ingress-controller.git
```

### 创建 Ingress Controller 对象

Kong Ingress Controller 安装所需的 YAML 文件都存在解压缩后的“deploy/single”目录下，提供“有数据库”和“无数据库”等多种部署方式。

如果“无数据库”方式，只需一个“all-in-one-dbless.yaml”就可以完成部署工作，也就是执行命令：

```bash
kubectl apply -f all-in-one-dbless.yaml
```

安装完成之后，Kong Ingress Controller 会创建一个新的名字空间“kong”，里面有默认的 Ingress Controller，以及对应的 Service：

```bash
kubectl get pod -n kong
```

```bash
kubectl get svc -n kong
```

要注意，运行命令“kubectl get pod”可以显示有 3 个 Pod 在运行：

- 其中 1 个 Pod 是 `ingress-kong`
- 还有 2 个 Pod 是 `proxy-kong`

使用 curl 命令访问服务 `kong-proxy` 的 NodePort（使用集群里任意节点的 IP 地址），可以验证 Kong Ingress Controller 是否工作正常：

```bash
curl 192.168.26.210:32301 -i
```

因为现在还没有为它配置任何 Ingress 资源，所以返回了状态码 404，这是正常的。

还可以用 `kubectl exec` 命令进入 Pod：

```bash
kubectl exec -it -n kong proxy-kong-547bf4c85-qbjrd -- sh
```

查看它的内部信息：

```bash
kong version
kong health
```

### 为 Ingress Controller Pod 定义 Service

无论底层使用的是 **NGINX Ingress Controller** 还是 **Kong Ingress Controller**，Ingress 控制器在 Kubernetes 架构中的角色和流量暴露逻辑都是一致的：

1. **原理相同**：
    
    - 你使用 `kubectl create ing` 创建的 `Ingress` 对象**只是路由规则**的声明，它告诉控制器“访问 `kong.test/` 时转给 `ngx-svc:80`”。
    - 实际上真正接收和转发流量的是 **Kong Ingress Controller Pod**（即 Kong 的 Data Plane 数据面网关）。
      
2. **打通内外流量的方式**：
    
    - 如果想让集群外部的真实流量访问进来，同样必须通过为 Kong 的 Ingress Controller Pod 创建一个 **`LoadBalancer`** 或 **`NodePort`** 类型的 Service，将端口暴露给集群外。
    - 如果仅在本地开发测试，也同样可以使用类似 `kubectl port-forward` 的命令，将本地端口映射到 Kong 的 Ingress Controller Pod 的代理端口（默认通常是 `8000` 或 `80`）：

```bash
kubectl port-forward -n kong $(podname) 8080:8000 &
```

```bash
kubectl port-forward -n kong \
    $(podname) 8080:80 &
```

---

# 利用 Ingress Class 创建新实例


## 定义 Ingress Class


```yaml
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: kong-ink

spec:
  controller: ingress-controllers.konghq.com/kong
```

## 定义 Ingress 对象

用 `kubectl create` 生成 YAML 模板文件，
使用“--rule”指定路由规则、使用“--class”指定 Ingress Class：

```bash
export out="--dry-run=client -o yaml"
kubectl create ing kong-ing \
    --rule="kong.test/=ngx-svc:80" --class=kong-ink $out
```

生成的 Ingress 对象如下，可以看到域名是“kong.test”，流量会转发到后端的 ngx-svc 服务：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kong-ing

spec:
  ingressClassName: kong-ink

  rules:
  - host: kong.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ngx-svc
            port:
              number: 80
```

## 定义 Ingress Controller

可以从 `all-in-one-dbless.yaml` 这个文件中分离出 Ingress Controller 的定义。

如果想让集群外部的真实流量访问进来，同样必须通过为 Kong 的 Pod 创建一个 **`LoadBalancer`** 或 **`NodePort`** 类型的 Service，将端口暴露给集群外。

可以查找 Deployment 对象，把它及相关的 Service 代码复制一份，并另存为 `kong-kic.yml` 。

参考帮助文档对 `kong-kic.yml` 做如下修改：

- Deployment、Service 对象的 name 都要重命名，如重命名为 `ingress-kong-dep`、`proxy-kong-dep`、`kong-admin-svc`、`kong-proxy-svc`。
- “`spec.selector`”和“`template.metadata.labels`”字段也要对应修改为应用的名字，一般来说和 Deployment 的名字一样，也就是 `ingress-kong-dep`、`proxy-kong-dep`。
- “`ingress-kong`”要用环境变量“`CONTROLLER_INGRESS_CLASS`”指定新的 Ingress Class 名字为“`kong-ink`”，同时用“`CONTROLLER_KONG_ADMIN_SVC`”“`CONTROLLER_PUBLISH_SERVICE`”指定 Service 的新名字为“`kong/kong-admin-svc`”“`kong/kong-proxy-svc`”。
- 可以根据需要将“`ingress-kong`”里的镜像改成任意支持的版本，如较旧的版本 `Kong:3.0` 或者较新的版本 `Kong:3.5`。
- 可以将“`kong-proxy`”Service 对象的类型改成 `NodePort`，方便后续测试。
- Kong Ingress Controller 里的 Pod 大量使用了环境变量来调整应用的行为，`proxy-kong` 中比较有用的一个环境变量是“`KONG_ROUTER_FLAVOR`”，用来切换内置路由器引擎。

确认 Ingress Controller 的 YAML 修改完毕，准备创建对象。

## 创建对象

创建 Ingress 和 Ingress Class 两个对象：

```bash
kubectl apply -f kong-ing.yml
```

创建 Ingress Controller 对象：

```bash
kubectl apply -f kong-kic.yml
```

## 验证对象

创建完成之后，Kong Ingress Controller 默认位于名字空间“kong”中，查看其状态需要用“-n”参数显式指定（否则只能查看“default”名字空间里的 Pod 的状态），里面有默认的 Ingress Controller，以及对应的 Service：

```bash
kubectl get pod -n kong
```

```bash
kubectl get svc -n kong
```

验证 Ingress Controller ：

```bash
kubectl get deploy -n kong
```

验证 Ingress ：

```bash
kubectl get ing
```

## 测试对象

使用 curl 命令测试时应该用“--resolve”或者“-H”参数指定 Ingress 定义的域名“kong.test”，否则 Kong Ingress Controller 会找不到路由：

```bash
curl --resolve kong.test:8080:127.0.0.1 http://kong.test:8080
```

```bash
curl 192.168.26.210:30105 -H 'Host: kong.test' -v
```

---

# Ingress 、Service、Pod 链路

访问 Pod 的方式主要有两种：

- 集群内 / Ingress：`服务名:Kong Ingress Controller 对外的 Service 端口`
- 集群外直接通过 NodePort：`节点IP:NodePort 端口`

## 后端转发链路

假如 Ingress 的 YAML 定义为：

```yaml
backend:
  service:
    name: ngx-svc
    port:
      number: 80
```

对应 Service 的 YAML 定义为：

```yaml
apiVersion: v1
kind: Service
metadata: 
  name: ngx-svc

spec:
  type: NodePort
  ports:
    - port: 80          # 后端 Service 在集群内部暴露的端口；Ingress 转发到这里
      targetPort: 8080  # Pod/容器实际监听的端口
      nodePort: 30105   # 仅当 Service 类型是 NodePort 或 LoadBalancer 时填写，是节点对外开放的端口
```

那么对应关系就会是：

```text
Ingress: ngx-svc:80
              ↓
Service port: 80
              ↓
Service targetPort: 8080
              ↓
Pod 容器监听: 8080
```

 Ingress 中的 `80` 是 Ingress 转发到后端 Service `ngx-svc` 时使用的 Service 端口，而`curl` 命令中的端口应当是 Kong Ingress Controller 对外暴露的代理端口。
 因此，`ngx-svc:80` 不等于 curl 必须用 `:80`。
 
具体用哪个端口取决于 Kong Ingress Controller 对外的 Service 的端口映射，例如：

```bash
kubectl get svc -A | grep kong
```

- 若 Kong 暴露的是 `8080`，继续使用 `kong.test:8080`
- 若 Kong 暴露的是 `80`，则使用 `kong.test:80`

另外，若访问的是默认 HTTP 端口 `80`，命令可简写为：

```bash
curl --resolve kong.test:80:127.0.0.1 http://kong.test/
```

## 完整转发链路

Ingress Controller Pod 的监听端口位于 Ingress 之前，负责接收客户端请求：

```text
Client 客户端
  ↓
Ingress Controller 对外 Service 的 nodePort: 30105
  ↓
Ingress Controller Service 的 port: 80
  ↓
Ingress Controller Pod 的 targetPort: 例如 8000 / 80
  ↓
根据 Ingress 规则匹配 kong.test
  ↓
ngx-svc:80
  ↓
ngx-svc 的 Service port: 80
  ↓
ngx-svc 的 targetPort: 8080
  ↓
ngx Pod 容器监听: 8080
```

因此，要确定 Ingress Controller Pod 实际监听哪个端口，要看的是 Ingress Controller 自己的 Service，而不是 `ngx-svc`：

```bash
kubectl get svc -A
```

例如 Kong 常见的一种配置可能是：

```yaml
# Kong Ingress Controller 对外的 Service
spec:
  type: NodePort
  ports:
  - name: proxy
    port: 80          # Controller Service 的集群内端口
    targetPort: 8000  # Kong Controller Pod 实际监听端口
    nodePort: 30105   # 节点对外端口
```

那么端口含义为：

| 位置                      | 端口      |
| ----------------------- | ------- |
| 浏览器/curl 访问节点           | `30105` |
| Kong 的 Service 端口       | `80`    |
| Kong Pod 内代理监听端口        | `8000`  |
| 后端 `ngx-svc` Service 端口 | `80`    |
| 后端 Nginx Pod 监听端口       | `8080`  |

此时测试应访问 Kong 的 NodePort，而不是后端 Service 的 NodePort：

```bash
curl --resolve kong.test:30105:127.0.0.1 http://kong.test:30105/
```

不过，Kong Pod 的实际监听端口可能是 `8000`、`80` 或其他值；以 Kong Ingress Controller 对外的 Service 中的 `targetPort` 为准。`ngx-svc` 的 `targetPort: 8080` 只描述后端 Nginx Pod，和 Kong Controller Pod 的监听端口是两回事。

---

# `annotations`

annotation 是 Kubernetes 为资源对象提供的一个方便扩展功能的手段，可以在不修改 Ingress 定义的前提下，让 Kong Ingress Controller 更好地利用内部的 Kong 来管理流量。

目前 Kong Ingress Controller 支持在 `Ingress` 和 `Service` 这两个对象上添加 annotation 。

 `annotations` 和 `labels` 的区别：

- `annotations` 添加的信息一般是给 Kubernetes 内部的各种对象使用的，有点像扩展属性；
- `labels` 主要面对的是 Kubernetes 外部的用户，用来筛选、过滤对象。

## 添加额外域名

Ingress 的域名允许使用通配符“\*”，如“\*.abc.com”，但问题在于“\*”只能是前缀而不能是后缀。

而 `konghq.com/host-aliases` 可以为 Ingress 规则添加额外的域名。

例如修改 Ingress 定义，在“metadata”字段里添加一个 annotation，可以让它除了支持“kong.test”还能够支持“kong.dev”“kong.ops”等域名：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kong-ing
  annotations:
    konghq.com/host-aliases: "kong.dev, kong.ops"
spec:
  ...
```

## 启用插件

插件是 Kong Ingress Controller 的特色功能，能够附加在流量转发的过程中，实现各种数据处理。这个插件机制是开放的，既可以使用官方插件，也可以使用第三方插件，还可以使用 Lua、Go、Rust 等语言编写符合自己特定需求的插件。

`konghq.com/plugins` 可以启用 Kong Ingress Controller 内置的各种插件。

### 定义插件对象

Response Transformer 插件实现了对响应数据的修改，能够添加、替换、删除响应头或者响应体。
例如，让 Response Transformer 插件添加一个新的响应头字段：

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: kong-add-resp-header-plugin

plugin: response-transformer
config:
  add:
    headers:
    - Resp-New-Header:kong-kic
```

Rate Limiting 插件实现了限速功能，能够以时、分、秒等单位任意限制客户端访问的次数。
例如，让 Rate Limiting 插件限制客户端每分钟只能发送两个请求：

```yaml
apiVersion: configuration.konghq.com/v1
kind: KongPlugin
metadata:
  name: kong-rate-limiting-plugin

plugin: rate-limiting
config:
  minute: 2
```

### 启用插件功能

定义好插件之后，就可以在 Ingress 对象里用“annotations”来启用插件功能：

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: kong-ing
  annotations:
    konghq.com/plugins: |
      kong-add-resp-header-plugin,
      kong-rate-limiting-plugin
```
