# ~~ Sakari 的 Linux 网络配置仓库~~
# 现在是 Maary 的 Linux 网络配置仓库了

[使用dae和clash实现的全局代理方案](https://sakari.top/2023/dae-clash/)

## 使用项目

- [dae](https://github.com/daeuniverse/dae)：高性能全局透明代理
- [Clash.Meta](https://github.com/MetaCubeX/Clash.Meta)：作为 dae 的上游提供代理

## 安装

- 一个正常工作的代理客户端，`clash`、`clash-meta`、`clash for windows`、`v2ray` 等均可。
下面以 [Clash.Meta](https://github.com/MetaCubeX/Clash.Meta) 示例：
    - 从 [release](https://github.com/MetaCubeX/Clash.Meta/releases) 中下载对应版本的二进制文件
    - 重命名二进制文件为 Clash-Meta，给予可执行权限，移动到 ```/usr/local/bin/```
    - 创建 ```/etc/Clash-Meta/``` 目录作为工作目录
    - 创建 service 文件 ```/etc/systemd/system/Clash-Meta.service```:
        ```
        [Unit]
        Description=Clash-Meta Daemon, Another Clash Kernel.
        After=network.target NetworkManager.service systemd-networkd.service iwd.service

        [Service]
        Type=simple
        User=root
        Group=root
        LimitNPROC=500
        LimitNOFILE=1000000
        CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE
        AmbientCapabilities=CAP_NET_ADMIN CAP_NET_RAW CAP_NET_BIND_SERVICE
        Restart=always
        ExecStartPre=/usr/bin/sleep 1s
        ExecStart=/usr/local/bin/Clash-Meta -d /etc/Clash-Meta

        [Install]
        WantedBy=multi-user.target
        ```

- [dae](https://github.com/daeuniverse/dae)：见 [吃鹅直通手册](https://github.com/daeuniverse/dae/blob/main/docs/getting-started/README_zh.md)，注意 dae 对内核版本有要求

## 配置

### Clash

本节可跳过，可使用现有的正常工作的配置文件，仅需要保证 clash 的端口和 dae node 中配置的一致。与 dae 共同使用时请保证代理的各种全局代理方案（tun、iptables 等）均已关闭

#### 转换订阅

自行搭建 [proxy-provider-converter](https://github.com/qier222/proxy-provider-converter)，或使用 [proxy-converter.sakari.top](https://proxy-converter.sakari.top/)，将 clash 订阅转换成 `Proxy Provider` 支持的格式，或直接使用 v2ray 订阅连接。

需要注意自行搭建的转换工具可能无法在 DIRECT 下访问，可以考虑自行下载然后根据 `clash/config.yaml` 中 `proxy-providers: path:` 下文件名进行修改放入到前面的 Clash 工作目录中，如按照本设置中应该为 `sub1.yaml`。

#### 修改配置

编辑 [clash/config.yaml](clash/config.yaml)，将转换后的链接填入 `url` 中

下载 [Yacd-meta](https://github.com/yaling888/yacd/archive/gh-pages.zip)，解压至配置文件中 `external-ui` 所在目录，权限应该设置为 ```755```。

根据需要修改分流规则

### dae

修改 [dae/config.dae](dae/config.dae)，将 `wan_interface` 的值修改为自己的网卡，可以使用 `ip a` 查看

根据使用的代理在 `routing` 下添加规则，如 `pname(clash) -> must_direct`

### 启用配置

启动 Clash Meta 和 Dae:

```sh
systemctl start Clash-Meta.service
systemctl start dae.service 
```

设置开机启动：

```sh
systemctl enable Clash-Meta.service
systemctl enable dae.service 
```