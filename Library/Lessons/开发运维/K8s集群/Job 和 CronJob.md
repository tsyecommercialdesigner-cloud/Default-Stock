---
title: Job 和 CronJob
created: 2026-08-06
source: 《Kubernetes 零基础实战》
tags:
  - Job
  - CronJob
---
---

# 离线业务 Job 和 CronJob

Kubernetes 的核心对象 Pod 可以编排一个或多个容器，让这些容器共享网络、存储等资源。

因为 Pod 比容器更能表示实际的应用，所以 Kubernetes 不会在容器层面来编排业务，而是把 Pod 作为在集群里调度运维的最小单位。

Kubernetes 以 Pod 为中心，延伸出了很多表示各种业务的资源对象。Pod 的功能已经足够完善了，为什么还要定义这些额外的对象呢？为什么不直接在 Pod 里添加功能，来处理业务需求呢？

这个问题体现了 Google 对管理大规模计算集群的深度思考，下面将讲解 Kubernetes 基于 Pod 的设计理念，先从离线业务对象——Job 和 CronJob 开始。

---

# 为什么不直接使用 Pod

Kubernetes 使用的是 RESTful API，将集群中的各种业务抽象为 HTTP 资源对象，在这个层次之上就可以使用面向对象的方式来考虑问题。面向对象编程（OOP）把一切视为高内聚的对象，强调对象之间互相通信来完成任务。

面向对象的设计思想虽然多用于软件开发，但却意外地适合 Kubernetes。因为 Kubernetes 使用 YAML 来描述资源，把业务简化成一个个的对象，内部有属性，外部有联系，也需要互相协作，只不过不需要编程，完全由 Kubernetes 自动处理（其实 Kubernetes 的 Go 语言内部实现就大量应用了面向对象的设计）。

面向对象的设计有许多基本原则，其中有两条比较恰当地描述了 Kubernetes 对象设计思路，一条是单一职责，另一条是组合优于继承。

**单一职责**的通俗理解是对象应该专注于做好一件事，不要贪大求全，保持足够小的粒度才方便复用和管理。

**组合优于继承**的通俗理解是应该尽量让对象在运行时产生联系，保持松耦合，而不要用硬编码的方式固定对象的关系。

基于这两条设计原则，再来看 Kubernetes 的资源对象就会很清晰。因为 Pod 已经是一个相对完善的对象，专门负责管理容器，那么就不应该盲目为它扩充功能，而是要保持它的独立性，容器之外的功能就再定义其他对象，把 Pod 作为它的一个成员。

这样每种 Kubernetes 对象就可以只关注自己的业务领域，做自己最擅长的事情，其他工作交给其他对象来处理，既有分工又有协作，从而以更小的成本实现更大的收益。

---

# 为什么要有 Job 和 CronJob

Kubernetes 里的两种对象 Job 和 CronJob 就是组合了 Pod，来实现对离线业务的处理。

假设当前运行了 Nginx 和 busybox 两个 Pod，它们分别代表了 Kubernetes 里的两大类业务。一类是像 Nginx 这样长时间运行的在线业务，另一类是像 busybox 这样短时间运行的离线业务[^1]。

[^1]: 离线业务的例子有很多，著名的例子有 MapReduce 和 Hadoop。

在线业务类型的应用有很多，例如 Nginx、Node.js、Tomcat、MySQL、Redis 等，一旦运行起来基本上不会停，也就是“永远在线”。

而离线业务类型的应用也不少，它们一般不直接服务于外部用户，只对内部用户有意义，例如日志分析、数据建模、音视频转码等，虽然计算量很大，但只会运行一段时间。离线业务的特点是必定会退出，不会无限期地运行下去，所以它的调度策略与在线业务存在很大的不同，需要考虑运行超时、状态检查、失败重试、获取计算结果等。

