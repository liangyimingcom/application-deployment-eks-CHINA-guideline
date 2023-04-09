

# 创建 EKS 集群



在创建 EKS 集群之前，我们还需要安装 eksctl 和 kubectl 工具。



## 安装 kubectl

```bash
sudo curl --silent --location -o /usr/local/bin/kubectl \
  https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl
```

参考文档：https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html

验证 kubectl 安装结果：

```bash
kubectl version --client
```



## 安装 eksctl

[eksctl](https://eksctl.io/) 是 Amazon EKS 的官方管理工具, Go 语言实现, 底层通过 CloudFormation 对 AWS 资源进行管理。

安装命令:

```bash
curl --silent --location "https://kgithub.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin
```

参考文档：https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html

验证 eksctl 安装结果:

```bash
eksctl version
```



## 通过 eksctl 创建 EKS 集群（指定的VPC下创建集群）

下面的命令会创建一个名字为 **mobile-app** 的 EKS 集群, 这个 EKS 集群会创建一个默认，同时也会创建 EC2 工作节点。

**由于是在指定的VPC下创建集群，避免了VPC子网没有正确打标签（tag）的问题。**后续创建的 pod 任务会运行在这个新创建的 VPC 中。



在Cloud9上传“eks_cluster_zhy_v125.yaml”后运行，创建生产环境的  EKS：

```bash
eksctl create cluster -f eks_cluster_zhy_v125.yaml
```



整个 EKS 集群创建过程大约 15 分钟, 输出信息如下图所示:

```bash
[ec2-user@ip-10-203-0-177 workspace]$ eksctl create cluster -f ./eks_cluster_zhy_v125.yaml
2023-04-06 17:43:09 [ℹ]  eksctl version 0.136.0
2023-04-06 17:43:09 [ℹ]  using region cn-northwest-1
2023-04-06 17:43:09 [✔]  using existing VPC (vpc-056f83f8db816668c) and subnets (private:map[private-one:{subnet-0b3fef6c1bb8b5306 cn-northwest-1a 10.201.0.0/19 0 } private-two:{subnet-0810e50caed049bfb cn-northwest-1b 10.201.96.0/19 0 }] public:map[public-one:{subnet-0998e8598dc0a8579 cn-northwest-1a 10.201.32.0/19 0 } public-two:{subnet-0ac31567324699565 cn-northwest-1b 10.201.64.0/19 0 }])
2023-04-06 17:43:09 [!]  custom VPC/subnets will be used; if resulting cluster doesn't function as expected, make sure to review the configuration of VPC/subnets
2023-04-06 17:43:09 [ℹ]  nodegroup "sprintcloud-gateway-nodegroups" will use "" [AmazonLinux2/1.25]
2023-04-06 17:43:09 [ℹ]  nodegroup "app-nodegroups" will use "" [AmazonLinux2/1.25]
2023-04-06 17:43:09 [ℹ]  using Kubernetes version 1.25
2023-04-06 17:43:09 [ℹ]  creating EKS cluster "mobile-app" in "cn-northwest-1" region with managed nodes
2023-04-06 17:43:09 [ℹ]  2 nodegroups (app-nodegroups, sprintcloud-gateway-nodegroups) were included (based on the include/exclude rules)
2023-04-06 17:43:09 [ℹ]  will create a CloudFormation stack for cluster itself and 0 nodegroup stack(s)
2023-04-06 17:43:09 [ℹ]  will create a CloudFormation stack for cluster itself and 2 managed nodegroup stack(s)
2023-04-06 17:43:09 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=cn-northwest-1 --cluster=mobile-app'
2023-04-06 17:43:09 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "mobile-app" in "cn-northwest-1"
2023-04-06 17:43:09 [ℹ]  CloudWatch logging will not be enabled for cluster "mobile-app" in "cn-northwest-1"
2023-04-06 17:43:09 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=cn-northwest-1 --cluster=mobile-app'
2023-04-06 17:43:09 [ℹ]  
2 sequential tasks: { create cluster control plane "mobile-app", 
    2 sequential sub-tasks: { 
        wait for control plane to become ready,
        2 parallel sub-tasks: { 
            create managed nodegroup "sprintcloud-gateway-nodegroups",
            create managed nodegroup "app-nodegroups",
        },
    } 
}
2023-04-06 17:43:09 [ℹ]  building cluster stack "eksctl-mobile-app-cluster"
2023-04-06 17:43:10 [ℹ]  deploying stack "eksctl-mobile-app-cluster"
2023-04-06 17:43:40 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:44:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:45:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:46:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:47:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:48:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:49:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:50:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:51:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:52:10 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-cluster"
2023-04-06 17:54:11 [ℹ]  building managed nodegroup stack "eksctl-mobile-app-nodegroup-app-nodegroups"
2023-04-06 17:54:11 [ℹ]  building managed nodegroup stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:54:11 [ℹ]  deploying stack "eksctl-mobile-app-nodegroup-app-nodegroups"
2023-04-06 17:54:11 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-app-nodegroups"
2023-04-06 17:54:11 [ℹ]  deploying stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:54:11 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:54:41 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-app-nodegroups"
2023-04-06 17:54:41 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:55:20 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-app-nodegroups"
2023-04-06 17:55:40 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:56:30 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:56:54 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-app-nodegroups"
2023-04-06 17:57:39 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-nodegroup-sprintcloud-gateway-nodegroups"
2023-04-06 17:57:39 [ℹ]  waiting for the control plane to become ready
2023-04-06 17:57:40 [✔]  saved kubeconfig as "/home/ec2-user/.kube/config"
2023-04-06 17:57:40 [ℹ]  no tasks
2023-04-06 17:57:40 [✔]  all EKS cluster resources for "mobile-app" have been created
2023-04-06 17:57:40 [ℹ]  nodegroup "sprintcloud-gateway-nodegroups" has 2 node(s)
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-60-138.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-75-246.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  waiting for at least 2 node(s) to become ready in "sprintcloud-gateway-nodegroups"
2023-04-06 17:57:40 [ℹ]  nodegroup "sprintcloud-gateway-nodegroups" has 2 node(s)
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-60-138.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-75-246.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  nodegroup "app-nodegroups" has 2 node(s)
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-117-82.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-6-137.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  waiting for at least 2 node(s) to become ready in "app-nodegroups"
2023-04-06 17:57:40 [ℹ]  nodegroup "app-nodegroups" has 2 node(s)
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-117-82.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:40 [ℹ]  node "ip-10-201-6-137.cn-northwest-1.compute.internal" is ready
2023-04-06 17:57:41 [ℹ]  kubectl command should work with "/home/ec2-user/.kube/config", try 'kubectl get nodes'
2023-04-06 17:57:41 [✔]  EKS cluster "mobile-app" in "cn-northwest-1" region is ready
```

![image-20210712002956454](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712002956454.png)



验证集群工作状态:

```bash
$ kubectl get nodes
NAME                                               STATUS   ROLES    AGE   VERSION
ip-10-201-117-82.cn-northwest-1.compute.internal   Ready    <none>   26m   v1.25.7-eks-a59e1f0
ip-10-201-6-137.cn-northwest-1.compute.internal    Ready    <none>   26m   v1.25.7-eks-a59e1f0
```

可以看到现在有两个 Node 节点在运行，分别承载的是两个 coreDNS Pod。

```bash
$ kubectl get po --all-namespaces -o wide
NAMESPACE     NAME                      READY   STATUS    RESTARTS   AGE     IP              NODE                                               NOMINATED NODE   READINESS GATES
kube-system   aws-node-kj2d2            1/1     Running   0          27m     10.201.117.82   ip-10-201-117-82.cn-northwest-1.compute.internal   <none>           <none>
kube-system   aws-node-xkjrf            1/1     Running   0          27m     10.201.6.137    ip-10-201-6-137.cn-northwest-1.compute.internal    <none>           <none>
kube-system   coredns-b5987d9dd-ktd6z   1/1     Running   0          3m31s   10.201.123.15   ip-10-201-117-82.cn-northwest-1.compute.internal   <none>           <none>
kube-system   coredns-b5987d9dd-q7dsk   1/1     Running   0          3m31s   10.201.12.90    ip-10-201-6-137.cn-northwest-1.compute.internal    <none>           <none>
kube-system   kube-proxy-jc5q2          1/1     Running   0          27m     10.201.117.82   ip-10-201-117-82.cn-northwest-1.compute.internal   <none>           <none>
kube-system   kube-proxy-s9dn6          1/1     Running   0          27m     10.201.6.137    ip-10-201-6-137.cn-northwest-1.compute.internal    <none>           <none>
```

查看 eksctl 为我们创建的默认  Profile，它指定了 default 和 kube-system 两个 namespace 中的 pod 都会运行在 EKS 上。



至此，我们成功的完成了 EKS 集群的创建。



## （可选）登录AWS Console控制台，查看 eks集群的配置情况

登录后并打开：https://cn-northwest-1.console.amazonaws.cn/eks/home?region=cn-northwest-1#



由于此EKS集群为C9的IAM用户创建，所以会显示“您的当前用户或角色无权访问此 EKS nodegroup 上的 Kubernetes 对象”，如下图：

![image-20230409170357066](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230409170357066.png)



这是由于在 kubectl 中配置的 AWS IAM 实体未经 Amazon EKS 进行身份验证时会遇到此错误，即当前用户或角色没有 Kubernetes RBAC 权限来描述集群资源。面对这种情况，我们需要在集群的身份验证配置映射增加当前用户的条目，即把你的 IAM 实体已映射到 aws-auth ConfigMap里。



<u>操作步骤如下：</u>

> 第一步：获取  aws-auth ConfigMap username 的arn信息

```bash
CLUSTER_NAME=mobile-app 
AWS_REGION=cn-northwest-1

#运行以下命令检查您的 IAM 实体是否在 aws-auth ConfigMap 中，并获取 username 的arn信息：
eksctl get iamidentitymapping --cluster $CLUSTER_NAME

```

![image-20230409231607271](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230409231607271.png)



> 第二步：通过运行以下命令自动映射您的 IAM 实体 到aws-auth ConfigMap里

```bash
#上一步获取的username 的arn信息
export EKSCREATEDUSERARN=arn:aws-cn:iam::XXXXXXXXXXXX:role/eksctl-mobile-app-nodegroup-app-n-NodeInstanceRole-1FM6UH617GBOF
#您当前登录AWS Console控制台的IAM用户arn；
export AWSLOGINUSERARN=arn:aws-cn:iam::XXXXXXXXXXXX:user/USERNAME 

#通过运行以下命令自动映射您的 IAM 实体：
eksctl create iamidentitymapping \
    --cluster $CLUSTER_NAME \
    --region $AWS_REGION \
    --arn  $AWSLOGINUSERARN \
    --group system:masters \
    --no-duplicate-arns \
    --username $EKSCREATEDUSERARN
```

![image-20230409232223379](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230409232223379.png)



> 第三步：通过运行以下命令， 查看新增后的 aws-auth ConfigMap 配置：

```bash
kubectl edit configmap aws-auth --namespace kube-system
```

![image-20230409232459343](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230409232459343.png)



> 第四步：登录AWS Console控制台，查看 eks集群的配置情况，可以正常显示，权限问题解决。

![image-20230410000657247](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230410000657247.png)







## （可选）部署一个nginx测试eks集群基本功能是否正常工作

>  配置环境变量用于后续使用

```bash
CLUSTER_NAME=mobile-app 
AWS_REGION=cn-northwest-1

STACK_NAME=$(eksctl get nodegroup --cluster ${CLUSTER_NAME} --region=${AWS_REGION} -o json | jq -r '.[].StackName')
echo $STACK_NAME
ROLE_NAME=$(aws cloudformation describe-stack-resources --stack-name $STACK_NAME --region=${AWS_REGION} | jq -r '.StackResources[] | select(.ResourceType=="AWS::IAM::Role") | .PhysicalResourceId')
echo $ROLE_NAME
export ROLE_NAME=${ROLE_NAME}
export STACK_NAME=${STACK_NAME}
```

>  参考 nginx-nlb.yaml, 创建一个nginx pod，并通过LoadBalancer类型对外暴露 *特别提醒80/443 在AWS China Region需要完成备案流程，请联系你的商业经理确保已开通，或者自行更改nginx-nlb.yaml的端口


```yaml
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

```bash
kubectl apply -f ./nginx-nlb.yaml 

## Check deployment status
kubectl get pods
kubectl get deployment nginx-deployment 

## Get the external access 确保 EXTERNAL-IP是一个有效的AWS Network Load Balancer的地址
kubectl get service service-nginx -o wide 

## 下面可以试着访问这个地址
ELB=$(kubectl get service service-nginx -o json | jq -r '.status.loadBalancer.ingress[].hostname')
curl -m3 -v $ELB
```

> 查看结果是否一致

```bash
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
[ec2-user@ip-10-203-0-177 workspace]$ 
```

>
> 清理

```
kubectl delete -f nginx-nlb.yaml 
```
