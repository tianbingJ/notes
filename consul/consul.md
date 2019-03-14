
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
查看
```
curl http://localhost:8500/v1/catalog/service/web

[
    {
        "ID": "ca509b80-74f1-8d85-5550-b40ab1316403",
        "Node": "6a0d494c4509",
        "Address": "127.0.0.1",
        "Datacenter": "dc1",
        "TaggedAddresses": {
            "lan": "127.0.0.1",
            "wan": "127.0.0.1"
        },
        "NodeMeta": {
            "consul-network-segment": ""
        },
        "ServiceKind": "",
        "ServiceID": "web",
        "ServiceName": "web",
        "ServiceTags": [
            "rails"
        ],
        "ServiceAddress": "",
        "ServiceWeights": {
            "Passing": 1,
            "Warning": 1
        },
        "ServiceMeta": {},
        "ServicePort": 80,
        "ServiceEnableTagOverride": false,
        "ServiceProxyDestination": "",
        "ServiceProxy": {},
        "ServiceConnect": {},
        "CreateIndex": 10,
        "ModifyIndex": 10
    }
]
```