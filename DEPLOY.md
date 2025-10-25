# 部署到 GitHub Pages 详细教程

## 📋 前置准备

1. 确保已安装 Git
2. 拥有 GitHub 账号
3. 项目代码已准备就绪

## 🚀 部署步骤

### 第一步：初始化 Git 仓库

在项目根目录打开终端，执行以下命令：

```bash
# 初始化 Git 仓库
git init

# 添加所有文件到暂存区
git add .

# 提交代码
git commit -m "Initial commit: 个人工具导航网站"
```

### 第二步：在 GitHub 创建仓库

1. 访问 GitHub：https://github.com
2. 点击右上角的 "+" 号，选择 "New repository"
3. 填写仓库信息：
   - **Repository name**: `navWeb` (或其他你喜欢的名字)
   - **Description**: 个人工具导航网站
   - **Public/Private**: 选择 Public（公开仓库才能使用免费的 GitHub Pages）
   - ⚠️ **不要勾选** "Add a README file"（因为我们已经有代码了）
4. 点击 "Create repository" 创建仓库

### 第三步：关联远程仓库并推送代码

创建仓库后，GitHub 会显示一些命令。在项目根目录执行：

```bash
# 关联远程仓库（替换 YOUR_USERNAME 为你的 GitHub 用户名）
git remote add origin https://github.com/YOUR_USERNAME/navWeb.git

# 重命名分支为 main（如果当前是 master）
git branch -M main

# 推送代码到 GitHub
git push -u origin main
```

### 第四步：配置 GitHub Pages

1. 在 GitHub 仓库页面，点击顶部的 **"Settings"** (设置)
2. 在左侧菜单找到 **"Pages"** 选项
3. 在 "Build and deployment" 部分：
   - **Source**: 选择 "GitHub Actions"
   - （不要选择 "Deploy from a branch"，因为我们使用自动化部署）
4. 保存设置

### 第五步：触发自动部署

由于我们已经配置了 GitHub Actions 工作流（`.github/workflows/deploy.yml`），
当你推送代码到 `main` 分支时，会自动触发部署。

查看部署状态：
1. 在仓库页面点击顶部的 **"Actions"** 标签
2. 可以看到正在运行的工作流
3. 等待部署完成（通常需要 2-5 分钟）
4. 部署成功后会显示绿色的 ✓ 标记

### 第六步：访问你的网站

部署成功后，你的网站地址为：

```
https://YOUR_USERNAME.github.io/navWeb/
```

（将 YOUR_USERNAME 替换为你的 GitHub 用户名）

你也可以在 Settings > Pages 页面找到网站地址。

## 🔄 后续更新

当你修改代码后，只需要执行以下命令即可自动重新部署：

```bash
# 添加修改的文件
git add .

# 提交修改
git commit -m "描述你的修改内容"

# 推送到 GitHub
git push
```

推送后，GitHub Actions 会自动构建并部署新版本。

## ⚠️ 常见问题

### 问题 1：推送代码时要求输入用户名和密码

**解决方案**：
- GitHub 已不再支持密码认证，需要使用个人访问令牌（Personal Access Token）
- 生成令牌：GitHub 设置 > Developer settings > Personal access tokens > Tokens (classic) > Generate new token
- 选择权限：至少勾选 `repo` 权限
- 复制生成的令牌，在推送时用令牌代替密码

### 问题 2：网站显示 404

**解决方案**：
- 检查 GitHub Pages 是否已启用
- 确认部署工作流是否成功运行
- 等待几分钟，GitHub Pages 需要时间生效
- 检查仓库是否为 Public

### 问题 3：网站样式或资源加载失败

**解决方案**：
- 检查 `vite.config.js` 中的 `base` 配置
- 如果仓库名不是 `navWeb`，需要修改 base 为 `'/你的仓库名/'`
- 或者使用自定义域名时，base 设置为 `'/'`

### 问题 4：数据文件加载失败

**解决方案**：
- 确保 `public/data/sites.json` 文件存在
- 检查浏览器控制台的错误信息
- 确认文件路径正确（应该是 `/data/sites.json`）

## 🎨 使用自定义域名（可选）

如果你有自己的域名，可以配置自定义域名：

1. 在 GitHub 仓库的 Settings > Pages 中找到 "Custom domain"
2. 输入你的域名（例如：nav.yourdomain.com）
3. 在你的域名 DNS 设置中添加 CNAME 记录：
   - 类型：CNAME
   - 名称：nav（或其他子域名）
   - 值：YOUR_USERNAME.github.io
4. 等待 DNS 生效（可能需要几分钟到几小时）
5. 勾选 "Enforce HTTPS" 启用 HTTPS

## 📝 配置文件说明

项目中的 `.github/workflows/deploy.yml` 文件定义了自动部署流程：

- **触发条件**：推送到 main 分支时自动运行
- **构建步骤**：
  1. 检出代码
  2. 安装 Node.js
  3. 安装依赖（npm install）
  4. 构建项目（npm run build）
  5. 上传构建产物
  6. 部署到 GitHub Pages

## 🎉 完成！

现在你的个人工具导航网站已经成功部署到 GitHub Pages 了！

你可以：
- 分享网站链接给朋友
- 在任何设备上访问你的导航网站
- 随时更新内容，自动重新部署

---

如有问题，欢迎查看 GitHub Actions 的运行日志获取详细错误信息。

