# GitHub Pages 个人网站部署指南

> 基于实际部署经验整理，项目：辣卡的抽屉（1807318.xyz）
> 整理日期：2026-05-29

---

## 一、前期准备

| 项目 | 说明 |
|------|------|
| GitHub 账号 | 用于托管代码和启用 Pages 服务 |
| 域名 | 在任意注册商购买（本例为 1807318.xyz，通过 Cloudflare 管理 DNS） |
| 代理工具 | 国内访问 GitHub 必备（Clash/v2ray 等），后续 git push 依赖代理 |

---

## 二、项目文件结构

```
C:\code\blog\
├── index.html        ← 主页面（纯 HTML）
├── style.css         ← 样式文件
├── script.js         ← 交互逻辑
├── CNAME             ← 自定义域名声明（内容就一行域名）
├── .gitignore        ← Git 忽略规则
└── assets/           ← 图片等静态资源（按需添加）
```

### CNAME 文件

无后缀纯文本文件，内容只有一行你的域名：

```
www.1807318.xyz
```

> 这个文件**必须存在**。没有它，每次 git push 后 GitHub 会重置自定义域名设置。

### .gitignore 文件

```gitignore
.workbuddy/
.DS_Store
Thumbs.db
desktop.ini
.vscode/
.idea/
*.swp
*.swo
*~
```

---

## 三、创建 GitHub 仓库

