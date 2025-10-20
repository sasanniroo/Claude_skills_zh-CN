---
name: artifacts-builder
description: 一套基于现代前端技术（React、Tailwind CSS、shadcn/ui）的工具，用来创建复杂的 claude.ai HTML 成品。适用于需要状态管理、路由或 shadcn/ui 组件的复杂作品，不适用于简单的单文件 HTML/JSX。
license: 完整条款见 LICENSE.txt
---

# 成品构建器

若要构建强大的前端 claude.ai 成品，请依次执行以下步骤：
1. 使用 `scripts/init-artifact.sh` 初始化前端仓库
2. 编辑生成的代码，开发你的成品
3. 通过 `scripts/bundle-artifact.sh` 将全部代码打包为单个 HTML 文件
4. 将成品展示给用户
5. （可选）对成品执行测试

**技术栈**：React 18 + TypeScript + Vite + Parcel（打包）+ Tailwind CSS + shadcn/ui

## 设计与风格指南

务必要避免常见的“AI 感”俗套：例如过度的居中布局、泛滥的紫色渐变、统一的圆角以及 Inter 字体。

## 快速上手

### 第 1 步：初始化项目

运行初始化脚本，创建新的 React 项目：
```bash
bash scripts/init-artifact.sh <project-name>
cd <project-name>
```

脚本会生成已配置好的项目，包含：
- ✅ React + TypeScript（基于 Vite）
- ✅ Tailwind CSS 3.4.1 与 shadcn/ui 主题体系
- ✅ 已配置路径别名（`@/`）
- ✅ 预装 40+ shadcn/ui 组件
- ✅ 包含所有 Radix UI 依赖
- ✅ 预配置用于打包的 Parcel（通过 `.parcelrc`）
- ✅ 兼容 Node 18+（自动检测并锁定 Vite 版本）

### 第 2 步：开发成品

编辑生成的文件以实现你的成品。具体实践可参考下方 **常见开发任务**。

### 第 3 步：打包为单个 HTML 文件

将 React 应用打包为单个 HTML 成品：
```bash
bash scripts/bundle-artifact.sh
```

脚本会生成 `bundle.html` —— 一个完全自包含的成品，所有 JavaScript、CSS 与依赖均已内联。该文件可直接在 Claude 对话中共享。

**前置要求**：项目根目录必须存在 `index.html`。

**脚本会执行以下操作**：
- 安装打包依赖（parcel、@parcel/config-default、parcel-resolver-tspaths、html-inline）
- 创建带路径别名支持的 `.parcelrc`
- 使用 Parcel 构建（不生成 source map）
- 通过 html-inline 将所有资源内联到单个 HTML 中

### 第 4 步：向用户分享成品

将打包后的 HTML 文件发送给用户，使其可作为 artifact 查看。

### 第 5 步：测试 / 可视化成品（可选）

此步骤完全可选，仅在必要或用户要求时执行。

如需测试或预览，可借助可用工具（包括其他技能，或内置的 Playwright、Puppeteer 等）。通常不建议在展示成品前先行测试，因为这会延迟用户看到成果的时间。如用户提出请求或出现问题，再进行后续测试更妥当。

## 参考资料

- **shadcn/ui 组件**：https://ui.shadcn.com/docs/components
