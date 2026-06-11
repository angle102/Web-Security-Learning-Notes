# Nmap



# 一、Nmap

## 1.1 工具定位

Nmap 是常用的端口扫描和服务识别工具。

主要作用：

```text
发现主机是否存活
发现开放端口
识别服务类型
识别服务版本
判断操作系统
执行基础 NSE 脚本检测
```

简单理解：

```text
Nmap = 资产和端口暴露面识别工具
```

---

## 1.2 使用场景

Nmap 常用于：

```text
1. 授权目标端口扫描
2. 服务版本识别
3. Web 服务发现
4. 数据库暴露检查
5. Redis / MongoDB / Elasticsearch 暴露检查
6. 内网主机服务枚举
7. 渗透测试报告资产清单整理
```

---

## 1.3 常用命令

### 基础扫描

```bash
nmap -Pn example.com
```

参数说明：

```text
-Pn：不进行主机发现，直接扫描端口
```

适合目标禁 Ping 的情况。

---

### 服务版本识别

```bash
nmap -sV -sC -Pn example.com
```

参数说明：

```text
-sV：识别服务版本
-sC：使用默认 NSE 脚本
-Pn：跳过主机发现
```

---

### 指定端口扫描

```bash
nmap -sV -sC -Pn -p 80,443,8080,8443 example.com
```

适合重点识别 Web 服务。

---

### 常见端口扫描

```bash
nmap -sS -Pn -T3 -p 21,22,80,443,3306,3389,5432,6379,8080,8443,8888,9200,27017 example.com
```

参数说明：

```text
-sS：SYN 扫描
-T3：中等速度，比较稳妥
-p：指定端口
```

---

## 1.4 常见端口价值

| 端口    | 服务                     | 测试价值             |
| ----- | ---------------------- | ---------------- |
| 21    | FTP                    | 匿名登录、文件泄露        |
| 22    | SSH                    | 远程管理、弱口令风险       |
| 80    | HTTP                   | Web 服务           |
| 443   | HTTPS                  | Web 服务           |
| 3306  | MySQL                  | 数据库暴露            |
| 5432  | PostgreSQL             | 数据库暴露            |
| 6379  | Redis                  | 未授权访问、Session 泄露 |
| 8080  | Web / Tomcat / Jenkins | 管理后台             |
| 8443  | HTTPS 管理端              | 后台、控制台           |
| 8888  | 宝塔面板                   | 运维面板             |
| 9200  | Elasticsearch          | 日志、数据泄露          |
| 27017 | MongoDB                | 数据库暴露            |

---

## 1.5 Nmap 结果分析

示例结果：

```text
80/tcp   open  http     nginx
443/tcp  open  https    nginx
3306/tcp open  mysql    MySQL 5.7
6379/tcp open  redis    Redis key-value store
8080/tcp open  http     Apache Tomcat
```

分析：

```text
80/443：存在 Web 服务，需要继续访问和目录扫描
3306：MySQL 暴露，需要判断是否仅内网访问、是否弱口令
6379：Redis 暴露，需要判断是否开启认证
8080：可能是 Tomcat 或其他后台，需要进一步访问
```

报告中应记录：

```text
目标 IP
开放端口
服务名称
服务版本
风险判断
下一步测试建议
```

---


