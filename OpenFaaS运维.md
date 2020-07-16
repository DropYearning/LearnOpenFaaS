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

## OpenFaaS常用命令

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

- 搜索Template Store：`faas-cli template store list -v` [需要搜索github,可能需要打开终端代理]

  - 查看某一个模板的详细内容:`faas-cli template store describe node10-express`

  - 拉取模板：`faas-cli template store pull node10-express`

    

## Kubernetes Lens的使用

- [Lens | The Kubernetes IDE](https://k8slens.dev/)是一款一个强大的 IDE，可以实时查看集群状态，实时查看日志流，方便排查故障。有了 `Lens`，你可以更方便快捷地使用你的集群，从根本上提高工作效率和业务迭代速度。
- 添加集群：
  - add clustet -> custom
  - 使用命令`k3d get-kubeconfig`获取集群使用的kubeconfig.yaml位置
  - 复制其内容至文本框内
- 添加成功后可以使用Lens监视k8s集群的状态
  - ![image-20200714223134706](/Users/brightzh/Library/Application Support/typora-user-images/image-20200714223134706.png)

- 参考
  - [Lens —— 最炫酷的 Kubernetes 桌面客户端 - 掘金](https://juejin.im/post/5ef2f503f265da02c8745f7d)
  - 