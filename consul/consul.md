
### Start agent
dev环境不持久化任何状态
```
consul agent -dev
```

### Cluster members
查看cluster成员
```
consul members
```
也可以通过http查看（强一致性）
```
curl localhost:8500/v1/catalog/nodes
```

### 服务注册
可以通过文件的形式注册, 编写文件./consul.d/web.json
```
{
	"service":{
		"name": "web",
		"tags": ["rails"],
		"port": 80
	}
}
```
启动
```
consul agent -dev -config-dir=./consul.d/web.json
```