<div align="center">
  <h1>🚀 SyncMein Skill: 跨端静默免密登录</h1>
  <p>
    <strong>终结单点登录（SSO）互斥，为 AI Agent 实现真正的会话克隆与纯态复用</strong>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Version-1.0.0-blue.svg" alt="Version" />
    <img src="https://img.shields.io/badge/Support-Playwright%20%7C%20Puppeteer-green.svg" alt="Support" />
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License" />
  </p>
</div>

<br/>

## CK免密登录上号 | 更适合AI Agent的通用登录方式 | syncmein-passwordless-login

## ✨ 为什么你需要 SyncMein Skill？

在自动化和 AI Agent 开发中，我们常常面临一个致命痛点：**登录太难，维持登录更难**。复杂的验证码、严格的设备环境检测，甚至“新登录把旧设备挤下线”的单设备限制，让自动化流程频频中断。

SyncMein Skill 彻底改变了这一现状。它不走寻常路，提供了一种更优雅的解法：

* 🎭 **真正的静默会话克隆**：全程不触发目标服务器的任何新登录验证（无验证码、无扫码），直接在底层复用已授权的会话状态。
* 🛡️ **完美规避登录互斥**：突破单设备/SSO限制，你的主账号（手机/电脑）和自动化 Agent 可以**同时在线，互不干扰，不被挤下线**。
* 🍪 **全量底层状态注入**：不仅是常规 Cookie，完美支持 `HttpOnly: true` 跨域安全写入，以及 LocalStorage / SessionStorage 的双重精准挂载，100% 还原真实浏览器上下文。
* 🤖 **无缝接轨 LLM Agent**：专为大模型工具链（Tool Calling）设计，只需提供一句口令，Agent 即可自动完成环境伪装与接管。

---

## 🎮 快速展示

*(💡 建议：在这里放一张 Agent 自动注入登录态并刷新页面的 GIF 动图，视觉冲击力最强！)*

**一句指令，即可唤醒免密登录：**
> "帮我用 SyncMein 口令 `C1ITBS9O7M` 登录一下 jd.com，注意别把我大号挤下线了。"

---

## 🚀 极简上手 (Zero to Hero)

只需两步，让你的 Agent 具备免密穿梭能力。

### 1. 准备工作
确保你的执行环境支持底层自动化框架（以 Playwright 为例）：
```bash
pip install playwright requests
playwright install

```

### 2. 注入 Skill

将核心技能代码挂载到你的 Agent 工作流中。我们采用了“先导航建立 Origin -> 底层注入 Cookie/Storage -> 刷新激活”的标准自动化链路，完美绕过风控。

```python
# 核心调用示例
from syncmein_skill import syncmein_passwordless_login

# 传入口令与目标站点，Agent 自动接管
result = syncmein_passwordless_login(
    token_code="YOUR_TOKEN_CODE",
    target_url="[https://www.jd.com/404](https://www.jd.com/404)", 
    context=browser_context,
    page=current_page
)
print(result) # ✅ 免密登录注入成功！

```

*(详细代码实现请查阅 `src/` 目录)*

---

## 🏗️ 工作原理

传统的免密方案大多依赖纯前端 JS (`document.cookie`) 注入，这会导致包含关键授权信息的 `HttpOnly` Cookie 丢失，复用失败。

SyncMein Skill 利用底层的 `BrowserContext` 级别的控制权：

1. 拦截并拉取云端同步的完整 Session 映射表。
2. 在浏览器沙盒外围，由宿主进程强制写入带有 `HttpOnly: true` 标记的密钥。
3. 执行页面级 JS 挂载 `Storage`。
最终达成 **不触发任何鉴权接口请求** 的纯态复用。

---

## 🤝 贡献与支持

欢迎提交 Pull Request 或 Issue。如果你觉得这个项目拯救了你的 Agent，请给它一个 ⭐️ **Star**！

你目前的这个项目是打算作为开源工具库发布在 GitHub 上，还是作为你们内部 Agent 平台的说明文档？

```
