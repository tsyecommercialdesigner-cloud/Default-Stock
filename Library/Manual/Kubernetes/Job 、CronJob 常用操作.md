---
title: CronJob、Job 常用操作
created: 2026-08-06
tags:
  - CronJob
  - Job
---
---

# Job 

## 创建 YAML 模板

 创建一个“echo-job”，命令如下：

```bash
export out="--dry-run=client -o yaml"
kubectl create job echo-job --image=busybox $out
```

它会生成一个基本的 YAML 文件：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: echo-job

spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - image: busybox
          name: echo-job
          imagePullPolicy: IfNotPresent
          command: ["/bin/echo"]
          args: ["hello", "world"]
```

以下是一个进阶的 YAML 文件：

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sleep-job

spec:
  activeDeadlineSeconds: 15
  backoffLimit: 2
  completions: 4
  parallelism: 2

  template:
    spec:
      restartPolicy: OnFailure
      containers:
        - image: busybox
          name: echo-job
          imagePullPolicy: IfNotPresent
          command:
            - sh
            - -c
            - sleep $((RANDOM % 10 + 1)) && echo done
```

## 创建 Job 对象

使用如下“`kubectl apply`”命令：

```bash
kubectl apply -f job.yml
```

---

# CronJob

## 创建 YAML 模板

直接使用命令“`kubectl create`”来创建 CronJob 的 YAML 模板文件：

```bash
export out="--dry-run=client -o yaml"
kubectl create cj echo-cj --image=busybox --schedule="" $out
```

编辑上面定义的这个 YAML 模板文件可以得到 CronJob 对象：

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-cj

spec:
  schedule: '*/1 * * * *'
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - image: busybox
              name: echo-job
              imagePullPolicy: IfNotPresent
              command: ["/bin/echo"]
              args: ["hello", "world"]
```

## 创建 CronJob 对象

使用如下“`kubectl apply`”命令：

```bash
kubectl apply -f cronjob.yml
```

---