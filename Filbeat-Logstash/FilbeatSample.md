# Filebeat 配置文件

以下是 Filebeat 的配置文件，用於收集和處理日誌，並將其發送到 Logstash。

## 配置文件內容
```yaml
filebeat.config:
  modules:
    path: ${path.config}/modules.d/*.yml
    reload.enabled: false

filebeat.inputs:
  - type: filestream
    id: my-nginx-logs
    enabled: true
    paths:
      - /home/admin/log/test/testAPILog/*/log_Host_Json_*.log
      # - /var/log/nginx/error.log
    fields:
      service: nginx
    fields_under_root: true

tags: ["host-log", "test", "test-api"]

output.logstash:
  hosts: ["172.21.11.xxx:5044"]