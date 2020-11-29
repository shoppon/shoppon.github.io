# 配置

下载docker镜像：`docker pull manictime/manictimeserver`

创建用户：`docker run -v /var/lib/manictimeserver/Data:/app/Data --rm --entrypoint dotnet manictime/manictimeserver ManicTimeServer.dll addadmin -u shopppon@gmail.com -p xxx`

启动服务：`docker run -d -v /var/lib/manictimeserver/Data:/app/Data -p 8086:8080 manictime/manictimeserver`

# 分析

