# Minikube

## Install Minikube on Ubuntu

- 1、**install docker：**
  
- [Install Docker Engine on Ubuntu | Docker Documentation](https://docs.docker.com/engine/install/ubuntu/)
  
- **2、install [VirtualBox on Ubuntu](https://phoenixnap.com/kb/install-virtualbox-on-ubuntu)**：

  - run the command:`sudo apt install virtualbox virtualbox-ext-pack`

- **3、Install Minikube**: 

  > 建议安装阿里云团队构建的版本

  - `curl -Lo minikube https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.12.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/`

  - 验证安装：`minikube version`
  - [minikube start | minikube](https://minikube.sigs.k8s.io/docs/start/)

- **4、Install Kubectl**：

  ```
  curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
  ```

  - `chmod +x ./kubectl`
  - `sudo mv ./kubectl /usr/local/bin/kubectl`
  - 验证安装：`kubectl version -o json`
  - [Install and Set Up kubectl | Kubernetes](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

- 5、**Start Minikube：**

  ```
  minikube start  --image-mirror-country cn \
  --iso-url=https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.5.0.iso \
  --registry-mirror=https://dockerhub.azk8s.cn \
  --memory=4096 
  ```

  运行输出：

  ```
  ð  Ubuntu 18.04 上的 minikube v1.12.0
  ✨  Automatically selected the docker driver
  ✅  正在使用镜像存储库 registry.cn-hangzhou.aliyuncs.com/google_containers
  ð  Starting control plane node minikube in cluster minikube
  ð  Pulling base image ...
  ð¥  Creating docker container (CPUs=2, Memory=4096MB) ...
  ð³  正在 Docker 19.03.2 中准备 Kubernetes v1.18.3…
  ð  Verifying Kubernetes components...
  ð  Enabled addons: default-storageclass, storage-provisioner
  ð  完成！kubectl 已经配置至 "minikube"
  ```

  

### Minikube安装参考

- [How to Install Minikube on Ubuntu 18.04 / 20.04](https://phoenixnap.com/kb/install-minikube-on-ubuntu)
- [Minikube - Kubernetes本地实验环境-阿里云开发者社区](https://developer.aliyun.com/article/221687)



## Install Minikube on MacOS

Wait to be done...



## minikube操作命令

- check the status of the cluster: `minikube status`

  - If your cluster is running, the output from `minikube status` should be similar to:

    ```
    host: Running
    kubelet: Running
    apiserver: Running
    kubeconfig: Configured
    ```

-  stop your cluster, run:`minikube stop`

-  clear minikube's local state:`minikube delete`

## Mirror in China

Since some container registries like `docker.io`, `gcr.io` are not accessible or very slow in China, we have set up container registry proxy servers for Azure China VMs:

> **Note**: currently *.azk8s.cn could only be accessed by Azure China IP, we don't provide public outside access any more. 

| global                                                       | proxy in China                                               | format                                                  | example                                                      |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------- | ------------------------------------------------------------ |
| [dockerhub](https://github.com/Azure/container-service-for-azure-china/blob/master/aks/hub.docker.com) (docker.io) | [dockerhub.azk8s.cn](http://mirror.azk8s.cn/help/docker-registry-proxy-cache.html) | `dockerhub.azk8s.cn/<repo-name>/<image-name>:<version>` | `dockerhub.azk8s.cn/microsoft/azure-cli:2.0.61` `dockerhub.azk8s.cn/library/nginx:1.15` |
| gcr.io                                                       | [gcr.azk8s.cn](http://mirror.azk8s.cn/help/gcr-proxy-cache.html) | `gcr.azk8s.cn/<repo-name>/<image-name>:<version>`       | `gcr.azk8s.cn/google_containers/hyperkube-amd64:v1.13.5`     |
| quay.io                                                      | [quay.azk8s.cn](http://mirror.azk8s.cn/help/quay-proxy-cache.html) | `quay.azk8s.cn/<repo-name>/<image-name>:<version>`      | `quay.azk8s.cn/deis/go-dev:v1.10.0`                          |
| mcr.microsoft.com                                            | mcr.azk8s.cn                                                 | `mcr.azk8s.cn/<repo-name>/<image-name>:<version>`       | `mcr.azk8s.cn/oss/kubernetes/hyperkube:v1.15.7`              |

- [container-service-for-azure-china/README.md at master · Azure/container-service-for-azure-china](https://github.com/Azure/container-service-for-azure-china/blob/master/aks/README.md)