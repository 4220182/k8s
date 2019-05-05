### 基于提示的自动发现（目前是BEAT）

一、Filebeat支持基于提供程序提示的自动发现。提示系统在Kubernetes Pod注释或具有前缀的Docker标签中查找提示co.elastic.logs。一旦容器启动，Filebeat将检查它是否包含任何提示并为其启动正确的配置。提示告诉Filebeat如何获取给定容器的日志。默认情况下，将使用docker输入从容器中检索日志。
```
    filebeat.autodiscover:
      providers:
        - type: kubernetes
          hints.enabled: true
```
默认所有容器日志都会被监控到，如果想不监控该容器的日志输出，需要在容器[nginx.yaml](nginx.yaml)增加annotations：
```
      annotations:
        co.elastic.logs/disable: "true"
```

二、但是，我们有时候需要，默认不监控所有日志，这就需要使用折中的方法实现：
增加processors->drop,只收集符合条件的容器日志.
如[filebeat-kubernetes-defaultNotRecordLogs.yaml](filebeat-kubernetes-defaultNotRecordLogs.yaml) ：
```
    filebeat.autodiscover:
      providers:
        - type: kubernetes
          hints.enabled: true
          include_annotations:
          - "filebeat.logging/game"

    processors:
      - add_kubernetes_metadata:
          in_cluster: true
      - drop_event:
          when:
            not.has_fields: ['kubernetes.annotations.filebeat_logging/game']
```
### 注意：
1. include_annotations和not.has_fields中的annotations定义，前者是使用"." 后者需要把前者的"."替换为"_",否则条件不成立，在filebeat 6.x中都是使用"."(参考[/filebeat/6.7.2/hints/filebeat-kubernetes.yaml](/filebeat/6.7.2/hints/filebeat-kubernetes.yaml)),不知是否是7.0.1的一个BUG。
2. include_annotations：“*” 无效，这是否也是7.0.1的BUG？

出现这样的差异，我是在去掉Drop_event再查看filebeat日志输出发现此问题。
```
# tail autodiscover-kubernetes.log -n 1 |jq .
{
  "@timestamp": "2019-05-05T12:29:55.320Z",
  "@metadata": {
    "beat": "filebeat",
    "type": "_doc",
    "version": "7.0.1"
  },
  "kubernetes": {
    "namespace": "default",
    "replicaset": {
      "name": "nginx-74f94564f"
    },
    "labels": {
      "pod-template-hash": "74f94564f",
      "app": "nginx"
    },
    "annotations": {
      "filebeat_logging/game": "mygame"
    },
......
```
[nginx.html](nginx.yaml) 增加：
annotations: 
  filebeat.logging/game: "mygame" ,如下：
```
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
      annotations:
        co.elastic.logs/disable: "false"
        filebeat.logging/game: "mygame"
```        

### 三、参考
<https://www.elastic.co/guide/en/beats/filebeat/7.0/configuration-autodiscover-hints.html>
<https://www.elastic.co/guide/en/beats/filebeat/7.0/filtering-and-enhancing-data.html>

