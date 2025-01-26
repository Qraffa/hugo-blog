---
title: "Raft篇-1"
date: 2025-01-27T00:46:50+08:00
url: post/2025/01/27/raftexample-1
tags:
- raft
categories:
- raft
---

# Raft篇-1

raft库只实现核心的raft算法部分，使用raft库时，用户需要自己实现其中的网络传输以及存储，以分别支持raft节点内部的通信和raft log的持久化等。

raftexample是etcd库中一个使用raft库的例子，其中实现了上述所需的网络传输和存储能力，以及提供了http服务给外部调用，包括读写数据等。

从raftexample的实现来看etcd是怎么设计和实现raft协议的。

### 启动结构

````go
newRaftNode(*id, strings.Split(*cluster, ","), *join, getSnapshot, proposeC, confChangeC)

newKVStore(<-snapshotterReady, proposeC, commitC, errorC)

serveHttpKVAPI(kvs, *kvport, confChangeC, errorC)
````

- newRaftNode：启动raft node节点，raft算法核心实现部分
- newKVStore：每个节点的存储实现
- serveHttpKVAPI：提供给外部的http接口，提供读写数据，增删节点能力（这个不是raft node节点之间的通信实现）

### 写入数据

首先在raftexample文档中可以看到写入数据是通过`PUT`请求实现的

`````bash
curl -L http://127.0.0.1:12380/my-key -XPUT -d foo
`````

PUT实现如下：

```go 
// contrib/raftexample/httpapi.go#L36
case r.Method == "PUT":
  h.store.Propose(key, string(v))

// Propose函数实现
// contrib/raftexample/kvstore.go#L70
s.proposeC <- buf.String()
```

最终是往 kvstore 的 proposeC channel中写入了数据，这个proposeC就是在启动时同时给raft node和kvstore用于初始化的channel。因此粗略得到如下关系图

![](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/raft/ectd_1_0.svg)

处理proposeC的流程

```go
// contrib/raftexample/raft.go#L402
func (rc *raftNode) serveChannels() {
	// ...
  for rc.proposeC != nil && rc.confChangeC != nil {
    case prop, ok := <-rc.proposeC:
          // contrib/raftexample/raft.go#L427
          rc.node.Propose(context.TODO(), []byte(prop))
```

`serveChannels`函数是在初始化raftNode时创建goroutine执行的；在这里最后是调用raft库的node的Propose函数，这里的具体实现往后在分析。

走到此处，我们已经串联了一个写入请求从 外部http请求 -> raftexample -> raft库。

### raft逻辑时钟

raft库的文档中写到使用库时需要定时调用`Node.Tick()`。

在raftexample中，实现如下：

```go
func (rc *raftNode) serveChannels() {
  // ...
  ticker := time.NewTicker(100 * time.Millisecond)
	// ...
  for {
      select {
      case <-ticker.C:
        rc.node.Tick()
```

设置一个定时器，每100ms调用一次`Node.Tick()`

在`Tick`函数实现中，实际是往node结构体的tickc channel中写入

```go
func (n *node) Tick() {
	select {
	case n.tickc <- struct{}{}:
```

然后再有node的主循环监听，并调用rawNode的Tick方法

```go
func (n *node) run() {
  // node.go#L433
	case <-n.tickc:
			n.rn.Tick()
```

在rawNode的Tick方法中，最终会调用到各节点的`tick`函数，该函数在节点不同状态时设置的不一样，比如leader节点的tick函数是`tickHeartbeat`，follower节点的tick函数是`tickElection`

简化中间channel流程的话，相当于是由Ticker定时起驱动，定时调用各个节点的tick函数。

![ectd_1_1](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/raft/ectd_1_1.svg)

### raftNode通信

使用raft库时，需要自己实现网络传输层能力，这里通过leader节点的心跳请求来分析下raftexample的网络传输是如何实现给raft库的。

上面已经了解了raft的逻辑时钟推进过程，接下来从leader节点视角来看，是如何发送心跳请求，以及raftexample是如何实现网络层传输；以及从follower节点视角来看，是如何处理网络请求的。

