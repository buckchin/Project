# 常用 Docker 指令

## 常用的指令
```bash
# 構建 Docker 鏡像並指定標籤
docker build -t imgrepo.test.net/line/lineapi .

# 將 Docker 鏡像推送到倉庫
docker push imgrepo.test.net/line/lineapi

# 列出正在運行的容器
docker ps

# 進入運行中容器的 bash 環境
docker exec -it line bash


# 列出所有容器（包括已停止的）
docker ps -a

# 從鏡像運行一個容器
docker run -d -p <主機端口>:<容器端口> <鏡像名稱>

# 停止運行中的容器
docker stop <容器ID>

# 刪除容器
docker rm <容器ID>

# 刪除 Docker 鏡像
docker rmi <鏡像名稱>

# 從倉庫拉取鏡像
docker pull <鏡像名稱>

# 查看容器日誌
docker logs <容器ID>

# 列出所有 Docker 鏡像
docker images
```