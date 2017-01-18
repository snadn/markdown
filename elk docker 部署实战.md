# elk docker 部署实战

elk(Elasticsearch + Logstash + Kibana) 是业界目前搭建日志统计监控系统的一套成熟的解决方案。
我为了后续前端的监控告警系统的搭建，最近对其进行了一下了解和尝试。

## 部署搭建

搭建 elk 整套系统，我选择了 docker 进行相关的部署。

部署涉及到两个镜像 [sebp/elk](https://hub.docker.com/r/sebp/elk/) 和 [prima/filebeat](https://hub.docker.com/r/prima/filebeat/)。

elk 是对日志进行处理和展示，filebeat 是对日志进行采集。

镜像的网站有详细的文档说明。我在此将我的安装步骤简要说明一下。

1. `docker pull sebp/elk` 获取 elk 镜像
2. `docker pull prima/filebeat` 获取 filebeat 镜像
3. 从 `https://github.com/spujadas/elk-docker/tree/master/nginx-filebeat`,
	获取 `logstash-beats.crt` 放到 `/etc/pki/tls/certs/logstash-beats.crt`
	获取 `filebeat.yml` 放到 `/etc/filebeat/filebeat.yml`
4. `sudo docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -d --name elk sebp/elk` 启动 elk 容器
5. `sudo docker run -v /etc/pki/tls/certs/logstash-beats.crt:/etc/pki/tls/certs/logstash-beats.crt -v /filebeat.yml:/etc/filebeat/filebeat.yml -v /var/log:/var/log -d --name filebeat --link prima/filebeat` 启动 filebeat 容器
6. 创建 `/etc/filebeat/filebeat.template.json` 内容为

	```
	{
		"template" : "*",
		"order" : 0,
		"settings" : {
			"number_of_shards" : 1
		},
		"mappings" : {
			"type1" : {
				"_source" : { "enabled" : false }
			}
		}
	}
	```
7. 执行 `curl -XPUT 'http://elk:9200/_template/filebeat?pretty' -d@/etc/filebeat/filebeat.template.json`

通过以上步骤应该就能在浏览器通过 ip:5601 访问到 Kibana，简单设置一下就可以看到日志的记录了。

## 日志处理

默认的配置，日志里面的有些字段并没有被展开，不便于我们进行搜索过滤。所以还需要我们进行一下相应的配置。针对 Logstash 的配置和相关知识，可以参看官方的文档 https://www.elastic.co/guide/en/logstash/current/index.html

通过文档我知道 grok、kv、useragent、urldecode 等几个 filter 插件是我需要的。

1. 执行 `docker exec -it elk bash` 进入 elk 容器
2. 执行 `vi /etc/logstash/conf.d/11-nginx.conf` 进行相应的插件的配置
3. 退出容器，执行 `docker restart elk` 重启容器就可以了

## 后续

elk 相关的东西还有很多是我要了解的，我也仅仅是才了解了一些皮毛，欢迎对这方面感兴趣的小伙伴和我一起探讨。