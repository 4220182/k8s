基于提示的自动发现（目前是BEAT）
Filebeat支持基于提供程序提示的自动发现。提示系统在Kubernetes Pod注释或具有前缀的Docker标签中查找提示co.elastic.logs。一旦容器启动，Filebeat将检查它是否包含任何提示并为其启动正确的配置。提示告诉Filebeat如何获取给定容器的日志。默认情况下，将使用docker输入从容器中检索日志。
```
    filebeat.autodiscover:
      providers:
        - type: kubernetes
          hints.enabled: true
```
默认是所有容器日志都会监控到，如果想不监控该容器的日志输出，需要在容器增加：
```
      annotations:
        co.elastic.logs/disable: "true"
```
但是，我们有时候需要，默认不监控所有日志，如果需要哪些容器监控日志，就这该容器加上：
```
      annotations:
        co.elastic.logs/disable: "false"
```
此功能应该在filebeat 7.x以上就可以实现，参考：

https://github.com/elastic/beats/pull/10911
https://www.elastic.co/guide/en/beats/filebeat/current/configuration-autodiscover-hints.html

其他实现方法：
https://discuss.elastic.co/t/filebeat-autodiscover-hints-breaking-template/137310/10

