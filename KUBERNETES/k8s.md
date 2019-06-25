# Kubernetes
## 命令 
### 进入某一个容器内部
```
kubectl exec -it ${pod_name} -n ${namespace} -- /bin/bash
eg: kubectl exec -it ins-order-545f988d7d-5bq88 -n jtpf-dev -- /bin/bash
```
### 查看某一个命名空间下的所有的pod
```
kubectl get po -n ${namespace}
eg: kubectl get po -n sequip-dev -o wide
```

### 查看某一个详情
```
kubectl describe po ${pod_name} -n ${namespace}
eg: kubectl describe po ins-batch -n sequip-dev
```
### 查看某一个node的详细信息
```
kubectl describe node ${node_name}
```

### 删除pod
```
kubectl delete po ${pod_name} -n ${namespace} --grace-period=0 --force
eg: kubectl delete po ins-batch-5787bccb97-74fjg -n jtpf-dev --grace-period=0 --force
```

### 删除所有驱赶失败的pod
```
kubectl get po  -n ${namespace} | grep Evicted | awk '{print $1}' | xargs kubectl delete po -n ${namespace}
eg: kubectl get po  -n jtpf-pre | grep Evicted | awk '{print $1}' | xargs kubectl delete po -n jtpf-pre --grace-period=0 --force
```

### 查看所有pod的网络IP是多少
```
命令逻辑: 获取所有的pod名字, 执行describe命令, 再过滤IP关键字
eg: kubectl get po  -n jtpf-dev | awk '{print $1}' | grep '-' | xargs kubectl describe po -n jtpf-dev | grep 'IP'
```

### 检查容器中的应用是否健康
```
kubectl exec -t ${another_pod} -- curl -I ${pod's cluster IP}
eg: kubectl exec -t ins-svc-95b5879fd-d5t9p -- curl -I 10.244.9.142
```


## 问题
### 容器不停重启
- 现象: 容器不停重启 
- 容器状态, Running -> OOMKilled -> CrashLoopBackOff
- 最后定位原因: jvm没有OOM, 是容器OOM, jvm设置800M, pod设置的limits是1024Mi, 容器中除jvm以外需要的内存超过200M, 导致容器OOM.

### 只启动一个容器, 但是swagger会有两种结果
- 现象: 多次请求, 返回的swagger页面的结果不一样
- 最后定位原因: work1机器有问题, 虽然pod删除了, 但是eureka中的服务依然在, 而且还能访问. 联系运维, 机器重启了一下就好了. 



