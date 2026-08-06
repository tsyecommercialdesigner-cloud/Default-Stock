---
title: ConfigMap 和 Secret
created: 2026-08-06
source: Cherry Studio
tags:
  - ConfigMap
  - Secret
---
---

# 配置信息 ConfigMap 和 Secret

之前介绍了 Kubernetes 里的 3 种 API 对象：Pod、Job 和 CronJob，虽然还没有讲到更高级的其他对象，但使用它们也可以在集群里编排运行一些实际的业务。

不过想让业务更顺利地运行，有一个问题不容忽视，那就是应用的配置管理。

通常来说应用程序会有一个配置文件，它把运行时需要的一些参数从代码中分离出来，让用户在实际运行的时候能更方便地调整优化，例如 Nginx 有 `nginx.conf`、Redis 有 `redis.conf`、MySQL 有 `my.cnf` 等。

学习容器技术的时候讲过，可以选择两种管理配置文件的方式：

- 第一种方式是编写 Dockerfile，用 `COPY` 把配置文件打包到镜像里；
- 第二种方式是在运行时使用 `docker cp` 或者 `docker run -v`，把本机的文件拷贝进容器。

但这两种方式都存在缺陷：

- 第一种方式相当于在镜像里固定了配置文件，不方便修改，不灵活；
- 第二种方式则显得有点“笨拙”，不适合在集群中自动化运维管理。

Kubernetes 针对这个问题的解决方案，是使用 YAML 来定义 API 对象，再组合起来实现动态配置。

应用程序有很多类别的配置信息，从数据安全的角度可以分成如下两类：

- **明文配置**，也就是不保密，可以任意查询修改，如服务端口、运行参数、文件路径等；
- **机密配置**，由于涉及敏感信息，不能随便查看，如密码、密钥、证书等。

这两类配置信息本质上都是字符串，只是出于安全性，在存放和使用方面有些差异，所以 Kubernetes 定义了两个 API 对象：

> - ConfigMap 用来管理明文配置；
> - Secret 用来管理机密配置。

两者组合一起，实现了对应用的灵活配置与定制。

---

# ConfigMap

执行命令 `kubectl create` 可以创建一个 ConfigMap 的 YAML 模板文件。注意，ConfigMap 有简写名字 `cm`，所以命令行里不必写出它的全称：

```bash
export out="--dry-run=client -o yaml"     # 定义 Shell 变量
kubectl create cm info $out
```

得到的模板文件大概如下：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: info
```

ConfigMap 的 YAML 文件和 Pod、Job 不一样，除了熟悉的 `apiVersion`、`kind`、`metadata`，没有其他字段，特别是没有 `spec` 字段。这是因为 ConfigMap 存储的是配置数据，是静态的字符串而不是容器，所以不需要用 `spec` 字段来说明运行时的状态。

既然 ConfigMap 要存储数据，就需要用另一个含义更明确的字段 `data`。

想要生成带有 `data` 字段的 YAML 模板文件，需要在 `kubectl create` 命令后面加上参数 `--from-literal`，表示从字面值生成一些数据[^1]：

[^1]: 如果已经存在一些配置文件，可以使用参数 `--from-file` 从文件自动创建 ConfigMap 或 Secret 对象。

```bash
kubectl create cm info --from-literal=k=v $out
```

>[!warning] 注意
>因为在 ConfigMap 里的数据都是键值对结构，
>所以 `--from-literal` 参数需要使用 `k=v` 的形式。

修改 YAML 模板文件，增加一些键值对，就得到一个比较完整的 ConfigMap 对象：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: info

data:
  count: '10'
  debug: 'on'
  path: '/etc/systemd'
  greeting: |
    say hello to kubernetes.
```

现在可以使用 `kubectl apply` 把这个 YAML 文件交给 Kubernetes，来创建 ConfigMap 对象：

```bash
kubectl apply -f cm.yml
```

创建成功后，仍然可以使用 `kubectl get`、`kubectl describe` 查看 ConfigMap 的状态：

```text
[K8S ~]$ kubectl get cm
NAME   DATA   AGE
info   4      13s

[K8S ~]$ kubectl describe cm info
Name:         info

Data
====
count:
----
10
debug:
----
on
greeting:
----
say hello to kubernetes.
path:
----
/etc/systemd
```

可以看到，ConfigMap 的键值对信息已经存入了 etcd，后续可以被其他 API 对象使用。

---

# Secret

`Secret` 对象与 `ConfigMap` 对象的结构和用法类似，不过在 Kubernetes 里 `Secret` 对象又细分出很多类，如以下 4 类：

