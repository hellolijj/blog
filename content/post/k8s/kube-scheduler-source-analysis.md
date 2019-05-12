---
title: "Kube Scheduler Source Analysis"
date: 2019-05-12T14:13:04+08:00
draft: false
---

kube-scheduler 源码分析

网上有很多 kubernetes 源码分析的博客。但是，我的源码分析记录我的思考和跟别人不一样的地方。

对于很多人来说，对于 kube-scheduler 来说，很多人只能说出预选和优选。包括我在面试的时候，也就说到这个层面，面试官说ok了。但是，对于一个开发者来说，kube-scheduler 的功能远不止于此。

参考文档：https://farmer-hutao.github.io/k8s-source-code-analysis/core/scheduler/before-scheduler-run.html

1、kube-scheduler 选举

kube-scheduler 启动的时候有一个参数 `--leader-elect` 这说明 kube-scheduler 的启动也有选举操作。关于选举，后期有时间再展开介绍。

2、schedluer 的结构体定义

关于scheduler的定义和解释如注视：Scheduler watch新创建的未被调度的pods，然后尝试寻找合适的node，回写一个绑定关系到api server.

```golang
// Scheduler watches for new unscheduled pods. It attempts to find
// nodes that they fit on and writes bindings back to the api server.
type Scheduler struct {
	config *factory.Config
}
```
这里关于 factory.Config 的详细结构，以后有时间展开介绍。

3、wait.Until() 每隔一段时间执行一次。

从`cmd/kube-scheduler/app/server.go#Run`里的 sched.Run 函数 到 	`pke/scheduler/scheduler.go#Run` 文件，创建了一个携程。就是启动一个协程，每隔一定的时间，就去运行声明的匿名函数，直到接收到结束信号 就关闭这个协程。
```golang
run := func(ctx context.Context) {
	sched.Run()
	<-ctx.Done()
}

func (sched *Scheduler) Run() {
	if !sched.config.WaitForCacheSync() {
		return
	}

	go wait.Until(sched.scheduleOne, 0, sched.config.StopEverything)
}
```

4、scheduleOne 串行化调度

查看 scheduleOne 函数，说明pod节点一个一个调度，调度完成后是将 pod node 的绑定关系 传给 api-server。scheduleOne 又是每隔一段时间运行一次。

在 scheduleOne 先获取 nextPod，然后对nextPod 判断。 DeletionTimestamp 是pod删除的时间搓。
```golang
if pod.DeletionTimestamp != nil {
	sched.config.Recorder.Eventf(pod, v1.EventTypeWarning, "FailedScheduling", "skip schedule deleting pod: %v/%v", pod.Namespace, pod.Name)
	klog.V(3).Infof("Skip schedule deleting pod: %v/%v", pod.Namespace, pod.Name)
	return
}
```
> 虽然这不是主干内容，但是我就是喜欢分析这些旁枝末节的东西。因为主干的东西，我看了，知道了，已经有人做过了。我就不做这件事情了。

5、进入调度过程

scheduler 的调度函数如下: 传入参数为 pod 对象。返回值为 ScheduleResult 对象。

```golang
func (sched *Scheduler) schedule(pod *v1.Pod) (core.ScheduleResult, error) {
	result, err := sched.config.Algorithm.Schedule(pod, sched.config.NodeLister)
	if err != nil {
		pod = pod.DeepCopy()
		sched.recordSchedulingFailure(pod, err, v1.PodReasonUnschedulable, err.Error())
		return core.ScheduleResult{}, err
	}
	return result, err
}
```

调度返回结构体
```golang
type ScheduleResult struct {
	// Name of the scheduler suggest host
	SuggestedHost string
	// Number of nodes scheduler evaluated on one pod scheduled
	EvaluatedNodes int
	// Number of feasible nodes on one pod scheduled
	FeasibleNodes int
}
```

sched.config.Algorithm.Schedule 是接口 ScheduleAlgorithm 的一个方法。scheduler接口如下
```golang
type ScheduleAlgorithm interface {
	Schedule(*v1.Pod, algorithm.NodeLister) (scheduleResult ScheduleResult, err error)
	Preempt(*v1.Pod, algorithm.NodeLister, error) (selectedNode *v1.Node, preemptedPods []*v1.Pod, cleanupNominatedPods []*v1.Pod, err error)
	Predicates() map[string]predicates.FitPredicate
	Prioritizers() []priorities.PriorityConfig
}
```

- Schedule 方法是调度
- Preempt 方法是抢占
- Predicates 方法是预选
- Prioritizers 方法是优选

对于接口的实现，位于`pkg/scheduler/core/generic_scheduler.go`中的 genericScheduler 结构体。


6、预选过程

genericScheduler.Schedule 函数简单的参数判断后，进入到预选过程。

```golang
filteredNodes, failedPredicateMap, err := g.findNodesThatFit(pod, nodes)
```
在 findNodesThatFit函数 中


预选定义了 check 的 匿名函数。通过 workqueue.ParallelizeUntil 函数并行计算 pod fit nodes 的过程。

```
workqueue.ParallelizeUntil(ctx, 16, len(allNodes), checkNode)
```

对于每一个 checkNode 函数，着重关注里面的 podFitsOnNode 函数。podFitsOnNode 函数里通过 predicate(pod, metaToUse, nodeInfoToUse) 函数来进行判断条件是否合法。

predicate 方法 位于 `pkg/scheduler/algorithm/predicates/predicates.go` 文件。

这里有系统预定义的各种判断条件。

7、优选过程