而这些业务特性与容器管理没有必然的联系，如果由 Pod 来实现就违反了单一职责的原则，所以应该把这部分功能分离到另外一个对象上实现，让这个对象来控制 Pod 的运行，完成附加的工作。

离线业务也可以分为两种：

- 一种是“临时任务”，运行完成就结束，下次有需求再重新安排；
- 另一种是“定时任务”，按时运行，不需要过多干预。

对应到 Kubernetes 里，“临时任务”就是 API 对象 Job，“定时任务”就是 API 对象 CronJob[^2]，使用这两个对象就能够在 Kubernetes 里调度管理任意的离线业务。

[^2]: 单词“Cron”和“Kubernetes”一样，也是源自希腊语，即 Chronos ，意为“时间”。

---
---

# 用 YAML 描述 Job 和 CronJob

Job 和 CronJob 都属于离线业务，具有一定的相似性，下面先介绍通常只运行一次的 Job 对象。

Job 的 YAML 文件头部包括如下 3 个必备字段：

- **`apiVersion` 字段**：不是“`v1`”，而是“`batch/v1`”[^2]。
- **`kind` 字段**：是“`Job`”，和对象的名字一致。
- **`metadata` 字段**：仍然要由“`name`”字段标记名字，也可以通过“`labels`”字段添加任意的标签。

[^2]: Job 和 CronJob 的“`apiVersion`”字段是“`batch/v1`”，表示它们不属于核心对象组（core group），而是批处理对象组（batch group）。

这些字段的说明都可以使用命令“`kubectl explain job`”查看。不过，想要生成 YAML 模板文件不能使用“`kubectl run`”命令，因为“`kubectl run`”命令只能创建 Pod，要创建 Pod 以外的其他 API 对象需要使用“`kubectl create`”命令，并在命令中加上对象的类型。

例如用 busybox 创建一个“echo-job”，命令如下：

```bash
export out="--dry-run=client -o yaml"    # 定义 Shell 变量
kubectl create job echo-job --image=busybox $out
```

它会生成一个基本的 YAML 文件，保存之后对 YAML 文件做如下修改，就有了第一个 Job 对象：

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

可以发现，Job 的描述与 Pod 很像，主要区别在于“`spec`”字段里多了一个“`template`”字段，“`template`”字段里又有一个“`spec`”字段。这种做法其实是在 Job 对象里应用了组合模式，“`template`”字段定义了一个应用模板，里面嵌入了一个 Pod，这样 Job 就可以从这个模板创建 Pod。

而这个 Pod 因为受 Job 的管理，不直接和 apiserver 通信，也就没必要重复“`apiVersion`”等头字段，只需要定义关键的“`spec`”字段，描述清楚容器相关的信息，可以说是“无头”的 Pod 对象。

其实，“echo-job”只是对 Pod 做了简单的包装，并没有太多的额外功能。

这个 Pod 对象的工作包括，在“`containers`”里定义名字和镜像，“`command`”执行“`/bin/echo`”命令，输出“`hello world`”。

不过，因为 Job 业务的特殊性，还要在“`spec`”里多加一个字段“`restartPolicy`”，定义 Pod 运行失败时的策略，“`OnFailure`”表示失败时重启容器，而“`Never`”则表示不重启容器，让 Job 重新调度生成一个新的 Pod。

---

# 用 kubectl 操作 Job

现在来创建 Job 对象，运行这个简单的离线作业，还是使用如下“`kubectl apply`”命令：

```bash
kubectl apply -f job.yml
```

Kubernetes 会从 YAML 的模板定义中提取 Pod，在 Job 的控制下运行 Pod。
如下命令，“`kubectl get job`”“`kubectl get pod`”分别用来查看 Job 和 Pod 的状态：

```bash
kubectl get job
```

```bash
kubectl get pod
```

返回结果为：

```text
[K8S ~]$ kubectl get job
NAME       COMPLETIONS   DURATION   AGE
echo-job   1/1           3s         18s

[K8S ~]$ kubectl get pod
NAME             READY   STATUS      RESTARTS   AGE
echo-job-829t5   0/1     Completed   0          22s
```

