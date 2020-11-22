### consul部署

> **生成 TLS Certificates**

* 文件consul/ca : ca-config.json、ca-csr.json、consul-csr.json

* 执行
	```bash
	cfssl gencert -initca ca/ca-csr.json | cfssljson -bare ca
	```
* 执行
	```yaml
	cfssl gencert \
	  -ca=ca.pem \
	  -ca-key=ca-key.pem \
	  -config=ca/ca-config.json \
	  -profile=default \
	  ca/consul-csr.json | cfssljson -bare consul
	```
* 工作目录中具有以下文件：
ca-key.pem
ca.pem
consul-key.pem
consul.pem

---
> **创建 Consul Secret 和 Configmap**

* 将Gossip加密密钥和TLS证书存储在密钥中
	``` yaml
	kubectl create secret generic consul -n manage \
	  --from-literal="gossip-encryption-key=${GOSSIP_ENCRYPTION_KEY}" \
	  --from-file=ca.pem \
	  --from-file=consul.pem \
	  --from-file=consul-key.pem
	```
* 将Consul服务器配置文件存储在ConfigMap中
	```yaml
	kubectl create configmap consul -n manage --from-file=configs/server.json
	```

---
> **部署**

* 持久化存储卷声明
	```yaml
	kubectl create -f pvc/pvc.yaml
	```
* 部署consul
	```yaml
	kubectl create -f serviceaccounts/
	kubectl create -f services/
	kubectl create -f statefulsets/
	```

---

> 检验

```yaml
kubectl get pods -n manage |grep consul
kubectl exec -it consul-0 -n manage /bin/sh
consul members
consul monitor -log-level debug
```
* web ui : http://ip:30601
