---
title: 如何改变Docker的数据目录
url: https://toutiao.io/posts/i9q3h6k
---

在 Linux 系统中 Docker 默认的数据目录为 `/var/lib/docker`, 如果系统磁盘空间
不够大我们需要给它移个位置

1. 停止 docker `systemctl stop docker`

2. 创建一个新的目录并把数据同步过去

    ```
    mkdir /srv/docker
    rsync -aqxP /var/lib/docker/ /srv/docker
    ```

    如果是新安装就不需要移数据了

3. 修改配置文件 `/etc/docker/daemon.json`, 把数据目录添加进去

    ```
    {
      "data-root": "/srv/docker"
    }
    ```

4. 启动系统 `systemctl start docker`.
