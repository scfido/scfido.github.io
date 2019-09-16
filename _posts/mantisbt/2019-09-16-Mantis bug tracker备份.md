
```sh
echo "开始备份"

docker exec -it MantisBT_mysql_1 mysqldump -umantisbt -pmantisbt --hex-blob --de
fault-character-set=utf8mb4 bugtracker > ./mantisbt.sql

echo "备份完成"
echo "请在本机执行以下命令将备份拷贝到本地"
echo "scp -r root@yourhost.com:/root/mantisbt.sql ./"
```