# 主机部署

Loggie使用Golang编译成二进制，可根据自身需求对接各类部署系统。  
这里我们提供一个使用systemd部署Loggie的参考。  

## 前置检查
- 操作系统：Linux
- 系统架构：amd64
- 发行版支持systemd

目前release仅包含GOOS=linux GOARCH=amd64生成的二进制可执行文件。其他系统和架构，请自行基于源码交叉编译。

## 下载二进制

```
mkdir /opt/loggie && curl https://github.com/loggie-io/installation/releases/download/v1.2.0/loggie-linux-amd64 -o /opt/loggie/loggie && chmod +x /opt/loggie/loggie
```

## 添加配置文件

请根据实际需求创建配置，以下为参考：

#### 创建loggie.yml

!!! example "loggie.yml"

    ```yaml
    cat << EOF > /opt/loggie/loggie.yml
    loggie:
      monitor:
        logger:
          period: 30s
          enabled: true
        listeners:
          filesource: ~
          filewatcher: ~
          reload: ~
          sink: ~
    
      reload:
        enabled: true
        period: 10s
    
      http:
        enabled: true
        port: 9196
    EOF
    ```


#### 创建pipelines.yml

!!! example "pipelines.yml"

    ```yaml
    cat << EOF > /opt/loggie/pipelines.yml
    pipelines:
      - name: local
        sources:
          - type: file
            name: demo
            paths:
              - /tmp/log/*.log
        sink:
          type: dev
          printEvents: true
          codec:
            pretty: true
    EOF
    ```

## 添加systemd配置

```yaml
cat << EOF > /lib/systemd/system/loggie.service
[Unit]
Description=Loggie
Documentation=https://loggie-io.github.io/docs/getting-started/install/node/

[Service]
MemoryMax=200M
ExecStart=/opt/loggie/loggie -config.system=/opt/loggie/loggie.yml -config.pipeline=/opt/loggie/pipelines.yml
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

## 启动

首先生效配置：
```bash
systemctl daemon-reload
```

然后设置为开机启动：
```bash
systemctl enable loggie
```

接着就可以正式启动Loggie了：

```bash
systemctl start loggie
```

启动后，你可以随时查看进程状态：
```bash
systemctl status loggie
```
