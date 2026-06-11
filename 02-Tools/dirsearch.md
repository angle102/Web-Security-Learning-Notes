# Dirsearch

# 一、dirsearch

## 1.1 工具定位

dirsearch 是常用的 Web 目录扫描工具。

主要作用：

```text
发现隐藏路径
发现后台入口
发现接口路径
发现备份文件
发现配置文件
发现上传目录
发现源码泄露目录
```

简单理解：

```text
dirsearch = Web 目录和敏感文件枚举工具
```

---

## 1.2 使用场景

dirsearch 常用于：

```text
1. 登录框后隐藏路径发现
2. 后台入口发现
3. 备份文件发现
4. Swagger / API 文档发现
5. .git / .env 泄露发现
6. 上传目录发现
7. Java Web 敏感路径发现
```

---

## 1.3 常用命令

### 基础扫描

```bash
dirsearch -u https://example.com -e php,html,js,json,txt,bak,zip -t 5
```

参数说明：

```text
-u：目标 URL
-e：扩展名
-t：线程数
```

---

### 降低线程扫描

```bash
dirsearch -u https://example.com -e php,js,json,txt -t 2 --delay 1
```

适合敏感业务系统，降低请求压力。

---

### 指定字典

```bash
dirsearch -u https://example.com -w wordlist.txt -e php,html,js,json,txt
```

---

## 4.4 高价值路径

| 路径                 | 可能含义           |
| ------------------ | -------------- |
| `/admin`           | 后台入口           |
| `/login`           | 登录页面           |
| `/backend`         | 后台目录           |
| `/manage`          | 管理系统           |
| `/console`         | 控制台            |
| `/api`             | API 接口         |
| `/swagger`         | Swagger 文档     |
| `/swagger-ui.html` | Swagger UI     |
| `/v2/api-docs`     | Swagger JSON   |
| `/openapi.json`    | OpenAPI 文档     |
| `/actuator`        | Spring Boot 监控 |
| `/upload`          | 上传目录           |
| `/uploads`         | 上传文件目录         |
| `/backup.zip`      | 备份压缩包          |
| `/www.zip`         | 网站源码包          |
| `/db.sql`          | 数据库备份          |
| `/.git/`           | Git 源码泄露       |
| `/.env`            | 环境变量泄露         |
| `/WEB-INF/web.xml` | Java Web 配置泄露  |

---

## 1.5 状态码分析

| 状态码     | 含义    | 价值          |
| ------- | ----- | ----------- |
| 200     | 正常访问  | 路径存在        |
| 301/302 | 跳转    | 可能是后台或登录入口  |
| 401     | 需要认证  | 资源存在，需要身份认证 |
| 403     | 禁止访问  | 目录可能存在      |
| 404     | 不存在   | 用于判断伪 404   |
| 500     | 服务端错误 | 可能触发后端逻辑    |

注意：

```text
目录扫描结果不能只看 200。
401、403、500 也经常有价值。
```

---

