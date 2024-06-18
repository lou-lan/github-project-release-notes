# kube-proxy

## 1.30.0

### Bug or Regression

- 修复了在 `1.26.0+` 版本中引入的 `kube-proxy` 回归问题，使 externalIPs 在 externalTrafficPolicy: Local 下正常工作。([#121919](https://github.com/kubernetes/kubernetes/pull/121919), [@uablrek](https://github.com/uablrek))
- kube-proxy：修复了在 nftables 模式下 LoadBalancerSourceRanges 无法正常工作的问题。（[#122614](https://github.com/kubernetes/kubernetes/pull/122614), [@tnqn](https://github.com/tnqn)）
- 修复了 `1.27` 及之后版本的 kube-proxy iptables 模式中的竞争条件，该条件可能导致某些更新丢失（例如，当服务获得新的端点时，新端点的规则可能要等到很久以后才会添加）。([#122204](https://github.com/kubernetes/kubernetes/pull/122204), [@danwinship](https://github.com/danwinship))

### Other (Cleanup or Flake)

- 将 kube-proxy 迁移到使用 [contextual logging](https://k8s.io/docs/concepts/cluster-administration/system-logs/#contextual-logging). ([#122197](https://github.com/kubernetes/kubernetes/pull/122197), [@fatsheep9146](https://github.com/fatsheep9146))
- kube-proxy 的 nftables 模式现在兼容内核 5.4。 ([#122296](https://github.com/kubernetes/kubernetes/pull/122296), [@tnqn](https://github.com/tnqn))


## 1.29.1

- 修复了在 1.27 及更高版本的 kube-proxy 的 iptables 模式中存在的竞争条件，该条件可能导致某些更新丢失（例如，当一个服务获得新的端点时，新的端点规则可能要到很久之后才会被添加）。([#122756](https://github.com/kubernetes/kubernetes/pull/122756), [@hakman](https://github.com/hakman)) [SIG Network]

## v1.28.0

### 紧急升级说明

- 停止在 `kubeadm upgrade plan --config` 期间接受 `kube-proxy` 和 `kubelet` 的组件配置。这是一种遗留行为，在升级时支持不佳，并且只能在计划阶段使用，以确定存储在集群中的这些组件的配置是否需要手动版本迁移。未来，`kubeadm` 将尝试替代的组件配置迁移方法。([#120788](https://github.com/kubernetes/kubernetes/pull/120788), [@chendave](https://github.com/chendave))
- `kube-proxy` 命令行文档已更新，以澄清 `--bind-address` 实际上与绑定到地址无关，而且你可能实际上并不想使用它。 ([#120274](https://github.com/kubernetes/kubernetes/pull/120274), [@danwinship](https://github.com/danwinship))

### API Change

- `kube-proxy` 现在有一个新的基于 nftables 的模式，可以通过运行以下命令启用：

  ```
  kube-proxy --feature-gates NFTablesProxyMode=true --proxy-mode nftables
  ```

  目前这是一个 alpha 级别的功能，虽然它可能不会破坏你的数据，但可能会有轻微的问题。（它通过了 e2e 测试，但还没有在实际环境中使用过。）

  目前它在功能上应该与 iptables 模式大致相同，除了它不支持（也永远不会支持）在 127.0.0.1 上的 Service NodePorts。（另请注意，当前没有特定于 nftables 配置的命令行参数；如果你想设置与 `--iptables-xxx` 选项等效的配置，需要使用配置文件。）

  由于这段代码还非常新，还没有进行大量优化；尽管预计它 *最终* 会比 iptables 后端性能更好，但目前为止几乎没有进行性能测试。（[#121046](https://github.com/kubernetes/kubernetes/pull/121046), [@danwinship](https://github.com/danwinship))

- `kube-proxy`：添加了一个用于配置 `nf_conntrack_tcp_be_liberal` sysctl（内核的 netfilter conntrack 子系统）的选项/标志。启用后，`kube-proxy` 将不会为无效的 conntrack 状态安装 `DROP` 规则，这当前会导致使用非对称路由的用户出现问题。（[#120354](https://github.com/kubernetes/kubernetes/pull/120354), [@aroradaman](https://github.com/aroradaman)）

  
### Feature

* 新增了一个 `--init-only` 命令行标志给 kube-proxy。设置该标志使 kube-proxy 执行其需要特权模式的初始配置，然后退出。`--init-only` 模式旨在在特权初始化容器中执行，以便主容器可以在更严格的 securityContext 下运行。（#120864，@uablrek）[SIG Network and Scalability]

* 修复了当 Node 对象更改其 PodCIDR 时，kube-proxy 在退出时出现恐慌的问题。（#120375，@pegasas）
* kube-proxy 仅在未设置 `nf_conntrack_tcp_be_liberal` 时，才会为无效的 conntrack 状态安装 DROP 规则。（#120412，@aojea）

### Bug or Regression

* 修复了 kube-proxy 的一个回归问题，即在具有 IPv4 和 IPv6 IP 的节点上，如果给出单栈 IPv6 配置选项，kube-proxy 可能会拒绝启动。（#121008，@danwinship）
* kube-proxy 中的 `--bind-address` 参数具有误导性，没有端口会使用该地址打开。相反，它会在内部转换为 "nodeIP"。如果未指定 `--bind-address` 或将其设置为 "any" 地址（0.0.0.0 或 ::），则两个协议族的 nodeIP 现在都从 Node 对象中获取。建议不要指定 `--bind-address`，特别是避免将其设置为 localhost（127.0.0.1 或 ::1）。（#119525，@uablrek）[SIG Network and Scalability]
* 现在，当双栈集群中仅有一个 IP 协议族出现问题时，kube-proxy 能更准确地报告其健康状况。（#118146，@aroradaman）

### Other (Cleanup or Flake)

* 通过允许设置比现有值更低的 sysctl 值，更改了 kube-proxy 的行为。（#120448，@aroradaman）
