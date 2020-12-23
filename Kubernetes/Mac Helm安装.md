### Helm 就相当于 kubernetes 环境下的 yum 包管理工具。

```shell
# 用 homebrew 安装 Helm
$ brew install kubernetes-helm

# 更新本地 charts repo
$ helm repo update

# 安装 mysql chart
$ helm install --name my-mysql stable/mysql

# 删除 mysql
$ helm delete my-mysql

# 删除 mysql 并释放该名字以便后续使用
$ helm delete --purge my-mysql
```

镜像源：

```
http://mirror.azure.cn/kubernetes/charts-incubator/
https://charts.jetstack.io
https://burdenbear.github.io/kube-charts-mirror/
```

