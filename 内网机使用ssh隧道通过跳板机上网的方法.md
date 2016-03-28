# 内网机使用ssh隧道通过跳板机上网的方法

1. 通过 `ssh -D port` 建立隧道。关于 ssh 的具体用法及更多需要的参数可网上搜索。
2. 安装 tsocks ，该软件提供一个 shell 环境，让 tcp 链接都通过指定的代理进行连接。
3. 安装 [dns-tcp-socks-proxy](https://github.com/jtripper/dns-tcp-socks-proxy)
	该工具可以让 dns 转换到 tcp 协议

因为懒，所以写的很简略😜