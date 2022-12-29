# harbor-guide
Harbor 教程

## 安装

### Argo CD
我们可以通过 [Argo CD](https://github.com/devops-ws/argo-cd-guide) 这种 GitOps 的方式来安装 Harbor：

```shell
cat <<EOF | kubectl apply -n argocd -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: harbor
  namespace: argocd
spec:
  project: default
  source:
    chart: harbor
    repoURL: https://helm.goharbor.io
    targetRevision: 1.10.2
    helm:
      releaseName: harbor
      parameters:
      - name: expose.type
        value: nodePort
      - name: expose.tls.enabled
        value: "false"
      - name: externalURL
        value: http://10.121.218.242:30002/             # 必须要设置外部访问地址
  destination:
    server: "https://kubernetes.default.svc"
    namespace: harbor
  syncPolicy:
    syncOptions:
    - CreateNamespace=true
    automated:
      prune: true
      selfHeal: true
EOF
```

## Docker Daemon
如果安装的 Harbor 没有证书签名的话，可以添加如下配置使得忽略：

```json
{
  "insecure-registries": ["10.121.218.242:30002"]
}
```

> Linux 下的文件路径为：/etc/docker/daemon.json
>
> 重启命令：systemctl restart docker

命令行登录：
```shell
docker login 10.121.218.242:30002 -u admin -pHarbor12345
```
