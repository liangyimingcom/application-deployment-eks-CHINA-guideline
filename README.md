# application deployment on EKS CHINA REGION guideline

## 第一次在EKS 中国区上部署应用的指南说明


第一次在EKS China上部署应用，总是会遇到很多疑问。

通常，对于初学者的学习曲线是：

1. 先学习：k8s 
2. 再学习：eks
3. 最后学习：应用如何使用 AWS原生服务，如ALB ingress controler 完成应用的公网发布等（使用EKS组件完成k8s适配部署增强）

这个学习路径特别好，可惜需要花费的时间太久，项目上线时间紧 - 耽误不得。



**因此，专门撰写了一个教程，针对于《第一次在EKS CHINA region 上部署应用》即可部署成功的指南与说明。**

如果有不清楚的，或者认为可以补充的内容，请给我留言



请按照步骤1-5完成实验，内容请见每个章节。

- 步骤 10-Landingzone 整体规划说明。
- 步骤 20-EKS 环境准备。
- 步骤 30-EKSVPC 规划与部署。
- 步骤 40-创建 EKS 集群。
- 步骤 50-部署 AWS LoadBalancer Controller.md 步骤 60-部署应用到 EKS 集群。

- 步骤 70-任何步骤创建失败后的环境清理。
