#### 下载安装包

```shell
$ cd /home/clickhouse/
$ wget https://github.com/Vertamedia/chproxy/releases/download/v1.14.0/chproxy-linux-amd64-v1.14.0.tar.gz
$ tar -xzvf chproxy-*.gz
$ mkdir -p /data/chproxy
$ /bin/cp -rf /home/clickhouse/chproxy /data/chproxy
$ chown -R clickhouse:clickhouse /data/chproxy
```

#### 编写配置文件

###### vim /data/chproxy/config.yml

```yaml
# 是否打印调试日志。默认情况下，调试日志将被禁用。
log_debug: true
# 在配置解析期间是否忽略安全检查。默认情况下的安全检查被启用。
hack_me_please: true 
server:
  http:
  	# 可以采用IP：port的形式；IP部分是可选的。
    listen_addr: ":9090"
    # 允许的网络或network_groups的列表，可以包含IP地址、IP子网掩码或`network_groups`中的名称，
    # 默认情况下，所有IP都接受请求。
    allowed_networks: ["office", "reporting-apps", "1.2.3.4", "172.0.0.0/8"]
    # 代理读取整个请求（包括正文）的最大持续时间。缺省值为1m。 
    read_timeout: 5m
    # 写超时时间，默认值是来自用户或群集的最大MaxExecutionTime + MaxQueueTime值。
    write_timeout: 10m
  https:
  	listen_addr: ":443"
  	# TLS证书和密钥文件的路径。
  	# cert_file: "cert_file"
  	# key_file: "key_file"
  	# 如果存在此部分编号，则将自动颁发和更新证书。如果存在此部分，则不需要cert_file和key_file。
    # Autocert需要应用侦听80端口，用于证书生成。
    autocert:
      # autocert证书缓存目录路径
      cache_dir: "certs_dir"
      # 允许代理响应的主机名列表。
      allowed_hosts: ["example.com"]
  # 普罗米修斯格式的指标可以通过/metrics端点公开访问。该配置可以限制对`/metrics`端点的访问。
  # 默认情况下，对`/metrics`端点的访问不受限制。
  metrics:
    allowed_networks: ["office"]

users:
  # 名称和密码用于通过BasicAuth或通过`user`/`password`查询参数授权访问。
  # 密码是可选的。默认情况下，使用空密码。
  - name: "distributed-read"
    password: "****"
    allowed_networks: ["office", "1.2.3.0/24"]
    # 来自用户的请求被路由到该集群。
    to_cluster: "distributed-read"
    # 代理用户
    to_user: "readonly"
    # 给定输入用户的每分钟请求数限制。默认情况下，没有每分钟的限制。
    requests_per_minute: 10
    # 是否拒绝通过HTTP的输入请求。
    deny_http: true
    # 是否拒绝通过HTTPS的输入请求。
    deny_https: true
    # 是否像`tabix`一样允许`CORS`请求。默认情况下，出于安全原因拒绝“ CORS”请求。
    allow_cors: true
    # 最大等待请求队列，这个选项可以用于处理请求突发情况；
    # 默认情况下，所有的请求都没有立即执行，在队列中等待。
    max_queue_size: 40
    # 队列中的请求的最大等待时间，仅当设置了max_queue_size时，此选项才有意义。
    # 默认情况下，请求在队列中等待最多10秒钟。
    max_queue_time: 25s
    # 为用户同时运行的查询的最大数量。默认情况下，对并发的数量没有限制.
    max_concurrent_queries: 4
    # 用户的最大查询持续时间。超时查询通过`KILL QUERY`被强制终止。默认情况下，查询持续时间没有限制。
    max_execution_time: 1m
    # 要使用的响应缓存配置名称。默认情况下，响应不缓存。
    cache: "shortterm"
    # 与每个代理请求一起发送到ClickHouse的可选参数组。这些参数可以在param_groups块中设置。
    # 默认情况下，没有额外的PARAMS发送到ClickHouse。
    params: "web"
    
  - name: "distributed-write"
    to_cluster: "distributed-write"
    to_user: "default"

  - name: "replica-write"
    to_cluster: "replica-write"
    to_user: "default"

clusters:
  # 集群名称在`to_cluster`中使用。
  - name: "first cluster"
    # 用来与群集节点进行通信的协议，当前支持的值为“http”或“https”，默认情况下使用“http”。
    scheme: "http"
    # 群集节点地址列表，请求平均分布在其中。
    nodes: ["127.0.0.1:8123", "shard2:8123"]
    # 过时：使用此间隔检查每个群集节点的可用性。默认情况下，每5秒检查一次每个节点。
    # 请使用`heartbeat.interval`。
    heartbeat_interval: 1m
    # 用于心跳请求的用户配置。clusters.users 中第一个用户的凭据将用于向Clickhouse发出心跳请求。
    heartbeat:
      # 检查所有群集节点的可用性的时间间隔，默认情况下，每5秒检查一次每个节点。
      interval: 1m
      # 群集节点的等待响应超时，默认为3s
      timeout: 10s
      # 设置要在运行状况检查中请求的URI的参数
      # 默认为"/?query=SELECT%201"
      request: "/?query=SELECT%201%2B1"
      # 来自运行状况检查请求的clickhouse的参考响应
      # 默认为"1\n"
      response: "2\n"
        
    # 群集用户的配置。    
    users:
      # 用户名在`to_user`中使用。
      - name: "default"
        password: "6lYaUiFi"
        max_concurrent_queries: 4
        max_execution_time: 1m
        
    # 使用该用户将杀死超时的查询。默认情况下，使用“default”用户。
    kill_query_user:
      name: "default"
      password: "***"

  - name: "second cluster"
    scheme: "https"
    # 群集可能包含多个副本，而不是扁平化节点。Chproxy选择副本间最小负载的节点。
    replicas:
      - name: "replica1"
        nodes: ["127.0.1.1:8443", "127.0.1.2:8443"]
      - name: "replica2"
        nodes: ["127.0.2.1:8443", "127.0.2.2:8443"]
    users:
      - name: "default"
        max_concurrent_queries: 4
        max_execution_time: 1m

      - name: "web"
        max_concurrent_queries: 4
        max_execution_time: 10s
        requests_per_minute: 10
        max_queue_size: 50
        max_queue_time: 70s
        allowed_networks: ["office"]

  - name: "distributed-read"
    nodes: [
      "clickhouse-node-06:8123",
      "clickhouse-node-09:8123"
    ]
    users:
    - name: "readonly"
      password: "6lYaUiFi"

# 查询参数的可选列表，随每个代理请求发送至ClickHouse。
# 这些列表可用于按用户覆盖ClickHouse设置。
param_groups:
  # 组名，可以在`user`级别传递到`params`选项中。
  - name: "cron-job"
    # List of key-value params to send
    params:
      - key: "max_memory_usage"
        value: "40000000000"
      - key: "max_bytes_before_external_group_by"
        value: "20000000000"

  - name: "web"
    params:
      - key: "max_memory_usage"
        value: "5000000000"
      - key: "max_columns_to_read"
        value: "30"
      - key: "max_execution_time"
        value: "30"
      
# 可选的网络列表，可作为`allowed_networks`的值。      
network_groups:
  - name: "office"
    # 每个项目都可以包含IP或IP子网掩码。
    networks: ["127.0.0.0/24", "10.10.0.1"]

  - name: "reporting-apps"
    networks: ["10.10.10.0/24"]

# 可选的响应缓存配置。可以使用不同的设置配置多个不同的高速缓存。
caches:
  # 缓存名称，可以将其传递到“user”级别的“cache”选项中。
  - name: "longterm"
    # 将存储缓存的响应的目录路径。
    dir: "/data/chproxy/cache/shortterm"
    # 最大缓存大小。可以使用Kb，Mb，Gb和Tb后缀。
    max_size: 150Mb
    # 缓存的响应的过期时间。
    expire: 130s
    # 当多个具有相同查询的请求同时命中`chproxy`且查询没有缓存的响应时，
    # 则只有一个请求将被代理到clickhouse。其他请求将等待在此宽限期内的缓存响应。
    # 这被称为“防雷群”问题的防护。默认情况下`grace_time`为5秒。负值将使保护免受“雷群”问题的影响。
    grace_time: 20s
  - name: "shortterm"
  	dir: "/path/to/shortterm/cachedir"
    max_size: 100Mb
    expire: 10s
```