可以看到，因为 Pod 被 Job 管理，所以不会反复重启报错，而是显示为“`Completed`”（表示任务完成），Job 里会列出运行成功的作业数量，这里只有一个作业，所以是“`1/1`”。

还可以看到，Pod 被自动关联了一个名字，用的是 Job 的名字（echo-job）加一个随机字符串（829t5），这是因为为 Job 的管理，免去了手动定义 Pod 对象名字的麻烦。使用命令“`kubectl logs`”，可以获取 Pod 的运行结果：

```bash
kubectl logs echo-job-829t5
```

```text
[K8S ~]$ kubectl logs echo-job-829t5
hello world
```

有些读者可能会觉得，经过了 Job、Pod 对容器的两次封装，虽然从概念上更清晰，但并没有带来什么实际的好处，和直接运行容器也差不了多少。

其实 Kubernetes 的这套 YAML 描述对象的框架提供了非常多的灵活性，可以在 Job 级别、Pod 级别添加任意的字段来定制业务，而这种优势是简单的容器技术无法相比的。

下面列出控制离线作业的一些重要字段，其他更详细的信息可以参考 Job 文档。

- `activeDeadlineSeconds`：设置 Pod 运行的超时时间。
- `backoffLimit`：设置 Pod 的失败重试次数。
- `completions`：Job 完成需要运行多少个 Pod，默认是 1 个。
- `parallelism`：与 completions 相关，表示允许并发运行的 Pod 数量，避免过多地占用资源。

要注意这 4 个字段并不在“`template`”字段下，而是在“`spec`”字段下，所以它们属于 Job 级别，可以用来控制模板里的 Pod 对象。

可以再创建一个 Job 对象，名字是“sleep-job”。它随机休眠一段时间再退出，模拟运行时间较长的作业（如 MapReduce）。Job 的参数设置为 15 s 超时，最多重试 2 次，需要运行完 4 个 Pod，但同一时刻最多并发运行 2 个 Pod：

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

使用“`kubectl apply`”创建 Job 之后，可以使用“`kubectl get pod -w`”来实时观察 Pod 的状态，查看 Pod 不断被排队、创建、运行的过程：

```text
[K8S ~]$ kubectl apply -f sleep-job.yml
job.batch/sleep-job created

[K8S ~]$ kubectl get pod -w
NAME              READY   STATUS
sleep-job-6qdfb   0/1     Completed
sleep-job-j7tr6   1/1     Running
sleep-job-j7tr6   0/1     Completed
sleep-job-jh78n   1/1     Running
sleep-job-jh78n   0/1     Completed
sleep-job-zqxd7   0/1     Pending
sleep-job-zqxd7   0/1     Completed
```

4 个 Pod 运行完毕后，再用“`kubectl get`”查看 Job 和 Pod 的状态：

```text
[K8S ~]$ kubectl get job
NAME        COMPLETIONS   DURATION   AGE
sleep-job   4/4           21s        21s

[K8S ~]$ kubectl get pod
NAME              READY   STATUS      RESTARTS   AGE
sleep-job-6qdfb   0/1     Completed   0          17s
sleep-job-j7tr6   0/1     Completed   0          24s
sleep-job-jh78n   0/1     Completed   0          15s
sleep-job-zqxd7   0/1     Completed   0          24s
```

可以看到，Job 的完成数量是 4，而 4 个 Pod 也都是完成状态，符合预期[^3]。

[^3]: 为了方便获取计算结果，Job 在运行结束后不会被立即删除。为避免过多的已完成 Job 消耗系统资源，可以使用字段“`ttlSecondsAfterFinished`”设置 Job 运行结束后的保留时间。