#### Leader节点

上述说到不同状态节点的tick函数各不相同，leader节点调用的是`tickHeartbeat`函数

```go
func (r *raft) becomeLeader() {
  // raft.go#L940
	r.tick = r.tickHeartbeat
```

关于raft算法leader节点心跳选举等过程，这里不展开，主要看如何处理网络请求

```go
func (r *raft) tickHeartbeat() {
  // raft.go#L883
	if r.heartbeatElapsed >= r.heartbeatTimeout {
		if err := r.Step(pb.Message{From: r.id, Type: pb.MsgBeat}); err != nil {
```

超过心跳超时后，这里会调用raft的`Step`函数

```go
// raft.go#L1085
func (r *raft) Step(m pb.Message) error {
  	// ...
  	// raft.go#L1257
		err := r.step(r, m)
```

看到`Step`函数，在中间我身略了一大部分，这部分是所有状态节点都会执行都通用逻辑，在最后部分，则会执行各自的`setp`函数，这个函数与上面的`tick`函数一样，不同状态节点的tick函数各不相同。

还是以leader节点，看看他的step函数，记住上面调用`Step`函数时，传递的消息类型是`MsgBeat`，所以这里我们先关注分析这个部分

```go
func (r *raft) becomeLeader() {
  // raft.go#L938
	r.step = stepLeader
}

// raft.go#L1267
func stepLeader(r *raft, m pb.Message) error {
	switch m.Type {
	case pb.MsgBeat:
		r.bcastHeartbeat()
```

最终实际是调用的`bcastHeartbeat`函数，这里有些实现细节不是此次分析关心的，我们先简单略过，这是大致的调用链路，中间我们先不关心，我们关注最后的`send`方法

```go
bcastHeartbeat -> bcastHeartbeatWithCtx -> sendHeartbeat -> r.send
```

在`send`方法中，我们此次传输的消息类型是`MsgBeat`，所以其他细节我们也先略过

```go
func (r *raft) send(m pb.Message) {
  // raft.go#L596
  r.msgs = append(r.msgs, m)
```

这里可以看到send函数实现中，只是将m添加到msgs中，即结束了；那么发送一定还在别的地方了。

在继续之前，我们又可以先更新一下我们的结构图

![ectd_1_2](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/raft/ectd_1_2.svg)

##### send msgs

回到raft库的文档中，提到使用时在创建raft node后还需要从`Node.Ready()`中读取数据，然后处理，其中包括发送其中的所有消息。

回到raftexample的实现中，在`serveChannels`这个协程中，实现了上述说的能力

```go
func (rc *raftNode) serveChannels() {
  case rd := <-rc.node.Ready():
  	// contrib/raftexample/raft.go#L459
  	rc.transport.Send(rd.Messages)
```

接下来来看Ready是什么，以及从Ready中读取到的是什么数据

```go
// node.go#L547
func (n *node) Ready() <-chan Ready { return n.readyc }
```

`Ready()`函数是返回node节点中的`readyc`这个channel

再进一步

```go
func (n *node) run() {
	var readyc chan Ready
	var rd Ready
	for {
		if advancec == nil && n.rn.HasReady() {
      // node.go#L363
			rd = n.rn.readyWithoutAccept()
			readyc = n.readyc
		}
  // ...
  // node.go#L435
  case readyc <- rd:
```

`readyc`中的数据是在raft node的主循环中构造和投递的

再进一步，看rd是如何构造的

```go
// rawnode.go#L140
func (rn *RawNode) readyWithoutAccept() Ready {
	r := rn.raft

	rd := Ready{
		Entries:          r.raftLog.nextUnstableEnts(),
		CommittedEntries: r.raftLog.nextCommittedEnts(rn.applyUnstableEntries()),
		Messages:         r.msgs,
	}
// ...
```

`readyWithoutAccept`函数是用于构造`readyc`中的数据，其中会将`raft.msgs`中写入到其中，也就是上面`send`函数写入到`msgs`中的数据

知道了消息数据从何而来，从何触发，接下来看raftexample是如何实现消息的发送的。

