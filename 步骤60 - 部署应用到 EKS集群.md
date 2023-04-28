# 步骤4 - 部署应用到 EKS



上个步骤3 中，我们已经部署了 AWS Load Balancer Controller，在本节实验中，我们将实现 Kubernetes Service 和 Ingress 资源与 AWS ALB/NLB 的集成。



## 选项1：Kubernetes Ingress 与 ALB 集成 (AWS的7层负载均衡器)

我们从创建一个 Kubernetes Ingress 资源开始，通过以下命令下载 2048 游戏 yaml 文件

```bash
curl -o 2048_full.yaml https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.7/docs/examples/2048/2048_full.yaml
```

该文件里创建了一个 game-2048 namespace，并在这个 namespace 里创建了 deployment-2048，service-2048 和 ingress-2048。 

其中 ingress 资源的配置如下，**ingressClassName 为 alb**，将创建一个 internet-facing 的 IP 模式的 ALB。

有关更多的 基于 ALB 的 Ingress 的使用和配置，可参考 https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/guide/ingress/annotations/

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  namespace: game-2048
  name: ingress-2048
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: service-2048
              port:
                number: 80

```



部署 2048-game 的 yaml 文件

```bash
kubectl apply -f 2048_full.yaml
```

输出示例如下：
```bash
$ kubectl apply -f 2048_full.yaml 
namespace/game-2048 created
deployment.apps/deployment-2048 created
service/service-2048 created
ingress.networking.k8s.io/ingress-2048 created

```



查看 Ingress 资源状态

```bash
$ kubectl describe ing -n game-2048
Name:             ingress-2048
Namespace:        game-2048
Address:          k8s-game2048-ingress2-bc39cec2ee-1451929837.cn-northwest-1.elb.amazonaws.com.cn
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host        Path  Backends
  ----        ----  --------
  *           
              /   service-2048:80 (10.201.109.72:80,10.201.113.69:80,10.201.117.165:80 + 2 more...)
Annotations:  alb.ingress.kubernetes.io/scheme: internet-facing
              alb.ingress.kubernetes.io/target-type: ip
Events:       <none>

```



查看 Pod 运行情况，可以看到 Pod 全部调度到了 Fargate 节点上

```bash
$ kubectl get po -n game-2048 -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP               NODE                                               NOMINATED NODE   READINESS GATES
deployment-2048-5686bb4958-8pqkr   1/1     Running   0          10h   10.201.117.165   ip-10-201-116-71.cn-northwest-1.compute.internal   <none>           <none>
deployment-2048-5686bb4958-8zc2g   1/1     Running   0          10h   10.201.23.239    ip-10-201-11-227.cn-northwest-1.compute.internal   <none>           <none>
deployment-2048-5686bb4958-v2jvb   1/1     Running   0          10h   10.201.109.72    ip-10-201-116-71.cn-northwest-1.compute.internal   <none>           <none>
deployment-2048-5686bb4958-xj6xv   1/1     Running   0          10h   10.201.5.118     ip-10-201-11-227.cn-northwest-1.compute.internal   <none>           <none>
deployment-2048-5686bb4958-zzc9j   1/1     Running   0          10h   10.201.113.69    ip-10-201-116-71.cn-northwest-1.compute.internal   <none>           <none>
[ec2-user@ip-10-203-0-177 temp1]$ 

```



在 AWS Console 上也可以查看 AWS Load Balancer Controller 创建的 ALB 和 目标组 信息，可以看到目标 Pod 以 IP 模式挂载到 ALB 后端，目标 IP 地址即是 service-2048 的 Backends Endpoint IP。

![image-20230410121536764](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230410121536764.png)

![image-20230410121611743](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230410121611743.png)



通过 Ingress 对应的 ALB 地址访问 service-2048，在浏览器打开 ALB URL：

http://k8s-game2048-ingress2-bc39cec2ee-1451929837.cn-northwest-1.elb.amazonaws.com.cn/

![image-20230410121710286](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230410121710286.png)

>清理

```bash
kubectl delete -f 2048_full.yaml
```











## 选项2：Kubernetes Service 与 NLB 模式 (AWS的4层负载均衡器)

- 参考 nginx-nlb.yaml, 创建一个nginx pod，并通过LoadBalancer类型对外暴露 

- 特别提醒80/443 在AWS China Region需要完成备案流程，请联系你的商业经理确保已开通，或者自行更改nginx-nlb.yaml的端口

  

  我们创建一个简单的 Nginx Service，指定使用 nlb 模式。首先创建 yaml 文件如下

```
cat << EOF > nginx-nlb.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: "service-nginx"
  annotations:
        service.beta.kubernetes.io/aws-load-balancer-type: nlb
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    
EOF

```

部署 Service，该 Service 会部署到默认的 default namespace。

```bash
kubectl apply -f nginx-nlb.yaml
```



查看 Service 和 NLB 的创建情况，以及 Pod 是否挂载

```bash
## Check deployment status
kubectl get pods
kubectl get deployment nginx-deployment 

## Get the external access 确保 EXTERNAL-IP是一个有效的AWS Network Load Balancer的地址
kubectl get service service-nginx -o wide 

## 下面可以试着访问这个地址
ELB=$(kubectl get service service-nginx -o json | jq -r '.status.loadBalancer.ingress[].hostname')
curl -m3 -v $ELB

```

查看结果是否一致

```
[ec2-user@ip-10-203-0-177 workspace]$ curl -m3 -v $ELB
*   Trying 161.189.90.53:80...
* Connected to ad04ec90ceede477d9985e5437940da0-8c94a65dd04d0a54.elb.cn-northwest-1.amazonaws.com.cn (161.189.90.53) port 80 (#0)
> GET / HTTP/1.1
> Host: ad04ec90ceede477d9985e5437940da0-8c94a65dd04d0a54.elb.cn-northwest-1.amazonaws.com.cn
> User-Agent: curl/7.79.1
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< Server: nginx/1.23.4
< Date: Thu, 06 Apr 2023 19:08:42 GMT
< Content-Type: text/html
< Content-Length: 615
< Last-Modified: Tue, 28 Mar 2023 15:01:54 GMT
< Connection: keep-alive
< ETag: "64230162-267"
< Accept-Ranges: bytes
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host ad04ec90ceede477d9985e5437940da0-8c94a65dd04d0a54.elb.cn-northwest-1.amazonaws.com.cn left intact

```



查看 Service 和 NLB 的创建情况，以及 Pod 是否挂载

在 AWS Console 上，查看 NLB 后面挂载的目标组，为 INSTANCE实例 模式；

![image-20230428110356240](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230428110356240.png)



>清理

```bash
kubectl delete -f nginx-nlb.yaml 
```





### 本章节完成
