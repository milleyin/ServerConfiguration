# SafetyCheck

> Lightweight, low-noise Linux security snapshot & integrity checker  
> 一个**不吵、不瞎报、可长期运行**的服务器安全巡检脚本

---

## 设计目标

SafetyCheck 不是入侵检测系统（IDS），也不是实时防御工具。它的目标只有一个：

**把一台服务器在某个时间点的“安全状态”，完整、可回溯地记录下来，并在出现“真正值得注意的变化”时提醒你。**

核心设计理念：

- **终端输出极简**
  - 只显示确实值得人介入的风险
  - 日常运行基本全是 `✅ OK`
- **日志完整快照**
  - 每次执行都会生成一份完整日志
  - 可回放任意时间点的系统状态
- **人为降噪，而不是规则堆砌**
  - nginx 新连接是正常的
  - ssh 是你自己连的
  - 内存抖动不是风险  
  这些不会打扰你

---

## 适用场景

- 单机 / 少量服务器的 DevOps / 技术负责人
- 不想天天盯安全告警，但又必须知道有没有“不对劲”
- 经历过一次入侵后，希望：
  - 有事后证据
  - 有“心理安全感”

---

## 检查内容一览

每次运行，SafetyCheck 会执行以下检查：

### 1. 进程变化（Process）

- 对比进程快照
- **仅在发现可疑行为特征时报警**，例如：
  - `/tmp` / `/dev/shm`
  - `curl | bash`
  - `bash -c` / `sh -c`
  - `nc` / `socat` / `python -c`

完整 diff 会写入日志，终端只输出可疑项。

### 2. 网络连接（Established）

- 仅关注新出现的已建立连接
- **默认排除**：
  - `sshd`
  - `nginx`（Web 服务新 IP 连入属于正常现象）

所有变化都会记录进日志。

### 3. 监听端口（Listen Ports）

- 新增监听端口会提示
- 用于发现后门服务悄悄监听

### 4. Cron 任务（高风险）

- 任意变更都会提示
- Cron 是典型持久化手段

### 5. 用户 / 权限 / sudo（高风险）

- 用户、组、sudoers 变更会提示
- 可用于发现新增账号、提权、sudo 权限漂移

### 6. SSH authorized_keys（高风险）

- 同时记录：
  - 路径
  - 权限
  - 所属用户/组
  - key 内容

任何变化都会提示。

### 7. systemd 自启动服务（高风险）

- 新启用的服务会提示
- 防止开机持久化

### 8. 防火墙规则（iptables）

- 使用 `iptables-save` 快照
- 规则变更会提示
- fail2ban 动态变化可追溯

### 9. 登录行为（仅记录）

- 最近 50 条登录记录
- 默认仅记录到日志，不在终端报警（避免自己登录刷屏）

---

## 目录结构

```text
~/safetyCheck/
├── baseline/        # 基线快照（首次运行自动生成）
│   ├── ps.txt
│   ├── net.txt
│   ├── listen.txt
│   ├── cron.txt
│   ├── userperm.txt
│   ├── sshkeys.txt
│   ├── systemd.txt
│   └── iptables.txt
│
└── logs/            # 每次运行生成一个完整日志
    └── YYYY-MM-DD_HH-MM-SS.log
```

---

## 使用方式

### 一行命令运行（推荐）

```bash
curl -fsSL https://raw.githubusercontent.com/milleyin/ServerConfiguration/main/StatusCheck/commonCheck | bash
```

### 建议运行频率

- 手动巡检：登录服务器后跑一次
- 自动化（可选）：每天 1 次（cron）
- 不建议太频繁（避免基线被动态噪音“带跑偏”）

---

## 使用建议

- 首次运行会建立 Baseline（正常现象）
- 看到 `⚠️ ATTENTION REQUIRED`：
  1. 先看终端输出（已尽量降噪）
  2. 再打开对应 `.log` 文件做完整分析
- 建议保留日志作为时间线证据，不要随手清理

---

## 设计哲学

SafetyCheck 刻意 **不做** 的事情：

- 不联网
- 不上传数据
- 不自动封禁
- 不做“智能决策”

它只做两件事：

1. **快照**
2. **对比**

最终决策仍然交给人。

---

## License

Private / Internal Use  
如需在其他环境使用，请自行审查与调整规则。
