# HTTP 与 TCP

## 目录

1. [TCP/IP 协议栈概述](#1-tcpip-协议栈概述)
2. [TCP 协议详解](#2-tcp-协议详解)
3. [HTTP 协议详解](#3-http-协议详解)
4. [HTTPS 请求流程](#4-https-请求流程)
5. [HTTP vs HTTPS 对比](#5-http-vs-https-对比)
6. [实战案例](#6-实战案例)

---

## 1. TCP/IP 协议栈概述

### 1.1 OSI 七层模型

```mermaid
flowchart TD
    OSI1_A[7. 应用层] --> OSI1_B[6. 表示层]
    OSI1_B --> OSI1_C[5. 会话层]
    OSI1_C --> OSI1_D[4. 传输层]
    OSI1_D --> OSI1_E[3. 网络层]
    OSI1_E --> OSI1_F[2. 数据链路层]
    OSI1_F --> OSI1_G[1. 物理层]
    
    style OSI1_A fill:#ffe0b2
    style OSI1_D fill:#c8e6c9
    style OSI1_E fill:#c8e6c9
```

### 1.2 TCP/IP 四层模型

| 层级 | 协议 | 职责 |
|------|------|------|
| **应用层** | HTTP、HTTPS、FTP、DNS | 应用程序通信 |
| **传输层** | TCP、UDP | 端到端传输 |
| **网络层** | IP、ICMP、ARP | 路由和寻址 |
| **链路层** | Ethernet、Wi-Fi | 物理介质传输 |

### 1.3 各层协议职责

```mermaid
flowchart LR
    subgraph 应用层
        IP1_A[HTTP/HTTPS]
    end
    
    subgraph 传输层
        IP1_B[TCP]
    end
    
    subgraph 网络层
        IP1_C[IP]
    end
    
    subgraph 链路层
        IP1_D[Ethernet]
    end
    
    IP1_A --> IP1_B --> IP1_C --> IP1_D
```

---

## 2. TCP 协议详解

### 2.1 TCP 特性

| 特性 | 说明 |
|------|------|
| **面向连接** | 建立连接后才传输数据 |
| **可靠传输** | 确认重传、序列号、校验和 |
| **面向字节流** | 无边界的字节序列 |
| **流量控制** | 滑动窗口机制 |
| **拥塞控制** | 慢启动、拥塞避免 |

### 2.2 TCP 三次握手（Three-way Handshake）

#### 三次握手流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Server as 服务器
    
    Client->>Server: SYN (seq=x)
    Note over Client,Server: 第一次握手<br/>客户端请求建立连接<br/>SYN=1, seq=x
    
    Server->>Client: SYN+ACK (seq=y, ack=x+1)
    Note over Client,Server: 第二次握手<br/>服务器确认并同意连接<br/>SYN=1, ACK=1, seq=y, ack=x+1
    
    Client->>Server: ACK (ack=y+1)
    Note over Client,Server: 第三次握手<br/>客户端确认，连接建立<br/>ACK=1, ack=y+1
```

#### 标志位说明

| 标志位 | 含义 |
|--------|------|
| **SYN** | Synchronize，请求同步 |
| **ACK** | Acknowledgment，确认 |
| **FIN** | Finish，结束连接 |

#### 为什么需要三次握手？

```mermaid
flowchart TD
    TCP1_A[三次握手目的] --> TCP1_B[确认双方能力]
    TCP1_A --> TCP1_C[同步序列号]
    TCP1_A --> TCP1_D[防止已失效连接]
    
    TCP1_B --> TCP1_B1[客户端: 我能发]
    TCP1_B --> TCP1_B2[服务器: 我能收/发]
    TCP1_B --> TCP1_B3[客户端: 我能收]
    
    TCP1_D --> TCP1_D1[避免旧连接报文]
    TCP1_D --> TCP1_D2[保证连接可靠性]
    
    style TCP1_B1 fill:#c8e6c9
    style TCP1_B2 fill:#c8e6c9
    style TCP1_B3 fill:#c8e6c9
```

#### 状态转换

```mermaid
flowchart TD
    TCP2_A[客户端] -->|SYN| TCP2_B(SYN_SENT)
    TCP2_B -->|SYN+ACK| TCP2_C(ESTABLISHED)
    
    TCP2_D[服务器] -->|SYN| TCP2_E(SYN_RCVD)
    TCP2_E -->|ACK| TCP2_C
    
    style TCP2_A fill:#ffe0b2
    style TCP2_D fill:#ffe0b2
    style TCP2_C fill:#c8e6c9
```

### 2.3 TCP 四次挥手（Four-way Handshake）

#### 四次挥手流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Server as 服务器
    
    Client->>Server: FIN (seq=x)
    Note over Client,Server: 第一次挥手<br/>客户端请求断开<br/>FIN=1, seq=x
    
    Server->>Client: ACK (ack=x+1)
    Note over Client,Server: 第二次挥手<br/>服务器确认收到<br/>ACK=1, ack=x+1
    
    Server->>Client: FIN (seq=y)
    Note over Client,Server: 第三次挥手<br/>服务器准备断开<br/>FIN=1, seq=y
    
    Client->>Server: ACK (ack=y+1)
    Note over Client,Server: 第四次挥手<br/>客户端确认，连接断开<br/>ACK=1, ack=y+1
```

#### 为什么需要四次挥手？

```mermaid
flowchart TD
    FIN1_A[四次挥手原因] --> FIN1_B[半关闭状态]
    FIN1_A --> FIN1_C[数据传输完成]
    
    FIN1_B --> FIN1_B1[客户端先关闭发送]
    FIN1_B --> FIN1_B2[服务器继续发送数据]
    FIN1_B --> FIN1_B3[服务器再关闭发送]
    
    style FIN1_B1 fill:#ffe0b2
    style FIN1_B2 fill:#c8e6c9
    style FIN1_B3 fill:#ffe0b2
```

#### TIME_WAIT 状态详解

```mermaid
flowchart TD
    TIME1_A[TIME_WAIT状态] --> TIME1_B[等待2MSL]
    TIME1_B --> TIME1_C[确保最后ACK到达]
    TIME1_B --> TIME1_D[防止旧连接干扰]
    
    TIME1_C --> TIME1_C1[MSL: 最大报文生存时间]
    TIME1_C --> TIME1_C2[通常2分钟]
    
    TIME1_D --> TIME1_D1[等待旧报文失效]
    TIME1_D --> TIME1_D2[避免端口冲突]
    
    style TIME1_B fill:#ffe0b2
```

### 2.4 TCP 状态机

#### 11 种状态转换

```mermaid
flowchart TD
    STATE1_A[CLOSED] -->|listen| STATE1_B(LISTEN)
    STATE1_B -->|SYN| STATE1_C(SYN_RCVD)
    STATE1_C -->|ACK| STATE1_D(ESTABLISHED)
    
    STATE1_A -->|connect/SYN| STATE1_E(SYN_SENT)
    STATE1_E -->|SYN+ACK| STATE1_D
    
    STATE1_D -->|close/FIN| STATE1_F(FIN_WAIT_1)
    STATE1_F -->|ACK| STATE1_G(FIN_WAIT_2)
    STATE1_F -->|FIN+ACK| STATE1_H(CLOSING)
    
    STATE1_G -->|FIN| STATE1_I(TIME_WAIT)
    STATE1_H -->|ACK| STATE1_I
    STATE1_I -->|2MSL| STATE1_A
    
    STATE1_D -->|FIN| STATE1_J(CLOSE_WAIT)
    STATE1_J -->|close/FIN| STATE1_K(LAST_ACK)
    STATE1_K -->|ACK| STATE1_A
    
    style STATE1_D fill:#c8e6c9
    style STATE1_I fill:#ffe0b2
```

#### 状态转换表

| 状态 | 说明 |
|------|------|
| **CLOSED** | 初始状态 |
| **LISTEN** | 监听连接 |
| **SYN_SENT** | 已发送 SYN |
| **SYN_RCVD** | 已收到 SYN |
| **ESTABLISHED** | 连接建立 |
| **FIN_WAIT_1** | 等待对方 FIN |
| **FIN_WAIT_2** | 等待对方 FIN |
| **CLOSE_WAIT** | 等待关闭 |
| **CLOSING** | 双方同时关闭 |
| **LAST_ACK** | 等待最后 ACK |
| **TIME_WAIT** | 等待 2MSL |

---

## 3. HTTP 协议详解

### 3.1 HTTP 特点

| 特点 | 说明 |
|------|------|
| **无状态** | 不保存会话状态 |
| **基于请求-响应** | 客户端发起请求，服务器响应 |
| **明文传输** | 数据不加密 |
| **灵活** | 支持多种方法和头部 |

### 3.2 HTTP 请求结构

```mermaid
flowchart TD
    HTTP1_A[HTTP请求] --> HTTP1_B[请求行]
    HTTP1_A --> HTTP1_C[请求头]
    HTTP1_A --> HTTP1_D[空行]
    HTTP1_A --> HTTP1_E[请求体]
    
    HTTP1_B --> HTTP1_B1[方法 + URL + 版本]
    HTTP1_C --> HTTP1_C1[Host, User-Agent, Accept...]
    HTTP1_E --> HTTP1_E1[POST数据]
```

#### 请求行示例

```
GET /index.html HTTP/1.1
```

#### 请求头示例

```http
Host: www.example.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)
Accept: text/html,application/xhtml+xml
Accept-Language: zh-CN,zh;q=0.9
```

### 3.3 HTTP 响应结构

```mermaid
flowchart TD
    HTTP2_A[HTTP响应] --> HTTP2_B[状态行]
    HTTP2_A --> HTTP2_C[响应头]
    HTTP2_A --> HTTP2_D[空行]
    HTTP2_A --> HTTP2_E[响应体]
    
    HTTP2_B --> HTTP2_B1[版本 + 状态码 + 原因短语]
    HTTP2_C --> HTTP2_C1[Content-Type, Content-Length...]
    HTTP2_E --> HTTP2_E1[HTML/JSON/二进制]
```

#### 状态行示例

```
HTTP/1.1 200 OK
```

#### 常见状态码

| 状态码 | 含义 |
|--------|------|
| **200** | 请求成功 |
| **301** | 永久重定向 |
| **302** | 临时重定向 |
| **400** | 请求错误 |
| **401** | 未授权 |
| **403** | 禁止访问 |
| **404** | 资源未找到 |
| **500** | 服务器错误 |
| **503** | 服务不可用 |

### 3.4 HTTP 方法

| 方法 | 含义 | 是否幂等 |
|------|------|---------|
| **GET** | 获取资源 | 是 |
| **POST** | 创建资源 | 否 |
| **PUT** | 更新资源 | 是 |
| **DELETE** | 删除资源 | 是 |
| **HEAD** | 获取响应头 | 是 |
| **OPTIONS** | 获取支持的方法 | 是 |
| **PATCH** | 部分更新 | 否 |

---

## 4. HTTPS 请求流程

### 4.1 HTTPS 原理

#### SSL/TLS 协议

```mermaid
flowchart TD
    HTTPS1_A[HTTPS] --> HTTPS1_B[SSL/TLS]
    HTTPS1_B --> HTTPS1_C[握手协议]
    HTTPS1_B --> HTTPS1_D[记录协议]
    
    HTTPS1_C --> HTTPS1_C1[协商加密参数]
    HTTPS1_C --> HTTPS1_C2[交换密钥]
    
    HTTPS1_D --> HTTPS1_D1[分段]
    HTTPS1_D --> HTTPS1_D2[压缩]
    HTTPS1_D --> HTTPS1_D3[加密]
    HTTPS1_D --> HTTPS1_D4[传输]
```

#### 对称加密与非对称加密

```mermaid
flowchart LR
    subgraph 对称加密
        ENC1_A[相同密钥] --> ENC1_B[加密/解密]
        ENC1_B --> ENC1_C[速度快]
    end
    
    subgraph 非对称加密
        ENC1_D[公钥/私钥] --> ENC1_E[公钥加密]
        ENC1_E --> ENC1_F[私钥解密]
        ENC1_F --> ENC1_G[速度慢]
    end
    
    style ENC1_C fill:#c8e6c9
    style ENC1_G fill:#ffe0b2
```

#### 数字证书

```mermaid
flowchart TD
    CERT1_A[数字证书] --> CERT1_B[证书内容]
    CERT1_A --> CERT1_C[CA签名]
    
    CERT1_B --> CERT1_B1[公钥]
    CERT1_B --> CERT1_B2[域名]
    CERT1_B --> CERT1_B3[有效期]
    CERT1_B --> CERT1_B4[颁发机构]
    
    CERT1_C --> CERT1_C1[防止篡改]
    CERT1_C --> CERT1_C2[验证身份]
```

### 4.2 HTTPS 完整请求流程

```mermaid
flowchart TD
    FLOW1_A[用户访问 https://example.com] --> FLOW1_B[TCP三次握手]
    FLOW1_B --> FLOW1_C[TLS握手]
    FLOW1_C --> FLOW1_D[HTTP请求]
    FLOW1_D --> FLOW1_E[HTTP响应]
    FLOW1_E --> FLOW1_F[TCP四次挥手]
    
    style FLOW1_B fill:#c8e6c9
    style FLOW1_C fill:#ffe0b2
    style FLOW1_D fill:#c8e6c9
```

### 4.3 TLS 握手详解

#### 完整 TLS 1.2 握手流程

```mermaid
sequenceDiagram
    participant Client as 客户端
    participant Server as 服务器
    
    Client->>Server: 1. Client Hello
    Note right of Client: 支持的协议版本<br/>支持的加密套件<br/>随机数<br/>Session ID
    
    Server->>Client: 2. Server Hello
    Note right of Server: 选定协议版本<br/>选定加密套件<br/>随机数<br/>Session ID
    
    Server->>Client: 3. Certificate
    Note right of Server: 服务器证书<br/>包含公钥
    
    Server->>Client: 4. Server Key Exchange
    Note right of Server: 额外密钥参数(可选)
    
    Server->>Client: 5. Server Hello Done
    Note right of Server: 服务器握手结束
    
    Client->>Server: 6. Client Key Exchange
    Note right of Client: 预主密钥(加密)
    
    Client->>Server: 7. Change Cipher Spec
    Note right of Client: 后续用新密钥
    
    Client->>Server: 8. Finished
    Note right of Client: 验证握手
    
    Server->>Client: 9. Change Cipher Spec
    Server->>Client: 10. Finished
```

#### TLS 握手阶段总结

| 阶段 | 客户端操作 | 服务器操作 |
|------|-----------|-----------|
| **1-2** | 发送支持的套件 | 选择加密套件 |
| **3-5** | - | 发送证书 |
| **6-8** | 发送预主密钥 | - |
| **9-10** | - | 确认加密 |

---

## 5. HTTP vs HTTPS 对比

### 对比表格

| 特性 | HTTP | HTTPS |
|------|------|-------|
| **安全性** | 明文传输，不安全 | 加密传输，安全 |
| **端口** | 80 | 443 |
| **证书** | 不需要 | 需要 CA 证书 |
| **性能** | 较快（无加密开销） | 较慢（加密开销） |
| **搜索引擎** | 不影响排名 | 优先收录 |
| **SEO** | 一般 | 更好 |
| **缓存** | 简单 | 需要配置 |
| **CDN支持** | 简单 | 需要特殊配置 |

### 性能对比

```mermaid
flowchart LR
    PERF1_A[HTTP请求] --> PERF1_B[直接传输]
    PERF1_B --> PERF1_C[响应]
    
    PERF1_D[HTTPS请求] --> PERF1_E[TCP握手]
    PERF1_E --> PERF1_F[TLS握手]
    PERF1_F --> PERF1_G[加密传输]
    PERF1_G --> PERF1_H[响应]
    
    style PERF1_B fill:#c8e6c9
    style PERF1_F fill:#ffe0b2
```

---

## 6. 实战案例

### 6.1 使用 Wireshark 抓包分析

#### 抓包步骤

```mermaid
flowchart TD
    WIRESHARK1_A[打开Wireshark] --> WIRESHARK1_B[选择网卡]
    WIRESHARK1_B --> WIRESHARK1_C[设置过滤规则]
    WIRESHARK1_C --> WIRESHARK1_D[开始捕获]
    WIRESHARK1_D --> WIRESHARK1_E[访问网站]
    WIRESHARK1_E --> WIRESHARK1_F[停止捕获]
    WIRESHARK1_F --> WIRESHARK1_G[分析数据包]
```

#### 过滤规则

```
tcp.port == 443 && tls
```

#### 分析 TCP 握手

```mermaid
flowchart TD
    CAP1_A[抓包结果] --> CAP1_B[筛选TCP]
    CAP1_B --> CAP1_C[找到SYN]
    CAP1_C --> CAP1_D[找到SYN+ACK]
    CAP1_D --> CAP1_E[找到ACK]
    CAP1_E --> CAP1_F[确认三次握手]
```

### 6.2 使用 curl 测试

#### HTTP 请求示例

```bash
curl http://example.com
```

#### HTTPS 请求示例

```bash
curl https://example.com
```

#### 详细输出

```bash
curl -v https://example.com
```

---

## 总结

### TCP 核心要点

```mermaid
flowchart LR
    TCP_SUM_A[TCP] --> TCP_SUM_B[三次握手建立连接]
    TCP_SUM_A --> TCP_SUM_C[可靠传输数据]
    TCP_SUM_A --> TCP_SUM_D[四次挥手断开连接]
    
    TCP_SUM_B --> TCP_SUM_B1[SYN → SYN+ACK → ACK]
    TCP_SUM_D --> TCP_SUM_D1[FIN → ACK → FIN → ACK]
    
    style TCP_SUM_B1 fill:#c8e6c9
    style TCP_SUM_D1 fill:#c8e6c9
```

### HTTPS 核心要点

```mermaid
flowchart LR
    HTTPS_SUM_A[HTTPS] --> HTTPS_SUM_B[TCP握手]
    HTTPS_SUM_A --> HTTPS_SUM_C[TLS握手]
    HTTPS_SUM_A --> HTTPS_SUM_D[加密传输]
    
    HTTPS_SUM_B --> HTTPS_SUM_B1[建立连接]
    HTTPS_SUM_C --> HTTPS_SUM_C1[协商密钥]
    HTTPS_SUM_D --> HTTPS_SUM_D1[安全通信]
    
    style HTTPS_SUM_B1 fill:#c8e6c9
    style HTTPS_SUM_C1 fill:#ffe0b2
    style HTTPS_SUM_D1 fill:#c8e6c9
```

### 一次 HTTPS 请求完整流程

1. **DNS 解析**：将域名转换为 IP
2. **TCP 三次握手**：建立连接
3. **TLS 握手**：协商加密参数、交换密钥
4. **发送 HTTP 请求**：加密后的请求数据
5. **接收 HTTP 响应**：加密后的响应数据
6. **TCP 四次挥手**：断开连接

---

## 参考资料

1. [TCP/IP 详解 卷1：协议](https://www.amazon.com/TCP-Illustrated-Vol-1-Protocols/dp/0201633469)
2. [HTTP 权威指南](https://www.amazon.com/HTTP-Definitive-Guide-Guides/dp/1565925092)
3. [TLS 1.3 规范](https://datatracker.ietf.org/doc/html/rfc8446)
