# Kubernetes自身证书管理

## 背景

Certificate Management with kubeadm，https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-certs

kubeadm管理2个pki，一个是k8s自身组件的，一个是etcd的

kubeadm pki中的证书有如下属性

- ca证书有效期10年
- 普通证书有效期1年

kubeadm集成了普通证书的renew，没有根证书的替换

## 更新证书

```bash
# 查看证书的到期时间
kubeadm alpha certs check-expiration

# 更新所有证书
kubeadm alpha certs renew all

# 把证书更新放到 crontab 管理
vim /etc/cron.d/yunion_kubeadm_renew_certs
@monthly root /usr/bin/kubeadm alpha certs renew all
```

## 其它问题解决方法

1. 查看 kubelet 的状态，发现启动报错，'/etc/kubernetes/bootstrap-kubelet.conf: no such file or directory':

```bash
# 如果发现 kubelet 服务状态为 failed 启动失败，无法启动
# 查看日志
systemctl status kubelet -l

# 发现有报错
# F1127 09:00:16.566510   27284 server.go:262] failed to run Kubelet:
# unable to load bootstrap kubeconfig: stat /etc/kubernetes/bootstrap-kubelet.conf: no such file or directory
 
# 解决办法
cp /etc/kubernetes/admin.conf /etc/kubernetes/bootstrap-kubelet.conf
systemctl restart kubelet
```

2. 如果访问控制台出现traefik 404 server not found，可以尝试重启它

```bash
kubectl -n kube-system rollout restart ds traefik-ingress-controller
```
