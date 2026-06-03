<div align="center">
  <h1>🚀 SyncMein Skill: 跨端静默免密登录（安全受限版）</h1>
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


> **⚠️ 重要安全声明：本插件仅限用户本人账号、本人设备使用。** 所有功能设计均以"用户自有账号多设备同步"和"受信任人员临时访问"为边界，严禁用于未经授权的账号共享、批量登录或任何违反目标平台服务协议的行为。


## ✨ 为什么你需要 SyncMein Skill？

在自动化和 AI Agent 开发中，我们常常面临一个致命痛点：**登录太难，维持登录更难**。复杂的验证码、严格的设备环境检测，甚至“新登录把旧设备挤下线”的单设备限制，让自动化流程频频中断。

SyncMein Skill 彻底改变了这一现状。它不走寻常路，提供了一种更优雅的解法：

* 🎭 **真正的静默会话克隆**：全程不触发目标服务器的任何新登录验证（无验证码、无扫码），直接在底层复用已授权的会话状态。
* 🛡️ **完美规避登录互斥**：突破单设备/SSO限制，你的主账号（手机/电脑）和自动化 Agent 可以**同时在线，互不干扰，不被挤下线**。
* 🍪 **全量底层状态注入**：不仅是常规 Cookie，完美支持 `HttpOnly: true` 跨域安全写入，以及 LocalStorage / SessionStorage 的双重精准挂载，100% 还原真实浏览器上下文。
* 🤖 **无缝接轨 LLM Agent**：专为大模型工具链（Tool Calling）设计，只需提供一句口令，Agent 即可自动完成环境伪装与接管。
* 🔒 **安全受限机制**：支持口令有效期、设备数量限制和随时撤销，最大程度降低口令泄露后的风险敞口。

### 授权使用场景

1. **个人多设备同步**：在自己的多台设备（如个人电脑、工作机、远程服务器）间同步同一账号的登录状态，避免频繁重新登录或被互踢。
2. **受信任人员临时访问**：将账号临时分享给受信任的人（如家人、密友）进行短期、特定目的的访问，且已明确告知对方使用范围和期限。

**禁止行为**：严禁将口令公开发布、出售、用于批量爬虫或任何超出个人/受信任小范围使用的场景。

---

## 🎮 快速展示

这里以本人开发的 [API 计费管理平台](https://open.kainy.cn/#readme) 测试账号为例：

**一句指令，即可唤醒免密登录：**
> "帮我用 SyncMein 口令 `R5M1193G0C` 登录一下 open.kainy.cn，注意别把我大号挤下线了。"

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

### 口令安全策略

![](https://s.gqmg.com/https://oray.kainy.cn:38400/_upload/./content/temp/2026/06/mpxedgz9.webp)

| 策略项　　　　　　　　　　　　　　 | 说明　　　　　　　　　　　　　　　　　　　　　 | 配置方式　　　　　　　　　　　　　 |
| ------------------------------------| ------------------------------------------------| ------------------------------------|
| **口令有效期**　　　　　　　　　　 | 口令到达预设时间后自动失效，接口拒绝返回数据　 | 生成口令时设置（如 1h / 24h / 7d） |
| **设备数量限制（即访问次数限制）** | 限制最多允许多少台设备通过该口令完成注入　　　 | 生成口令时设置（如 1 / 3 / 5）　　 |
| **撤销机制**　　　　　　　　　　　 | 用户可随时在口令管理页面删除口令，立即切断访问 | 插件口令管理页面手动删除　　　　　 |
| **过期自动清理**　　　　　　　　　 | 系统后台自动扫描并清理已过期口令，无需手动维护 | 系统自动执行　　　　　　　　　　　 |

在插件生成口令时，可灵活组合上述策略，例如生成一个 **"有效期 2 小时、仅限 1 台设备（一次性使用）"** 的高限制口令，用于临时受信任访问；或生成一个 **"有效期 7 天、限 3 台设备"** 的口令，用于个人多设备长期同步。

---

## 🔐 口令管理与撤销

![](https://s.gqmg.com/https://oray.kainy.cn:38400/_upload/./content/temp/2026/06/mpxeg0ju.webp)

在插件的口令管理页面，您可以：

- **查看所有有效口令**：包括创建时间、剩余有效期、已用设备数 / 上限、是否一次性等信息。
- **删除口令（撤销访问授权）**：点击删除后，该口令立即失效，所有基于该口令的登录态访问将被阻断。
- **批量清理**：一键清理所有已过期或不再需要的口令。

> 💡 **建议**：完成临时访问或设备同步后，及时删除不再需要的口令，缩小潜在攻击面。

---

## 🤝 贡献与支持

欢迎提交 Pull Request 或 Issue。如果你觉得这个项目拯救了你的 Agent，请给它一个 ⭐️ **Star**！
客服微信： SyncMein 💁
