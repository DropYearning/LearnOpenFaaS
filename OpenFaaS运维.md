# OpenFaaS运维

## OpenFaaS集群的暂停与恢复：

- 暂停：
  - 在k3d中暂停集群的运行: `k3d stop`
  - 关闭docker
- 恢复：
  - 打开docker
  - 启动k3d集群：`k3d start`
  - 映射OpenFaaS网关：`kubectl port-forward svc/gateway -n openfaas 31112:8080`
  - 映射Grafana：`kubectl port-forward deployment/grafana 3000:3000 -n openfaas`
  - 打开 http://127.0.0.1:31112/ui/ 登陆后台或者使用`faas list`验证运行正常
  - 登陆 http://127.0.0.1:3000/dashboard/db/openfaas, with username `admin` password `admin`

## OpenFaaS常用命令：

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

  