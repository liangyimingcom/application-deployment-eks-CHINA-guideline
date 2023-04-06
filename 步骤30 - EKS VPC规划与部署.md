# AWS VPC环境规划 - VPC的设计与部署



**提示：如果您在任何步骤创建失败，环境清理请参考 [步骤5]()。**



## 第一步：对要部署的应用，进行VPC规划设计

我们以部署在AWS 宁夏区域的一个名称为“MobileAPP”的应用为例。



**VPC CIDR规划：: 10.201.0.0/16 (65534)**

**VPC子网规划 - Subnet- EKS/k8s/worknode cluster  -** 

|      | **Description**         | **Subnets**            | **Type** | **memo** |
| ---- | ----------------------- | ---------------------- | -------- | :------: |
|      |                         |                        |          |          |
| 1    | External  subnet - AZ a | 10.201.32.0/19 | public   |   8190   |
| 2    | External  subnet - AZ b | 10.201.64.0/19         | public   |   8190   |
| 3    | External  subnet - AZ c | 10.201.128.0/19        | public   |   8190   |
| 4    | Internal  subnet - AZ a | 10.201.0.0/19          | private  |   8190   |
| 5    | Internal  subnet - AZ b | 10.201.96.0/19         | private  |   8190   |
| 6    | Internal  subnet - AZ c | 10.201.160.0/19        | private  |   8190   |

 **VPC子网规划 -  Subnet - RDS/EC2(安装 redis/mongodb)**    

|      | **Description**         | **Subnets**         | **Type** | **memo** |
| ---- | ----------------------- | ------------------- | -------- | -------- |
|      |                         |                     |          |          |
| 1    | Internal subnet - AZ  a | 10.201.192.0/24 | private  | 254     |
| 2    | Internal subnet - AZ  b | 10.201.193.0/24     | private  | 254     |

**VPC子网规划说明：EKS自动创建VPC分区，k8s worknodes分为二类子网**

- 外网 (External subnet 1&2)： 公共子网
- 内网 (Internal subnet 1&2)： 私有子网





## 第二步：对规划VPC，进行部署，同时完成EKS的标签要求



使用Cloudformation 创建  MobileAPP PROD VPC
1）打开ZHY的CF：https://cn-northwest-1.console.amazonaws.cn/cloudformation/home?region=cn-northwest-1#/stacks/create

2）创建新的stacks，填入：https://s3.us-west-2.amazonaws.com/amazon-eks/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml

![image-20210712001115457](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001115457.png)



3）创建新的stacks name = Mobileappvpo01 ，填入IP地址（参考 VPC规划设计）

![image-20230407010022462](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407010022462.png)



![image-20230407005451936](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407005451936.png)



4）确认无误后，点击创建CF。然后检查VPC是否创建成功。

![image-20210712001313036](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001313036.png)



5）创建RDS/REDIS子网；（图例仅供参考）

![image-20210712001347851](https://raw.githubusercontent.com/liangyimingcom/storage/master/uPic/image-20210712001347851.png)

![image-20230407011351864](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407011351864.png)





6）检查VPC是否正常，如下图：（图例仅供参考）

![image-20230407011630323](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407011630323.png)



## 第三步：准备 EKS集群的Yaml配置文件

**Yaml文件名：《[eks_cluster_zhy_v125.yaml](https://github.com/liangyimingcom/application-deployment-eks-CHINA-guideline/blob/main/cf-template/eks_cluster_zhy_v125.yaml)》** 。在Cloud9里面”New Terminal“，然后复制粘贴如下代码创建文件。

```yaml
cat << EOF > eks_cluster_zhy_v125.yaml
---
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: mobile-app
  region: cn-northwest-1

vpc:
  id: vpc-056f83f8db816668c
  cidr: "10.201.0.0/16"
  subnets:
    public:
      public-one:
        id: subnet-0998e8598dc0a8579
      public-two:
        id: subnet-0ac31567324699565
    private:
      private-one:
        id: subnet-0b3fef6c1bb8b5306
      private-two:
        id: subnet-0810e50caed049bfb

managedNodeGroups:
  # private node group
  - name: app-nodegroups
    instanceType: m6i.large
    desiredCapacity: 2
    privateNetworking: true

EOF
```



1) 在Cloud9中打开yaml文件，然后修改VPC-ID，如AWS Console一致：

![image-20230407011935705](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407011935705.png)



2）VPC-ID来自于AWS Console的VPC界面，请在这里查阅：

![image-20230407012018390](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407012018390.png)



3）请把改好的YAML文件保存好，用于下一步的操作。



**注意：使用Cloudformation 创建  PROD VPC的主要目的是为了完成VPC打标签（Tag）的工作。没有正确配置标签的VPC子网（subnet）是无法在EKS下工作的。**

正确的标签参考如下：

![image-20230407012113995](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230407012113995.png)