1. 登录 [github.com](https://github.com)
2. 点右上角 **+** → **New repository**
3. Repository name **必须**填 `你的用户名.github.io`（如 `laka.github.io`）
4. 选 **Public**（私有仓库需 Pro 才能用 Pages）
5. **不要**勾选任何初始化选项（README、.gitignore、License）
6. 点 **Create repository**

---

## 四、本地初始化并推送代码

### 4.1 配置 Git 身份

首次在这台电脑使用 Git，需要先设置用户名和邮箱：

```bash
git config --global user.name "你的用户名"
git config --global user.email "你的邮箱"
```

> 建议邮箱用 GitHub 提供的 noreply 地址：`用户名@users.noreply.github.com`，保护隐私。

### 4.2 配置代理（国内必须）

```bash
# 先确认你的代理端口（Clash 默认 7890，但可能不同）
# 查看方法：打开 Clash 面板，看 "HTTP 端口"

git config --global http.proxy http://127.0.0.1:你的代理端口
git config --global https.proxy http://127.0.0.1:你的代理端口
```

> 本例中代理端口为 7897（非默认值），需在 Clash 面板中确认实际端口。
>
> 可用以下命令快速查看本机代理端口：
> ```powershell
> Get-NetTCPConnection -State Listen | Where-Object { $_.LocalAddress -eq '127.0.0.1' -and $_.LocalPort -ge 7890 -and $_.LocalPort -le 7900 }
> ```

### 4.3 初始化并推送

```bash
cd C:\code\blog

# 初始化 Git 仓库
git init

# 添加所有文件
git add .

# 首次提交（必须先 commit 才能 push）
git commit -m "init: 个人网站初始版本"

# 设置主分支名（GitHub 默认要求 main）
git branch -M main

# 关联远程仓库
git remote add origin https://github.com/你的用户名/你的用户名.github.io.git

# 推送
git push -u origin main
```

### 4.4 常见报错及解决

| 报错 | 原因 | 解决 |
|------|------|------|
| `src refspec main does not match any` | 没有 commit 就 push | 先 `git add .` 再 `git commit -m "xxx"` |
| `Author identity unknown` | 未设置 Git 用户名/邮箱 | 执行 `git config user.name/email` |
| `Failed to connect to github.com port 443` | 网络不通 | 配置代理，确认端口号正确 |
| `via 127.0.0.1 after xxx ms: Could not connect` | 代理端口不对 | 确认实际代理端口并更新配置 |

---

## 五、开启 GitHub Pages

1. 打开仓库页面 → **Settings**（仓库的，不是头像的）
2. 左侧菜单 → **Pages**
3. Build and deployment：
   - **Source**：`Deploy from a branch`
   - **Branch**：`main`，目录 `/ (root)`
4. 点 **Save**

保存后等 1-2 分钟，页面顶部会出现：

> Your site is live at `https://你的用户名.github.io/`

此时已可通过 `https://你的用户名.github.io` 访问网站。

---

## 六、配置自定义域名

### 6.1 GitHub 侧设置

在 **Settings → Pages** 页面：

1. 找到 **Custom domain** 输入框
2. 填入域名（如 `www.1807318.xyz`）
3. 点 **Save**
4. 等待 DNS 检查通过（显示绿色勾 "DNS check successful"）

### 6.2 DNS 解析配置

去你的域名 DNS 管理后台添加记录：

**绑定 www 子域名（推荐）：**

| 记录类型 | 主机记录 | 记录值 | TTL |
|---------|---------|--------|-----|
| CNAME | www | 你的用户名.github.io | 600 |
| A | @ | 185.199.108.153 | 600 |
| A | @ | 185.199.109.153 | 600 |
| A | @ | 185.199.110.153 | 600 |
| A | @ | 185.199.111.153 | 600 |

**只绑定裸域名（不带 www）：**

| 记录类型 | 主机记录 | 记录值 | TTL |
|---------|---------|--------|-----|
| A | @ | 185.199.108.153 | 600 |
| A | @ | 185.199.109.153 | 600 |
| A | @ | 185.199.110.153 | 600 |
| A | @ | 185.199.111.153 | 600 |

### 6.3 Cloudflare 注意事项

如果域名 DNS 托管在 Cloudflare，**必须关闭代理**（橙色云朵 → 灰色云朵）：

1. 登录 Cloudflare Dashboard → 选择域名
2. 进入 **DNS** 标签页
3. 对每条 CNAME 和 A 记录，点击橙色云朵图标使其变灰
4. 状态应显示为 **"DNS only"**，不是 "Proxied"

> **原因**：开启 Cloudflare 代理后，域名解析到 Cloudflare 的 IP 而非 GitHub Pages 的 IP。GitHub 无法验证域名所有权，无法签发 Let's Encrypt SSL 证书，Enforce HTTPS 将无法开启。

验证 DNS 是否正确解析：

```bash
nslookup www.1807318.xyz
# 应返回 185.199.108~111.153 中的 IP
# 如果返回 104.21.x.x 或 172.67.x.x → Cloudflare 代理未关
```

---

## 七、开启 HTTPS

DNS 生效后，GitHub 会自动向 Let's Encrypt 申请 SSL 证书：

1. 回到 **Settings → Pages**
2. 等待 **Enforce HTTPS** 复选框变为可勾选状态
3. 勾选 **Enforce HTTPS**

> **如果 Enforce HTTPS 仍然灰色无法勾选**：
> - DNS 刚改完需要等待传播（5~30 分钟）
> - 证书签发也需要时间（几分钟到数小时）
> - 可以尝试清空 Custom domain → Save → 重新填入 → Save，触发重新检查
> - 用 `nslookup` 确认域名已解析到 GitHub IP，而非 Cloudflare IP

---

## 八、HTML 中的域名替换

`index.html` 中所有 `yourdomain.com` 占位符需替换为真实域名：

```html
<!-- 需要替换的位置 -->
<link rel="canonical" href="https://www.1807318.xyz/">
<meta property="og:url" content="https://www.1807318.xyz/">
<meta property="og:image" content="https://www.1807318.xyz/assets/og.png">
```

JSON-LD 结构化数据中的 `url` 和 `sameAs` 也要替换为真实链接。

替换后推送：

```bash
git add .
git commit -m "fix: 替换域名为真实值"
git push
```

---

## 九、日常更新流程

每次修改网站内容后：

```bash
cd C:\code\blog
git add .
git commit -m "更新说明"
git push
```

推送后 **30 秒内** 自动上线，无需任何构建步骤。

---

## 十、完整时间线参考

以本次部署为例，记录各步骤实际耗时：

| 步骤 | 耗时 | 备注 |
|------|------|------|
| 创建仓库 | 1 分钟 | — |
| 初始化 Git + 推送代码 | 5 分钟 | 含排查 commit 报错 |
| 开启 GitHub Pages | 2 分钟 | 等 1-2 分钟生效 |
| DNS 配置 | 5 分钟 | 在 Cloudflare 添加记录 |
| DNS 传播 | 5~30 分钟 | 取决于 TTL 和注册商 |
| 关闭 Cloudflare 代理 | 1 分钟 | 橙色云朵 → 灰色 |
| HTTPS 证书签发 | 数分钟~数小时 | 首次可能较慢 |
| **总计** | **约 1~2 小时** | 大部分时间在等 |

---

## 十一、问题排查速查表

| 现象 | 检查方法 | 解决方案 |
|------|---------|---------|
| `username.github.io` 显示 404 | Settings → Pages 确认分支和目录 | 确保选 main + /root |
| 自定义域名显示 "improperly configured" | `nslookup` 检查域名解析 | DNS 记录未添加或不正确 |
| DNS check successful 但 Enforce HTTPS 灰色 | 检查是否有 Cloudflare 代理 | 关闭橙色云朵，等 DNS 传播 |
| git push 连接超时 | 检查代理是否开启及端口 | `git config --global http.proxy` |
| 推送后自定义域名丢失 | 检查仓库根目录是否有 CNAME | 确保 CNAME 文件存在且未被 .gitignore |
| 国内访问慢 | — | 配 Cloudflare CDN（需开启代理+Full SSL） |

---

## 附录：关键 GitHub Pages IP

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

> 这些 IP 由 GitHub 官方提供，如变更请查阅 [https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
