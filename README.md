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

