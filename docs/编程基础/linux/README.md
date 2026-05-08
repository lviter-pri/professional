# Linux系统

Linux是开源的类Unix操作系统，是服务器端最流行的操作系统之一。掌握Linux是后端开发工程师的必备技能。

## 一、Linux概述

### 1.1 Linux特点

- **开源免费**：源代码公开，可自由修改和分发
- **多用户多任务**：支持多个用户同时使用
- **稳定性高**：适合长期运行的服务器
- **安全性强**：权限管理严格
- **丰富的命令行工具**：强大的文本处理能力

### 1.2 发行版

| 发行版 | 特点 | 适用场景 |
|--------|------|----------|
| **CentOS** | 稳定、企业级 | 服务器 |
| **Ubuntu** | 用户友好、社区活跃 | 桌面/服务器 |
| **Debian** | 稳定、软件丰富 | 服务器 |
| **Red Hat** | 商业支持、企业级 | 企业服务器 |
| **Arch Linux** | 轻量、滚动更新 | 技术爱好者 |

## 二、文件系统

### 2.1 目录结构

```
/          # 根目录
├── bin/   # 基础命令二进制文件
├── etc/   # 配置文件
├── home/  # 用户主目录
├── lib/   # 系统库文件
├── opt/   # 可选软件安装目录
├── tmp/   # 临时文件
├── usr/   # 用户程序
├── var/   # 可变数据（日志等）
└── root/  # root用户主目录
```

### 2.2 文件权限

```
-rw-r--r--  1 user group  1024 May  8 10:00 file.txt
 ↑↑↑ ↑↑↑ ↑↑↑
 │││ │││ └─ 其他用户权限
 │││ │└─ 组权限
 │││ └─ 用户权限
 │└─ 组（group）
 └─ 用户（user）
```

权限说明：
- `r`：读取权限（4）
- `w`：写入权限（2）
- `x`：执行权限（1）

## 三、常用命令

### 3.1 文件操作

| 命令 | 功能 | 示例 |
|------|------|------|
| `ls` | 列出目录内容 | `ls -la` |
| `cd` | 切换目录 | `cd /home/user` |
| `pwd` | 显示当前目录 | `pwd` |
| `mkdir` | 创建目录 | `mkdir -p dir/subdir` |
| `rm` | 删除文件/目录 | `rm -rf dir` |
| `cp` | 复制文件 | `cp file1 file2` |
| `mv` | 移动/重命名 | `mv old new` |

### 3.2 文本处理

| 命令 | 功能 | 示例 |
|------|------|------|
| `cat` | 查看文件内容 | `cat file.txt` |
| `less` | 分页查看 | `less file.txt` |
| `grep` | 搜索内容 | `grep "pattern" file` |
| `sed` | 文本替换 | `sed 's/old/new/g' file` |
| `awk` | 文本处理 | `awk '{print $1}' file` |

### 3.3 系统管理

| 命令 | 功能 | 示例 |
|------|------|------|
| `top` | 查看进程 | `top` |
| `ps` | 列出进程 | `ps aux` |
| `kill` | 终止进程 | `kill -9 pid` |
| `df` | 磁盘使用 | `df -h` |
| `free` | 内存使用 | `free -h` |
| `systemctl` | 服务管理 | `systemctl restart nginx` |

### 3.4 用户管理

| 命令 | 功能 | 示例 |
|------|------|------|
| `useradd` | 添加用户 | `useradd user` |
| `userdel` | 删除用户 | `userdel user` |
| `groupadd` | 添加组 | `groupadd group` |
| `chown` | 修改所有者 | `chown user:group file` |
| `chmod` | 修改权限 | `chmod 755 file` |

## 四、Shell脚本基础

### 4.1 脚本结构

```bash
#!/bin/bash
# 这是注释

echo "Hello World"

# 变量
name="Linux"
echo "Hello $name"

# 条件判断
if [ -f file.txt ]; then
    echo "文件存在"
else
    echo "文件不存在"
fi

# 循环
for i in {1..5}; do
    echo $i
done
```

### 4.2 常用命令

| 命令 | 说明 |
|------|------|
| `echo` | 输出内容 |
| `read` | 读取输入 |
| `if` | 条件判断 |
| `for` | 循环 |
| `while` | 循环 |
| `case` | 多分支 |

## 五、系统运维

### 5.1 日志管理

- `/var/log/messages`：系统日志
- `/var/log/auth.log`：认证日志
- `/var/log/syslog`：系统日志（部分发行版）

### 5.2 定时任务

```bash
# 编辑定时任务
crontab -e

# 格式：分 时 日 月 周 命令
0 2 * * * /backup/script.sh
```

### 5.3 网络配置

```bash
# 查看网络接口
ip addr

# 测试网络
ping google.com

# 查看端口占用
netstat -tlnp
```

---

## 本目录包含的文档

- [CentOS基础操作命令](centos/基础操作命令.md)
