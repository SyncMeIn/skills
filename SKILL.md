---
name: syncmein-passwordless-login
version: 1.0.0
trigger_keywords: ["免密登录", "SyncMein", "注入登录态", "口令登录"]
description: 配合 SyncMein 插件，在常用浏览器生成的ck口令。AI Agent 通过本skill，获取 Cookie/Storage 并注入浏览器，实现免密登录（静默会话克隆，不会挤下原设备）。本插件仅限用户本人账号、本人设备使用。
---

# SyncMein 口令免密登录注入技能

## 描述
通过调用 SyncMein 接口，利用用户提供的免密口令（tokenCode）和目标域名（domain），获取包含 HttpOnly 标志在内的完整 Cookie 和 Storage 数据，并通过底层自动化 API（如 Playwright/Puppeteer）将其无缝注入到当前浏览器上下文中，从而绕过严格的登录检测，实现自动化免密登录。

> **⚠️ 安全提醒：本插件仅限用户本人账号、本人设备使用。** 任何未经授权的账号共享、口令传播或超出授权范围的访问，均可能违反目标平台服务协议并带来安全风险。

**核心优势**：实现真正的"静默会话克隆"（Silent Session Cloning）。全程直接复用底层会话状态，无需向目标服务器发起新的登录验证请求，完美规避具有单点登录（SSO）或单设备限制网站的安全策略，**彻底解决"当前环境登录会导致之前设备被挤下线"的互斥问题**，实现多环境无缝协同操作。

## 触发条件
当用户提供 SyncMein 口令，并要求对某个目标网站进行"免密登录"、"口令登录"或"注入会话数据"时触发使用。

## 参数
- `tokenCode` — SyncMein 免密口令（用户提供）
- `domain` — 目标网站主域名（如 `jd.com`、`nr.kainy.cn`）

## API Endpoint
```
GET https://kainy.cn/SyncMein/api/cookie/getWithToken?tokenCode={tokenCode}&domain={domain}&withStorage=1
```

返回 JSON 包含：
- `cookies[]` — Cookie 数组（含 httpOnly、secure 等标志）
- `localStorage` — key-value 对象
- `sessionStorage` — key-value 对象

## 执行流程

### 1. 解析参数
从用户指令提取 `tokenCode` 和 `domain`。

### 2. 获取会话数据
```
curl "https://kainy.cn/SyncMein/api/cookie/getWithToken?tokenCode={tokenCode}&domain={domain}&withStorage=1"
```

### 3. 浏览器环境初始化
通过 CDP 或 Puppeteer 控制浏览器导航至目标域名轻量页面（推荐 `https://{domain}/404` 或登录页），建立 Origin 环境，确保 Storage 能准确挂载到目标域名下。

**CDP 模式**（推荐，用于远程浏览器）：
```javascript
const wsUrl = await fetch('http://192.168.9.103:9222/json/version').then(r => r.json()).then(d => d.webSocketDebuggerUrl);
const browser = await puppeteer.connect({ browserWSEndpoint: wsUrl });
const page = await browser.newPage();
await page.goto(`https://${domain}/404`, { waitUntil: 'domcontentloaded', timeout: 15000 });
```

### 4. 注入 Cookie 和 Storage
4a. **清理 Cookie**：剔除非标准字段（如 `storeId`、`id`），只保留标准 Cookie 属性：
```javascript
const cleanCookies = cookies.map(c => {
  const { storeId, id, ...rest } = c;
  // 确保 domain 以 . 开头
  if (rest.domain && !rest.domain.startsWith('.')) rest.domain = '.' + rest.domain;
  return rest;
});
```

4b. **注入 Cookie**（通过 CDP 协议，支持 httpOnly）：
```javascript
const client = await page.target().createCDPSession();
for (const cookie of cleanCookies) {
  await client.send('Network.setCookie', cookie);
}
```

4c. **注入 Storage**：
```javascript
await page.evaluate((ls, ss) => {
  if (ls) Object.entries(ls).forEach(([k, v]) => localStorage.setItem(k, v));
  if (ss) Object.entries(ss).forEach(([k, v]) => sessionStorage.setItem(k, v));
}, localStorage, sessionStorage);
```

### 5. 刷新激活
```javascript
await page.goto(`https://${domain}`, { waitUntil: 'domcontentloaded', timeout: 15000 });
```

### 6. 验证与报告
检查页面是否显示已登录状态，向用户反馈注入结果。

## 安全策略与撤销机制

- **不落盘**：Cookie 和 Storage 数据仅在内存中临时持有，禁止写入任何文件、缓存或持久化存储。
- **不打印**：日志、调试输出、错误堆栈中禁止包含明文 Cookie、Token 或 Storage 内容；如需记录，只能使用脱敏摘要（如 `cookie_key_hash`）。
- **不进入 Agent 对话上下文**：解析出的 Cookie/Storage 字段不得通过 `think`、工具结果、文本回复等任何途径传入大模型上下文。
- **注入后即焚**：完成浏览器注入后，立即从内存中清除原始会话数据；仅保留非敏感的任务结果（如“登录成功”）反馈给用户。
- **用户撤销权**：用户可随时要求撤销已授权口令或清理当前浏览器环境；收到请求后必须立即清除对应 Cookie 并断开相关会话。

## Pitfalls

- **Cookie domain 格式**：部分接口返回的 domain 不带前导 `.`，需要补上否则 `Network.setCookie` 会失败。
- **storeId/id 字段**：接口返回的 Cookie 可能包含 `storeId`、`id` 等非标准字段，必须剔除后才能注入。
- **先导航再注入**：必须先让浏览器页面到达目标域名（即使是 404 页），再注入 Storage，否则 localStorage/sessionStorage 会挂到错误的 origin 下。
- **httpOnly Cookie**：普通的 `page.setCookie()` 无法设置 httpOnly cookie，必须用 CDP 协议的 `Network.setCookie`。
- **页面关闭检查**：关闭注入用的 tab 前，务必检查 `browser.pages()` 数量，必须保留至少 1 个 tab，否则 Chrome 会退出导致 CDP 断开。
- **多账号场景**：同一 domain 不同账号的注入会互相覆盖，如需多账号并行，需要使用不同的 CDP 浏览器实例或 Profile。
- **口令失效处理**：调用接口前需预判断口状态（过期、超设备数、已被撤销），对失效口令给出明确提示，避免反复重试。

## 示例

**用户**: 帮我用 SyncMein 口令 C1ITBS9O7M 登录一下 jd.com。

**AI**: 正在为您执行 jd.com 的免密登录操作...
1. 调用 SyncMein 接口获取会话数据 ✅
2. 浏览器导航至 jd.com 建立环境 ✅
3. 注入 Cookie（含 httpOnly）和 Storage ✅
4. 刷新页面激活登录态 ✅

✅ 登录态已成功激活！由于采用静默状态复用，原设备登录状态不受影响（不会被挤下线）。
