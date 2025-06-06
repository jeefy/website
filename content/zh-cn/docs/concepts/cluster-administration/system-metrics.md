---
title: Kubernetes 系统组件指标
content_type: concept
weight: 70
---
<!--
title: Metrics For Kubernetes System Components
reviewers:
- brancz
- logicalhan
- RainbowMango
content_type: concept
weight: 70
-->

<!-- overview -->

<!--
System component metrics can give a better look into what is happening inside them. Metrics are
particularly useful for building dashboards and alerts.

Kubernetes components emit metrics in [Prometheus format](https://prometheus.io/docs/instrumenting/exposition_formats/).
This format is structured plain text, designed so that people and machines can both read it.
-->
通过系统组件指标可以更好地了解系统组个内部发生的情况。系统组件指标对于构建仪表板和告警特别有用。

Kubernetes 组件以
[Prometheus 格式](https://prometheus.io/docs/instrumenting/exposition_formats/)生成度量值。
这种格式是结构化的纯文本，旨在使人和机器都可以阅读。

<!-- body -->

<!--
## Metrics in Kubernetes

In most cases metrics are available on `/metrics` endpoint of the HTTP server. For components that
don't expose endpoint by default, it can be enabled using `--bind-address` flag.

Examples of those components:
-->
## Kubernetes 中组件的指标  {#metrics-in-kubernetes}

在大多数情况下，可以通过 HTTP 服务器的 `/metrics` 端点来获取组件的度量值。
对于那些默认情况下不暴露端点的组件，可以使用 `--bind-address` 参数来启用。

这些组件的示例：

* {{< glossary_tooltip term_id="kube-controller-manager" text="kube-controller-manager" >}}
* {{< glossary_tooltip term_id="kube-proxy" text="kube-proxy" >}}
* {{< glossary_tooltip term_id="kube-apiserver" text="kube-apiserver" >}}
* {{< glossary_tooltip term_id="kube-scheduler" text="kube-scheduler" >}}
* {{< glossary_tooltip term_id="kubelet" text="kubelet" >}}

<!--
In a production environment you may want to configure [Prometheus Server](https://prometheus.io/)
or some other metrics scraper to periodically gather these metrics and make them available in some
kind of time series database.

Note that {{< glossary_tooltip term_id="kubelet" text="kubelet" >}} also exposes metrics in
`/metrics/cadvisor`, `/metrics/resource` and `/metrics/probes` endpoints. Those metrics do not
have the same lifecycle.

If your cluster uses {{< glossary_tooltip term_id="rbac" text="RBAC" >}}, reading metrics requires
authorization via a user, group or ServiceAccount with a ClusterRole that allows accessing
`/metrics`. For example:
-->
在生产环境中，你可能需要配置
[Prometheus 服务器](https://prometheus.io/)或某些其他指标搜集器以定期收集这些指标，
并使它们在某种时间序列数据库中可用。

请注意，{{< glossary_tooltip term_id="kubelet" text="kubelet" >}} 还会在 `/metrics/cadvisor`，
`/metrics/resource` 和 `/metrics/probes` 端点中公开度量值。这些度量值的生命周期各不相同。

如果你的集群使用了 {{< glossary_tooltip term_id="rbac" text="RBAC" >}}，
则读取指标需要通过基于用户、组或 ServiceAccount 的鉴权，要求具有允许访问
`/metrics` 的 ClusterRole。例如：

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
  - nonResourceURLs:
      - "/metrics"
    verbs:
      - get
```

<!--
## Metric lifecycle

Alpha metric →  Stable metric →  Deprecated metric →  Hidden metric → Deleted metric

Alpha metrics have no stability guarantees. These metrics can be modified or deleted at any time.

Stable metrics are guaranteed to not change. This means:

* A stable metric without a deprecated signature will not be deleted or renamed
* A stable metric's type will not be modified

Deprecated metrics are slated for deletion, but are still available for use.
These metrics include an annotation about the version in which they became deprecated.
-->
## 指标生命周期  {#metric-lifecycle}

Alpha 指标 →  稳定的指标 →  弃用的指标 →  隐藏的指标 → 删除的指标

Alpha 指标没有稳定性保证。这些指标可以随时被修改或者删除。

稳定的指标可以保证不会改变。这意味着：

* 稳定的、不包含已弃用（deprecated）签名的指标不会被删除（或重命名）
* 稳定的指标的类型不会被更改

已弃用的指标最终将被删除，不过仍然可用。
这类指标包含注解，标明其被废弃的版本。

<!--
For example:

* Before deprecation
-->
例如：

* 被弃用之前：

  ```
  # HELP some_counter this counts things
  # TYPE some_counter counter
  some_counter 0
  ```

<!--
* After deprecation
-->
* 被弃用之后：

  ```
  # HELP some_counter (Deprecated since 1.15.0) this counts things
  # TYPE some_counter counter
  some_counter 0
  ```

<!--
Hidden metrics are no longer published for scraping, but are still available for use. To use a
hidden metric, please refer to the [Show hidden metrics](#show-hidden-metrics) section.

Deleted metrics are no longer published and cannot be used.
-->
隐藏的指标不会再被发布以供抓取，但仍然可用。
要使用隐藏指标，请参阅[显式隐藏指标](#show-hidden-metrics)节。

删除的指标不再被发布，亦无法使用。

<!--
## Show hidden metrics

As described above, admins can enable hidden metrics through a command-line flag on a specific
binary. This intends to be used as an escape hatch for admins if they missed the migration of the
metrics deprecated in the last release.

The flag `show-hidden-metrics-for-version` takes a version for which you want to show metrics
deprecated in that release. The version is expressed as x.y, where x is the major version, y is
the minor version. The patch version is not needed even though a metrics can be deprecated in a
patch release, the reason for that is the metrics deprecation policy runs against the minor release.
-->
## 显示隐藏指标   {#show-hidden-metrics}

如上所述，管理员可以通过设置可执行文件的命令行参数来启用隐藏指标，
如果管理员错过了上一版本中已经弃用的指标的迁移，则可以把这个用作管理员的逃生门。

`show-hidden-metrics-for-version` 参数接受版本号作为取值，
版本号给出你希望显示该发行版本中已弃用的指标。
版本表示为 x.y，其中 x 是主要版本，y 是次要版本。补丁程序版本不是必须的，
即使指标可能会在补丁程序发行版中弃用，原因是指标弃用策略规定仅针对次要版本。

<!--
The flag can only take the previous minor version as it's value. All metrics hidden in previous
will be emitted if admins set the previous version to `show-hidden-metrics-for-version`. The too
old version is not allowed because this violates the metrics deprecated policy.

Take metric `A` as an example, here assumed that `A` is deprecated in 1.n. According to metrics
deprecated policy, we can reach the following conclusion:
-->
此参数的取值只能使用前一个次要版本。如果管理员将前一个版本设置为 `show-hidden-metrics-for-version`，
则前一个版本中隐藏的度量值会再度生成。不允许使用过旧的版本，因为那样会违反指标弃用策略。

以指标 `A` 为例，此处假设 `A` 在 1.n 中已弃用。根据指标弃用策略，我们可以得出以下结论：

<!--
* In release `1.n`, the metric is deprecated, and it can be emitted by default.
* In release `1.n+1`, the metric is hidden by default and it can be emitted by command line
  `show-hidden-metrics-for-version=1.n`.
* In release `1.n+2`, the metric should be removed from the codebase. No escape hatch anymore.

If you're upgrading from release `1.12` to `1.13`, but still depend on a metric `A` deprecated in
`1.12`, you should set hidden metrics via command line: `--show-hidden-metrics=1.12` and remember
to remove this metric dependency before upgrading to `1.14`
-->
* 在版本 `1.n` 中，这个指标已经弃用，且默认情况下可以生成。
* 在版本 `1.n+1` 中，这个指标默认隐藏，可以通过命令行参数 `show-hidden-metrics-for-version=1.n` 来再度生成。
* 在版本 `1.n+2` 中，这个指标就将被从代码中移除，不会再有任何逃生窗口。

如果你要从版本 `1.12` 升级到 `1.13`，但仍依赖于 `1.12` 中弃用的指标 `A`，则应通过命令行设置隐藏指标：
`--show-hidden-metrics=1.12`，并记住在升级到 `1.14` 版本之前移除此指标依赖项。

<!--
## Component metrics

### kube-controller-manager metrics

Controller manager metrics provide important insight into the performance and health of the
controller manager. These metrics include common Go language runtime metrics such as go_routine
count and controller specific metrics such as etcd request latencies or Cloudprovider (AWS, GCE,
OpenStack) API latencies that can be used to gauge the health of a cluster.
-->
## 组件指标  {#component-metrics}

### kube-controller-manager 指标  {#kube-controller-manager-metrics}

控制器管理器指标可提供有关控制器管理器性能和运行状况的重要洞察。
这些指标包括通用的 Go 语言运行时指标（例如 go_routine 数量）和控制器特定的度量指标，
例如可用于评估集群运行状况的 etcd 请求延迟或云提供商（AWS、GCE、OpenStack）的 API 延迟等。

<!--
Starting from Kubernetes 1.7, detailed Cloudprovider metrics are available for storage operations
for GCE, AWS, Vsphere and OpenStack.
These metrics can be used to monitor health of persistent volume operations.

For example, for GCE these metrics are called:
-->
从 Kubernetes 1.7 版本开始，详细的云提供商指标可用于 GCE、AWS、Vsphere 和 OpenStack 的存储操作。
这些指标可用于监控持久卷操作的运行状况。

比如，对于 GCE，这些指标称为：

```
cloudprovider_gce_api_request_duration_seconds { request = "instance_list"}
cloudprovider_gce_api_request_duration_seconds { request = "disk_insert"}
cloudprovider_gce_api_request_duration_seconds { request = "disk_delete"}
cloudprovider_gce_api_request_duration_seconds { request = "attach_disk"}
cloudprovider_gce_api_request_duration_seconds { request = "detach_disk"}
cloudprovider_gce_api_request_duration_seconds { request = "list_disk"}
```

<!--
### kube-scheduler metrics
-->
### kube-scheduler 指标   {#kube-scheduler-metrics}

{{< feature-state for_k8s_version="v1.21" state="beta" >}}

<!--
The scheduler exposes optional metrics that reports the requested resources and the desired limits
of all running pods. These metrics can be used to build capacity planning dashboards, assess
current or historical scheduling limits, quickly identify workloads that cannot schedule due to
lack of resources, and compare actual usage to the pod's request.
-->
调度器会暴露一些可选的指标，报告所有运行中 Pod 所请求的资源和期望的约束值。
这些指标可用来构造容量规划监控面板、访问调度约束的当前或历史数据、
快速发现因为缺少资源而无法被调度的负载，或者将 Pod 的实际资源用量与其请求值进行比较。

<!--
The kube-scheduler identifies the resource [requests and limits](/docs/concepts/configuration/manage-resources-containers/)
configured for each Pod; when either a request or limit is non-zero, the kube-scheduler reports a
metrics timeseries. The time series is labelled by:

- namespace
- pod name
- the node where the pod is scheduled or an empty string if not yet scheduled
- priority
- the assigned scheduler for that pod
- the name of the resource (for example, `cpu`)
- the unit of the resource if known (for example, `cores`)
-->
kube-scheduler 组件能够辩识各个 Pod 所配置的资源
[请求和约束](/zh-cn/docs/concepts/configuration/manage-resources-containers/)。
在 Pod 的资源请求值或者约束值非零时，kube-scheduler 会以度量值时间序列的形式
生成报告。该时间序列值包含以下标签：

- 名字空间
- Pod 名称
- Pod 调度所处节点，或者当 Pod 未被调度时用空字符串表示
- 优先级
- 为 Pod 所指派的调度器
- 资源的名称（例如，`cpu`）
- 资源的单位，如果知道的话（例如，`cores`）

<!--
Once a pod reaches completion (has a `restartPolicy` of `Never` or `OnFailure` and is in the
`Succeeded` or `Failed` pod phase, or has been deleted and all containers have a terminated state)
the series is no longer reported since the scheduler is now free to schedule other pods to run.
The two metrics are called `kube_pod_resource_request` and `kube_pod_resource_limit`.

The metrics are exposed at the HTTP endpoint `/metrics/resources`. They require
authorization for the `/metrics/resources` endpoint, usually granted by a
ClusterRole with the `get` verb for the `/metrics/resources` non-resource URL.

On Kubernetes 1.21 you must use the `--show-hidden-metrics-for-version=1.20`
flag to expose these alpha stability metrics.
-->
一旦 Pod 进入完成状态（其 `restartPolicy` 为 `Never` 或 `OnFailure`，且其处于
`Succeeded` 或 `Failed` Pod 阶段，或者已经被删除且所有容器都具有终止状态），
该时间序列停止报告，因为调度器现在可以调度其它 Pod 来执行。
这两个指标称作 `kube_pod_resource_request` 和 `kube_pod_resource_limit`。

这些指标通过 HTTP 端点 `/metrics/resources` 公开出来。
访问 `/metrics/resources` 端点需要鉴权，通常通过对
`/metrics/resources` 非资源 URL 的 `get` 访问授予访问权限。  

在 Kubernetes 1.21 中，你必须使用 `--show-hidden-metrics-for-version=1.20`
参数来公开 Alpha 级稳定性的指标。

<!--
### kubelet Pressure Stall Information (PSI) metrics
-->
### kubelet 压力阻塞信息（PSI）指标

{{< feature-state for_k8s_version="v1.33" state="alpha" >}}

<!--
As an alpha feature, Kubernetes lets you configure kubelet to collect Linux kernel
[Pressure Stall Information](https://docs.kernel.org/accounting/psi.html)
(PSI) for CPU, memory and IO usage.
The information is collected at node, pod and container level.
The metrics are exposed at the `/metrics/cadvisor` endpoint with the following names:
-->
作为一个 Alpha 阶段的特性，Kubernetes 允许你配置 kubelet 以基于 CPU、内存和 IO 的使用情况收集 Linux
内核的[压力阻塞信息（PSI）](https://docs.kernel.org/accounting/psi.html)。
此信息是在节点、Pod 和容器级别进行收集的。
这些指标通过 `/metrics/cadvisor` 端点暴露，指标名称如下：

```
container_pressure_cpu_stalled_seconds_total
container_pressure_cpu_waiting_seconds_total
container_pressure_memory_stalled_seconds_total
container_pressure_memory_waiting_seconds_total
container_pressure_io_stalled_seconds_total
container_pressure_io_waiting_seconds_total
```

<!--
You must enable the `KubeletPSI` [feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
to use this feature. The information is also exposed in the
[Summary API](/docs/reference/instrumentation/node-metrics#psi).
-->
要使用此特性，你必须启用 `KubeletPSI` [特性门控](/zh-cn/docs/reference/command-line-tools-reference/feature-gates/)。
此信息也会通过 [Summary API](/zh-cn/docs/reference/instrumentation/node-metrics#psi) 暴露。

<!--
#### Requirements

Pressure Stall Information requires:

- [Linux kernel versions 4.20 or later](/docs/reference/node/kernel-version-requirements#requirements-psi).
- [cgroup v2](/docs/concepts/architecture/cgroups)
-->
#### 要求

启用压力阻塞信息需满足以下条件：

- [Linux 内核版本为 4.20 或更高](/zh-cn/docs/reference/node/kernel-version-requirements#requirements-psi)
- [cgroup v2](/zh-cn/docs/concepts/architecture/cgroups)

<!--
## Disabling metrics

You can explicitly turn off metrics via command line flag `--disabled-metrics`. This may be
desired if, for example, a metric is causing a performance problem. The input is a list of
disabled metrics (i.e. `--disabled-metrics=metric1,metric2`).
-->
## 禁用指标 {#disabling-metrics}

你可以通过命令行参数 `--disabled-metrics` 来关闭某指标。
在例如某指标会带来性能问题的情况下，这一操作可能是有用的。
参数值是一组被禁用的指标（例如：`--disabled-metrics=metric1,metric2`）。

<!--
## Metric cardinality enforcement

Metrics with unbounded dimensions could cause memory issues in the components they instrument. To
limit resource use, you can use the `--allow-metric-labels` command line option to dynamically
configure an allow-list of label values for a metric.

In alpha stage, the flag can only take in a series of mappings as metric label allow-list.
Each mapping is of the format `<metric_name>,<label_name>=<allowed_labels>` where 
`<allowed_labels>` is a comma-separated list of acceptable label names.
-->
## 指标顺序性保证    {#metric-cardinality-enforcement}

具有无限维度的指标可能会在其监控的组件中引起内存问题。
为了限制资源使用，你可以使用 `--allow-metric-labels` 命令行选项来为指标动态配置允许的标签值列表。

在 Alpha 阶段，此选项只能接受一组映射值作为可以使用的指标标签。
每个映射值的格式为`<指标名称>,<标签名称>=<可用标签列表>`，其中
`<可用标签列表>` 是一个用逗号分隔的、可接受的标签名的列表。

<!--
The overall format looks like:

```
--allow-metric-labels <metric_name>,<label_name>='<allow_value1>, <allow_value2>...', <metric_name2>,<label_name>='<allow_value1>, <allow_value2>...', ...
```
-->
最终的格式看起来会是这样：

```
--allow-metric-labels <指标名称>,<标签名称>='<可用值1>,<可用值2>...', <指标名称2>,<标签名称>='<可用值1>, <可用值2>...', ...
```

<!--
Here is an example:
-->
下面是一个例子：

```none
--allow-metric-labels number_count_metric,odd_number='1,3,5', number_count_metric,even_number='2,4,6', date_gauge_metric,weekend='Saturday,Sunday'
```

<!--
In addition to specifying this from the CLI, this can also be done within a configuration file. You
can specify the path to that configuration file using the `--allow-metric-labels-manifest` command
line argument to a component. Here's an example of the contents of that configuration file:
-->
除了从 CLI 中指定之外，还可以在配置文件中完成此操作。
你可以使用组件的 `--allow-metric-labels-manifest` 命令行参数指定该配置文件的路径。
以下是该配置文件的内容示例：

```yaml
"metric1,label2": "v1,v2,v3"
"metric2,label1": "v1,v2,v3"
```

<!--
Additionally, the `cardinality_enforcement_unexpected_categorizations_total` meta-metric records the
count of unexpected categorizations during cardinality enforcement, that is, whenever a label value
is encountered that is not allowed with respect to the allow-list constraints.
-->
此外，`cardinality_enforcement_unexpected_categorizations_total`
元指标记录基数执行期间意外分类的计数，即每当遇到允许列表约束不允许的标签值时。

## {{% heading "whatsnext" %}}

<!--
* Read about the [Prometheus text format](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md#text-based-format)
  for metrics
* See the list of [stable Kubernetes metrics](https://github.com/kubernetes/kubernetes/blob/master/test/instrumentation/testdata/stable-metrics-list.yaml)
* Read about the [Kubernetes deprecation policy](/docs/reference/using-api/deprecation-policy/#deprecating-a-feature-or-behavior)
-->
* 阅读有关指标的 [Prometheus 文本格式](https://github.com/prometheus/docs/blob/master/content/docs/instrumenting/exposition_formats.md#text-based-format)
* 参阅[稳定的 Kubernetes 指标](https://github.com/kubernetes/kubernetes/blob/master/test/instrumentation/testdata/stable-metrics-list.yaml)的列表
* 阅读有关 [Kubernetes 弃用策略](/zh-cn/docs/reference/using-api/deprecation-policy/#deprecating-a-feature-or-behavior)
