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

## Lab 3 - Introduction to functions

- 创建新函数有两种方法:
  - 使用内置或社区代码模板搭建一个函数(默认) [openfaas/templates: OpenFaaS Classic templates](https://github.com/openfaas/templates)
  - 使用现有的二进制文件作为函数(高级)

### hello-world function in Python

```
(base) ➜  lab3 git:(master) ✗ faas new --lang python3 hello-openfaas --prefix="<your-docker-username-here>"
Folder: hello-openfaas created.
  ___                   _____           ____
 / _ \ _ __   ___ _ __ |  ___|_ _  __ _/ ___|
| | | | '_ \ / _ \ '_ \| |_ / _` |/ _` \___ \
| |_| | |_) |  __/ | | |  _| (_| | (_| |___) |
 \___/| .__/ \___|_| |_|_|  \__,_|\__,_|____/
      |_|


Function created in folder: hello-openfaas
Stack file written: hello-openfaas.yml

Notes:
You have created a Python3 function using the Classic Watchdog.

To include third-party dependencies create a requirements.txt file.

For high-throughput applications, we recommend using the python3-flask
or python3-http templates.
```

- 该创建命令生成一下文件：

  ```
  ./hello-openfaas.yml //This file is used to manage the function - it has the name of the function, the Docker image and any other customisations needed.
  ./hello-openfaas 
  ./hello-openfaas/handler.py //The handler for the function - you get a req object with the raw request and can print the result of the function to the console.
  ./hello-openfaas/requirements.txt //Use this file to list any pip modules you want to install, such as requests or urllib
  ```

- hello-openfaas.yaml：

  ```
  version: 1.0
  provider:
    name: openfaas
    gateway: http://127.0.0.1:31112
  functions:
    hello-openfaas: # 函数名字
      lang: python3 # 编程语言
      handler: ./hello-openfaas # The folder used to build from is called handler, this must be a folder not a file
      image: hello-openfaas:latest # The Docker image name to be used is under the field image
  ```

- 修改./hello-openfaas/handler.py，使其返回Hello OpenFaaS：

  ```
  def handle(req):
      """handle a request to the function
      Args:
          req (str): request body
      """
  
      return "Hello OpenFaaS"
  ```

- `faas-cli up` command combines build, push and deploy commands of `faas-cli` in a single command：`faas-cli up -f hello-openfaas.yml`

  ```
  Deploying: hello-openfaas.
  WARNING! Communication is not secure, please consider using HTTPS. Letsencrypt.org offers free SSL/TLS certificates.
  
  Deployed. 202 Accepted.
  URL: http://127.0.0.1:31112/function/hello-openfaas.openfaas-fn
  ```

- `faas invoke hello-openfaas`

### Example function: astronaut-finder

- 本实例将创建一个名为“宇航员搜索者”的函数，它可以随机抽取国际空间站(ISS)上某个太空人的名字。

- 在./astronaut-finder/requirements.txt中添加pip模块`requests`

- 修改handler.py：

  ```python
  import requests
  import random
  
  def handle(req):
      r = requests.get("http://api.open-notify.org/astros.json")
      result = r.json()
      index = random.randint(0, len(result["people"])-1)
      name = result["people"][index]["name"]
  
      return "%s is in space" % (name)
  ```

- build the function:`faas-cli build -f ./astronaut-finder.yml`

  > 如果yaml文件名为stack.yaml，则只需要使用命令`faas build`即可，`stack.yml` is the default file-name for the CLI.

- Push the function to dockerhub:`faas-cli push -f ./astronaut-finder.yml`

- Push the function to openfaas: `faas-cli deploy -f ./astronaut-finder.yml`

- `faas-cli invoke astronaut-finder`

  ```
  (base) ➜  lab3 git:(master) ✗ faas invoke astronaut-finder
  Reading from STDIN - hit (Control + D) to stop.
  Anatoly Ivanishin is in space
  (base) ➜  lab3 git:(master) ✗ faas invoke astronaut-finder
  Reading from STDIN - hit (Control + D) to stop.
  Ivan Vagner is in space
  ```

### Troubleshooting: find the container's logs

- 对于k8s部署的openfaas，查看日志：`kubectl logs deployment/astronaut-finder -n openfaas-fn`

  ```
  (base) ➜  lab3 git:(master) ✗ kubectl logs deployment/astronaut-finder -n openfaas-fn
  2020/07/15 01:31:11 Version: 0.18.1	SHA: b46be5a4d9d9d55da9c4b1e50d86346e0afccf2d
  2020/07/15 01:31:11 Timeouts: read: 5s, write: 5s hard: 0s.
  2020/07/15 01:31:11 Listening on port: 8080
  2020/07/15 01:31:11 Writing lock-file to: /tmp/.lock
  2020/07/15 01:31:11 Metrics listening on port: 8081
  2020/07/15 01:34:37 Forking fprocess.
  2020/07/15 01:34:38 Wrote 24 Bytes - Duration: 1.445268 seconds
  2020/07/15 01:34:40 Forking fprocess.
  2020/07/15 01:34:41 Wrote 26 Bytes - Duration: 0.990424 seconds
  2020/07/15 01:36:18 Forking fprocess.
  2020/07/15 01:36:19 Wrote 24 Bytes - Duration: 1.079265 seconds
  2020/07/15 01:36:48 Forking fprocess.
  2020/07/15 01:36:49 Wrote 30 Bytes - Duration: 0.874071 seconds
  2020/07/15 01:36:52 Forking fprocess.
  2020/07/15 01:36:53 Wrote 24 Bytes - Duration: 1.072044 seconds
  2020/07/15 01:44:56 Forking fprocess.
  2020/07/15 01:44:57 Wrote 24 Bytes - Duration: 1.433524 seconds
  ```

### Troubleshooting: verbose output with write_debug

- 可以配置函数的YAML文件，使其在被调用时反馈verbose信息

- 例如修改 astronaut-finder的YAML文件：

  ```yaml
  version: 1.0
  provider:
    name: openfaas
    gateway: http://127.0.0.1:31112
  functions:
    astronaut-finder:
      lang: python3
      handler: ./astronaut-finder
      image: dockerbbbz/astronaut-finder:latest
      environment:
        write_debug: true
  ```

- 重新部署:`faas-cli deploy -f ./astronaut-finder.yml`

- 再次调用 astronaut-finder函数

- 重新查看kubectl日志：`kubectl logs deployment/astronaut-finder -n openfaas-fn`，可以输出详细的运行日志，包括python的报错内容：

  ```
  2020/07/15 01:49:30 Forking fprocess.
  2020/07/15 01:49:30 Query
  2020/07/15 01:49:30 Path  /
  2020/07/15 01:49:37 Success=false, Error=exit status 1
  2020/07/15 01:49:37 Out=Traceback (most recent call last):
    File "/home/app/python/urllib3/connectionpool.py", line 670, in urlopen
      httplib_response = self._make_request(
    File "/home/app/python/urllib3/connectionpool.py", line 426, in _make_request
      six.raise_from(e, None)
    File "<string>", line 3, in raise_from
    File "/home/app/python/urllib3/connectionpool.py", line 421, in _make_request
      httplib_response = conn.getresponse()
    File "/usr/local/lib/python3.8/http/client.py", line 1332, in getresponse
      response.begin()
    File "/usr/local/lib/python3.8/http/client.py", line 303, in begin
      version, status, reason = self._read_status()
    File "/usr/local/lib/python3.8/http/client.py", line 272, in _read_status
      raise RemoteDisconnected("Remote end closed connection without"
  http.client.RemoteDisconnected: Remote end closed connection without response
  
  During handling of the above exception, another exception occurred:
  
  Traceback (most recent call last):
    File "/home/app/python/requests/adapters.py", line 439, in send
      resp = conn.urlopen(
    File "/home/app/python/urllib3/connectionpool.py", line 724, in urlopen
      retries = retries.increment(
    File "/home/app/python/urllib3/util/retry.py", line 403, in increment
      raise six.reraise(type(error), error, _stacktrace)
    File "/home/app/python/urllib3/packages/six.py", line 734, in reraise
      raise value.with_traceback(tb)
    File "/home/app/python/urllib3/connectionpool.py", line 670, in urlopen
      httplib_response = self._make_request(
    File "/home/app/python/urllib3/connectionpool.py", line 426, in _make_request
      six.raise_from(e, None)
    File "<string>", line 3, in raise_from
    File "/home/app/python/urllib3/connectionpool.py", line 421, in _make_request
      httplib_response = conn.getresponse()
    File "/usr/local/lib/python3.8/http/client.py", line 1332, in getresponse
      response.begin()
    File "/usr/local/lib/python3.8/http/client.py", line 303, in begin
      version, status, reason = self._read_status()
    File "/usr/local/lib/python3.8/http/client.py", line 272, in _read_status
      raise RemoteDisconnected("Remote end closed connection without"
  urllib3.exceptions.ProtocolError: ('Connection aborted.', RemoteDisconnected('Remote end closed connection without response'))
  
  During handling of the above exception, another exception occurred:
  
  Traceback (most recent call last):
    File "index.py", line 19, in <module>
      ret = handler.handle(st)
    File "/home/app/function/handler.py", line 5, in handle
      r = requests.get("http://api.open-notify.org/astros.json")
    File "/home/app/python/requests/api.py", line 76, in get
      return request('get', url, params=params, **kwargs)
    File "/home/app/python/requests/api.py", line 61, in request
      return session.request(method=method, url=url, **kwargs)
    File "/home/app/python/requests/sessions.py", line 530, in request
      resp = self.send(prep, **send_kwargs)
    File "/home/app/python/requests/sessions.py", line 643, in send
      r = adapter.send(request, **kwargs)
    File "/home/app/python/requests/adapters.py", line 498, in send
      raise ConnectionError(err, request=request)
  requests.exceptions.ConnectionError: ('Connection aborted.', RemoteDisconnected('Remote end closed connection without response'))
  
  2020/07/15 01:49:46 Forking fprocess.
  2020/07/15 01:49:46 Query
  2020/07/15 01:49:46 Path  /
  2020/07/15 01:49:47 Duration: 0.908570 seconds
  Bob Behnken is in space
  ```

### Managing multiple functions

- CLI的YAML文件允许将函数组合到一起，这在使用一组连续的函数调用时非常有用

- 创建第一个函数：`faas-cli new --lang python3 first`

- 使用参数`--append`创建第二个函数：`faas-cli new --lang python3 second --append=./first.yml`

- 可以发现， 创建第二个函数时不会生成`./seconed.yaml`，而是会在`./first.yaml`后追加相应内容

- 为了方便起见，可以修改 `first.yml` 为 `example.yml`：`mv first.yml example.yml`

  ```yml
  version: 1.0
  provider:
    name: openfaas
    gateway: http://127.0.0.1:31112
  functions:
    first:
      lang: python3
      handler: ./first
      image: first:latest
  
    second:
      lang: python3
      handler: ./second
      image: second:latest
  ```

- 在CLI中构建函数栈（stack of functions）时可以使用以下参数：
  - 并行构建多个函数：`faas-cli build -f ./example.yml --parallel=2`
  - 只构建某一个函数：`faas-cli build -f ./example.yml --filter=second`
  - 通过HTTP(s)部署远程功能堆栈（yaml）文件：``faas-cli deploy -f https://....`

### Variable Substitution in YAML File (optional exercise)

### Custom binaries as functions (optional exercise)

- 可以将自定义二进制文件或容器用作功能，但是大多数时候使用语言模板应涵盖所有最常见的情况。

- Any binary or existing container can be made a serverless function by adding the OpenFaaS function watchdog.

  > OpenFaaS supports Windows binaries too? Like C#, VB or PowerShell

## Lab4 Go deeper with functions

### Inject configuration through environmental variables

- 能够控制函数在运行时的行为非常有用，我们可以通过至少两种方式做到这一点：

  - **Set environmental variables at deployment time**：We did this with `write_debug` in [Lab 3](https://github.com/openfaas/workshop/blob/master/lab3.md) - you can also set any custom environmental variables you want here too - for instance if you wanted to configure a language for your *hello world* function you may introduce a `spoken_language` variable.

  - **Use HTTP context - querystring / headers**：The other option which is more dynamic and can be altered at a per-request level is the use of querystrings and HTTP headers, both can be passed through the `faas-cli` or `curl`.

    > These headers become exposed through environmental variables so they are easy to consume within your function. So any header is prefixed with `Http_` and all `-` hyphens are replaced with an `_` underscore.

- 例子：lists off all environmental variables

  - 目标：Deploy a function that prints environmental variables using a built-in BusyBox command:

  - `faas-cli deploy --name env --fprocess="env" --image="functions/alpine:latest"`

  - 使用CLI打印所有环境变量：`echo "" | faas-cli invoke env --query workshop=1`

    ```
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    HOSTNAME=env-5999678959-w9fjh
    fprocess=env
    HUBSTATS_PORT=tcp://10.43.25.25:8080
    ASTRONAUT_FINDER_SERVICE_HOST=10.43.137.146
    WORDCOUNT_PORT_8080_TCP_PROTO=tcp
    FIGLET_PORT_8080_TCP_PORT=8080
    MARKDOWN_PORT_8080_TCP_PROTO=tcp
    HELLO_OPENFAAS_PORT_8080_TCP_PROTO=tcp
    ENV_SERVICE_PORT_HTTP=8080
    KUBERNETES_PORT_443_TCP_ADDR=10.43.0.1
    FIGLET_PORT_8080_TCP_PROTO=tcp
    ENV_SERVICE_HOST=10.43.163.164
    HUBSTATS_PORT_8080_TCP_PROTO=tcp
    ASTRONAUT_FINDER_PORT=tcp://10.43.137.146:8080
    ASTRONAUT_FINDER_PORT_8080_TCP_ADDR=10.43.137.146
    MARKDOWN_PORT=tcp://10.43.234.169:8080
    HELLO_OPENFAAS_PORT_8080_TCP=tcp://10.43.17.182:8080
    ENV_PORT_8080_TCP=tcp://10.43.163.164:8080
    KUBERNETES_PORT_443_TCP_PROTO=tcp
    WORDCOUNT_SERVICE_PORT=8080
    WORDCOUNT_PORT=tcp://10.43.68.210:8080
    NODEINFO_SERVICE_PORT=8080
    NODEINFO_PORT_8080_TCP_PROTO=tcp
    HUBSTATS_SERVICE_PORT=8080
    HUBSTATS_PORT_8080_TCP=tcp://10.43.25.25:8080
    MARKDOWN_PORT_8080_TCP=tcp://10.43.234.169:8080
    HUBSTATS_PORT_8080_TCP_ADDR=10.43.25.25
    KUBERNETES_PORT_443_TCP=tcp://10.43.0.1:443
    BASE64_PORT_8080_TCP_ADDR=10.43.193.94
    BASE64_SERVICE_HOST=10.43.193.94
    BASE64_PORT_8080_TCP=tcp://10.43.193.94:8080
    BASE64_PORT_8080_TCP_PROTO=tcp
    BASE64_PORT_8080_TCP_PORT=8080
    NODEINFO_PORT_8080_TCP=tcp://10.43.153.161:8080
    ECHOIT_PORT_8080_TCP=tcp://10.43.70.57:8080
    ASTRONAUT_FINDER_SERVICE_PORT=8080
    ASTRONAUT_FINDER_PORT_8080_TCP_PROTO=tcp
    WORDCOUNT_PORT_8080_TCP=tcp://10.43.68.210:8080
    FIGLET_PORT=tcp://10.43.69.188:8080
    FIGLET_SERVICE_PORT_HTTP=8080
    HELLO_OPENFAAS_SERVICE_PORT=8080
    ENV_PORT_8080_TCP_PORT=8080
    ASTRONAUT_FINDER_PORT_8080_TCP_PORT=8080
    KUBERNETES_PORT=tcp://10.43.0.1:443
    BASE64_SERVICE_PORT_HTTP=8080
    WORDCOUNT_SERVICE_PORT_HTTP=8080
    FIGLET_SERVICE_HOST=10.43.69.188
    HELLO_OPENFAAS_PORT=tcp://10.43.17.182:8080
    ENV_PORT_8080_TCP_ADDR=10.43.163.164
    HUBSTATS_SERVICE_HOST=10.43.25.25
    HUBSTATS_PORT_8080_TCP_PORT=8080
    MARKDOWN_SERVICE_PORT=8080
    NODEINFO_SERVICE_HOST=10.43.153.161
    KUBERNETES_SERVICE_PORT_HTTPS=443
    WORDCOUNT_PORT_8080_TCP_ADDR=10.43.68.210
    ECHOIT_SERVICE_HOST=10.43.70.57
    KUBERNETES_PORT_443_TCP_PORT=443
    MARKDOWN_SERVICE_HOST=10.43.234.169
    MARKDOWN_PORT_8080_TCP_PORT=8080
    HELLO_OPENFAAS_SERVICE_PORT_HTTP=8080
    HELLO_OPENFAAS_PORT_8080_TCP_ADDR=10.43.17.182
    ASTRONAUT_FINDER_SERVICE_PORT_HTTP=8080
    BASE64_PORT=tcp://10.43.193.94:8080
    WORDCOUNT_PORT_8080_TCP_PORT=8080
    FIGLET_PORT_8080_TCP_ADDR=10.43.69.188
    HELLO_OPENFAAS_SERVICE_HOST=10.43.17.182
    ENV_PORT=tcp://10.43.163.164:8080
    NODEINFO_PORT_8080_TCP_PORT=8080
    ECHOIT_PORT=tcp://10.43.70.57:8080
    WORDCOUNT_SERVICE_HOST=10.43.68.210
    FIGLET_SERVICE_PORT=8080
    HUBSTATS_SERVICE_PORT_HTTP=8080
    ECHOIT_SERVICE_PORT_HTTP=8080
    ECHOIT_PORT_8080_TCP_PROTO=tcp
    ECHOIT_PORT_8080_TCP_ADDR=10.43.70.57
    KUBERNETES_SERVICE_HOST=10.43.0.1
    HELLO_OPENFAAS_PORT_8080_TCP_PORT=8080
    ENV_SERVICE_PORT=8080
    ENV_PORT_8080_TCP_PROTO=tcp
    ECHOIT_SERVICE_PORT=8080
    BASE64_SERVICE_PORT=8080
    NODEINFO_SERVICE_PORT_HTTP=8080
    NODEINFO_PORT=tcp://10.43.153.161:8080
    NODEINFO_PORT_8080_TCP_ADDR=10.43.153.161
    KUBERNETES_SERVICE_PORT=443
    FIGLET_PORT_8080_TCP=tcp://10.43.69.188:8080
    MARKDOWN_SERVICE_PORT_HTTP=8080
    MARKDOWN_PORT_8080_TCP_ADDR=10.43.234.169
    ECHOIT_PORT_8080_TCP_PORT=8080
    ASTRONAUT_FINDER_PORT_8080_TCP=tcp://10.43.137.146:8080
    HOME=/home/app
    Http_X_Forwarded_For=127.0.0.1:48298
    Http_X_Forwarded_Host=127.0.0.1:31112
    Http_User_Agent=Go-http-client/1.1
    Http_Accept_Encoding=gzip
    Http_Content_Type=text/plain
    Http_Method=POST
    Http_ContentLength=-1
    Http_Content_Length=-1
    Http_Query=workshop=1
    Http_Path=/
    Http_Host=env.openfaas-fn.svc.cluster.local:8080
    ```

  - 使用curl打印所有环境变量：`curl -X GET $OPENFAAS_URL/function/env/some/path -d ""`

    - `curl -X`：指定是什么命令（POST、GET...）
    - `curl -d`：使用POST方式发送数据

  - 可以在curl请求中添加header：`curl $OPENFAAS_URL/function/env --header "X-Output-Mode: json" -d ""`

    - `curl -H` or `curl --header`：自定义头信息传递给服务器

  - 如果是在Python代码中也可以使用 `os.getenv("Http_X_Output_Mode")`

### Security: read-only filesystems

- OpenFaaS 使用容器的安全特性之一是能够使我们的执行环境的根文件系统**只读**

- 例子：生成一个函数将文件保存到函数的文件系统

  - `faas-cli new --lang python3 ingest-file --prefix=your-name`

  - 修改handler.py:

    ```python
    import os
    import time
    
    def handle(req):
        # Read the path or a default from environment variable
        path = os.getenv("save_path", "/home/app/") # 如果存在，返回环境变量 key 的值，否则返回 default。 key ， default 和返回值均为 str 字符串类型。
    
        # generate a name using the current timestamp
        t = time.time()
        file_name = path + str(t)
    
        # write a file
        with open(file_name, "w") as f:
            f.write(req)
            f.close()
    
        return file_name
    ```

  - Build the example:`faas-cli up -f ingest-file.yml`

  - Invoke:`echo "Hello function" > message.txt`

    ```
    (base) ➜  lab4 git:(master) ✗ cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
    /home/app/1594781697.7065554 // 写入成功
    ```

- 可以在YML文件中设置该函数对应的文件系统为只读：

  ```yml
  version: 1.0
  provider:
    name: openfaas
    gateway: http://127.0.0.1:31112
  functions:
    ingest-file:
      lang: python3
      handler: ./ingest-file
      image: dockerbbbz/ingest-file:latest
      readonly_root_filesystem: true
  ```

  - 再次部署：`faas-cli up -f ingest-file.yml`

  - 只读文件系统不能被写入:

    ```
    Server returned unexpected status code: 500 - exit status 1
    Traceback (most recent call last):
      File "index.py", line 19, in <module>
        ret = handler.handle(st)
      File "/home/app/function/handler.py", line 13, in handle
        with open(file_name, "w") as f:
    OSError: [Errno 30] Read-only file system: '/home/app/1594782267.3650436'
    ```

- 但是可以为函数设置一个临时的空间来存放临时文件：

  - 修改YML，添加save_path字段：

    ```yml
    version: 1.0
    provider:
      name: openfaas
      gateway: http://127.0.0.1:31112
    functions:
      ingest-file:
        lang: python3
        handler: ./ingest-file
        image: dockerbbbz/ingest-file:latest
        readonly_root_filesystem: true
        environment:
            save_path: "/tmp/"
    ```

  - 再次部署:`faas-cli up -f ingest-file.yml`

  - 因为在YAML中设置了环境变量save_path，之后所有的文件都将被写入/tmp/：

    ```
    (base) ➜  lab4 git:(master) ✗ cat message.txt | faas-cli invoke -f ingest-file.yml ingest-file
    /tmp/1594782503.402575
    ```

### Making use of logging

- OpenFaaS 看门狗通过传入 HTTP 请求并通过标准 I/O 流 stdin 和 stdout 读取 HTTP 响应来进行操作。这意味着作为一个函数运行的进程不需要知道任何关于 web 或 HTTP 的信息。

- 但是当函数错误运行，stderr非空时，stderr信息不会被打印到日志中，而是会输出在调用结果中，这会影响函数的输出格式：

  - 例如，修改lab3中的hello-openfaas：

    ```python
    import sys
    import json
    
    def handle(req):
        sys.stderr.write("This should be an error message.\n")
        return json.dumps({"Hello": "OpenFaaS"})
    ```

  - 再次调用该函数，返回：

    ```
    This should be an error message.
    {"Hello": "OpenFaaS"}
    ```

  - 查看日志：`kubectl logs deployment/hello-openfaas -n openfaas-fn`，日志中不会显示stderr的信息

- OpenFaaS提供了一种解决方案，可以将错误消息打印到日志并保持函数响应结果清晰，仅返回标准输出。 为此，应使用`combining_output`标志。

  - 修改hello-openfaas.yml:

    ```yml
    version: 1.0
    provider:
      name: openfaas
      gateway: http://127.0.0.1:31112
    functions:
      hello-openfaas:
        lang: python3
        handler: ./hello-openfaas
        image: dockerbbbz/hello-openfaas:latest
        environment:
          combine_output: false
    ```

  - 重新部署：`faas-cli up -f hello-openfaas.yml`

  - 重新调用和打印日志：

    ```
    (base) ➜  lab3 git:(master) ✗ echo | faas-cli invoke hello-openfaas
    {"Hello": "OpenFaaS"}
    (base) ➜  lab3 git:(master) ✗ kubectl logs deployment/hello-openfaas -n openfaas-fn
    2020/07/15 03:22:26 Version: 0.18.1	SHA: b46be5a4d9d9d55da9c4b1e50d86346e0afccf2d
    2020/07/15 03:22:26 Timeouts: read: 5s, write: 5s hard: 0s.
    2020/07/15 03:22:26 Listening on port: 8080
    2020/07/15 03:22:26 Writing lock-file to: /tmp/.lock
    2020/07/15 03:22:26 Metrics listening on port: 8081
    2020/07/15 03:22:53 Forking fprocess.
    2020/07/15 03:22:53 stderr: This should be an error message. //stderr信息
    2020/07/15 03:22:53 Wrote 22 Bytes - Duration: 0.104804 seconds
    ```

### Create Workflows

- 有时候我们需要串接多个函数作为一个工作流，其中前一个函数的输出将作为后一个函数的输入

- 有两种方式可以实现函数的Workflows：

  - 第一种方式：通过在客户端调用时将前一个函数的输出作为后一个函数的输入来实现

    - Example：例如将函数Nodeinfo的输出作为Markdown converter的输入

      ```
      (base) ➜  lab3 git:(master) ✗ echo -n "" | faas-cli invoke nodeinfo | faas-cli invoke markdown
      <p>Hostname: nodeinfo-6d58b7fb8f-9d5dk</p>
      
      <p>Platform: linux
      Arch: x64
      CPU count: 4
      Uptime: 21733</p>
      ```

    - Pros:

      - requires no code - can be done with CLI programs
      - fast for development and testing
      - easy to model in code

      Cons:

      - additional latency - each function goes back to the server
      - chatty (more messages)

  - 第二种方式：通过在后一个函数的代码中调用第一个函数来实现

    - 当从函数访问 API 网关等服务时，使用环境变量配置主机名是最佳实践，这一点很重要，原因有二: 名称可能会改变，在 Kubernetes 有时需要一个后缀。
    - Examples：
      - 在实验3中，我们引入了请求模块，并使用它调用一个远程 API 来获取国际空间站上宇航员的名字。我们可以使用相同的技术来调用部署在 OpenFaaS 上的另一个函数。
      - 在这里我们部署另外一个函数SentimentAnalysis，该函数可以用于分析句子的情绪是积极的还是消极的。`faas-cli store deploy SentimentAnalysis`
    - Pros:
      - functions can make use of each other directly
      - low latency since the functions can access each other on the same network
    - Cons:
      - requires a code library for making the HTTP request

