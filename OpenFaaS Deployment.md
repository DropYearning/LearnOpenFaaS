# OpenFaaS Deployment

## Deploy OpenFaaS on Ubuntu 

### Install the `faas-cli`

```
# MacOS and Linux users
# If you run the script as a normal non-root user then the script
# will download the faas-cli binary to the current folder
$ curl -sL https://cli.openfaas.com | sudo sh

# Windows users with (Git Bash)
$ curl -sL https://cli.openfaas.com | sh
```

###  Deploy with `arkade` (fastest option)

The `arkade install` command installs OpenFaaS using its official helm chart, but without using `tiller`, a [component which is insecure by default](https://engineering.bitnami.com/articles/running-helm-in-production.html). arkade can also install other important software for OpenFaaS users such as `cert-manager` and `nginx-ingress`. It's the easiest and quickest way to get up and running.

- Get arkade

  ```
  # For MacOS / Linux:
  curl -SLsf https://dl.get-arkade.dev/ | sudo sh
  
  # For Windows (using Git Bash)
  curl -SLsf https://dl.get-arkade.dev/ | sh
  ```

- Install the OpenFaaS `app`

  ```
  # If you're using a local Kubernetes cluster or a VM, then run:
  arkade install openfaas
  
  # If you're using a managed cloud Kubernetes service which supplies LoadBalancers, then run the following:
  arkade install openfaas --load-balancer
  ```

- 启动minikube集群：`minikube start`

  - 初次启动minikube请执行:

    ```
    minikube start  --image-mirror-country cn \
    --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.0.iso \
    --registry-mirror=https://dockerhub.azk8s.cn \
    --memory=4096 
    ```

- arkade安装OpenFaaS成功的输出：

  ```
  Using kubeconfig: /home/mastlabzl/.kube/config
  Using helm3
  Node architecture: "amd64"
  Client: "x86_64", "Linux"
  2020/07/13 20:52:59 User dir established as: /home/mastlabzl/.arkade/
  "openfaas" has been added to your repositories
  
  Hang tight while we grab the latest from your chart repositories...
  ...Successfully got an update from the "openfaas" chart repository
  Update Complete. ⎈ Happy Helming!⎈
  VALUES values.yaml
  Command: /home/mastlabzl/.arkade/bin/helm3/helm [upgrade --install openfaas openfaas/openfaas --namespace openfaas --values /tmp/charts/openfaas/values.yaml --set clusterRole=false --set operator.create=false --set gateway.replicas=1 --set queueWorker.replicas=1 --set basic_auth=true --set gateway.directFunctions=true --set openfaasImagePullPolicy=IfNotPresent --set faasnetes.imagePullPolicy=Always --set basicAuthPlugin.replicas=1 --set ingressOperator.create=false --set queueWorker.maxInflight=1 --set serviceType=NodePort]
  Release "openfaas" does not exist. Installing it now.
  NAME: openfaas
  LAST DEPLOYED: Mon Jul 13 20:53:08 2020
  NAMESPACE: openfaas
  STATUS: deployed
  REVISION: 1
  TEST SUITE: None
  NOTES:
  To verify that openfaas has started, run:
  
    kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"
  =======================================================================
  = OpenFaaS has been installed.                                        =
  =======================================================================
  
  # Get the faas-cli
  curl -SLsf https://cli.openfaas.com | sudo sh
  
  # Forward the gateway to your machine
  kubectl rollout status -n openfaas deploy/gateway
  kubectl port-forward -n openfaas svc/gateway 8080:8080 &
  
  # If basic auth is enabled, you can now log into your gateway:
  PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
  echo -n $PASSWORD | faas-cli login --username admin --password-stdin
  
  faas-cli store deploy figlet
  faas-cli list
  
  # For Raspberry Pi
  faas-cli store list \
   --platform armhf
  
  faas-cli store deploy figlet \
   --platform armhf
  
  # Find out more at:
  # https://github.com/openfaas/faas
  
  Thanks for using arkade!
  ```

- 使用`kubectl get pods -n openfaas`检查部署：

  ```
  ➜  bin kubectl get pods -n openfaas
  NAME                                 READY   STATUS    RESTARTS   AGE
  alertmanager-57bd4559d7-ns9bc        1/1     Running   0          13h
  basic-auth-plugin-7d4956689b-m4cwc   1/1     Running   0          13h
  faas-idler-b85f98fb7-dsbt2           1/1     Running   3          13h
  gateway-59b667b794-47wxg             2/2     Running   0          13h
  nats-5cd4dff7c8-h6b9d                1/1     Running   0          13h
  prometheus-bcc84d4d5-td8hh           1/1     Running   0          13h
  queue-worker-6cb888d49c-4ztqx        1/1     Running   0          13
  ```


## Deploy OpenFaaS on Mac OS

- 安装Docker For Mac[Docker CE for Mac Edge Edition](https://store.docker.com/editions/community/docker-ce-desktop-mac)

- 安装OpenFaaS CLI：`curl -sLSf https://cli.openfaas.com | sudo sh`

  - 检查安装：`faas version`

- 安装Kubectl：

  ```
  export VER=$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)
  curl -LO https://storage.googleapis.com/kubernetes-release/release/$VER/bin/darwin/amd64/kubectl
  chmod +x kubectl
  mv kubectl /usr/local/bin/
  ```

- 使用k3d启动一个k8s集群：

  - 安装k3d：`brew install k3d`
  
  - 添加KUBECONFIG环境变量：`export KUBECONFIG="$(k3d get-kubeconfig --name='k3s-default')"`
  
  - 使用k3d启动一个集群：`k3d create`
  
  - 检验部署：`kubectl cluster-info`
  
    ```
    (base) ➜  /etc kubectl cluster-info
    Kubernetes master is running at https://localhost:6443
    CoreDNS is running at https://localhost:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
    Metrics-server is running at https://localhost:6443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
    
    To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
    ```
  
- 使用Arkade安装OpenFaaS：

  - 在Mac OS下安装Arkade：`curl -SLsf https://dl.get-arkade.dev/ | sudo sh`

  - 使用Arkade安装OpenFaaS：`arkade install openfaas`

  - 安装输出：

    ```
    (base) ➜  /etc arkade install openfaas
    Using kubeconfig: /Users/brightzh/.config/k3d/k3s-default/kubeconfig.yaml
    Using helm3
    Node architecture: "amd64"
    Client: "x86_64", "Darwin"
    2020/07/14 17:14:49 User dir established as: /Users/brightzh/.arkade/
    "openfaas" has been added to your repositories
    
    Hang tight while we grab the latest from your chart repositories...
    ...Successfully got an update from the "stable" chart repository
    ...Successfully got an update from the "openfaas" chart repository
    Update Complete. ⎈ Happy Helming!⎈
    VALUES values.yaml
    Command: /Users/brightzh/.arkade/bin/helm3/helm [upgrade --install openfaas openfaas/openfaas --namespace openfaas --values /var/folders/nv/cnrwk4bs17gd2t6_ns_92hjw0000gn/T/charts/openfaas/values.yaml --set faasnetes.imagePullPolicy=Always --set basicAuthPlugin.replicas=1 --set gateway.replicas=1 --set queueWorker.replicas=1 --set gateway.directFunctions=true --set operator.create=false --set openfaasImagePullPolicy=IfNotPresent --set ingressOperator.create=false --set queueWorker.maxInflight=1 --set basic_auth=true --set serviceType=NodePort --set clusterRole=false]
    Release "openfaas" does not exist. Installing it now.
    NAME: openfaas
    LAST DEPLOYED: Tue Jul 14 17:15:01 2020
    NAMESPACE: openfaas
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    To verify that openfaas has started, run:
    
      kubectl -n openfaas get deployments -l "release=openfaas, app=openfaas"
    =======================================================================
    = OpenFaaS has been installed.                                        =
    =======================================================================
    
    # Get the faas-cli
    curl -SLsf https://cli.openfaas.com | sudo sh
    
    # Forward the gateway to your machine
    kubectl rollout status -n openfaas deploy/gateway
    kubectl port-forward -n openfaas svc/gateway 8080:8080 &
    
    # If basic auth is enabled, you can now log into your gateway:
    PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
    echo -n $PASSWORD | faas-cli login --username admin --password-stdin
    
    faas-cli store deploy figlet
    faas-cli list
    
    # For Raspberry Pi
    faas-cli store list \
     --platform armhf
    
    faas-cli store deploy figlet \
     --platform armhf
    
    # Find out more at:
    # https://github.com/openfaas/faas
    
    Thanks for using arkade!
    ```

- 登陆OpenFaaS gateway：
  - `kubectl port-forward svc/gateway -n openfaas 8080:8080` 这个命令将打开一个从集群到本地计算机的隧道，以便您可以访问 OpenFaaS 网关。
  - `export OPENFAAS_URL="127.0.0.1"`
  - 生成登陆密码：`PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)`
  - 输出生成的密码：`echo -n $PASSWORD `
  - 在CLI中登陆：`faas-cli login --username admin --password-stdin` # This command logs in and saves a file to ~/.openfaas/config.yml

- 验证OpenFaaS部署完毕:
  - 列出当前OpenFaaS部署的所有函数：`faas-cli list`
- 在.bashrc中保存OpenFaaS的URL：`export OPENFAAS_URL="127.0.0.1:8080"`



## Reference

- [Deployment guide for Kubernetes - OpenFaaS](https://docs.openfaas.com/deployment/kubernetes/)
- [workshop/lab1b.md at master · openfaas/workshop](https://github.com/openfaas/workshop/blob/master/lab1b.md)

