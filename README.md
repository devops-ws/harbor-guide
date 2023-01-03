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

## Harbor 用户配置
用户可以通过配置环境变量 `CONFIG_OVERWRITE_JSON` 来[修改用户配置](https://goharbor.io/docs/2.5.0/install-config/configure-user-settings-cli/)：

```json
CONFIG_OVERWRITE_JSON: '{"auth_mode":"oidc_auth", "oidc_name":"dex", "oidc_endpoint":"https://10.121.218.184:31392/api/dex", "oidc_client_id":"harbor", "oidc_client_secret":"rick", "oidc_admin_group":"infra-admin", "oidc_scope": "openid,offline_access,profile,groups,email", "oidc_user_claim":"username", "oidc_verify_cert":false, "oidc_auto_onboard":true}'
```

* `oidc_auto_onboard` 为 True 的话，用户登录后无需设置账户名，会自动从 `oidc_user_claim` 获取
