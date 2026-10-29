<h1 align="center">Qoder2API-Hub (多租户增强版)</h1>

<p align="center">
  <b>企业级·多租户·高可用的 Qoder (qoder.com.cn / qoder.com) -> OpenAI / Anthropic 协议网关</b><br>
  支持开放注册 · 自绑账号(BYOA) · 独立 API Key 与模型白名单 · 全员自动代签 · In-flight 削峰与 RPM 限流
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.9+-blue.svg?style=flat-square" alt="Python 3.9+">
  <img src="https://img.shields.io/badge/Dependencies-Zero-ff69b4.svg?style=flat-square" alt="Zero Dependencies">
  <img src="https://img.shields.io/badge/Multi--Tenant-Supported-brightgreen.svg?style=flat-square" alt="Multi-Tenant">
  <img src="https://img.shields.io/badge/License-MIT-green.svg?style=flat-square" alt="MIT">
</p>

---

## 🌟 二改核心升级亮点

本项目基于开源 `qoder2api-hub` 进行深度二次开发，将原本面向单机个人的网关重构升级为**支持多人共享与生产级管控的 Hub 平台**：

- 👥 **完全物理隔离的多用户体系**：开放用户注册与登录，首个注册用户自动成为系统管理员，多用户数据完全隔离。
- 🔑 **用户自主 API Key 与模型授权**：用户可为不同客户端创建独立的 API Key，并精准绑定该 Key 允许调用的**模型白名单**。
- 🔀 **智能私有池 + 公共池双层调度 (BYOA)**：
  - 用户自绑 Qoder 账号（私有池）：优先调用，**完全不限量**；
  - 公共池兜底：新用户无号自动借用公共池免费体验。
- 🛡️ **公共池每日配额防护**：普通用户每日使用公共池 Credits 设有限额（默认 100/天），防止单人脚本把公共池薅干。
- ⚡ **全站“防漏签”一键代签系统**：
  - 内置调度器每日自动为全站账号批量签到并领满福利包；
  - 管理员可在后台随时点击**「⚡ 为所有账号一键签到」**，用户哪怕几个月不上线，账号额度也不会流失。
- 🚦 **生产级请求并发与滑动窗口速率限制**：
  - 单 Key 活跃并发控制（In-flight Guard，默认 3 并发），流式结束原子释放，防打爆账号；
  - 60 秒精确滑动窗口 RPM 限制（默认 30 RPM），削峰填谷并带标准 `Retry-After` 退避通知。
- 💎 **纯 Python 标准库实现**：**零第三方 pip 依赖**，COSY 签名、AES-256-GCM 与 RSA 纯手搓，极速轻量部署。

📖 **深入了解技术架构与原理？请阅读：[📘 完整技术架构与设计文档 (docs/TECHNICAL_DOCS.md)](docs/TECHNICAL_DOCS.md)**

---

## 🚀 快速启动

需要 Python 3.9+。

### 1. 启动多用户模式（推荐）

```bash
# 克隆代码
git clone <your-repo-url>
cd qoder2api-hub

# 赋予执行权限并以多租户模式启动
chmod +x *.sh
QD_MULTIUSER_DB=./data/multiuser.db ./start-qoder-proxy.sh
```

Windows 用户可以直接执行：
```bat
set QD_MULTIUSER_DB=.\data\multiuser.db
python qoder_proxy.py --port 8790
```

启动后：
- **Web 控制台**：`http://127.0.0.1:8790/`
- **OpenAI 兼容接口**：`http://127.0.0.1:8790/v1`
- **首个注册账号**：系统会自动将其提升为超级管理员。

---

## 🛠️ 环境变量配置

| 环境变量 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `QD_MULTIUSER_DB` | `./data/multiuser.db` | **多租户开关**。指向 SQLite 文件即可开启多用户隔离模式 |
| `QD_MAX_CONCURRENCY_PER_KEY` | `3` | 单个 API Key 最大允许同时在飞的并发连接数（`0` 为不限） |
| `QD_MAX_RPM_PER_KEY` | `30` | 单个 API Key 每分钟最大请求次数（`0` 为不限） |
| `QD_PUBLIC_POOL_DAILY_LIMIT` | `100.0` | 普通用户每日从公共池消耗的 Credits 上限（`-1` 为不限） |
| `PORT` | `8790` | 网关监听端口 |
| `HOST` | `127.0.0.1` | 监听地址（对外开放需设为 `0.0.0.0`） |

---

## 🔌 客户端接入指南

| 客户端 / 工具 | Base URL | 协议类型 | 推荐模型 |
| :--- | :--- | :--- | :--- |
| **Cherry Studio / Chatbox** | `http://<IP>:8790/v1` | OpenAI Chat Completions | `Qwen3.8-Flash` / `Qwen3.8-Max` |
| **NextChat / 沉浸式翻译** | `http://<IP>:8790/v1` | OpenAI Chat Completions | `Qwen3.8-Flash` |
| **Cursor / VSCode** | `http://<IP>:8790/v1` | OpenAI Chat Completions | `Qwen3.8-Max` |
| **Claude Code** | `http://<IP>:8790` *(不带/v1)* | Anthropic Messages | 官方同源模型 |

### cURL 快速验证：
```bash
curl http://127.0.0.1:8790/v1/chat/completions \
  -H "Authorization: Bearer qd-你的APIKey" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "Qwen3.8-Flash",
    "messages": [{"role": "user", "content": "你好！"}],
    "stream": true
  }'
```

---

## 📌 二改与免责声明

1. **开源致谢**：本项目底层 Qoder 通信协议与逆向逆推逻辑基于开源社区作品 `qoder2api-hub`，向原项目的技术探索致以敬意。
2. **二次开发范围**：多用户体系、API Key 模型白名单隔离、私有+公共双池路由、集中式防漏签系统及请求并发/RPM 流控防护均由二次开发者独立设计并重写。
3. **免责声明**：本项目仅供计算机网络协议逆向、多租户架构设计及个人自动化运维的学习与技术交流使用，不提供、不销售任何官方账号或商用额度。使用者请自觉遵守 Qoder 服务协议。

---

[MIT License](LICENSE)
