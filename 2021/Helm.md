# 内部Helm系统部署

在内部安装了一个Helm系统，并接入到Rancher中，后续可以通过helm来管理产品的部署。

本文记录了Helm部署的步骤及过程，以便后续运维使用。

整个Helm体系分为两个部分，Helm Client和Helm Server (Tiller)。
<img src="https://p1-tt.byteimg.com/origin/pgc-image/a786e213b1c94614b8ba2f7e93e4e21c?from=pc"/>
<div style="text-align: center;">Helm总体架构</div>

## 1、Helm的部署

在需要部署Helm的服务器上下载了Helm二进制包
https://get.helm.sh/helm-v2.17.0-linux-amd64.tar.gz

然后解压后，将helm拷贝到`/usr/local/bin`下面，以便直接使用

## 2、Helm的初始化

在本地初始化Helm，需要连接到Kubernetes中，所以必须在 $HOME/.kub 中保存好集群的 config 文件，初始化之后

helm version
~~~text
Client: &version.Version{SemVer:"v2.17.0", GitCommit:"a690bad98af45b015bd3da1a41f6218b1a451dbe", GitTreeState:"clean"}
Error: Get "http://localhost:8080/api/v1/namespaces/kube-system/pods?labelSelector=app%3Dhelm%2Cname%3Dtiller": dial tcp 127.0.0.1:8080: connect: connection refused
~~~

~~~text
helm home
/home/masterserver/.helm
~~~

出现上述问题，说明没有部署 Tiller，先使用命令 `helm init` 部署 Tiller。

Tiller部署到`Kubernetes`集群，Helm可以部署到集群，也可以部署到某个节点或物理机，它们之间的关系请参考上图“Helm总体架构”，该命令会在`kubernetes`集群kube-system名字空间部署
`ghcr.io/helm/tiller:v2.17.0`的镜像并启动服务。

<img src="https://p6-tt.byteimg.com/origin/pgc-image/2c4107d0e9804f2f9c2e68aa3662b9e5?from=pc"/>
<div style="text-align: center;">Tiller在集群的状态</div>



## 3、给Tiller进行授权

Tiller如果要部署应用，需要在Kubernetes中进行授权

~~~bash
kubectl create serviceaccount --namespace kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}’
~~~

更新一下当前的helm配置
~~~bash
helm init --service-account tiller --upgrade
~~~

更新仓库
~~~bash
helm repo update
~~~

## 4、启动Helm的服务

完成部署和初始化后，现在需要Helm对外提供接口服务，Web接口，可以通过命令
~~~bash
 helm serve —url 0.0.0.0:8879 &
~~~

在本机的8879端口启动Web监听服务

<img src="https://p1-tt.byteimg.com/origin/pgc-image/fe04e2d7b95341a4ab1e1161611ec486?from=pc"/>
<div style="text-align: center;">Tiller在集群的状态</div>


这样`Rancher`才能引入这个Helm仓库

<img src="https://p1-tt.byteimg.com/origin/pgc-image/01d8b235918143e0a1d572d199396cc6?from=pc"/>
<div style="text-align: center;">Rancher中设置Helm Charts仓库</div>


## 5、创建自己的Helm Chart

创建自己的Helm Chart使用Helm命令即可，以下列出所有相关命令

5.1 创建一个chart，名称为nginx-fixed
~~~bash
helm create nginx-fixed
~~~

5.2 修改nginx-fixed目录下面的Chart.yaml和相关文件，如下

~~~text
apiVersion: v1
appVersion: "1.0"
description: 修改版的nginx
name: nginx-fixed
icon: http://ip-address/ncw/portal/static/img/a7_login_blue.2f737478.png
version: 1.1.0
~~~

5.3 helm package打包或者直接压缩nginx-fixed文件夹为tgz，两者其实差不多（但有不同）

~~~bash
helm package
~~~
或者
~~~bash
tar czfv nginx-fixed.tgz nginx-fixed
~~~

5.4 将helm chart压缩包拷贝到helm的repo里面

~~~bash
cp -f nginx-fixed.tgz ~/.helm/repository/local/
~~~

5.5 重新索引本地repo

~~~bash
helm repo index /home/pubserver/.helm/repository/local/
~~~

5.6 其他机器增加仓库（使用其他机器进行测试）
~~~bash
helm repo add auditcharts http://helm-deploy-server-ip:8879
~~~

5.7 更新库索引
~~~bash
helm repo update
~~~
~~~text
Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "auditcharts" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete.
~~~


5.8 检索测试，如nginx

~~~bash
helm search nginx
~~~
~~~text
NAME CHART VERSION APP VERSION DESCRIPTION
auditcharts/nginx-fixed 1.1.0 1.0 修改版的nginx
stable/nginx-ingress 1.41.3 v0.34.1 DEPRECATED! An nginx Ingress controller that uses ConfigM...
stable/nginx-ldapauth-proxy 0.1.6 1.13.5 DEPRECATED - nginx proxy with ldapauth
stable/nginx-lego 0.3.1 Chart for nginx-ingress-controller and kube-lego
stable/gcloud-endpoints 0.1.2 1 DEPRECATED Develop, deploy, protect and monitor your APIs…
~~~


## 6、通过Rancher或Helm部署

6.1 通过Rancher的应用商店中直接安装应用

<img src="https://p1-tt.byteimg.com/origin/pgc-image/ac173d0353204a01ac41f1516bd8df0a?from=pc"/>
<div style="text-align: center;">Rancher 引入私有Helm仓库</div>


6.2 通过helm客户端部署

~~~bash
helm install auditcharts/nginx-fixed
~~~

~~~text
NAME: womping-dog
LAST DEPLOYED: Mon Jul 12 15:21:12 2021
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME READY UP-TO-DATE AVAILABLE AGE
womping-dog-nginx-fixed 0/1 1 0 1s

==> v1/Pod(related)
NAME READY STATUS RESTARTS AGE
womping-dog-nginx-fixed-6bdd79748c-m68m9 0/1 ContainerCreating 0 0s

==> v1/Service
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
womping-dog-nginx-fixed ClusterIP 10.43.23.34 <none> 80/TCP 1s

==> v1/ServiceAccount
NAME SECRETS AGE
womping-dog-nginx-fixed 1 1s

NOTES:
1. Get the application URL by running these commands:
 export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=nginx-fixed,app.kubernetes.io/instance=womping-dog" -o jsonpath="{.items[0].metadata.name}")
 echo "Visit http://127.0.0.1:8080 to use your application"
 kubectl port-forward $POD_NAME 8080:80

~~~

> 发现整个 `2020` 年都没有更新BLOG，拿一篇日记凑数吧😝