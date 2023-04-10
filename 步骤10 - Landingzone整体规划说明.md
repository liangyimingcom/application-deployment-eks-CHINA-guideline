# Landingzone整体规划说明：

由于是生产环境，在开始创建k8s/EKS之前，需要做好landingzone规划。



规划分为三个VPC，分别为：

**1.** **Prod VPC:**

容纳生产环境。

**2. Non-Prod VPC:**

部署Non-Prod VPC，用于容纳非生产环境，包括开发、测试以及其他比如准生产环境等。

**3. Management VPC**

当VPC数量扩张或者共享服务比如跳板/管理机、运维管理、安全管控组件等增加时，可以单另采用共享服务VPC容纳这些服务，供多个应用系统VPC共享；

共享服务VPC需与各个应用VPC采用VPC peering打通。

**VPC Peering：**

VPC的IP段不得有重叠，保证以后在需要互联互通时采用VPC Peering 打通。



![image-20230406225021053](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230406225021053.png)



##  规划与设计 Management VPC (运维管理) 环境

**VPC CIDR规划： 10.203.0.0/16 (65534)**

|      | **Description**         | **Subnets**   | **Type** | **memo** |
| ---- | ----------------------- | ------------- | -------- | -------- |
|      |                         |               |          |          |
| 1    | External subnet  - AZ a | 10.203.0.0/24 | public   | 256      |
| 2    | External subnet - AZ b  | 10.203.1.0/24 | public   | 256      |
| 3    | Internal subnet -  AZ a | 10.203.2.0/24 | private  | 256      |
| 4    | Internal subnet - AZ b  | 10.203.3.0/24 | private  | 256      |
|      |                         |               |          |          |
|      |                         |               |          |          |





## 创建 Management VPC (运维管理) 环境

使用 AWS 用户登录Console控制台: https://cn-northwest-1.console.amazonaws.cn/console/home?region=cn-northwest-1

**请确认，并将 AWS Region 切换成您的 Workshop 使用的区域 - 宁夏区域。**



创建 Management VPC 环境, 步骤如下面截图所示:

[步骤-1] 进入VPC服务（https://cn-northwest-1.console.amazonaws.cn/vpc/home?region=cn-northwest-1#CreateVpc:createMode=vpcWithResources），按照规划部分输入ip cidr段如下：

![image-20230406232042938](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230406232042938.png)



[步骤-2] 点击创建，如下图：

![image-20230406232153411](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230406232153411.png)



[步骤-3] 创建成功，查看 公共子网的路由只想IGW，私有子网暂时不用，不创建NAT Gateway节省费用。

![image-20230406232322142](https://raw.githubusercontent.com/liangyimingcom/storage/master/PicGo/image-20230406232322142.png)



[步骤-4] Management VPC (运维管理) 创建完成后，我们可以在里面创建Cloud9管理运维服务器，请看下一章节；





### 本章节完成
