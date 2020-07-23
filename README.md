# K8S-常见问题排查方法

1. 同一个服务多个pod，其中有一个pod状态异常。
   原因：升级新服务，但是新服务的pod出现异常，所以老的pod一直保留，防止服务挂了。只有新服务升级正常running了，老的pod会自动删掉。
   思路：解决pod异常
```
kubecetl get pods | grep {serviceName}
kubectl logs -f {podName}
```
2. 查找某个ip被哪个服务占用
```
kubectl get pods --all-namespaces -o wide | grep {ip}
```

3. pod 内部抓包
```
$ kubectl get pod mypod -o json
>>> "containerID": "docker://ddaaad0f556d2b1e5d4298bcc22c1701ff15e82c7a335b340334d852abe9af2e",
>>> "hostIP": "10.193.90.92"

$ docker exec ddaaad0f556d2b1e5d4298bcc22c1701ff15e82c7a335b340334d852abe9af2e /bin/bash -c 'cat /sys/class/net/eth0/iflink'
>>> 13

$ for i in /sys/class/net/veth*/ifindex; do grep -l 13 $i; done
>>> /sys/class/net/veth235ab8ff/ifindex

$ tcpdump -i veth235ab8ff -w filename.pcap

source link: https://community.pivotal.io/s/article/How-to-get-tcpdump-for-containers-inside-Kubernetes-pods
```

4. 用sed替换文件内容时需要引用变量
```
sed -ir 's/^\s*"username".*/ "username": "'$userName'",/' config.json 

其中-r表示需要传入文件
-r 表示用正则表达式
以'$userName'引用变量，示例中由于要把引用得到的内容用双引号括起来
\s 表示空格
^ 表示以其后所跟的内容开头

参考链接：https://stackoverflow.com/questions/16965274/regular-expression-to-find-spaces
                https://www.grymoire.com/Unix/Sed.html#uh-4a
```

5. 服务配置文件
```
kubectl describe cm {serviceName}-config
```

6. 获取服务版本
```
kubectl describe pods --all-namespaces | grep Image: | uniq | awk '{print $2}' | awk -F '/' '{print $3}'| sort
helm list | awk '{print $9}'
```

7. 获取异常容器
```
kubectl get pods -n kube-system | grep -v Running
```

8. 删除异常pod
```
kubectl delete pod {podName} -n {namespace}
```

9. 所有pod的image版本号
```
for file in `kubectl get pods --all-namespaces -o jsonpath="{.items[*].spec.containers[*].image}"`;do echo $file;done | sort | uniq
```
10. 查看某个pod的yaml
```
kubectl get pod -n default xxxx -o yaml
```

11. kubectl命令dashboard
   Rancher
   
12. request 所调度的节点上至少应该分配到的cpu和内存资源；limits 真正能使用的资源

13. 查看节点
```
kubectl get nodes
```

14. 查看节点详情, 查看cpu、内存使用情况，该节点上有哪些pod
```
kuebctl decribe node xxxx
```

15. 从pod拷贝文件
```
kubectl cp pod:dirname.log  out.log
```

15. 进入交互方式
```
kubectl exec -it -n default xxx bash
```

16. 当前pod的资源使用情况
```
kubectl top -n xxxx
```

17. 查看资源字段具体含义
```
kubectl expain deployments.spec.minReadySeconds
```

18. 重点说明
```
179854568
一个pod可以包含多个container，共享网络，ip、端口
ip连通性说明：
域名只在k8s pod内联通
ingress通过node port暴露服务
内部访问路径：pod -> service name -> service ip(optional) -> pod ip
外部访问路径: ingress port + path -> pod ip
deployment时无状态服务，不需要持久化数据，statefulsets适用于有状态的服务，需要持久化数据到磁盘，daemonsets适用于每个node的一个pod的无状态服务的情况
```

19. 上次启动为何报错
```
kubectl logs xxx -p
```

20. Grafna  监控内存、CPU、网络、
   内存曲线不波动，可能是分配不合理（分配比较多）；服务不是内存型的(java)
   和top性能命令行的差异，指标选择的差异(container_memory_working_set_bytes)