显然，声明式的 Job 对象让离线业务的描述变得非常直观，简单的几个字段就可以很好地控制作业的并行度和完成数量，Kubernetes 把这些都自动实现了，不需要人工监控干预。

---

# 用 kubectl 操作 CronJob

学习了 Job 对象之后，再学习 CronJob 对象比较容易，可以直接使用命令“`kubectl create`”来创建 CronJob 的 YAML 模板文件。

```bash
export out="--dry-run=client -o yaml"    # 定义 Shell 变量
kubectl create cj echo-cj --image=busybox --schedule="" $out
```

需要注意两点。第一，CronJob 的名字有些长，Kubernetes 提供了简写“`cj`”，可以使用命令“`kubectl api-resources`”看到这个简写；第二，CronJob 需要定时运行，所以还需要在命令行里指定参数“`--schedule`”。

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

这里需要重点关注“`spec`”字段，它里面有 3 个“`spec`”嵌套层次：

- 第一个“`spec`”是 CronJob 自己的对象声明；
- 第二个“`spec`”从属于“`jobTemplate`”，定义了一个 Job 对象；
- 第三个“`spec`”从属于“`template`”，定义了 Job 里运行的 Pod。

所以，CronJob 其实是组合了 Job 而生成的新对象，是一种重复嵌套结构。

除了 Job 对象的“`jobTemplate`”字段，CronJob 对象的新字段是“`schedule`”，用来定义任务周期运行的规则。

它使用的是标准的 Cron 语法，指定分钟、小时、天、月、周，和 Linux 中的 `crontab` 相同。
在这个 CronJob 对象中定义的是每分钟运行一次，Cron 语法的具体含义读者可以参考 Kubernetes 官网文档[^4]。

[^4]: 如果认为 Cron 时间设置语法不好理解，可以在 [crontab.guru](https://crontab.guru) 网站中查看各表达式的含义。

除了名字不同，CronJob 和 Job 的用法几乎一样，使用“`kubectl apply`”创建 CronJob，使用“`kubectl get cj`”“`kubectl get pod`”分别查看 CronJob 和 Pod 的状态[^5]：

[^5]: 出于节约资源的考虑，CronJob 不会无限期保留已经运行的 Job，默认只保留 3 个最近的执行结果，但可以通过“`successfulJobsHistoryLimit`”字段来修改配置。

```text
[K8S ~]$ kubectl apply -f cronjob.yml
cronjob.batch/echo-cj created

[K8S ~]$ kubectl get cj
NAME      SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
echo-cj   */1 * * * *   False     0        3s              35s

[K8S ~]$ kubectl get pod
NAME                     READY   STATUS      RESTARTS   AGE
echo-cj-28185203-nrdrq   0/1     Completed   0          6s
```

---

# 小结

本节介绍了 Kubernetes 中资源对象设计的原则，它强调单一职责和组合优于继承，简单来说就是对象嵌套对象。

通过这种嵌套方式，Kubernetes 里的 API 对象形成了一个控制链：

> CronJob 使用定时规则控制 Job
> 	Job 使用并发数量控制 Pod
> 		Pod 再定义参数控制容器 Container
> 			Container 容器再隔离控制进程 Process
> 				Process 进程最终实现业务功能 Feature

这种层层递进的形式有点像设计模式里的装饰模式（decorator pattern），控制链中的每个环节各司其职，在 Kubernetes 的统一指挥下完成任务。

本节的内容要点如下。

- Pod 是 Kubernetes 的最小调度单元，为了保持它的独立性，不应该向它添加额外的功能。
- Kubernetes 为离线业务提供了 Job 和 CronJob 两种 API 对象，分别处理临时任务和定时任务。
- Job 的关键字段是“`spec.template`”，定义了运行业务的 Pod 模板，其他重要字段有“`completions`”“`parallelism`”等。
- CronJob 的关键字段是“`spec.jobTemplate`”和“`spec.schedule`”，分别定义了 Job 模板和定时运行的规则。

---



