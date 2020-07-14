# OpenFaaS Workshop

## Lab 1 - Prepare for OpenFaaS

[workshop/lab1.md at master · openfaas/workshop](https://github.com/openfaas/workshop/blob/master/lab1.md)

see 

## Lab 2 - Test things out

- `Mkdir lab2`

### Use the UI Portal

- 部署https://raw.githubusercontent.com/openfaas/faas/master/stack.yml中的实例函数

  ```
  faas-cli deploy -f https://raw.githubusercontent.com/openfaas/faas/master/stack.yml
  ```

  > 如果修改了端口，需要手动拉取yaml文件，修改其中的端口之后再部署。

- ![OHJMlA](https://gitee.com/pxqp9W/testmarkdown/raw/master/imgs/2020/07/OHJMlA.png)
  - Status - whether the function is ready to run. You will not be able to invoke the function from the UI until the status shows Ready.
  - Replicas - the amount of replicas of your function running in the cluster
  - Image - the Docker image name and version as published to the Docker Hub or Docker repository
  - Invocation count - this shows how many times the function has been invoked and is updated every 5 seconds

### Deploy via the Function Store

- 可以直接在UI中部署OpenFaaS Store中的函数

### Learn about the CLI

- 列出所有函数：`faas-cli list`，同时显示被调用的次数和副本数量

- 列出OpenFaaS中的所有pods：`kubectl get pods -n openfaas`

- verbose list：`faas-cli list --verbose`或`faas list -v`，比`faas-cli list`额外输出一列images

- 在CLI中调用函数：

  ```
  (base) ➜  lab2 git:(master) ✗ faas invoke figlet
  Reading from STDIN - hit (Control + D) to stop.
  123
   _ ____  _____
  / |___ \|___ /
  | | __) | |_ \
  | |/ __/ ___) |
  |_|_____|____/
  
  # 或者使用前一个命令的输出作为 faas invoke的输入
   uname -a | faas-cli invoke figlet
   cat lab2.md | faas-cli invoke markdown
   echo Hi | faas-cli invoke markdown
  ```

  

### Monitoring dashboard

OpenFaaS使用Prometheus自动跟踪您的功能指标。 借助Grafana等免费和开源工具，这些指标可以变成有用的dashboad。

- 在OpenFaaS Kubernetes命名空间中运行Grafana：

  ```
  kubectl -n openfaas run \
  --image=stefanprodan/faas-grafana:4.6.3 \
  --port=3000 \
  grafana
  ```

- 使用 NodePort 暴露 Grafana：

  ```
  kubectl -n openfaas expose pod grafana \
  --type=NodePort \
  --name=grafana
  ```

  > grafana pod名字不一定是grafana，需要使用命令`kubectl get pods -n openfaas`来查看grafana的具体名字。

- Grafana节点端口地址：

  ```
  $ GRAFANA_PORT=$(kubectl -n openfaas get svc grafana -o jsonpath="{.spec.ports[0].nodePort}")
  $ GRAFANA_URL=http://IP_ADDRESS:$GRAFANA_PORT/dashboard/db/openfaas
  ```

- 映射grafana端口：`kubectl port-forward deployment/grafana 3000:3000 -n openfaas`
- 访问 http://127.0.0.1:3000/dashboard/db/openfaas with username `admin` password `admin`
- ![img](https://camo.githubusercontent.com/24915ac87ecf8a31285f273846e7a5ffe82eeceb/68747470733a2f2f7062732e7477696d672e636f6d2f6d656469612f4339636145364358554141585f36342e6a70673a6c61726765)