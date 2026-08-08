---
title: PersistentVolume 常用操作
created: 2026-08-08
tags:
  - Persistent_Volume
  - Storage_Class
---
---

# PersistentVolume

## 定义 PersistentVolume 对象

下面给出一个 PV YAML 示例，可以把它作为模板文件，创建自己的 PV：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-10m-pv

spec:
  storageClassName: host-test
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Mi
  hostPath:
    path: /tmp/host-10m-pv
```

`storageClassName`字段：

> 是对存储类型的抽象 StorageClass，由于这个 PV 是手动管理的，于是名字可以自由定义。

`accessModes` 定义虚拟盘的读写权限：

> * `ReadOnlyMany`：存储卷只读不可写，可以被任意节点上的 Pod 多次挂载。
> * `ReadWriteOnce`：存储卷可读可写，但只能被一个节点上的 Pod 挂载。
> * `ReadWriteMany`：存储卷可读可写，可以被任意节点上的 Pod 多次挂载。
> - `ReadWriteOncePod`：只允许单个 Pod 读写，要求存储系统必须符合 CSI（Container Storage Interface）规范才可以使用此属性。

要注意，前 3 种访问模式限制的对象是节点而不是 Pod，因为存储是系统级别的概念，不属于 Pod 里的进程。

`capacity` 表示存储设备的容量：

> - Kubernetes 定义存储容量使用的是国际标准，以 1000 为基数，而日常习惯使用的 KB、MB、GB 的基数是 1024，要写成 Ki、Mi、Gi，一定别写错，否则单位不一致会导致实际容量对不上。
> - KB、MB、GB 与 KiB、MiB、GiB 的混用由来已久，大概是由早期 Windows 误用引起的，而 Mac 一直使用的是 1000 作为基数的 MB、GB，各种磁盘厂商的标称容量也用的是 MB、GB。

`hostPath` 字段：

> - 指定存储卷的本地路径，也就是在节点上创建的目录。
> - 只在本节点存储，如果 Pod 重建时被调度到了其他节点，那么即使加载了本地目录，也不会是之前的存储位置，持久化功能也就失效了。
> - HostPath 类型的 PV 对象一般只能用来做测试，或者用于 DaemonSet 这样与节点关系比较密切的应用。

用以上 4 个字段把 PV 的类型、访问模式、容量、存储位置都描述清楚，一个存储设备就创建好了。

## 定义 PersistentVolumeClaim 对象

PVC 不表示实际的存储，而是一个“申请”或者“声明”，“`spec`” 字段描述的是对存储的期望状态。

以下 YAML 文件描述了一个 PVC，要求使用一个 5 MB 的存储设备，访问模式是 `ReadWriteOnce`：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: host-5m-pvc

spec:
  storageClassName: host-test
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Mi
```

Kubernetes 根据 PVC 的描述寻找匹配 StorageClass 和容量的 PV，然后把 PV 和 PVC 绑定在一起。

---

# 在 Pod 里使用 PersistentVolume

## 定义 Pod 对象

先在 “`spec.volumes`” 里定义存储卷，然后在 “`volumes`” 里用字段 “`persistentVolumeClaim`” 指定 PVC 的名字，最后通过 “`containers.volumeMounts`” 挂载进容器。

下面就是 Pod 的 YAML 文件，把存储卷挂载到了 Nginx 容器的 “`/tmp`” 目录下：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: host-pvc-pod

spec:
  volumes:
  - name: host-pvc-vol
    persistentVolumeClaim:
      claimName: host-5m-pvc

  containers:
  - name: ngx-pvc-pod
    image: nginx:alpine
    ports:
    - containerPort: 80
    volumeMounts:
    - name: host-pvc-vol
      mountPath: /tmp
```

## 创建对象

创建PersistentVolume 对象：

```bash
kubectl apply -f hostpv.yml
```

创建PersistentVolumeClaim 对象：

```bash
kubectl apply -f hostpvc.yml
```

创建 Pod 对象：

```bash
kubectl apply -f hostpvcpod.yml
```

## 验证挂载结果

查看 PV 和 PVC 对象的状态：

```bash
kubectl get pv
```

```bash
kubectl get pvc
```

进入 Pod 验证 PV 是否挂载成功：

```bash
kubectl exec -it host-pvc-pod -- sh
```

```bash
echo 'aaa' > /tmp/a.txt
exit
```

登录数据节点检查：

```bash
cat /tmp/host-10m-pv/a.txt
```

---

# 在 Pod 里使用静态网络存储

## 定义 PersistentVolume 对象

NFS 支持多个节点同时访问一个共享目录。定义使用 NFS 系统网络存储的 PV 对象：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-1g-pv

spec:
  storageClassName: nfs
  accessModes:
  - ReadWriteMany
  capacity:
    storage: 1Gi

  nfs:
    path: /tmp/nfs/1g-pv
    server: 192.168.26.208
```

使用时，需要确保：

