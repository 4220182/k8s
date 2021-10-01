##创建测试httpbin
```
kubectl apply -f httpbin.yaml
```

##创建证书：
```
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -subj '/CN=istio-cert' -nodes
kubectl create secret tls tls-cert --cert=cert.pem --key=key.pem -n istio-system
```

##创建gateway
```
kubectl apply -f gateway.yaml
```

##创建虚拟机
```
kubectl apply -f vs.yaml
```

##客户端访问：
```
curl http://httpbin.tmp.com/status/200  -i                                                                                          23:54:55
HTTP/1.1 200 OK
server: istio-envoy
date: Thu, 30 Sep 2021 15:54:58 GMT
content-type: text/html; charset=utf-8
access-control-allow-origin: *
access-control-allow-credentials: true
content-length: 0
x-envoy-upstream-service-time: 4
```

流量图：
client --> gateway -> httpbin

