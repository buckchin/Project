# Logstash 配置文件

以下是 Logstash 的配置文件，用於從 Beats 接收日誌，處理並過濾特定標籤的日誌數據，最終輸出到 Elasticsearch。

## 配置文件內容
```ruby
input {
  beats {
    port => 5044
    type => "beats-log"
  }
}

filter {
  if [type] == "beats-log" {

    # ======================= Process iis-log Start ===========================
    if "iis-log" in [tags] {
      if "test" in [tags] or "testn" in [tags] {
        grok {
          match => {
            "message" => "%{TIMESTAMP_ISO8601:datetime} %{IP:siteIP} %{WORD:method} %{NOTSPACE:uriStem} %{NOTSPACE:uriQuery} %{NUMBER:port} %{NOTSPACE:username} %{IPORHOST:clientHost} %{NOTSPACE:userAgent} %{NOTSPACE:referer} %{NUMBER:httpStatus} %{NUMBER:httpSubtatus} %{NUMBER:win32Status} %{NUMBER:sentBytes} %{NUMBER:receivedBytes} %{NUMBER:timeTaken} %{NOTSPACE:oIP} %{NOTSPACE:tid} %{NOTSPACE:lid} %{NOTSPACE:uid}"
          }
        }
        mutate {
          remove_field => ["username"]
        }
      }
      else if "test_website" in [tags] {
        grok {
          match => {
            "message" => "%{TIMESTAMP_ISO8601:datetime} %{IPV6:siteIP} %{WORD:method} %{URIPATH:uriStem} %{NOTSPACE:uriQuery} %{NUMBER:port} - %{IPV6:clientHost} %{DATA:userAgent} %{URI:referer} %{NUMBER:httpStatus} %{NUMBER:httpSubStatus} %{NUMBER:win32Status} %{NUMBER:timeTaken}"
          }
        }
      }
    }

    if "analytics-log" in [tags] {
      mutate {
        add_field => {
          "[@metadata][index]" => "analytics-log-%{+YYYY.MM.dd}"
          "[@metadata][ilm]" => "log-ilm-policy"
        }
      }
    }
  }
}

output {
  if [type] == "beats-log" {
    elasticsearch {
      hosts => ["localhost"]
      ilm_enabled => true
      index => "%{[@metadata][index]}"
      ilm_policy => "%{[@metadata][ilm]}"
    }
  }
}