- NFS 服务端已经安装并启动 NFS 服务。
- `/tmp/nfs` 已在 NFS 服务端通过 `/etc/exports` 导出。
- 所有 Kubernetes Node 已安装 NFS 客户端工具，例如 Ubuntu/Debian 上的 `nfs-common`、CentOS/RHEL 上的 `nfs-utils`。
- Kubernetes Node 能访问 `192.168.26.208` 的 NFS 服务端口。
- “`spec.nfs`” 的 IP 地址一定要正确，路径一定要存在（由管理员事先创建好），

否则 Pod 按照 PV 对象的描述会无法挂载 NFS 共享目录，导致无法正常运行。

## 定义 PersistentVolumeClaim 对象

定义申请存储资源的 PVC 对象：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-static-pvc

spec:
  storageClassName: nfs
  accessModes:
  - ReadWriteMany

  resources:
    requests:
      storage: 1Gi
```

## 定义 Pod 对象

定义把 PVC 对象挂载成一个存储卷的 Pod 对象：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-static-pod

spec:
  volumes:
  - name: nfs-pvc-vol
    persistentVolumeClaim:
      claimName: nfs-static-pvc

  containers:
  - name: nfs-pvc-test
    image: nginx:alpine
    ports:
    - containerPort: 80

    volumeMounts:
    - name: nfs-pvc-vol
      mountPath: /tmp
```

## 创建对象

创建PersistentVolume 对象：

```bash
kubectl apply -f nfspv.yml
```

创建PersistentVolumeClaim 对象：

```bash
kubectl apply -f nfspvc.yml
```

创建 Pod 对象：

```bash
kubectl apply -f nfspvcpod.yml
```

## 验证挂载结果

进入 Pod：

```bash
kubectl exec -it nfs-static-pod -- sh
```

操作 NFS 共享目录进行测试：

```bash
echo '1234' > /tmp/n.txt
exit
```

退出 Pod，查看 NFS 服务器的 “`/tmp/nfs/1g-pv`” 目录验证结果。

---

# 在 Pod 里使用动态网络存储

Provisioner 对象是一个能够自动管理存储、创建 PV 对象的应用，消除了原来系统管理员的手动工作，让创建 PV 对象的工作实现了自动化。

有了 Provisioner 对象，用户就不再需要手动定义 PV 对象了，只需要在 PVC 对象里指定 StorageClass 对象，再关联到 Provisioner 对象。

## 定义 StorageClass 对象

NFS 默认的 StorageClass 定义如下：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client

provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
parameters:
  archiveOnDelete: "false"
```

可以不使用默认的 StorageClass，根据自己的需求定制具有不同存储特性的 StorageClass：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-client-retained

provisioner: k8s-sigs.io/nfs-subdir-external-provisioner
parameters:
  onDelete: "retain"
```

`provisioner` 字段：

> - 指定了应该使用哪个 Provisioner 对象；
> - NFS Provisioner 的名字其实是由它的 YAML 文件的环境变量 “PROVISIONER_NAME” 指定的，如果觉得名字太长也可以修改，但必须同步修改关联的 StorageClass。

`parameters` 字段：

> - 用来调节 Provisioner 运行的参数，需要参考文档来确定其具体值；
> - 可以使用一些源自 PV 对象的存储回收策略，例如`archiveOnDelete`、`onDelete` 等；
> - `archiveOnDelete: "false"` ：自动回收存储空间；
> - `onDelete: "retain"` ：暂时保留分配的存储，之后再手动删除。

## 定义 PersistentVolumeClaim 对象

定义一个使用 NFS 默认的 StorageClass ，并向系统申请 10 MB 的存储空间的 PVC 对象：

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-dyn-10m-pvc

spec:
  storageClassName: nfs-client
  accessModes:
  - ReadWriteMany

  resources:
    requests:
      storage: 10Mi
```

## 定义 Pod 对象

定义一个挂载此 PVC 的 Pod 对象：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nfs-dyn-pod

spec:
  volumes:
  - name: nfs-dyn-10m-vol
    persistentVolumeClaim:
      claimName: nfs-dyn-10m-pvc

  containers:
  - name: nfs-dyn-test
    image: nginx:alpine
    ports:
    - containerPort: 80

    volumeMounts:
    - name: nfs-dyn-10m-vol
      mountPath: /tmp
```


## 创建对象

创建PersistentVolumeClaim 对象：

```bash
kubectl apply -f nfsdynpvc.yml
```

创建 Pod 对象：

```bash
kubectl apply -f nfsdynpod.yml
```

然后 Kubernetes 就会自动找到 NFS Provisioner，在 NFS 的共享目录上创建合适的 PV 对象。

可以查看自动创建的 PV：

```bash
kubectl get pv
```

## 验证挂载结果

进入 Pod：

```bash
kubectl exec -it nfs-dyn-pod -- sh
```

操作 NFS 共享目录进行测试：

```bash
echo '1234' > /tmp/n.txt
exit
```

退出 Pod，去 NFS 服务器上查看共享目录，会发现多出了一个目录，名字与这个自动创建的 PV 一样，但加上了名字空间和 PVC 的前缀。