```go
// contrib/raftexample/raft.go#L459
rc.transport.Send(rd.Messages)

// server/etcdserver/api/rafthttp/transport.go#L192
p.send(m)

// server/etcdserver/api/rafthttp/peer.go#L245
func (p *peer) send(m raftpb.Message) {
writec, name := p.pick(m)
select {
case writec <- m:

// server/etcdserver/api/rafthttp/pipeline.go#L100
func (p *pipeline) handle() {
case m := <-p.msgc:
  start := time.Now()
  err := p.post(pbutil.MustMarshal(&m))
```

为了节省篇幅，只将关键流程标注，梳理上述流程看到，最终实际是通过http请求将消息发送给各个节点；也就是raftexample实现中还有一个http服务，是用于给raft node节点之间通信的。

最后总结下上述流程：

1. raft写入消息，实际是写入到节点内的`msgs`中
2. 主循环构造需要待发送的消息写入到`readyc`中
3. 外部实现轮训`readyc`读取其中的数据，将消息通过transport模块发送
4. transport模块实际是http服务，用于raft node节点之间通信

![ectd_1_3](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/raft/ectd_1_3.svg)

#### Follower节点

上述流程我们已经从Leader节点将消息发送到trasport，并通过http请求发送给其他各节点；接下来从Follower节点来看消息是如何被接收和处理的。

来到raftexample实现中，此处就是启动transport http服务的部分

```go
// contrib/raftexample/raft.go#L325
go rc.serveRaft()

// server/etcdserver/api/rafthttp/transport.go#L158
func (t *Transport) Handler() http.Handler {
	pipelineHandler := newPipelineHandler(t, t.Raft, t.ClusterID)
```

在此实现中注册了若干handler，此处我们先只关注`pipelineHandler`

```go
func (h *pipelineHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
  // server/etcdserver/api/rafthttp/http.go#L140
	if err := h.r.Process(context.TODO(), m); err != nil {
```

此处和我们启动传统http服务类似，接收消息然后处理等，这里的关键是调用了raftNode的`Process`函数

```go
// contrib/raftexample/raft.go#L499
func (rc *raftNode) Process(ctx context.Context, m raftpb.Message) error {
	return rc.node.Step(ctx, m)
}
```

在Process函数中，最终会调用到`node.Step()`函数

```go
// node.go#L473
func (n *node) Step(ctx context.Context, m pb.Message) error {
	return n.step(ctx, m)
}

// node.go#L498
func (n *node) step(ctx context.Context, m pb.Message) error {
	return n.stepWithWaitOption(ctx, m, false)
}

// node.go#L508
func (n *node) stepWithWaitOption(ctx context.Context, m pb.Message, wait bool) error {
  // node.go#L519
  ch := n.propc
	pm := msgWithResult{m: m}
	select {
	case ch <- pm:
    if !wait {
			return nil
		}
```

为了省略篇幅，列举了核心流程，最终会将消息发给`node.propc`，此处我们是Follower视角，因此先不考虑MsgProp类型消息，以及此时`wait=false`，即我们将消息写入到propc后，直接返回

再来看怎么处理`propc`中的消息，来到node的主循环中，

```go
func (n *node) run() {
  if lead != r.lead {
				propc = n.propc
  }
 	select {
		case pm := <-propc:
			err := r.Step(m)
```

此处我们在主循环中，从propc中读取数据并且调用`raft.Step()`函数处理，后续关于Step的处理就与上面说的Leader节点处理心跳时的流程一样了。

因此总结下：

1. transport 启动http服务，监听来自其他节点的请求消息
2. 处理http请求，并将消息投递给`node.propc`
3. 在主循环中读取`node.propc`，并调用`raft.Step()`函数，执行真正的函数处理逻辑

![ectd_1_4](https://qraffa-1304595678.cos.ap-guangzhou.myqcloud.com/img/raft/ectd_1_4.svg)

### 总结

至此，我们粗略的了解了raftexample是如何使用raft库的，包括如何向raft写入数据，如何触发raft的逻辑时钟推进，以及如何实现网络层，以实现raft节点之间的通信。