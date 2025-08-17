# PureWiki

PureWiki 是一个基于 Vue 3 和 Vite 构建的现代化静态 Wiki 系统.

## 功能特性

### 📖 内容管理
- **Markdown 渲染**：支持完整的 Markdown 语法，包括 GFM 扩展
- **自动目录生成**：基于标题自动生成页面导航目录 (TOC)
- **代码高亮**：使用 highlight.js 提供多语言代码高亮
- **Git 集成**：显示文件贡献者信息和最后修改时间

### 🎨 用户界面
- **响应式设计**：完美适配桌面端和移动端
- **暗黑模式**：支持明暗主题切换，自动检测系统偏好
- **可视化组件**：
  - 固定顶部导航栏
  - 侧边导航菜单
  - 移动端抽屉式导航
  - 浮动目录按钮（移动端）

### 🖼️ 媒体功能
- **图片灯箱**：点击图片查看大图，支持缩放和保存
- **手势支持**：移动端支持双指缩放
- **鼠标滚轮缩放**：桌面端 Ctrl+滚轮缩放图片

### 📝 内容增强
- **代码复制**：代码块一键复制功能
- **语言标识**：显示代码块编程语言
- **更新日志**：首页展示项目更新历史
- **贡献者信息**：显示文档贡献者列表

## 技术栈

- **前端框架**：Vue 3 (Composition API)
- **构建工具**：Vite
- **路由管理**：Vue Router 4
- **Markdown 处理**：
  - marked - Markdown 解析
  - marked-gfm-heading-id - 标题 ID 生成
- **代码高亮**：highlight.js
- **样式**：原生 CSS（支持暗黑模式）
- **YAML 解析**：js-yaml

## 项目结构

```
PureWiki/
├── src/
│   ├── components/          # Vue 组件
│   │   ├── App.vue         # 主应用组件
│   │   ├── WikiPage.vue    # Wiki 页面渲染器
│   │   ├── TopNavBar.vue   # 顶部导航栏
│   │   ├── NavigationMenu.vue # 侧边导航菜单
│   │   ├── Changelog.vue   # 更新日志组件
│   │   └── Footer.vue      # 页脚组件
│   ├── router/             # 路由配置
│   ├── generated/          # 自动生成的内容文件
│   └── main.js            # 应用入口
├── docs/                   # Markdown 文档源文件
├── public/                 # 静态资源
│   └── config.json        # 站点配置文件
├── scripts/
│   └── generate-content.mjs # 内容生成脚本
└── package.json
```

## 快速开始

### 环境要求
- Node.js 16+
- npm 或 yarn

### 安装依赖
```bash
npm install
```

### 创建文档
```
cd docs
rm -rf *
git clone <docs repo> .
```

### 开发模式
```bash
npm run dev
```

### 生成内容
```bash
npm run generate
```

### 构建生产版本
```bash
npm run build
```

### 预览构建结果
```bash
npm run preview
```

## 配置说明

### 站点配置 (`public/config.json`)
```json
{
  "siteTitle": "站点标题",
  "logoSrc": "/logo.png",
  "showBetaTag": true,
  "navLinks": [
    { "text": "首页", "url": "/" },
    { "text": "GitHub", "url": "https://github.com/..." }
  ],
  "footerText": "© 2025 站点名称"
}
```

### 文档前置元数据
在 Markdown 文件开头添加 YAML front matter：
```yaml
---
title: 页面标题
hide: true  # 从导航菜单中隐藏
---
```

## 内容管理

### 添加新页面
1. 在 `docs/` 目录下创建 `.md` 文件
2. 运行 `npm run generate` 重新生成内容
3. 页面将自动添加到导航菜单

### 文档组织
- 支持多级目录结构
- `index.md` 文件作为目录默认页面
- 文件名和目录名会自动转换为导航标题

### 隐藏页面
在文档前置元数据中添加 `hide: true` 可以隐藏页面不在导航中显示。

## 部署

项目构建后会在 `dist/` 目录生成静态文件，可以部署到任何静态网站托管服务：

- GitHub Pages
- Netlify
- Vercel
- 传统 Web 服务器

## 开发指南

### 添加新组件
1. 在 `src/components/` 创建 Vue 组件
2. 在需要的地方导入使用
3. 遵循现有的代码风格和约定

### 修改样式
- 使用 CSS 变量进行主题配置
- 遵循响应式设计原则
- 确保暗黑模式兼容性

### 扩展功能
- Markdown 渲染：修改 `scripts/generate-content.mjs`
- 页面组件：编辑 `WikiPage.vue`
- 导航结构：调整 `NavigationMenu.vue`

## 贡献

欢迎提交 Issue 和 Pull Request 来改进 PureWiki！

## 许可证

GPL v3 License