- 访问私有镜像仓库的认证信息；
- 身份识别的凭证信息；
- HTTPS 通信的证书和私钥；
- 一般的机密信息（格式由用户自行解释）。

下面只使用一般的机密信息这类，创建 YAML 模板文件的命令是 `kubectl create secret generic`，同样，也要使用参数 `--from-literal` 增加一些键值对：

```bash
export out="--dry-run=client -o yaml"
kubectl create secret generic user --from-literal=name=root $out
```

得到的 Secret 对象大概如下：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: user

data:
  name: cm9vdA==
```

Secret 对象和 ConfigMap 非常相似，只是 `kind` 字段由 `ConfigMap` 变成了 `Secret`，后面同样也是 `data` 字段，里面也是键值对数据。

不过，Secret 对象不能像 ConfigMap 对象那样直接保存明文，所以上述 `name` 字段的值是一串“乱码”，而不是在命令行里定义的 `root`。

这串“乱码”就是 Secret 与 ConfigMap 的不同之处，不让用户直接看到原始数据，起到一定的保密作用。不过它的“加密”方式其实非常简单，只是进行了 Base64 编码，算不上真正的加密，所以我们完全可以绕开 kubectl，用 Linux 的工具 `base64` 来对数据编码，然后写入 YAML 文件，例如：

```bash
[K8S ~]$ echo -n "123456" | base64
MTIzNDU2
```

要注意这条命令里的 `echo`，必须加上参数 `-n` 去掉字符串里隐含的换行符，否则 Base64 编码得到的字符串就是错误的。

重新编辑 Secret 的 YAML 文件，为它添加两条数据，添加方式既可以是使用参数 `--from-literal` 自动编码，也可以是手动编码：

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: user

data:
  name: cm9vdA==    # root
  pwd: MTIzNDU2     # 123456
  db: bXlzcWw=      # mysql
```

Secret 的创建和查看对象操作与 ConfigMap 一样，使用 `kubectl apply`、`kubectl get`、`kubectl describe`：

```bash
[K8S ~]$ kubectl apply -f secret.yml
secret/user created

[K8S ~]$ kubectl get secrets
NAME   TYPE     DATA   AGE
user   Opaque   3      59s

[K8S ~]$ kubectl describe secret user
Name:         user
Type:         Opaque

Data
====
db:    5 bytes
name:  4 bytes
pwd:   6 bytes
```

这样一个存储敏感信息的 Secret 对象就创建好了，而且使用 `kubectl describe` 不能直接看到数据，只能看到数据的大小。

---

# 加载为环境变量

编写 YAML 文件创建好 ConfigMap 和 Secret 对象后，下面介绍如何在 Kubernetes 里使用它们。

因为 ConfigMap 和 Secret 只是本质上存储在 etcd 里的字符串，所以如果想要在运行时加载这些数据，必须以某种方式注入 Pod，让应用程序来读取。Kubernetes 的处理方式和 Docker 一样，也是加载为环境变量和文件两种途径。

下面介绍比较简单的环境变量方式。之前提到描述容器的字段 `containers` 里有一个 `env` 字段，它定义了 Pod 里容器能够看到了环境变量。

当时只使用了简单的 `value`，把环境变量的值写死了在 YAML 文件里，实际上它还可以使用另一个 `valueFrom` 字段，从 ConfigMap 或者 Secret 对象里获取值，这样就实现了把配置信息以环境变量的形式注入 Pod，也就实现了配置与应用解耦。

由于 `valueFrom` 字段在 YAML 文件里的嵌套层次比较深，建议初次使用这个字段要先看一下 `kubectl explain` 对它的说明：

```bash
kubectl explain pod.spec.containers.env.valueFrom
```

`valueFrom` 字段指定了环境变量值的来源，可以是 `configMapKeyRef` 或者 `secretKeyRef`，再进一步指定应用的 ConfigMap 和 Secret 的 `name` 和它里面的 `key`，要当心的是这个 `name` 字段是 API 对象（即 `ConfigMap` 和 `Secret` ）的名字，而不是键值对的名字。

下面就列出了引用了 ConfigMap 和 Secret 对象的 Pod，为了方便阅读，把 `env` 字段提到了前面：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-pod

spec:
  containers:
    - env:
        - name: COUNT
          valueFrom:
            configMapKeyRef:
              name: info
              key: count
        - name: GREETING
          valueFrom:
            configMapKeyRef:
              name: info
              key: greeting
        - name: USERNAME
          valueFrom:
            secretKeyRef:
              name: user
              key: name
        - name: PASSWORD
          valueFrom:
            secretKeyRef:
              name: user
              key: pwd

      image: busybox
      name: busy
      imagePullPolicy: IfNotPresent
      command: ["/bin/sleep", "300"]
