# 步骤3 - 部署 AWS LoadBalancer Controller



[AWS Load Balancer Controller](https://github.com/kubernetes-sigs/aws-load-balancer-controller) 是一个控制器，可帮助管理 Kubernetes 集群的 Elastic Load Balancer。

- 它通过部署和配置 Application Load Balancers - ALB 来提供 Kubernetes Ingress 资源。
- 它通过部署和配置 Network Load Balancers - NLB 来提供 Kubernetes Service 资源。



AWS ALB 和 NLB 可以和部署在 Fargate 上的 Service/Ingress 进行集成，通过 IP 模式将流量从负载均衡器直接转发到 Pod IP 上。其中，

- 对于 Kubernetes Service，可以使用 AWS 网络负载均衡器 (NLB，IP 模式) 对跨 Pod 的 4 层网络流量进行负载均衡。
- 对于 Kubernetes Ingress，可以使用 AWS 应用负载均衡器 (ALB，IP 模式) 对跨 Pod 的 7 层网络流量进行负载均衡。

参考资料：https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html



### 创建 IAM 策略

#### 下载 IAM 策略文件

用于为 AWS Load Balancer Controller 配置 [IRSA](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) 权限 

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.4.7/docs/install/iam_policy_cn.json
```

创建成功后输出示例如下：

```bash
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  8425  100  8425    0     0   2428      0  0:00:03  0:00:03 --:--:--  2428
```

#### 部署OIDC：

```bash
#请使用自己的cluster name替换下面命令中的mobile-app
export CLUSTERNAME=mobile-app
eksctl utils associate-iam-oidc-provider --cluster $CLUSTERNAME --approve
```

创建成功后输出示例如下：

```bash
2023-04-09 17:23:02 [ℹ]  will create IAM Open ID Connect provider for cluster "mobile-app" in "cn-northwest-1"
2023-04-09 17:23:02 [✔]  created IAM Open ID Connect provider for cluster "mobile-app" in "cn-northwest-1"
```

#### 使用下载到策略文件创建 IAM 策略

执行如下命令

```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
--policy-document file://iam_policy_cn.json
```

以宁夏区域部署为例，创建成功后输出示例如下：

```bash
$ aws iam create-policy \
>     --policy-name AWSLoadBalancerControllerIAMPolicy \
> --policy-document file://iam_policy_cn.json
{
    "Policy": {
        "PolicyName": "AWSLoadBalancerControllerIAMPolicy",
        "PolicyId": "ANPAU5GINBISSQ4WRZHJA",
        "Arn": "arn:aws-cn:iam::xxxxxxxxxxx:policy/AWSLoadBalancerControllerIAMPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2023-04-09T17:26:22+00:00",
        "UpdateDate": "2023-04-09T17:26:22+00:00"
    }
}

```





### 创建 IAM role 和 service account

为 AWS Load Balancer Controller 创建一个 IAM role 和 Service Account 并将两者进行关联，以便 AWS Load Balancer Controller 所在 Pod 拥有相应的 IAM 权限（如创建 ELB、TargetGroup 等）。

执行如下命令，注意将 cluster 名称和 policy-arn 替换成你自己的值。

```bash
#请使用自己的cluster name替换下面命令中的mobile-app，REGION缺省为宁夏区域，AWSACCOUNTID为你的AWS账号ID
export CLUSTERNAME=mobile-app
export REGION=cn-northwest-1
export AWSACCOUNTID=xxxxxxxxxxxx

eksctl create iamserviceaccount \
  --cluster=$CLUSTERNAME \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws-cn:iam::$AWSACCOUNTID:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve


```

创建成功后 eksctl 输出示例如下：

```bash
$ eksctl create iamserviceaccount \
>   --cluster=$CLUSTERNAME \
>   --namespace=kube-system \
>   --name=aws-load-balancer-controller \
>   --role-name AmazonEKSLoadBalancerControllerRole \
>   --attach-policy-arn=arn:aws-cn:iam::$AWSACCOUNTID:policy/AWSLoadBalancerControllerIAMPolicy \
>   --approve
2023-04-09 17:34:53 [ℹ]  1 iamserviceaccount (kube-system/aws-load-balancer-controller) was included (based on the include/exclude rules)
2023-04-09 17:34:53 [!]  serviceaccounts that exist in Kubernetes will be excluded, use --override-existing-serviceaccounts to override
2023-04-09 17:34:53 [ℹ]  1 task: { 
    2 sequential sub-tasks: { 
        create IAM role for serviceaccount "kube-system/aws-load-balancer-controller",
        create serviceaccount "kube-system/aws-load-balancer-controller",
    } }2023-04-09 17:34:53 [ℹ]  building iamserviceaccount stack "eksctl-mobile-app-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2023-04-09 17:34:53 [ℹ]  deploying stack "eksctl-mobile-app-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2023-04-09 17:34:53 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2023-04-09 17:35:23 [ℹ]  waiting for CloudFormation stack "eksctl-mobile-app-addon-iamserviceaccount-kube-system-aws-load-balancer-controller"
2023-04-09 17:35:23 [ℹ]  created serviceaccount "kube-system/aws-load-balancer-controller"

```

创建完成后，可以开始部署 AWS Load Balancer Controller。



### 部署 AWS Load Balancer Controller

可以通过 Helm 或者手工的方式部署，在本次实验中我们使用 Helm 的方式。

#### 安装 Helm

官方给出的Helm在线安装命令如下：

> ~~curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash~~

由于国内与海外网络原因，海外最新安装包大概率无法下载，建议使用上传后安装的方法，即：从github上把离线的helm安装包上传到Cloud9中，然后进行本地安装。

执行如下命令：

```bash
#从github上把离线的helm安装包上传到Cloud9中
wget https://kgithub.com/liangyimingcom/application-deployment-eks-CHINA-guideline/raw/e84bc1033eec526b10ed72f62e16eae2fdddf4d7/image/helm-v3.11.2-linux-amd64.tar.gz

#进行本地安装
tar -zxvf helm-v3.11.2-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/

```

验证 Helm 安装

```bash
$ helm version --short
v3.11.2+g912ebc1
```



#### 安装 Controller

添加helm仓库，添加 eks-chart，需等待一段时间

```bash
helm repo add eks https://aws.github.io/eks-charts
```

更新helm仓库

```bash
helm repo update
```



使用 Helm 安装 Controller，将 cluster name 替换成你的集群名称

```bash

#请使用自己的cluster name替换下面命令中的mobile-app
export CLUSTERNAME=mobile-app

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=$CLUSTERNAME \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set enableShield=false \
  --set enableWaf=false \
  --set enableWafv2=false


```

以宁夏区域部署为例，创建成功后输出示例如下：

```bash
$ helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
>   -n kube-system \
>   --set clusterName=mobile-app \
>   --set serviceAccount.create=false \
>   --set serviceAccount.name=aws-load-balancer-controller \
>   --set enableShield=false \
>   --set enableWaf=false \
>   --set enableWafv2=false
NAME: aws-load-balancer-controller
LAST DEPLOYED: Sun Apr  9 17:41:54 2023
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!
```

查看部署状态和日志

```bash
$ kubectl get deployment -n kube-system aws-load-balancer-controller
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
aws-load-balancer-controller   2/2     2            2           19s
```


查看 Controller pod 名称

```bash
$ kubectl get po  -n kube-system | grep load-balancer
aws-load-balancer-controller-55fc9998d-ldpnt   1/1     Running   0              10h
aws-load-balancer-controller-55fc9998d-lfr2w   1/1     Running   0              10h
```



查看 Controller 日志，注意将 pod name 替换成你自己的 Controller pod 名称

```bash
$ kubectl logs -n kube-system aws-load-balancer-controller-55fc9998d-ldpnt

{"level":"info","ts":1681062124.0686626,"msg":"version","GitVersion":"v2.4.7","GitCommit":"2ba14d1e232c1f2aa02063a3edd2ef855ba468d9","BuildDate":"2023-02-23T07:33:14+0000"}
{"level":"info","ts":1681062124.087766,"logger":"controller-runtime.metrics","msg":"metrics server is starting to listen","addr":":8080"}
{"level":"info","ts":1681062124.0900154,"logger":"setup","msg":"adding health check for controller"}
{"level":"info","ts":1681062124.0901632,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/mutate-v1-pod"}
{"level":"info","ts":1681062124.0902758,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/mutate-elbv2-k8s-aws-v1beta1-targetgroupbinding"}
{"level":"info","ts":1681062124.0903578,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/validate-elbv2-k8s-aws-v1beta1-targetgroupbinding"}
{"level":"info","ts":1681062124.090421,"logger":"controller-runtime.webhook","msg":"registering webhook","path":"/validate-networking-v1-ingress"}
{"level":"info","ts":1681062124.0904868,"logger":"setup","msg":"starting podInfo repo"}
{"level":"info","ts":1681062126.0907125,"msg":"starting metrics server","path":"/metrics"}
I0409 17:42:06.090671       1 leaderelection.go:243] attempting to acquire leader lease kube-system/aws-load-balancer-controller-leader...
{"level":"info","ts":1681062126.090831,"logger":"controller-runtime.webhook.webhooks","msg":"starting webhook server"}
{"level":"info","ts":1681062126.0913892,"logger":"controller-runtime.certwatcher","msg":"Updated current TLS certificate"}
{"level":"info","ts":1681062126.0915668,"logger":"controller-runtime.webhook","msg":"serving webhook server","host":"","port":9443}
{"level":"info","ts":1681062126.0916717,"logger":"controller-runtime.certwatcher","msg":"Starting certificate watcher"}
I0409 17:42:06.103185       1 leaderelection.go:253] successfully acquired lease kube-system/aws-load-balancer-controller-leader
{"level":"info","ts":1681062126.1915586,"logger":"controller.ingress","msg":"Starting EventSource","source":"channel source: 0xc0005065a0"}
{"level":"info","ts":1681062126.1917267,"logger":"controller.ingress","msg":"Starting EventSource","source":"channel source: 0xc0005065f0"}
{"level":"info","ts":1681062126.1917841,"logger":"controller.ingress","msg":"Starting EventSource","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1917977,"logger":"controller.ingress","msg":"Starting EventSource","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1918027,"logger":"controller.ingress","msg":"Starting EventSource","source":"channel source: 0xc000506640"}
{"level":"info","ts":1681062126.1918113,"logger":"controller.ingress","msg":"Starting EventSource","source":"channel source: 0xc000506690"}
{"level":"info","ts":1681062126.1915424,"logger":"controller.targetGroupBinding","msg":"Starting EventSource","reconciler group":"elbv2.k8s.aws","reconciler kind":"TargetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1918488,"logger":"controller.targetGroupBinding","msg":"Starting EventSource","reconciler group":"elbv2.k8s.aws","reconciler kind":"TargetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1918628,"logger":"controller.targetGroupBinding","msg":"Starting EventSource","reconciler group":"elbv2.k8s.aws","reconciler kind":"TargetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1918402,"logger":"controller.ingress","msg":"Starting EventSource","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1919234,"logger":"controller.ingress","msg":"Starting EventSource","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1919477,"logger":"controller.ingress","msg":"Starting Controller"}
{"level":"info","ts":1681062126.1918874,"logger":"controller.targetGroupBinding","msg":"Starting EventSource","reconciler group":"elbv2.k8s.aws","reconciler kind":"TargetGroupBinding","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.192047,"logger":"controller.targetGroupBinding","msg":"Starting Controller","reconciler group":"elbv2.k8s.aws","reconciler kind":"TargetGroupBinding"}
{"level":"info","ts":1681062126.1916137,"logger":"controller.service","msg":"Starting EventSource","source":"kind source: /, Kind="}
{"level":"info","ts":1681062126.1921155,"logger":"controller.service","msg":"Starting Controller"}
{"level":"info","ts":1681062126.2931652,"logger":"controller.targetGroupBinding","msg":"Starting workers","reconciler group":"elbv2.k8s.aws","reconciler kind":"TargetGroupBinding","worker count":3}
{"level":"info","ts":1681062126.2933955,"logger":"controller.service","msg":"Starting workers","worker count":3}
{"level":"info","ts":1681062126.2934783,"logger":"controller.ingress","msg":"Starting workers","worker count":3}
{"level":"info","ts":1681062242.4170349,"logger":"backend-sg-provider","msg":"creating securityGroup","name":"k8s-traffic-mobileapp-73ff9edb02"}
{"level":"info","ts":1681062242.7008169,"logger":"controllers.ingress","msg":"Auto Create SG","LB SGs":[{"$ref":"#/resources/AWS::EC2::SecurityGroup/ManagedLBSecurityGroup/status/groupID"},"sg-0159358c140ad00be"],"backend SG":"sg-0159358c140ad00be"}

```





### 本章节完成



