---
title: "Let’s Encrypt: 如何通过 Certbot 更新证书后自动重启 Nginx"
url: https://toutiao.io/posts/5tqmrrc
---

Let's Encrypt 证书的有效期只有90天，所以每三个月需要续签一次，通常我们会设置一个 cronjob 来自动的完成，　但续签后需要重启 Web 服务 (比如Nginx), 不然证书过期后一直用的都是原来那个证书．

如果有个 Hook 机制的话就可以做到自动重启 Nginx了，　查阅certbot的文档后发现果然可以做到 https://certbot.eff.org/docs/using.html#renewing-certificates

> When Certbot detects that a certificate is due for renewal,  `--pre-hook` and `--post-hook` hooks run before and after each attempt to renew it. If you want your hook to run only after a successful renewal, use `--deploy-hook` in a command like this.

> ``certbot renew --deploy-hook /path/to/deploy-hook-script``

更方便的我们可以把 `deploy-hook` 加到配置文件 (`/etc/letsencrypt/cli.ini`) 中， 作为全局默认配置.

```
deploy-hook = systemctl reload nginx
```

这样每次成功续签后就可以自动重启Nginx，　使其加载最新的证书保证应用正常服务啦．