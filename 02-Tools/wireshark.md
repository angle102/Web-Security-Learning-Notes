# Wireshark

# 七、Wireshark

## 7.1 工具定位

Wireshark 是常用的网络抓包和流量分析工具。

主要作用：

```text
抓取网络流量
分析协议交互
查看 HTTP 请求
分析 DNS 查询
分析 TCP 连接
提取文件内容
还原通信过程
辅助流量取证
```

简单理解：

```text
Wireshark = 网络流量显微镜
```

---

## 7.2 使用场景

Wireshark 常用于：

```text
1. CTF 流量分析
2. 应急响应流量排查
3. 恶意通信分析
4. HTTP 文件传输分析
5. 邮件协议分析
6. DNS 请求分析
7. 登录明文协议排查
8. 入侵溯源取证
9. 数据包还原
```

---

## 7.3 常用过滤器

### HTTP 流量

```text
http
```

### 指定 IP

```text
ip.addr == 192.168.1.10
```

### 指定源 IP

```text
ip.src == 192.168.1.10
```

### 指定目标 IP

```text
ip.dst == 192.168.1.10
```

### 指定端口

```text
tcp.port == 80
```

### DNS 流量

```text
dns
```

### TCP 流量

```text
tcp
```

### 查找 GET 请求

```text
http.request.method == "GET"
```

### 查找 POST 请求

```text
http.request.method == "POST"
```

### 查找包含关键词的 HTTP 请求

```text
http contains "password"
```

---

## 7.4 常见分析思路

### HTTP 分析

重点看：

```text
请求 URL
请求方法
Host
Cookie
User-Agent
POST 数据
响应状态码
返回内容
上传和下载文件
```

---

### DNS 分析

重点看：

```text
可疑域名
频繁查询
DGA 域名特征
内网主机解析记录
恶意 C2 域名
```

---

### TCP 流分析

可以右键数据包：

```text
Follow → TCP Stream
```

作用：

```text
还原完整通信内容
查看 HTTP 请求和响应
查看明文账号密码
查看文件传输内容
```

---

### 文件提取

如果是 HTTP 文件传输，可以尝试：

```text
File → Export Objects → HTTP
```

作用：

```text
导出 HTTP 传输的图片、压缩包、脚本、文档等对象
```

---

## 7.5 Wireshark 输出内容

可以整理到报告中的内容：

```text
通信双方 IP
通信时间
协议类型
请求路径
关键请求包
上传或下载文件
异常域名
异常端口
可疑 Payload
攻击行为时间线
```

---

# 八、工具在完整渗透流程中的位置

| 阶段     | 工具                  | 主要作用          |
| ------ | ------------------- | ------------- |
| 信息收集   | Nmap、dirsearch      | 端口、服务、路径发现    |
| Web 测试 | Burp Suite          | 抓包、改包、重放、验证漏洞 |
| 漏洞扫描   | Nessus、OpenVAS      | 自动化漏洞发现       |
| 流量分析   | Wireshark           | 协议分析、取证、文件提取  |
| 报告整理   | Markdown、Excel、截图工具 | 输出证据和修复建议     |

完整流程示例：

```text
Nmap 发现 80、443、8080 端口
        ↓
访问 Web 服务，Burp Suite 抓包
        ↓
dirsearch 发现 /admin、/api、/swagger
        ↓
Burp Suite 验证登录、越权、上传漏洞
        ↓
Nessus / OpenVAS 扫描主机漏洞和配置风险
        ↓
Wireshark 分析异常流量或靶场数据包
        ↓
整理漏洞复现步骤和报告
```

---

