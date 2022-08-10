# Plex Media Server

家庭影音服务器

[Github](https://github.com/plexinc/pms-docker)

[Docker Hub](https://hub.docker.com/r/plexinc/pms-docker/)

```bash
kubectl create ns plex
kubectl apply -f plex.yaml
```

Plex 是我头疼了好久的一个 app。这里说一下主要问题和注意点。

### 数据持久化

Plex 的持久化要映射两个文件夹，分别是 config 和 transcode。由于我不用服务器转码（毕竟没有显卡，cpu转码太卡），，所以并没有映射 transcode。然后就是媒体文件夹，在这里我映射到了 media 下。

我在 config 文件夹这里卡了好久，一开始我是将 config 和 media 都通过 SMB 映射到了 NAS 上，映射没有任何问题，app也可以启动，但是配置完成之后所有文件都无法播放。开始时我怀疑是 SMB 的问题，于是换了 NFS，还是不行，然后试着把 config 映射去掉，果然就没问题了。  
在此我怀疑是 SMB 的 IO 性能不够，于是我选择了将 config 挂载到了 Local Volume 上。当然这样做也有问题，那就是数据并没有持久化到 NAS 上，宿主机（虚拟机）挂了就没了。而且数据只在 Server 01 上边，POD 也只能在 Server 01 上边去启动，并不能做到 HA。

### 网络配置
要做到让 Plex 在家庭网络外访问，必须要让外部网络能够访问 Plex 的 32400 端口。

尝试用 ClusterIP + Ingress 走 443，行不通，traefik 去转发 TCP 32400 不知道为什么也不行。  
用 NodePort，在宿主机打开32400端口，可行，~~但是有两层 NAT，损失严重~~，而且这样初次配置要用两层 SSH 转发，Pod 不自带 SSH Server 要手动 exec 进去安装，安装完成后为了要干净的 Pod 还要删除重建。极其复杂，PASS。
最后选择了 hostNetwork 模式，直接绑定到宿主机的网络上，简单方便，只有一层路由器 NAT，在路由器上做一个 WAN -> vlan2 的端口转发即可。
> 最后配置完成后还是切换回了 NodePort 模式。

### 初次配置
由于 K3s 集群和 PC 不在同一网段里，是无法直接去配置的，解决方法有很多种，最简单的办法，找个在 vlan2 网段的带桌面环境及浏览器的机器即可。但是我没有，我选择用 SSH 隧道。
```bash
ssh -L 32400:172.16.2.11:32400 172.16.2.11
```
然后访问 localhost:32400 即可