```

需要确认：`info` ConfigMap 和 `user` Secret 必须已在同一 `namespace` 中，并且包含对应的键。

这个 Pod 的名字是 `env-pod`，镜像就是 `busybox`，执行命令 `sleep` 休眠 300 s，可以在休眠时使用命令 `kubectl exec` 进入 Pod 观察环境变量。

需要重点关注的是 `env` 字段，里面定义了 4 个环境变量，“COUNT”“GREETING”“USERNAME”“PASSWORD”[^2]。

[^2]: Linux 系统对环境变量的命名有限制，不能使用 `-`、`.` 等特殊字符，所以在创建 ConfigMap 和 Secret 的时候需要注意，否则无法以环境变量的形式注入 Pod。

对于明文配置数据，“COUNT”“GREETING”引用的是 ConfigMap 对象，所以使用字段 `configMapKeyRef`，其中字段 `name` 是 ConfigMap 对象的名字，也就是之前创建的 `info`，而字段 `key` 分别是 `info` 对象里的 `count` 和 `greeting`。

同样的，对于机密配置数据，“USERNAME”“PASSWORD”引用的是 Secret 对象，要使用字段 `secretKeyRef`，再用字段 `name` 指定 Secret 对象的名字 `user`，用字段 `key` 对应里面的 `name` 和 `pwd`。

可见，ConfigMap 和 Secret 在 Pod 里的组合关系不像 Job 和 CronJob 那么简单、直接。

可以看出 Pod 与 ConfigMap、Secret 是松耦合关系，它们不是直接嵌套包含，而是使用 `keyRef` 字段间接引用对象，同一段配置信息可以在不同的对象间共享。

清楚了环境变量的注入方式之后，可以用 `kubectl apply` 创建 Pod，再用 `kubectl exec` 进入 Pod，验证环境变量是否生效：

```bash
[K8S ~]$ kubectl exec -it env-pod -- sh

/ # echo $COUNT
10

/ # echo $GREETING
say hello to kubernetes.

/ # echo $USERNAME $PASSWORD
root 123456
```

可以看到，在 Pod 里执行 `echo` 命令准确输出了两个 YAML 文件里定义的配置信息，证明 Pod 对象成功组合了 ConfigMap 对象和 Secret 对象。

---

# 挂载存储卷

下面介绍将 ConfigMap 和 Secret 对象加载为文件的方式。

## 定义存储卷

Kubernetes 为 Pod 定义了一个概念 Volume，可以翻译为存储卷。如果把 Pod 理解为一个虚拟机，那么 Volume 就相当于虚拟机里的磁盘[^3]。

[^3]: “Volume”这个词也许是来源于早期计算机存储使用的磁带设备都是成卷的，而 “mount” 操作自然就是把磁带“挂载”到磁带机。

可以为 Pod 挂载（mount）多个存储卷，里面存放供 Pod 访问的数据，这种方式有点类似 `docker run -v`，虽然用法复杂了一些，但功能也更强大了。

在 Pod 里挂载存储卷很容易，只需要在 `spec` 里增加一个 `volumes` 字段，再定义存储卷的名字和引用的 ConfigMap 和 Secret 对象就可以了。要注意的是存储卷属于 Pod，不属于容器，所以它和字段 `containers` 是同级别的，都属于 `spec`。

如下 YAML 文件定义了两个 Volume，分别引用 ConfigMap 和 Secret 对象，名字是 `cm-vol` 和 `sec-vol`：

```yaml
spec:
  volumes:
    - name: cm-vol
      configMap:
        name: info
    - name: sec-vol
      secret:
        secretName: user
```

## 在容器中挂载存储卷

有了存储卷的定义之后，就可以使用 `volumeMounts` 字段在容器里挂载了。正如 `volumeMounts` 的字面含义，可以把定义好的存储卷挂载到容器的某个路径下，需要使用 `mountPath`“`name`”字段明确指定挂载路径和存储卷的名字。

```yaml
containers:
  - volumeMounts:
      - mountPath: /tmp/cm-items
        name: cm-vol
      - mountPath: /tmp/sec-items
        name: sec-vol
