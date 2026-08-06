---
title: ConfigMap 、Secret 常用操作
created: 2026-08-06
tags:
  - ConfigMap
  - Secret
---
---

# ConfigMap

## 创建基本 YAML 模板

执行命令 `kubectl create` 可以创建一个 ConfigMap 的 YAML 模板文件：

```bash
export out="--dry-run=client -o yaml"
kubectl create cm info $out
```

得到如下模板：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: info
```

## 创建带有 `data` 的 YAML 模板

如果想要生成带有 `data` 字段的 YAML 模板文件，需要在 `kubectl create` 命令后面加上参数 `--from-literal`，表示从字面值生成一些数据[^1]：

[^1]: 如果已经存在一些配置文件，可以使用参数 `--from-file` 从文件自动创建 ConfigMap 或 Secret 对象。

```bash
export out="--dry-run=client -o yaml"
kubectl create cm info --from-literal=k=v $out
```

因为在 ConfigMap 里的数据都是键值对结构，所以 `--from-literal` 参数需要使用 `k=v` 的形式。

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

## 创建 ConfigMap 对象

现在可以使用 `kubectl apply` 把这个 YAML 文件交给 Kubernetes，来创建 ConfigMap 对象：

```bash
kubectl apply -f cm.yml
```


---

# Secret

## 创建基本 YAML 模板

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

>[!warning] 注意
>这种“加密”方式其实非常简单，只是进行了 Base64 编码，算不上真正的加密。

## 使用 Base64 编码

如果要绕开 kubectl，转而用 Linux 的工具 `base64` 来对数据编码，然后写入 YAML 文件，可以：

```bash
[K8S ~]$ echo -n "123456" | base64
MTIzNDU2
```

>[!warning] 注意
>这条命令里的 `echo`，必须加上参数 `-n` 去掉字符串里隐含的换行符，
>否则 Base64 编码得到的字符串就是错误的。

## 重新编辑 YAML 文件

重新编辑 Secret 的 YAML 文件，为它添加两条数据。添加方式既可以用手动编码，

也可以用 `kubectl create secret generic` 配合 `--from-literal` 自动进行 Base64 编码：

```
kubectl create secret generic user \
  --from-literal=name=root \
  --from-literal=pwd=123456 \
  --from-literal=db=mysql \
  --dry-run=client -o yaml > secret.yml
```

生成的 `secret.yml` 大致为：

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

>[!warning] 注意
>直接重新生成 `secret.yml` 时，要把原有的 `name=root` 也带上，
>否则生成的文件中不会保留原先的 `data.name`。


## 创建 Secret 对象

使用 `kubectl apply` 创建 `Secret` 对象：

```bash
kubectl apply -f secret.yml
```

这样一个存储敏感信息的 Secret 对象就创建好了。

---

# 使用 ConfigMap 和 Secret 的两种方式

环境变量的用法简单，更适合存放简短的字符串；
而存储卷更适合存放大数据量的配置文件，在 Pod 里加载成文件后可供应用直接读取使用[^2]。

[^2]: 受 etcd 的限制，ConfigMap 和 Secret 对象的大小不能超过 1 MB。

## 方式一：加载为环境变量

使用 `kubectl explain` 查看 Kubernetes 对 `valueFrom` 的说明：

```bash
kubectl explain pod.spec.containers.env.valueFrom
```

`valueFrom` 字段指定了环境变量值的来源，可以是 `configMapKeyRef` 或者 `secretKeyRef`，再进一步指定应用的 ConfigMap 和 Secret 的 `name` 和它里面的 `key`，要当心的是这个 `name` 字段是 API 对象（即 `ConfigMap` 和 `Secret` ）的名字，而不是键值对的名字。

在 `pod.yml` 中配置环境变量引用：

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

需要注意：

- Linux 系统对环境变量的命名有限制，不能使用 `-`、`.` 等特殊字符，所以在创建 ConfigMap 和 Secret 的时候需要注意，否则无法以环境变量的形式注入 Pod。
- `info` ConfigMap 和 `user` Secret 必须已在同一 `namespace` 中，并且包含对应的键。

##  方式二：挂载存储卷

Pod 的完整 YAML 描述：

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

可以使用 `volumes.configMap.items` 指定某个 Key 对应的文件名（或相对路径）：

```yaml
spec:
  volumes:
    - name: cm-vol
      configMap:
        name: info
        items:
          - key: count
            path: app-count.txt
          - key: greeting
            path: message.txt
```

其中：

- `key` 是 ConfigMap 中已有的键名，例如 `count` ；
- `path` 是相对于 `mountPath` 的路径，不能以 `/` 开头，支持使用子目录。


---

# 验证配置结果

首先使用 `kubectl apply` 创建 Pod：

```bash
kubectl apply -f pod.yml
```

再用 `kubectl exec` 进入 Pod 内部，验证环境变量是否生效：

```bash
kubectl exec -it env-pod -- sh
```

或

```bash
kubectl exec -it vol-pod -- sh
```

进而查看配置信息的加载形式：

```shell
ls /tmp/cm-items/
cat /tmp/cm-items/greeting
ls /tmp/sec-items/
cat /tmp/sec-items/pwd
```

可以看到，ConfigMap 和 Secret 对象都变成了目录的形式，而键值对变成了一个个的文件，文件名是 Key、文件内容是 Value。