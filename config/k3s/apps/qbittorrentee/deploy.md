# qBittorrent-Enhanced-Edition
一个 BT 下载工具

[Github](https://github.com/c0re100/qBittorrent-Enhanced-Edition)

[Github Docker](https://github.com/SuperNG6/Docker-qBittorrent-Enhanced-Edition)

```bash
kubectl create ns qbittorrentee
kubectl apply -f qbittorrentee.yaml
```

这里虽然配置了 persistentVolumeClaimRetentionPolicy，但是 pvc 还是不会自己删除，我猜可能是 K3s 还暂未支持这个特性。如果删除应用的话需要手动回收 pvc。

或者可以尝试手动编写 pcv，不使用 volumeClaimTemplates。