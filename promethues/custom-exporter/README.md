# 在 kubernetest + Prometheus operator 部署自定义exporter
本库使用最简单的方式用golang构建了一个自定义的prometheus exporter，然后部署到kubernetes上
作为自定义的监控

## 使用
代码很简单，只有一个main.go文件，通过makefile（做这个实验才学的，写的很初级，希望能帮我完善一下，谢谢）直接进行编译，注意本次使用使用了三台机器，我所在的k8s-master，以及k8s-node1，k8s-node2，并且这三者采用ssh的免密码通信。k8s-master作为编译机器，但不需要装go环境，我是通过容器来进行编译的，具体可参考我的`Dockerfile`

我通过在k8s-master机器上执行`make delive VERSION=0.0.1 TAG_LATEST=1` 来构建镜像，并且将本地镜像打包成`tar`并使用`scp`拷贝到k8s-node1，k8s-node2上，然后使用`docker load < xxx.tar`将镜像直接装到两台机器上
我这样做是不想污染我的docker hub仓库，毕竟这只是一个实验而已。

将Makefile中的有关文件拷贝复制的部分，按照你自己需求重新修改一下，比如修改ssh的地址等，就可以执行了（makefile没写好的缘故，不然应该直接就可以弄点变量上手了），原理就是上面的原理。

最后，到`kube`这个目录下 直接执行所有的yaml文件就可以了。

## 原理

Prometheus operator在部署的时候会产生一个新的k8s资源(crd)就是servicemonitor，我们需要做的就是如下：
1. 将自己的exporter使用deployment或者statefulset等方式发布到k8s上
2. 给自己的应用创建一个service，可以使用nodeport或者clusterip都行
3. 创建一个servicemonitor来关联这个service就好
4. （可选）创建一个prometheusrules来定制自己的prometheus rules就好

Prometheus operator会检查到有新的servicemonitor然后触发让prometheus更新配置文件（其实prometheus这个pod里有三个container，一个就是传统的prometheus服务器，一个是负责监听时间然后更新配置文件，再一个是将配置文件更新传递到configmap里），注意Prometheus operator和prometheus是两样东西，Prometheus operator是来管理像servicemonitor、prometheusrules这种k8s资源，然后来配置prometheus服务器的，而prometheus才是真正意义上的用于监控的prometheus，或者说是prometheus server，就是可以通过9090端口访问，会有一个dashboard的东西

## 问题解决
见 [troubleshoot](./troubleshoot.md)
