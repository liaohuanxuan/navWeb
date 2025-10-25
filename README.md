# 个人工具导航网站

一个简洁、美观、易用的个人工具导航网站，帮助你高效整理和管理常用的在线工具和资源。

## ✨ 特性

- 🎨 **现代化设计** - 简洁美观的界面，流畅的交互体验
- 🔍 **智能搜索** - 支持按标题、描述、标签进行模糊搜索
- 📂 **分类管理** - 清晰的分类体系，快速定位所需工具
- 📱 **响应式布局** - 完美适配桌面端和移动端
- ⚡ **快速加载** - 纯静态网站，加载速度极快
- 🚀 **易于部署** - 支持 GitHub Pages 一键部署

## 🛠️ 技术栈

- **前端框架**: Vue 3 (Composition API)
- **构建工具**: Vite
- **样式**: 原生 CSS (CSS Variables)
- **图标**: Font Awesome
- **数据存储**: JSON 文件
- **部署**: GitHub Pages

## 📦 快速开始

### 安装依赖

```bash
npm install
```

### 本地开发

```bash
npm run dev
```

访问 `http://localhost:5173` 查看网站

### 构建生产版本

```bash
npm run build
```

构建后的文件将输出到 `dist` 目录

### 预览生产版本

```bash
npm run preview
```

## 📝 自定义内容

### 修改网站数据

编辑 `public/data/sites.json` 文件来添加、修改或删除网站链接。

数据结构示例：

```json
{
  "categories": [
    {
      "id": "dev-tools",
      "name": "开发工具",
      "icon": "fas fa-code",
      "sites": [
        {
          "id": "github",
          "name": "GitHub",
          "description": "全球最大的代码托管平台",
          "url": "https://github.com",
          "icon": "fab fa-github",
          "tags": ["代码托管", "开源", "协作"]
        }
      ]
    }
  ]
}
```

### 修改样式

编辑 `src/style.css` 文件，可以通过修改 CSS 变量来快速调整主题：

```css
:root {
  --primary-color: #4f46e5;
  --secondary-color: #10b981;
  --background: #f8fafc;
  /* ... 更多变量 */
}
```

## 🚀 部署到 GitHub Pages

1. 在 GitHub 上创建一个新仓库
2. 将代码推送到仓库的 `main` 分支
3. 在仓库设置中启用 GitHub Pages
4. 选择 GitHub Actions 作为部署源
5. 推送代码后会自动触发部署

## 📂 项目结构

```
navWeb/
├── .github/
│   └── workflows/
│       └── deploy.yml        # GitHub Actions 部署配置
├── public/
│   └── data/
│       └── sites.json        # 网站数据
├── src/
│   ├── App.vue              # 主应用组件
│   ├── main.js              # 应用入口
│   └── style.css            # 全局样式
├── doc/
│   └── 需求.md              # 需求文档
├── index.html               # HTML 入口
├── package.json             # 项目配置
├── vite.config.js           # Vite 配置
└── README.md                # 项目说明
```

## 🎯 功能说明

### 搜索功能
- 在顶部搜索框输入关键词
- 支持搜索网站名称、描述和标签
- 实时过滤显示匹配结果

### 分类筛选
- 点击分类按钮快速筛选
- 点击"全部"显示所有内容
- 选中的分类会高亮显示

### 网站卡片
- 显示网站图标、名称和描述
- 显示相关标签
- 点击卡片在新标签页打开网站
- 悬停时有动画效果

## 📄 License

MIT License

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

---

Made with ❤️ by [Your Name]

