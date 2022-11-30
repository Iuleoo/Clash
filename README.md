# Clash
My Clash Rule

多订阅地址聚合使用本地配置文件，统一使用管理，非常方便

## 本地聚合配置文件说明：

配置文件在 Local Config 文件夹内

大部分服务商可能不提供 Clash的专属订阅格式，均需要 base64 解码，所以下下面这种填法，无法使用，下文有解决方法

```yaml
proxy-providers:
	foo:
    	type: http
    	path: ./profiles/1667995624784.yml
    	url: 订阅地址
```

## proxy-provider

> `proxy-providers` 能够动态加载代理服务器列表，这个功能帮助我们在不修改 `config.yaml` 启动配置文件情况下实现更新代理功能。是不是觉得很强大呢？

需要指定代理订阅源支持两种类型：

- `http`类型： Clash 在`启动时`从指定的 URL 加载服务器列表。如果设置了`interval`该选项，Clash 会定期从远程拉取服务器列表。（如果是base64加密的订阅源无法使用此项）
- `file`类型： Clash 在`启动时`从文件系统上的指定位置加载服务器列表。

上面说，服务商给的都是base64，所以我们将每个订阅都先通过Clash获取至本地，然后打开这三个配置文件所在的本地文件位置

![Snipaste_2022_11_30_5121](https://raw.githubusercontent.com/Iuleoo/Clash/836d5a7d9a4ee03ee9498e46b7389ac96da1d61f/img/Snipaste_2022_11_30_5121.jpg)

然后对应 `path` 下面👇方式填入即可，然后后续更新订阅我们只需要直接Clash更新那几个订阅节点即可

```yaml
proxy-providers:
  foo:
    type: file	# 一定得是file类型
    path: ./profiles/1667728624784.yml	# 获取的配置文件路径
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 3600
  foo1:
    type: file
    path: ./profiles/1669041315728.yml
    filter: 香港|HK|日本|台湾|韩国|新加坡|印度
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 3600
  foo2:
    type: file
    path: ./profiles/1668072848721.yml
    filter: IPLC
    health-check:
      enable: true
      url: http://www.gstatic.com/generate_204
      interval: 3600
```

**避免的坑：**

- 不建议直接使用`http`类型订阅源！不建议！不建议！ 为什么呢？ 因为URL地址早晚会被屏蔽或无法访问，此时Clash无法更新代理并且**重启后无法再次启动**。
- 使用`cron`类似的调度配合`file`类型实现更新订阅源，即使URL地址无法访问也不会影响`Clash`服务正常运行。
- `proxy-group`放后面！后面！后面！使用`proxy-providers`的`proxy-group`要放在`proxy-providers`后面！不然会告诉你找不到`provider`或者`proxy`的错误信息。

> 注意： `Clash premium版本`支持 `ruleset` 远程 URL 地址更新，与 `http` 方式类似，但不同点是如果 `ruleset` 本地文件如果存在，启动 Clash 时不会因 `ruleset` 的 URL 地址无法访问受影响。

## 使用rule-providers规则集

> 规则集就是用于识别哪些网站需要使用代理的规则文件，有了规则集就可以制作出`透明代理`了，在内网不进行任何设置的情况下自动根据访问网址选择代理或者直连。

介绍了规则集的作用，再来介绍规则集的使用方式：

- rule-providers 是`premium`版本支持的功能！如果提示你不支持`rule-providers`，那就是你的`Clash`版本用错了。
- 规则集也是`人为生成`的，不会包含所有的网络地址，因此就需要决定找不到匹配规则时是`直连(黑名单规则)`还是使用`代理(白名单规则)`。
- **白名单模式**，意为「没有命中规则的网络流量，统统使用代理」，适用于服务器线路网络质量稳定、快速，不缺服务器流量的用户。
- **黑名单模式**，意为「只有命中规则的网络流量，才使用代理」，适用于服务器线路网络质量不稳定或不够快，或服务器流量紧缺的用户。通常也是软路由用户、家庭网关用户的常用模式。

> 提示：家庭内部使用时选择`黑名单模式`可以让你避免很多闹心的网络异常问题，假如你家里有很多联网设备，比如智能音响、智能电器等需要联网服务器，这些服务器地址通常不会匹配在规则集中，因此使用`黑名单模式`可以避免这些设备通过代理方式联网失败。

一般我会在下面提供的规则集基础上自己维护一个自己的规则集，用于添加自己所需的部分网站规则

下面是 `Clash` 提供的每日更新规则集：

```yaml
# 专业版支持 rule-providers
rule-providers:
  reject:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/reject.txt"
    path: ./ruleset/reject.yaml
    interval: 86400
  icloud:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/icloud.txt"
    path: ./ruleset/icloud.yaml
    interval: 86400
  apple:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/apple.txt"
    path: ./ruleset/apple.yaml
    interval: 86400
  google:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/google.txt"
    path: ./ruleset/google.yaml
    interval: 86400
  proxy:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/proxy.txt"
    path: ./ruleset/proxy.yaml
    interval: 86400
  direct:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/direct.txt"
    path: ./ruleset/direct.yaml
    interval: 86400
  private:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/private.txt"
    path: ./ruleset/private.yaml
    interval: 86400
  gfw:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/gfw.txt"
    path: ./ruleset/gfw.yaml
    interval: 86400
  greatfire:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/greatfire.txt"
    path: ./ruleset/greatfire.yaml
    interval: 86400
  tld-not-cn:
    type: http
    behavior: domain
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/tld-not-cn.txt"
    path: ./ruleset/tld-not-cn.yaml
    interval: 86400
  telegramcidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/telegramcidr.txt"
    path: ./ruleset/telegramcidr.yaml
    interval: 86400
  cncidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/cncidr.txt"
    path: ./ruleset/cncidr.yaml
    interval: 86400
  lancidr:
    type: http
    behavior: ipcidr
    url: "https://cdn.jsdelivr.net/gh/Loyalsoldier/clash-rules@release/lancidr.txt"
    path: ./ruleset/lancidr.yaml
    interval: 86400
```

### 白名单模式rule配置

```yaml
rules:
  - PROCESS-NAME,v2ray,DIRECT
  - PROCESS-NAME,xray,DIRECT
  - PROCESS-NAME,naive,DIRECT
  - PROCESS-NAME,trojan,DIRECT
  - PROCESS-NAME,trojan-go,DIRECT
  - PROCESS-NAME,ss-local,DIRECT
  - PROCESS-NAME,privoxy,DIRECT
  - PROCESS-NAME,leaf,DIRECT
  - PROCESS-NAME,v2ray.exe,DIRECT
  - PROCESS-NAME,xray.exe,DIRECT
  - PROCESS-NAME,naive.exe,DIRECT
  - PROCESS-NAME,trojan.exe,DIRECT
  - PROCESS-NAME,trojan-go.exe,DIRECT
  - PROCESS-NAME,ss-local.exe,DIRECT
  - PROCESS-NAME,privoxy.exe,DIRECT
  - PROCESS-NAME,leaf.exe,DIRECT
  - PROCESS-NAME,Surge,DIRECT
  - PROCESS-NAME,Surge 2,DIRECT
  - PROCESS-NAME,Surge 3,DIRECT
  - PROCESS-NAME,Surge 4,DIRECT
  - PROCESS-NAME,Surge%202,DIRECT
  - PROCESS-NAME,Surge%203,DIRECT
  - PROCESS-NAME,Surge%204,DIRECT
  - PROCESS-NAME,Thunder,DIRECT
  - PROCESS-NAME,DownloadService,DIRECT
  - PROCESS-NAME,qBittorrent,DIRECT
  - PROCESS-NAME,Transmission,DIRECT
  - PROCESS-NAME,fdm,DIRECT
  - PROCESS-NAME,aria2c,DIRECT
  - PROCESS-NAME,Folx,DIRECT
  - PROCESS-NAME,NetTransport,DIRECT
  - PROCESS-NAME,uTorrent,DIRECT
  - PROCESS-NAME,WebTorrent,DIRECT
  - PROCESS-NAME,aria2c.exe,DIRECT
  - PROCESS-NAME,BitComet.exe,DIRECT
  - PROCESS-NAME,fdm.exe,DIRECT
  - PROCESS-NAME,NetTransport.exe,DIRECT
  - PROCESS-NAME,qbittorrent.exe,DIRECT
  - PROCESS-NAME,Thunder.exe,DIRECT
  - PROCESS-NAME,ThunderVIP.exe,DIRECT
  - PROCESS-NAME,transmission-daemon.exe,DIRECT
  - PROCESS-NAME,transmission-qt.exe,DIRECT
  - PROCESS-NAME,uTorrent.exe,DIRECT
  - PROCESS-NAME,WebTorrent.exe,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,icloud,DIRECT
  - RULE-SET,apple,DIRECT
  - RULE-SET,google,DIRECT
  - RULE-SET,proxy,PROXY
  - RULE-SET,direct,DIRECT
  - RULE-SET,telegramcidr,PROXY
  - GEOIP,,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,PROXY
```

### 黑名单模式rule配置

```yaml
rules:
  - PROCESS-NAME,v2ray,DIRECT
  - PROCESS-NAME,xray,DIRECT
  - PROCESS-NAME,naive,DIRECT
  - PROCESS-NAME,trojan,DIRECT
  - PROCESS-NAME,trojan-go,DIRECT
  - PROCESS-NAME,ss-local,DIRECT
  - PROCESS-NAME,privoxy,DIRECT
  - PROCESS-NAME,leaf,DIRECT
  - PROCESS-NAME,v2ray.exe,DIRECT
  - PROCESS-NAME,xray.exe,DIRECT
  - PROCESS-NAME,naive.exe,DIRECT
  - PROCESS-NAME,trojan.exe,DIRECT
  - PROCESS-NAME,trojan-go.exe,DIRECT
  - PROCESS-NAME,ss-local.exe,DIRECT
  - PROCESS-NAME,privoxy.exe,DIRECT
  - PROCESS-NAME,leaf.exe,DIRECT
  - PROCESS-NAME,Surge,DIRECT
  - PROCESS-NAME,Surge 2,DIRECT
  - PROCESS-NAME,Surge 3,DIRECT
  - PROCESS-NAME,Surge 4,DIRECT
  - PROCESS-NAME,Surge%202,DIRECT
  - PROCESS-NAME,Surge%203,DIRECT
  - PROCESS-NAME,Surge%204,DIRECT
  - PROCESS-NAME,Thunder,DIRECT
  - PROCESS-NAME,DownloadService,DIRECT
  - PROCESS-NAME,qBittorrent,DIRECT
  - PROCESS-NAME,Transmission,DIRECT
  - PROCESS-NAME,fdm,DIRECT
  - PROCESS-NAME,aria2c,DIRECT
  - PROCESS-NAME,Folx,DIRECT
  - PROCESS-NAME,NetTransport,DIRECT
  - PROCESS-NAME,uTorrent,DIRECT
  - PROCESS-NAME,WebTorrent,DIRECT
  - PROCESS-NAME,aria2c.exe,DIRECT
  - PROCESS-NAME,BitComet.exe,DIRECT
  - PROCESS-NAME,fdm.exe,DIRECT
  - PROCESS-NAME,NetTransport.exe,DIRECT
  - PROCESS-NAME,qbittorrent.exe,DIRECT
  - PROCESS-NAME,Thunder.exe,DIRECT
  - PROCESS-NAME,ThunderVIP.exe,DIRECT
  - PROCESS-NAME,transmission-daemon.exe,DIRECT
  - PROCESS-NAME,transmission-qt.exe,DIRECT
  - PROCESS-NAME,uTorrent.exe,DIRECT
  - PROCESS-NAME,WebTorrent.exe,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,tld-not-cn,PROXY
  - RULE-SET,gfw,PROXY
  - RULE-SET,greatfire,PROXY
  - RULE-SET,telegramcidr,PROXY
  - MATCH,DIRECT
```

