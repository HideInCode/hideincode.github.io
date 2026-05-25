---
title: Nginx相关
date: 2019-07-01
categories:
- 开发部署
tags:
- devops
---



# 安装部署

## docker下安装

1. docker pull nginx 

2. docker run --name nginx -p 9000:80 -d nginx 先启动一个容器用来复制配置

3. 生成挂载文件地址

   ```bash
   mkdir -p /opt/docker/nginx/conf
   mkdir -p /opt/docker/nginx/html
   mkdir -p /opt/docker/nginx/logs
   
   docker cp nginx:/etc/nginx/nginx.conf /opt/docker/nginx/conf/nginx.conf
   docker cp nginx:/etc/nginx/conf.d /opt/docker/nginx/conf/conf.d
   docker cp nginx:/usr/share/nginx/html /opt/docker/nginx
   ```

   

4. 干掉这个nginx

   ``` bash
   docker stop nginx 
   docker rm nginx
   ```

5. 以80端口启动，一行命令。

   ```bash
   docker run -p 80:80 --name nginx --restart=always -v /opt/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /opt/docker/nginx/conf/conf.d:/etc/nginx/conf.d -v /opt/docker/nginx/html:/usr/share/nginx/html -v /opt/docker/nginx/logs:/var/log/nginx -d nginx
   
   ```

   

## 常见问题

### 配置遗漏

1. 每句配置结尾有分号！！！

### 转发不到

1. 查看error.log，请求路径是否符合预期。

## 斜杠问题

**请求为**：test/a/b

**配置为**：

```bash
location test/ {
	proxy_pass http://localhost:8080
}
```



|                                     | location:test/                 | location:test                  |
| ----------------------------------- | ------------------------------ | ------------------------------ |
| proxy_pass:http://localhost:8080    | http://localhost:8080/test/a/b | http://localhost:8080/test/a/b |
| proxy_pass:http://localhost:8080/   | http://localhost:8080/a/b      | http://localhost:8080//a/b     |
| proxy_pass:http://localhost:8080/a  | http://localhost:8080/testa/b  | http://localhost:8080/test/a/b |
| proxy_pass:http://localhost:8080/a/ | http://localhost:8080/test/a/b | http://localhost:8080/test/a/b |

**结论**

- proxy_pass代理地址端口后无任何字符，转发后地址：代理地址+访问URL目录部分
- proxy_pass代理地址端口后有目录(包括 / )，转发后地址：代理地址+访问URL目录部分去除location匹配目录（示例中的"v1"或"v1/"）