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

## proxy-provider 订阅源

需要指定代理订阅源支持两种类型：

- `http`类型： Clash 在`启动时`从指定的 URL 加载服务器列表。如果设置了`interval`该选项，Clash 会定期从远程拉取服务器列表。（如果是base64加密的订阅源无法使用此项）
- `file`类型： Clash 在`启动时`从文件系统上的指定位置加载服务器列表。

上面说，服务商给的都是base64，所以我们将每个订阅都先通过Clash获取至本地，然后打开这三个配置文件所在的本地文件位置

![Snipaste_2022_11_30_5121](img\Snipaste_2022_11_30_5121.jpg)

然后对应 `path` 填入即可，格式：

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