```

写好 `volumes` 和 `volumeMounts` 字段后，配置信息就可以加载为文件。

## 通过 `volumes.configMap.items` 给文件改名

可以在 `volumes.configMap.items` 字段里用 `key`、`path` 精确地指定 ConfigMap 里每个 Key-Value 加载的路径名，也就是给文件改名。
把 ConfigMap 挂载为卷后，ConfigMap 的每个 `data` 键通常会变成一个文件：

```yaml
data:
  count: "3"
  greeting: "hello"
```

默认挂载到 `/etc/config` 后会是：

```
/etc/config/count
/etc/config/greeting
```

`volumes.configMap.items` 让你指定某个 Key 对应的文件名（或相对路径）。例如：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
    - name: busy
      image: busybox
      command: ["/bin/sh", "-c", "sleep 300"]
      volumeMounts:
        - name: config-volume
          mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: info
        items:
          - key: count
            path: app-count.txt
          - key: greeting
            path: message.txt
```

挂载完成后：

```
/etc/config/app-count.txt   # 文件内容是 ConfigMap 的 count 值
/etc/config/message.txt     # 文件内容是 ConfigMap 的 greeting 值
```

这里：

- `key`：ConfigMap 中已有的键名，例如 `count`
- `path`：挂载目录下要生成的文件路径，例如 `app-count.txt`

`path` 是相对于 `mountPath` 的路径，不能以 `/` 开头；也可以使用子目录：

```
- key: greeting
  path: settings/message.txt
```

这样则会生成：

```
/etc/config/settings/message.txt
```

---

# 两者对比

可以看到，挂载存储卷的方式和加载为环境变量不太相同：

- 加载为环境变量的方式直接引用了 ConfigMap 和 Secret 对象；
- 挂载存储卷的方式多加了一个环节，需要先用存储卷引用 `ConfigMap` 和 `Secret` 对象，然后在容器里挂载存储卷。

这种方式的好处在于：

> 以存储卷的概念统一抽象了所有的存储，不仅现在能支持 ConfigMap 和 Secret 对象，以后还能支持临时卷、持久卷、动态卷、快照卷等许多形式的存储，扩展性非常好。

下面列出了 Pod 的完整 YAML 描述：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: vol-pod

spec:
  volumes:
    - name: cm-vol
      configMap:
        name: info
    - name: sec-vol
      secret:
        secretName: user

  containers:
    - volumeMounts:
        - mountPath: /tmp/cm-items
          name: cm-vol
        - mountPath: /tmp/sec-items
          name: sec-vol

      image: busybox
      name: busy
      imagePullPolicy: IfNotPresent
      command: ["/bin/sleep", "300"]
```

执行命令 `kubectl apply` 创建 Pod 对象之后，还是使用 `kubectl exec` 进入 Pod，查看配置信息的加载形式：

```bash
[K8S ~]$ kubectl exec -it vol-pod -- sh

/ # ls /tmp/cm-items/
count debug greeting path

/ # cat /tmp/cm-items/greeting
say hello to kubernetes.

/ # cat /tmp/sec-items/pwd
123456
```

可以看到，ConfigMap 和 Secret 对象都变成了目录的形式，而键值对变成了一个个的文件，文件名是 Key、文件内容是 Value。

因为这种形式上的差异，以存储卷的方式来使用 ConfigMap 和 Secret 对象，就和加载为环境变量的方式不太一样。环境变量的用法简单，更适合存放简短的字符串，而存储卷更适合存放大数据量的配置文件，在 Pod 里加载成文件后可供应用直接读取使用[^5]。

[^5]: 受 etcd 的限制，ConfigMap 和 Secret 对象的大小不能超过 1 MB。

---

# 小结

本节介绍了在 Kubernetes 里管理配置信息的两种 API 对象：ConfigMap 和 Secret，它们分别用来保存明文信息和机密敏感信息。这两个对象存储在 etcd 里，在需要的时候可以注入 Pod 使用，其使用方式可根据具体场景灵活选择加载为环境变量或者文件。

本节的内容要点如下：

- ConfigMap 保存了一些键值对格式的字符串数据，使用的字段是 `data`，而不是 `spec`；
- Secret 与 ConfigMap 类似，也使用字段 `data` 保存字符串数据，但它要求数据必须是 Base64 编码，起到一定的加密效果；
- 在 Pod 的 `env.valueFrom` 字段中可以引用 ConfigMap 和 Secret 对象，把它们加载为应用可以访问的环境变量；
- 在 Pod 的 `spec.volumes` 字段中可以引用 ConfigMap 和 Secret 对象，把它们加载为存储卷，然后在 `spec.containers.volumeMounts` 字段中加载为文件的形式。

---
