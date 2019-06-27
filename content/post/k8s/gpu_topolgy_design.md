---
title: "Gpu Topolgy Scheduler Design"
date: 2019-06-27T21:00:04+08:00
draft: false
---

gsoc 设计文档更新版。

## device plugin

针对 device 

```
// NvidiaDevicePlugin implements the Kubernetes device plugin API
type NvidiaDevicePlugin struct {
	devs        []*pluginapi.Device
	realDevNames []string
	devNameMap   map[string]uint
	devIndxMap   map[uint]string
	socket      string
	gpuTopology gpuTopology

	stop   chan interface{}
	health chan *pluginapi.Device

	server *grpc.Server
}

// gpuTopologyType
type gpuTopologyType nvml.P2PLinkType

// gpuTopology
type gpuTopology map[uint]map[uint]gpuTopologyType
```

device-plugin 启动的时候将 gpu topology 写入node 的 annotation里
```
node, err := clientset.CoreV1().Nodes().Get(nodeName, metav1.GetOptions{})

if err != nil {
    return err
}

newNode := node.DeepCopy()
for gpu1, temp := range topology {
    for gpu2, topo := range temp {
        envGsocGpuTopology := ENV_GOSC_GPU_TOPOLOGY_PRIFIX + fmt.Sprintf("_%d",gpu1 )+ fmt.Sprintf("_%d",gpu2)
        newNode.ObjectMeta.Annotations[envGsocGpuTopology] = fmt.Sprint(uint(topo))
    }
}
_, err = clientset.CoreV1().Nodes().Update(newNode)

```

## 针对 scheduler-extender 的设计

### scheuler 的配置文件修改

管理的资源类型改为 aliyun.com/gpu-count

```
{
  "kind": "Policy",
  "apiVersion": "v1",
  "extenders": [
    {
      "urlPrefix": "http://127.0.0.1:32766/gpushare-scheduler",
      "filterVerb": "filter",
      "bindVerb":   "bind",
      "enableHttps": false,
      "nodeCacheCapable": true,
      "managedResources": [
        {
          "name": "aliyun.com/gpu-count",
          "ignoredByScheduler": false
        }
      ],
      "ignorable": false
    }
  ]
}
```

### 针对 deviceInfo

```
type DeviceInfo struct {
	idx    int
	podMap map[types.UID]*v1.Pod
	rwmu   *sync.RWMutex
	isUsed bool
}
```

### 关于 gpu 的topology 边

```
package cache

// gpu edge single gpu is not include
type Edge struct {
	gpu1     uint
	gpu2     uint
	distance uint
}

type Edges []*Edge

func (e Edges) Len() int {
	return len(e)
}

func (e Edges) Less(i, j int) bool {
	return e[i].distance < e[j].distance
}

func (e Edges) Swap(i, j int) {
	e[i], e[j] = e[j], e[i]
}
```

### node info

```
func NewNodeInfo(node *v1.Node) *NodeInfo {
	log.Printf("debug: NewNodeInfo() creates nodeInfo for %s", node.Name)

	devMap := map[int]*DeviceInfo{}
	for i := 0; i < utils.GetGPUCountInNode(node); i++ {
		devMap[i] = newDeviceInfo(i) // 这里的i 表示什么？device id 吗？
	}

	gpuTopology := utils.GetGPUTopologyInNode(node)

	gpuEdges := Edges{}
	totalGpu := len(devMap)

	if totalGpu >= 2 {
		for i := 0; i < totalGpu; i++ {
			for j := i + 1; j < totalGpu; j++ {
				gpuEdges = append(gpuEdges, &Edge{uint(i), uint(j), gpuTopology[uint(i)][uint(j)]})
			}
		}
	}

	sort.Sort(gpuEdges)

	nodeInfo := &NodeInfo{
		name:        node.Name,
		node:        node,
		devs:        devMap,
		gpuCount:    utils.GetGPUCountInNode(node),
		gpuTopology: gpuTopology,
		gpuEdges:    gpuEdges,
		rwmu:        new(sync.RWMutex),
	}

	log.Printf("debug: node %s has nodeinfo %v", node.Name, nodeInfo)
	return nodeInfo
}
```

