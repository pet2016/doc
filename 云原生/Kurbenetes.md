

#### 有状态服务statefulset

1. 稳定性，唯一的网络标识
2. 稳定性，持久化存储
3. 有序的部署和扩展
4. 有序的删除和终止
5. 有序的自动回滚和更新

Service是K8s中用于定义服务的对象，它为一组Pod提供一个稳定的网络入口，通过标签选择器将流量引导到这些Pod。Service的IP地址和端口是稳定的，其他服务可以通过该IP地址和端口访问服务

Endpoint是Service背后真实运行应用程序的Pod的地址和端口的集合

K8s内置了一个DNS服务，允许在集群内使用域名进行服务发现。Service的名称将映射到DNS中，从而允许其他服务使用该域名来访问服务。例如，在一个Pod中，可以通过backend-service.default.svc.cluster.local来访问上述定义的backend-service


Kubernetes的服务发现机制工作原理如下：

Pod注册： 当Pod启动时，它会向K8s API服务器注册自己的IP地址和端口号。
Service创建： 创建一个Service对象时，K8s会为该服务分配一个Cluster IP，并为其创建一个DNS记录。
Endpoint更新： K8s通过Label Selector将Service与匹配的Pod关联起来，并更新相应的Endpoints对象。
DNS解析： 其他Pod可以通过Service名称或Endpoint的DNS记录来解析服务的IP地址。

Kubernetes服务发现机制带来了多重优势：

弹性和动态扩展： 服务发现使得新的Pod能够动态地加入或离开服务，而其他服务无需修改配置即可感知这些变化。
解耦服务： 通过Service对象，服务之间的通信不再依赖于具体的IP地址和端口号，而是通过Service名称和DNS解析进行，提高了服务的解耦性。
负载均衡： Service对象自动提供了负载均衡，将流量分发到后端Pod。这有助于确保各个Pod能够均匀地处理请求。
DNS解析： Kubernetes内置了DNS服务，使得在集群内部使用域名进行服务发现变得十分方便。

etcd 是 Kubernetes 中至
关重要的组件，承担了 配置中心 和 注册中心 的双重角色。通过其强一致性和高可用性，etcd 能够确保 Kubernetes 集群中的资源信息始终是最新且可靠的。Kubernetes 的各个组件通过 API Server 使用 watch 机制 实时监控资源的变化，确保集群中的所有资源能够及时响应和动态管理。

关键点回顾：
etcd 数据结构：etcd 使用层次化的键值对结构来存储 Kubernetes 集群中的资源和配置信息，确保数据的高效组织和查询。
watch 机制：Kubernetes 组件通过 API Server 发起 watch 请求来监听资源状态的变化，API Server 通过流式传输实时推送变更事件。
高可用性与故障恢复：etcd 通过 Raft 共识算法和领导者选举机制保证数据的一致性和高可用性，支持多节点复制和自动故障恢复。
