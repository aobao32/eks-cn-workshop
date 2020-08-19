# 实验（二）部署ALB Ingress

## 一、部署ALB Ingress

### 1、为EKS生成IAM的OIDC授权

执行如下命令。请注意替换集群名称和区域为实际操作的环境。

```
eksctl utils associate-iam-oidc-provider \
    --region cn-northwest-1 \
    --cluster eksworkshop \
    --approve
```

返回结果如下表示成功。

```
[ℹ]  eksctl version 0.25.0
[ℹ]  using region cn-northwest-1
[ℹ]  will create IAM Open ID Connect provider for cluster "eksworkshop" in "cn-northwest-1"
[✔]  created IAM Open ID Connect provider for cluster "eksworkshop" in "cn-northwest-1"
```

### 2、创建IAM Policy（策略）

下载已经预先配置好的IAM策略到本地，注意请保持iam-policy.json的文件名。

```
wget https://s3.cn-north-1.amazonaws.com.cn/myworkshop-lxy/eksworkshop/iam-policy.json
```

创建IAM Policy，执行如下命令。

```
aws iam create-policy \
    --policy-name ALBIngressControllerIAMPolicy \
    --policy-document file://iam-policy.json
```

返回如下结果表示成功。

```
{
    "Policy": {
        "PolicyName": "ALBIngressControllerIAMPolicy",
        "PolicyId": "ANPAWDS54SIWH6DVPEYUB",
        "Arn": "arn:aws-cn:iam::420029960748:policy/ALBIngressControllerIAMPolicy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 0,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "CreateDate": "2020-08-10T04:28:25Z",
        "UpdateDate": "2020-08-10T04:28:25Z"
    }
}
```

此时需要记住这个新建Policy的ARN ID，下一步创建角色时候将会使用。

### 3、创建EKS Service Account

执行如下命令。

```
kubectl apply -f https://s3.cn-north-1.amazonaws.com.cn/myworkshop-lxy/eksworkshop/rbac-role.yaml
```

执行如下命令。请替换region、cluster、attach-policy-arn三个参数为本次实验环境的参数。其他参数保持不变。

```
eksctl create iamserviceaccount \
    --region cn-northwest-1 \
    --name alb-ingress-controller \
    --namespace kube-system \
    --cluster eksworkshop \
    --attach-policy-arn arn:aws-cn:iam::420029960748:policy/ALBIngressControllerIAMPolicy \
    --override-existing-serviceaccounts \
    --approve
```

返回以下结果表示部署成功。本步骤可能需要1-3分钟时间。

```
[ℹ]  eksctl version 0.25.0
[ℹ]  using region cn-northwest-1
[ℹ]  1 iamserviceaccount (kube-system/alb-ingress-controller) was included (based on the include/exclude rules)
[!]  metadata of serviceaccounts that exist in Kubernetes will be updated, as --override-existing-serviceaccounts was set
[ℹ]  1 task: { 2 sequential sub-tasks: { create IAM role for serviceaccount "kube-system/alb-ingress-controller", create serviceaccount "kube-system/alb-ingress-controller" } }
[ℹ]  building iamserviceaccount stack "eksctl-eksworkshop-addon-iamserviceaccount-kube-system-alb-ingress-controller"
[ℹ]  deploying stack "eksctl-eksworkshop-addon-iamserviceaccount-kube-system-alb-ingress-controller"
[ℹ]  serviceaccount "kube-system/alb-ingress-controller" already exists
[ℹ]  updated serviceaccount "kube-system/alb-ingress-controller"
```

### 4、部署ALB Ingress控制器

下载配置文件到本地。

```
wget https://s3.cn-north-1.amazonaws.com.cn/myworkshop-lxy/eksworkshop/alb-ingress-controller.yaml
```

修改这个配置文件，找到其中的如下几个参数，替换为本次实验实际环境的实际参数。注意被配置文件中还有若干以 # 开头的注释，注释可以忽略不管。替换VPC ID请注意，请输入EKS集群的VPC ID，不要输入默认VPC或者其他VPC ID。

```
      - args:
        - --ingress-class=alb
        - --cluster-name=eksworkshop
        - --aws-vpc-id=vpc-08f86b63bfc45208b
        - --aws-region=cn-northwest-1
```

修改配置文件，保存退出。接下来确认配置文件在当前目录下，然后执行命令。

```
kubectl apply -f alb-ingress-controller.yaml
```

执行如下命令确认部署是否成功。

```
kubectl get pods -n kube-system
```

可以看到其中有ALB Ingress的控制器，就表示成功。

```
NAME                                      READY   STATUS    RESTARTS   AGE
alb-ingress-controller-6494895bbf-njfcl   1/1     Running   0          6s
```

至此ALB Ingress配置完成。此时只是配置好了EKS Ingress，如果立刻去查看EC2控制台的ELB界面，是看不到ALB的。接下来的步骤部署应用时候将随应用一起创建ALB。

## 二、部署使用ALB Ingress的测试应用

### 1、创建测试应用

执行如下命令，使用上一步创建的ALB Ingress部署测试应用。这一个Ngnix应用与上一个实验的Nginx的区别是，采用Cluster IP + ALB Ingress的网络方式对外暴露应用。

```
kubectl apply -f https://s3.cn-north-1.amazonaws.com.cn/myworkshop-lxy/eksworkshop/nginx-alb-ingress.yaml
```

由此将在Default namespace下创建出来nginx应用。

### 2、测试访问ALB

执行如下命令获取ALB Ingress的地址。命令后边没有加 -n 的参数表示是在default namespace，如果是在其他name space下需要使用 -n namespace名字 的方式声明要查询的命名空间。

```
kubectl get ingress
```

返回如下结果表示创建正常。

```
NAME          HOSTS   ADDRESS                                                                          PORTS   AGE
alb-ingress   *       546056ac-default-albingres-d740-1614037804.cn-northwest-1.elb.amazonaws.com.cn   80      4m21s
```

用浏览器访问以上的ALB地址，由此即可访问到测试应用，看到 Welcome to nginx! 即表示访问成功。 

此外，也可以在命令行界面下，使用curl访问。

```
ALB=$(kubectl get ingress -o json | jq -r '.items[0].status.loadBalancer.ingress[].hostname')
echo $ALB
curl -m3 -v $ALB
```

也可以看到访问正常。

### 3、删除实验环境

```
kubectl delete -f https://s3.cn-north-1.amazonaws.com.cn/myworkshop-lxy/eksworkshop/nginx-alb-ingress.yaml
```

至此实验完成。

## 三、参考文档

ALB Ingress [https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html]()

AWS GCR Workshop [https://github.com/aws-samples/eks-workshop-greater-china/tree/master/china/2020_EKS_Launch_Workshop]()