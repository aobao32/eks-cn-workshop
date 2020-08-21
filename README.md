# AWS中国区EKS动手实验

## 一、实验说明

实验环境：中国区AWS账号，ZHY区域

实验需求：

* MacOS，带包括 awscli，eksctl，kubectl等环境
* 如果客户端环境不是MacOS，建议在AWS上创建一个EC2使用Amazon Linux 2系统，并通过此EC2进行后续操作

## 二、实验内容

[实验1：创建集群](https://github.com/aobao32/eks-cn-workshop/blob/master/01-%E5%88%9B%E5%BB%BA%E9%9B%86%E7%BE%A4.md)

[实验2：部署ALB Ingress](https://github.com/aobao32/eks-cn-workshop/blob/master/02-%E9%83%A8%E7%BD%B2ALB%20Ingress.md)

[实验3：调整集群配置](https://github.com/aobao32/eks-cn-workshop/blob/master/03-%E8%B0%83%E6%95%B4%E9%9B%86%E7%BE%A4%E9%85%8D%E7%BD%AE.md)

[实验4：部署单体应用2048游戏](https://github.com/aobao32/eks-cn-workshop/blob/master/04-%E9%83%A8%E7%BD%B2%E5%8D%95%E4%BD%93%E5%BA%94%E7%94%A82048%E6%B8%B8%E6%88%8F.md)

[实验5：部署微服务应用](https://github.com/aobao32/eks-cn-workshop/blob/master/05-%E9%83%A8%E7%BD%B2%E5%BE%AE%E6%9C%8D%E5%8A%A1%E5%BA%94%E7%94%A8.md)

## 三、参考资料

结合AWS EKS ECR Workshop内容：

[https://github.com/aws-samples/eks-workshop-greater-china](https://github.com/aws-samples/eks-workshop-greater-china)

对脚本做了适应中国区的调整，且组合了部分脚本，降低实验参与者对Linux Shell的技术背景和难度，简化操作过程。