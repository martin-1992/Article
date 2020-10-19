## Raft-Notes
 
### [leader 选举](https://github.com/martin-1992/Article/tree/main/Raft-Notes/leader%20%E9%80%89%E4%B8%BE)
　　Raft 使用心跳机制来触发 leader 选举的。Raft 有两种类性的 RPC，请求投票 RPC 和附加日志 RPC，心跳包属于空内容的附加日志 RPC。

![avatar](./leader%20选举/photo_1.png)

![avatar](./leader%20选举/photo_2.png)

### [日志复制](https://github.com/martin-1992/Article/tree/main/Raft-Notes/%E6%97%A5%E5%BF%97%E5%A4%8D%E5%88%B6)
　　分布式的目的，就是将日志（消息）同步到其它服务节点（副本）。保证 leader 挂了，也不会丢失数据，其它从服务器成为 leader 后还能正常提供服务。Raft 的 leader 选举机制，就是选出合适的 leader，继续执行日志复制。

![avatar](./日志复制/photo_1.png)
