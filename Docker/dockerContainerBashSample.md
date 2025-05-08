# 重新啟用最新版本容器的 Docker 指令

以下指令用於停止並移除現有容器，刪除舊鏡像，然後運行最新版本的容器。

## Bash 指令
```bash
# 停止並移除名為 "line" 的容器
docker stop line && docker rm line

# 刪除指定的 Docker 鏡像
docker rmi docker.testzone.net/line

# 運行新容器，使用最新鏡像並配置環境
docker run -d --restart always --network=msg \
  -e "ASPNETCORE_ENVIRONMENT=Lab" \
  -e "http_proxy=http://172.21.11.11:8080" \
  -e "https_proxy=http://172.21.11.11:8080" \
  -e "no_proxy=.testzone.net" \
  -v /home/admin/log/test/:/var/log \
  -v /mnt/test/new_media:/app/wwwroot/new_media \
  --name testline docker.testzone.net/line
  ```