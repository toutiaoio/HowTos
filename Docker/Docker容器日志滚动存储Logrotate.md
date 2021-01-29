---
title: Docker容器日志滚动存储Logrotate
url: https://toutiao.io/posts/aho8gq2
---

默认情况下， 容器的 stdout 和 sdterr 写在 `/var/lib/docker/containers/[container-id]/[container-id]-json.log` 中的 json 中, 没有维护的情况下将会占用大量的磁盘空间

所以我们需要做日志 rotation , 修改配置文件 `/etc/docker/daemon.json`

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "5"
  }
}
```

然后重启应用

```
$ systemctl daemon-reload

$ systemctl restart docker
```
