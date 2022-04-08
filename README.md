> 该项目是通过golang实现的日志收集
> Email：2685246582@qq.com

# 1 环境准备

- windows10
- go1.17
- kafka-3.1.0（包含zookeeper）
- kibana-7.8.0
- elasticsearch-7.8.0
- etcd-3.5.2
- influxdb-2.1.1
- grafana-8.4.5

# 2 项目架构图

## 2.1 总架构图

![日志收集系统架构图 (1)](https://gitee.com/kkite/blogimg/raw/master/202204081400664.png)

- 该图为整个项目的架构图
- 虚线表示还未实现的可扩展功能

## 2.2 LogAgent工作流程

![LogAgent工作流程](https://gitee.com/kkite/blogimg/raw/master/202204081725359.png)

## 2.3 LogTransfer工作流程

![LogTransfer工作流程](https://gitee.com/kkite/blogimg/raw/master/202204081522987.png)

## 2.4 SysTransfer工作流程

![SysTransfer工作流程](https://gitee.com/kkite/blogimg/raw/master/202204081522834.png)

# 3 项目实现功能

- `etcd `: 存放各个节点的配置信息，实现各个节点的配置管理。
- `Log Agent` : 实现从节点收集日志，并发送到指定的 `kafka` 的 `topic`。
- `kafka` : 存放来自各个节点的日志（数据持久化）。
- `log transfer` : 消费 `kafka` 上指定 `topic` 的日志，并发往 `ElasticSearch`。
- `ElasticSearch` : 对日志建立索引，存放日志信息。
- `Kibana` : 提供友好的web页面实现对`ElasticSearch` 中数据的检索与展示。

**其他功能**

- `Sys Agent` : 收集节点上的`cpu`、`disk`、`net`等信息，然后发送到kafka
- `sys transfer` : 从 `kafka` 拿到系统信息数据，然后发送到`influxDB` 
- `influxDB `: 时序数据库，收到从 `sys transfer` 发来的系统数据
- `Grafana` : 从 `influxdb` 查询到数据，并在web页面进行展示，实现实时监控各个节点系统信息的功能。

> 注意：本项目并没有完全实现其他功能，只实现了通过 sys agent 收集系统数据，发送到influxdb，最后通过 grafana展示的功能。

# 4 项目启动

> 可能一些服务下载后需要修改配置，下面就没细说了
>
> 服务启动：在各个解压后的文件下，进入cmd启动，注意路径

## 4.1 从各个官网上下载好对应的服务

## 4.2 项目下载

下载地址：https://github.com/ZhanfeiKang/logCollection.git

## 4.3 启动各项服务

1. 启动zookeeper

   ```bash
   bin\windows\zookeeper-server-start.bat config\zookeeper.properties
   ```

2. 启动kafka

   ```bash
   bin\windows\kafka-server-start.bat config\server.properties
   ```

3. 启动etcd

   ```bash
   etcd.exe
   ```

4. 启动es

   ```bash
   bin\elasticsearch.bat
   ```

5. 启动kibana

   ```bash
   bin\kibana.bat
   ```

## 4.4 打开项目工程

**1. etcd_demo文件：**

- 修改存储在etcd中的日志配置项信息（path,topic）
- path：日志文件在主机上的存放位置
- topic：该path的文件对应kafka中的topic

**2. logagent文件：**

- 修改 `conf` 下的 `config.ini` 配置文件：
- kafka：
  - address：kafka服务的地址
  - chan_size：channel大小
- etcd：
  - address：kafka服务的地址
  - collect_key：主机配置项，对应存储在的etcd中的key

**3. 运行 etcd_demo：**

- 运行`etcd_demo`下的`main.go`文件
- 注意写入到etcd中的`path`-`topic`

**4. 运行-收集日志发往kafka：**

- 运行`loagent`下的`main.go`，即可监听对应的日志文件

- 测试：

  - 找到第3步path路径下的log文件，然后打开文件

  - 在该log文件中随便写入内容回车保存

  - 可观察`logagent`命令行中是否有数据发送到kafka

    > 也可以运行kafka自带的消费者从kafka消费写入的log信息，进行验证

**5. `log_transfer`文件：**

- 修改 `config` 下的 `logtransfer.ini` 配置文件：
- kafka：
  - address：kafka服务的地址
  - topic：从kafka中消费的topic
- es：
  - address：elasticsearch服务的地址
  - index：发送到es对应的index
  - max_chan_size：管道大小
  - goroutine_num：开多少个协程去消费

**6. 运行-从kafka中消费信息，然后发送到es：**

- 运行 `log_transfer` 下的 `main.go` 文件，即可从kafka对应的topic中消费，然后发送到es

- 测试：

  - 保持第4步程序的运行
  - 然后执行第4步中的测试，在对应的log文件写入信息
    - 注意：写入json格式的数据，因为要保存到es中去

  - 在网页访问 `kibana` 服务，检索对应的 `index` ，是否能查询到刚刚写入的数据。

## 4.5 influxdb2 + grafana的操作

>  这里懒得写了，我参考我写的博客: 
>
> [influxdb2.x与Grafana的使用（Go开发）](https://kkite.gitee.io/2022/04/07/influxdb2-x%E4%B8%8EGrafana%E7%9A%84%E4%BD%BF%E7%94%A8%EF%BC%88Go%E5%BC%80%E5%8F%91%EF%BC%89/)

# 5 源码分析

> 未完待续~
