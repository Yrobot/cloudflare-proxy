# Cloudflare Proxy

> 基于 Cloudflare Workers 的全功能 HTTP/HTTPS 代理服务

[![Deploy to Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://sink.proddig.com/cloudflare-proxy-github)

## 特性

- 🌐 **多种访问方式** - Web 界面、查询参数、路径方式、标准 HTTP 代理
- 🔒 **HTTPS 支持** - 完整支持 HTTPS 网站代理
- 🔄 **智能重定向** - 自动处理 301/302 等重定向
- 🛠️ **路径修复** - 自动修复 HTML 中的相对路径
- 🌍 **CORS 支持** - 完整的跨域资源共享支持
- 📱 **响应式设计** - 完美适配移动端和桌面端
- ⚡ **零成本运行** - Cloudflare Workers 免费版每天 10 万次请求

## 页面展示

![screenshot](./screenshot.png)

## 安装方式

### 方式一：一键部署（推荐）

点击上方 "Deploy to Cloudflare Workers" 按钮，按照提示完成部署。

### 方式二：使用 Wrangler CLI

```bash
# 1. 安装 Wrangler
npm install -g wrangler

# 2. 登录 Cloudflare
wrangler login

# 3. 克隆仓库
git clone https://github.com/Yrobot/cloudflare-proxy.git
cd cloudflare-proxy

# 4. 部署
wrangler deploy
```

### 方式三：使用 Cloudflare Dashboard

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 进入 **Workers & Pages**
3. 点击 **Create Application** > **Create Worker**
4. 将 `worker.js` 的内容复制粘贴到编辑器
5. 点击 **Save and Deploy**

### 获取访问地址

部署成功后，你会获得一个 Worker URL：

```
https://your-worker-name.your-subdomain.workers.dev
```

**注意** 这个自带的域名很多区域是无法直接访问的，建议绑定一个自己的域名使用

## 使用方式

### 方式 1: Web 界面

直接访问你的 Worker URL，在网页界面输入目标网址：

```
https://$YOUR-PROXY-DOMAIN/
```

### 方式 2: 查询参数

在 URL 后添加 `?url=` 参数：

```bash
https://$YOUR-PROXY-DOMAIN/?url=https://example.com
```

### 方式 3: 路径方式

直接在路径中指定目标网址：

```bash
# 完整 URL（带协议）
https://$YOUR-PROXY-DOMAIN/https://example.com

# 简写（自动添加 https://）
https://$YOUR-PROXY-DOMAIN/example.com
```

### 方式 4: HTTP 代理

设置为系统代理，适用于命令行工具：

```bash
# Linux/macOS
export HTTP_PROXY=https://$YOUR-PROXY-DOMAIN
export HTTPS_PROXY=https://$YOUR-PROXY-DOMAIN

# Windows (PowerShell)
$env:HTTP_PROXY="https://$YOUR-PROXY-DOMAIN"
$env:HTTPS_PROXY="https://$YOUR-PROXY-DOMAIN"

# 使用代理访问
curl https://api.github.com
```

## 使用场景

### 1. GitHub 文件加速

加速 raw.githubusercontent.com 文件下载：

```bash
# 原始地址（可能很慢）
https://raw.githubusercontent.com/user/repo/main/file.txt

# 使用代理（加速访问）
https://$YOUR-PROXY-DOMAIN/https://raw.githubusercontent.com/user/repo/main/file.txt
```

### 2. Docker 镜像加速

配置 Docker 镜像代理源：

```bash
# 在 /etc/docker/daemon.json 中配置
{
  "registry-mirrors": [
    "https://$YOUR-PROXY-DOMAIN/https://registry-1.docker.io"
  ]
}

# 重启 Docker
sudo systemctl restart docker
```

### 3. OpenAI API 代理

代理 OpenAI API 请求：

```javascript
// 设置代理基础 URL
const openai = new OpenAI({
  baseURL: "https://$YOUR-PROXY-DOMAIN/https://api.openai.com/v1",
  apiKey: "your-api-key",
});

// 或使用 fetch
fetch("https://$YOUR-PROXY-DOMAIN/https://api.openai.com/v1/chat/completions", {
  method: "POST",
  headers: {
    Authorization: "Bearer your-api-key",
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    model: "gpt-3.5-turbo",
    messages: [{ role: "user", content: "Hello!" }],
  }),
});
```

### 4. 前端 CORS 代理

解决前端跨域问题：

```javascript
// 直接访问会遇到 CORS 错误
fetch("https://api.example.com/data")
  .then((res) => res.json())
  .then((data) => console.log(data));

// 使用代理解决 CORS
fetch("https://$YOUR-PROXY-DOMAIN/https://api.example.com/data")
  .then((res) => res.json())
  .then((data) => console.log(data));
```

### 5. CI/CD 环境

在 GitHub Actions 中使用：

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    env:
      HTTP_PROXY: https://$YOUR-PROXY-DOMAIN
      HTTPS_PROXY: https://$YOUR-PROXY-DOMAIN
    steps:
      - name: Run tests
        run: npm test
```

## 注意与提醒

### 🚨 重要提示

1. **使用自定义域名**

   - Cloudflare 默认的 `*.workers.dev` 域名在某些地区可能无法访问
   - **强烈建议**绑定自己的域名以获得更好的访问体验
   - 在 Worker 设置中点击 **Triggers** > **Add Custom Domain** 添加自定义域名

2. **关于请求头**

   - 代理会转发所有请求头和内容

3. **访问限制**
   - Cloudflare Workers 免费版每天 10 万次请求
   - 单个请求不能超过 100MB
   - CPU 时间限制：免费版 10ms，付费版 50ms

### 🔒 安全配置

#### 添加 API Key 验证

```javascript
export default {
  async fetch(request) {
    // 检查 API Key
    const apiKey = request.headers.get("X-API-Key");
    if (apiKey !== "your-secret-key") {
      return new Response("Unauthorized", { status: 401 });
    }

    // 原有逻辑...
  },
};
```

## 免责声明

本项目仅供学习和研究使用，使用者需遵守以下规定：

1. **合法使用** - 仅用于访问合法内容，不得用于访问违法或侵权内容
2. **服务条款** - 使用时需遵守 Cloudflare Workers 服务条款
3. **责任自负** - 使用本代理产生的任何后果由使用者自行承担
4. **商业用途** - 如需商业使用，请确保符合相关法律法规
5. **隐私保护** - 建议不要通过代理传输个人敏感信息

## 常见问题

### Q: 为什么有些网站无法访问？

A: 可能原因：

- 网站有防爬虫机制，检测到了代理访问
- 网站使用了 WebSocket 等 Cloudflare Workers 不支持的协议
- 超过了请求大小或时间限制

### Q: 如何提高访问速度？

A: 建议：

- 使用自定义域名，选择离用户更近的 DNS 服务器
- 对于静态资源，考虑使用 Cloudflare 的缓存功能

### Q: 可以代理 WebSocket 吗？

A: 不可以。Cloudflare Workers 目前不支持 WebSocket 连接。

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

[GPL-3 License](LICENSE)

## 相关链接

- [Cloudflare Workers 文档](https://developers.cloudflare.com/workers/)
- [项目 GitHub](https://github.com/Yrobot/cloudflare-proxy)

---

**如果这个项目对你有帮助，请在 [GitHub](https://github.com/Yrobot/cloudflare-proxy) 上给我们一个 ⭐️**
