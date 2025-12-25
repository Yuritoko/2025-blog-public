若依项目后端Docker部署

1.Dockerfile

注意：第一次构建会生成一次镜像，后面再执行docker-compose会复用这个镜像，如果jar包更改了请把这个镜像删了再执行docker-compose。

```dockerfile
# 使用 btgoose/jdk17 镜像，这里可以使用官方jdk，比如openJdk
FROM btgoose/jdk17

# 设置工作目录
WORKDIR /app

# 创建应用目录
RUN mkdir -p /ruoyi/uploadPath

# 添加应用JAR包 - 修正路径问题
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar

# 暴露端口
EXPOSE 8080

# 修正入口点路径
ENTRYPOINT ["java", "-jar", "app.jar"]
```

2.docker-compose

会自动拉取并且构建

```dockerfile
version: '3.8'

services:
  # MySQL服务
  mysql:
    image: mysql:8.0
    container_name: ruoyi-mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: ry-vue
      TZ: Asia/Shanghai
    command:
      - --character-set-server=utf8mb4
      - --collation-server=utf8mb4_unicode_ci
      - --default-time-zone=+08:00
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d
      - ./mysql/init:/docker-entrypoint-initdb.d
    networks:
      - ruoyi-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      timeout: 20s
      retries: 10

  # Redis服务
  redis:
    image: redis:7-alpine
    container_name: ruoyi-redis
    restart: unless-stopped
    command: redis-server --appendonly yes --requirepass ""
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - ruoyi-network

  # RuoYi应用
  ruoyi:
    build: .  # 自动执行 Dockerfile
    container_name: ruoyi-app
    restart: always
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
    ports:
      - "8080:8080"
    networks:
      - ruoyi-network  # 统一网络名称


# 数据卷定义
volumes:
  mysql_data:
  redis_data:
  upload_data:

# 网络定义
networks:
  ruoyi-network:
    driver: bridge
```

3.把application.yml的mysql和redis的localhost改为容器内的主机名（mysql、redis）

4.执行maven clean package 获取项目的jar包

5.执行docker-compose。

6.项目结构：

doc/
├── mysql/
│   └── init/
│       └──初始化sql，能自动执行
├── Dockerfile

├──docker-compose.yml

└──target

​	└── ruoyi-admin.jar