### 针对 gpu topology 返回合适的 device list 
```
// 根据 gpu topology 返回 device list
func (n *NodeInfo) prim(pod *v1.Pod, req int) (ids []uint, found bool) {
	found = false
	if req <= 0 || req < n.getAvailableGPUs() {
		return
	}

	// req == 1, 随机返回device id
	if req == 1 {
		for _, dev := range n.devs {
			if dev.isUsed == false {
				ids = append(ids, uint(dev.idx))
				found = true
				return
			}
		}
	}

	ids, ok := n.getUnusedShortestTwoDevices()

	if !ok {
		return
	}

	if req == 2 {
		found = true
		return
	}

	// 寻找接下来的点

	// 顶点到集合ids的最短距离
	d := []int{}
	for i := 0; i < len(n.devs); i++ {
		d = append(d, 100)
	}

	d[ids[0]] = 0
	n.devs[int(ids[0])].isUsed = true
	d[ids[1]] = 0
	n.devs[int(ids[0])].isUsed = true

	// 循环 req - 2此
	for c := 2; c <= req; c++ {
		u := -1 // u使得d[u]最小
		min := 100
		for i := 0; i < len(n.devs); i++ {
			if n.devs[i].isUsed == false && d[i] < min {
				u = i
				min = d[i]
			}

		}
		if u == -1 { // 剩下的点和集合s不连通
			n.devs[int(ids[0])].isUsed = false
			n.devs[int(ids[0])].isUsed = false
			return
		}
		n.devs[u].isUsed = true
		ids = append(ids, uint(u))

		// 更新接下来的点到集合到最短距离
		for v := 0; v < len(n.devs); v++ {
			if n.devs[v].isUsed == false && int(n.gpuTopology[uint(u)][uint(v)]) < d[v] {
				d[v] = int(n.gpuTopology[uint(u)][uint(v)])
			}
		}
	}

	found = true
	return
}

// get Unused Shortest TwoDevices
func (n *NodeInfo) getUnusedShortestTwoDevices() (ids []uint, isShortestDistance bool) {
	isShortestDistance = false
	for i := 0; i < len(n.gpuEdges); i++ {
		if n.devs[int(n.gpuEdges[i].gpu1)].isUsed == false && n.devs[int(n.gpuEdges[i].gpu2)].isUsed == false {
			ids = []uint{n.gpuEdges[i].gpu1, n.gpuEdges[i].gpu2}
			isShortestDistance = true
			break
		}
	}
	return
}
```

### 给  pod 的 annotation 上标注分配的device 信息
```
EnvResourceIndex      = "ALIYUN_COM_GPU_IDX"       // 在 annotation 标记使用哪些gpuid 格式 1,2,4 or 2
EnvResourceByPod      = "ALIYUN_COM_GPU_COUNT_POD" // 在 annotation 标记这个pod 使用了几个 gpu  格式: 1 or 0 or 4
EnvAssignedFlag       = "ALIYUN_COM_GPU_ASSIGNED"
EnvResourceAssumeTime = "ALIYUN_COM_GPU_ASSUME_TIME"
```
### 将 node 上 annotion 转为 gpu topology 结构

// key 表示 设备的id /dev/nvidia%d
func GetGPUTopologyInNode(node *v1.Node) map[uint]map[uint]uint {
	topology := make(map[uint]map[uint]uint)
	if !IsGPUTopologyNode(node) {
		return topology
	}

	if !ok {
		return int(0)
	for k, v := range node.Annotations {
		if strings.HasPrefix(k, "GSOC_DEV_") {
			var gpu1, gpu2, topo uint
			fmt.Sscanf(k, "GSOC_DEV_%d_%d", &gpu1, &gpu2)
			fmt.Sscanf(v, "%d", &topo)
			topology[gpu1] = map[uint]uint{gpu2: topo}
		}
	}

	return int(val.Value())
	return topology
}


### 针对训练任务使用不同 device 分配

这个问题暂时还没有搞清楚。