#### 创建脚本

###### vim /data/chproxy/restart.sh

```bash
#!/bin/bash
cd /data/chproxy
echo "stop chproxy ..."
ps -ef | grep chproxy | head -2 | tail -1 | awk '{print $2}' | xargs kill -9
echo "start chproxy ..."
sudo -u clickhouse nohup ./chproxy -config=./config.yml >> ./logs/chproxy.out 2>&1 &
pid=$(ps -ef | grep chproxy | head -2 | tail -1 | awk '{print $2}')
echo "pid: $pid"
```

###### vim /data/chproxy/start.sh

```bash
#!/bin/bash
cd /data/chproxy
echo "start chproxy ..."
sudo -u clickhouse nohup ./chproxy -config=./config.yml >> ./logs/chproxy.out 2>&1 &
pid=$(ps -ef | grep chproxy | head -2 | tail -1 | awk '{print $2}')
echo "pid: $pid"
```

###### vim /data/chproxy/stop.sh

```bash
#!/bin/bash
ps -ef | grep chproxy | head -2 | tail -1 | awk '{print $2}' | xargs kill -9
```

###### vim /data/chproxy/status.sh

```bash
#! /bin/bash
pid=$(ps -ef | grep chproxy | head -2 | tail -1 | awk '{print $2}')
echo "pid: $pid"
```

###### vim /data/chproxy/config_reload.sh

```shell
#! /bin/bash
pid=$(ps -ef | grep chproxy | head -2 | tail -1 | awk '{print $2}')
echo "pid: $pid"
kill -1  $pid
echo "chproxy config reload success!"
```

