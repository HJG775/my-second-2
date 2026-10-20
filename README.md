# 新网页上线模板（GitHub Pages）

复制这个文件夹，改掉里面的 `index.html`，就是你的新网站。
外形、内容随便改，只有两样东西别删：`.github/workflows/deploy.yml` 和 `.gitignore`。

---

## 一、通用五步（每次新建网站都一样的套路）

```bash
# 1. 本地准备（复制本模板后进入文件夹）
cd 你的新项目文件夹
git init
git add .
git commit -m "初始化"

# 2. 网页上先建空仓库：浏览器打开 github.com/new
#    名称自定义，选 Public，三个勾选框都不勾

# 3. 连上远端并推送（仓库名换成你的）
git remote add origin git@github.com:你的用户名/你的仓库名.git
git branch -M main
git push -u origin main

# 4. 网页上开启部署（只做一次，以后不用再管）
#    仓库 → 设置 → 页面 → Source 选 GitHub Actions

# 5. 等待 Actions 变绿，访问
#    https://你的用户名.github.io/你的仓库名/
```

之后每次更新内容，只要：

```bash
git add . && git commit -m "改了什么" && git push
```

---

## 二、三种项目类型的区别

### 1）纯 HTML / CSS / JS（今天这种，最简单）

仓库里直接放源码，`path: ./` 原样上传。本模板就是这一种，不用改任何东西。

### 2）Vite / React / Vue 等需要构建的前端框架

源码不能直接部署，必须先 `npm run build` 生成 `dist/`，再把 `dist/` 上传。
把 `deploy.yml` 的前半段改成：

```yaml
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - run: npm ci
      - run: npm run build

      - uses: actions/configure-pages@v5

      - uses: actions/upload-pages-artifact@v3
        with:
          path: dist          # ← 关键：改成构建产物目录
```

**还要注意项目路径**：网站会挂在 `用户名.github.io/仓库名/` 下面，不是根目录。
Vite 项目必须在 `vite.config.js` 里加：

```js
export default { base: '/仓库名/' }
```

不加的话，页面能打开但样式和图片全丢（404）——这是新手最常踩的坑。

### 3）Next.js 等 SSR 框架

GitHub Pages 只能托管静态文件，必须先用 `output: 'export'` 导出静态站点，
或者改用 Vercel / Cloudflare Pages。Pages 不适合这类项目。

---

## 三、常见坑对照表

| 现象 | 原因 | 解法 |
|---|---|---|
| Actions 报 `Please ensure the repository is enabled` | 没开 Pages 就推送了 | 设置 → 页面 → Source 选 GitHub Actions，再重跑 |
| 页面能打开但没样式、图片 404 | 资源路径以 `/` 开头，但站点在子路径下 | 用相对路径，或设置框架的 `base` |
| 显示的是 README 而不是网页 | 没有 `index.html`，或它在子目录里 | 把 `index.html` 放到仓库根目录 |
| 改了代码但线上没变 | 忘了 push，或 Actions 还在跑 | 先 `git status` 确认已提交，再看 Actions 是否变绿 |
| `Permission denied (publickey)` | SSH 钥匙没配好 | 重新添加公钥到 GitHub，`ssh -T git@github.com` 验证 |
| 私有仓库开了 Pages 仍 404 | 免费账号不支持私有仓库 Pages | 把仓库改成 Public |

---

## 四、网址规则

- 项目站：`https://用户名.github.io/仓库名/`
- 用户主页（仓库名必须是 `用户名.github.io`）：`https://用户名.github.io/`

想换自定义域名，在「设置 → 页面 → 自定义域」里填，然后按提示配 DNS。
