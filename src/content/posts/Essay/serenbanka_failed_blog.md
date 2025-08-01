---
title: 记一次用GitHub Pages搭建web千恋万花
published: 2025-08-01
description: "记一次用GitHub Pages搭建web千恋万花——从入门到失败的踩坑之旅"
image: "./serenbanka_failed_blog.webp"
tags: [Essay, 随笔, 记录]
category: 记录
draft: false
series: 随笔
---

# 记一次用GitHub Pages搭建web千恋万花——从入门到失败的踩坑之旅

## 前言

最近发现了一个有趣的项目——serenbanka-Web版，这是一个基于Vue.js开发的web版千恋万花。作为一名前端开发者，看到能在浏览器里运行galgame，立马就心动了。心想着用GitHub Pages免费部署一个应该不难，结果...现实总是这么残酷。

今天就来记录一下这次踩坑经历，希望能给有同样想法的小伙伴们一些参考。

## 项目初探

### 项目简介

这个web版千恋万花项目的技术栈如下：
- **前端框架**：Vue.js 2.x
- **构建工具**：Webpack
- **样式预处理**：SCSS
- **状态管理**：Vuex
- **路由管理**：Vue Router

项目结构大致如下：
```
serenbanka-Vue/
├── public/
├── src/
│   ├── assets/         # 静态资源
│   ├── components/     # Vue组件
│   ├── views/          # 页面组件
│   ├── store/          # Vuex状态管理
│   └── router/         # 路由配置
├── package.json        # 依赖配置
└── vue.config.js       # Vue CLI配置
```

看起来是个标准的Vue项目，理论上应该可以轻松构建部署。

## 第一次尝试：直接Fork部署

### 操作流程

按照常规思路，我先尝试了最简单的方式：

1. 在GitHub上制作了一个仓库
2. 进入项目设置页面：`Settings` → `Pages`
3. 选择部署源：`Deploy from a branch`
4. 选择分支：`main`
5. 点击保存，等待部署

### 遇到的问题

几分钟后收到部署成功的邮件通知，兴冲冲地点开链接，结果页面显示：

```
404 - File not found
```

**问题分析：**

GitHub Pages直接部署的是源代码，而Vue项目需要先构建才能生成可访问的静态文件。源码中的`.vue`文件浏览器是无法直接解析的。

正确的流程应该是：`源码` → `构建` → `生成dist目录` → `部署dist目录`

## 第二次尝试：本地构建后推送

### 环境准备

既然需要构建，那就在本地进行：

```bash
# 克隆项目
git clone https://github.com/yCENzh/serenbanka-Web.git
cd serenbanka-Web

# 查看Node.js版本
node -v  # v16.14.0

# 安装依赖
npm install
```

### 依赖安装问题

第一个坑就来了：

```bash
npm WARN deprecated stable@0.1.8: Modern JS already guarantees Array#sort() is a stable sort
npm WARN deprecated urix@0.1.0: Please see https://github.com/lydell/urix#deprecated
npm WARN deprecated resolve-url@0.2.1: https://github.com/lydell/resolve-url#deprecated
npm ERR! peer dep missing: webpack@^4.0.0 || ^5.0.0, required by webpack-dev-server@4.15.1
```

**问题分析：**
- 项目使用的是较老版本的依赖
- 一些包已经被标记为过时（deprecated）
- 存在peer dependency警告

**解决尝试：**

```bash
# 清理npm缓存
npm cache clean --force

# 删除node_modules重新安装
rm -rf node_modules package-lock.json
npm install

# 尝试使用yarn
yarn install

# 强制安装忽略警告
npm install --legacy-peer-deps
```

最终通过`--legacy-peer-deps`参数勉强安装成功。

### 构建过程的问题

接下来尝试构建：

```bash
npm run build
```

结果又遇到了新的问题：

```bash
ERROR in ./src/assets/images/bg/bg1.jpg
Module parse failed: Unexpected character '�' (1:0)
You may need an appropriate loader to handle this file type.

ERROR in ./src/components/GameDialog.vue
Module build failed (from ./node_modules/vue-loader/lib/index.js):
SyntaxError: Unexpected token
```

**主要问题：**
1. **图片资源路径问题**：Webpack无法正确处理某些图片文件
2. **Vue组件语法错误**：部分组件代码存在语法问题
3. **音频文件格式**：浏览器兼容性问题
4. **资源文件过大**：构建时内存不足

### 问题修复尝试

针对这些问题，我尝试了以下修复方案：

```javascript
// vue.config.js 配置修改
module.exports = {
  publicPath: process.env.NODE_ENV === 'production' ? './' : '/',
  outputDir: 'dist',
  assetsDir: 'static',
  lintOnSave: false,
  configureWebpack: {
    resolve: {
      alias: {
        '@': path.resolve(__dirname, 'src')
      }
    }
  }
}
```

```bash
# 增加Node.js内存限制
export NODE_OPTIONS="--max-old-space-size=4096"
npm run build
```

经过一番折腾，本地构建总算成功了，生成了`dist`目录。

### 部署dist目录

有两种方式将构建后的文件部署到GitHub Pages：

**方式一：使用gh-pages分支**
```bash
# 安装gh-pages工具
npm install -g gh-pages

# 部署到gh-pages分支
gh-pages -d dist
```

**方式二：手动创建部署分支**
```bash
# 创建新的分支用于部署
git checkout --orphan gh-pages
git rm -rf .
cp -r dist/* .
git add .
git commit -m "Deploy to GitHub Pages"
git push origin gh-pages
```

部署成功后，在项目设置中将Pages源改为`gh-pages`分支。

### 新的问题出现

虽然页面能正常访问了，但又遇到了新问题：

1. **资源加载失败**：图片、音频文件无法正常加载
2. **路由问题**：刷新页面后出现404错误
3. **CORS问题**：某些资源因为跨域被阻止加载

## 第三次尝试：GitHub Actions自动化部署

为了解决手动构建的繁琐，我配置了GitHub Actions自动化构建部署。

### 配置Actions工作流

在`.github/workflows/deploy.yml`中配置：

```yaml
name: Build and Deploy
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout 🛎️
      uses: actions/checkout@v3

    - name: Setup Node.js 💿
      uses: actions/setup-node@v3
      with:
        node-version: '16'
        cache: 'npm'

    - name: Install dependencies 📦
      run: npm ci --legacy-peer-deps

    - name: Build 🔧
      run: npm run build
      env:
        NODE_OPTIONS: "--max-old-space-size=4096"

    - name: Deploy 🚀
      uses: peaceiris/actions-gh-pages@v3
      if: github.ref == 'refs/heads/main'
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist
```

### Actions执行失败

推送代码后，Actions开始运行，但很快就失败了：

```bash
Error: Process completed with exit code 137
```

**退出码137的含义**：进程被SIGKILL信号终止，通常是内存不足导致的。

查看详细日志发现：
- GitHub Actions的runner内存限制为7GB
- 项目构建时需要加载大量游戏资源文件
- 内存使用超限被系统强制终止

尝试了多种优化方案都无法解决这个问题。

## 失败原因深度分析

### 技术层面的限制

经过多次尝试后，我总结了失败的主要原因：

**1. 项目规模问题**
```
项目总大小：约2.5GB
├── 音频文件：1.8GB
├── 图片资源：600MB
├── 视频文件：100MB
└── 代码文件：< 10MB
```

**2. GitHub限制**
- 单个仓库建议不超过1GB
- Actions运行时内存限制
- Pages服务的带宽限制

**3. 浏览器兼容性**
- 大文件加载性能问题
- 音频格式兼容性
- 移动端适配问题

### 根本性问题

这类项目其实不太适合GitHub Pages：

1. **资源密集型**：游戏包含大量多媒体资源
2. **版权风险**：包含原作游戏素材
3. **用户体验**：加载时间过长影响体验
4. **维护成本**：项目依赖较老，维护困难

## 更好的解决方案：雨云云服务器

在GitHub Pages屡次受挫后，我开始考虑其他部署方案。这时想到了雨云这个云服务提供商。

### 为什么选择雨云？

**优势对比：**

| 维度 | GitHub Pages | 雨云云服务器 |
|------|-------------|-------------|
| **部署难度** | 需要复杂构建配置 | 支持直接上传部署 |
| **资源限制** | 严格的大小和带宽限制 | 根据套餐灵活配置 |
| **访问速度** | 国外节点，国内较慢 | 国内多节点，速度快 |
| **环境控制** | 无法自定义运行环境 | 完全的环境控制权 |
| **技术支持** | 仅社区支持 | 官方客服技术支持 |
| **成本** | 免费但功能受限 | 灵活计费，性价比高 |

### 雨云部署的优势

**1. 环境自由度高**
- 可以安装任意版本的Node.js
- 能够解决复杂的依赖冲突
- 支持自定义Nginx配置
- 可以进行性能调优

**2. 资源配置灵活**
- 不受GitHub的严格限制
- 可根据实际需求选择配置
- 支持弹性扩容
- 带宽和存储空间充足

**3. 国内访问体验更佳**
- 服务器位于国内，延迟低
- 不受国际网络波动影响
- 针对国内用户优化
- CDN加速支持

### 快速开始使用雨云

如果你也想部署类似的项目，推荐尝试雨云：

**注册步骤：**

1. **访问官网**：通过[雨云官网](https://www.rainyun.com/yCENzh_)进入（这个链接会自动带上邀请码，注册时享受5折优惠）

2. **注册账号**：
   - 填写邮箱和密码
   - 完成手机验证
   - 邀请码会自动填充，享受首单5折

3. **选择配置**：
   - 根据项目需求选择合适的服务器配置
   - 推荐至少2核4G配置用于前端项目构建
   - 选择国内节点保证访问速度

4. **部署项目**：
   - 上传项目文件
   - 配置Node.js环境
   - 设置Nginx反向代理
   - 绑定域名（可选）

**性价比分析：**

对于个人开发者来说，雨云的定价还是很合理的：
- 最低配置服务器月费不到30元
- 相比于GitHub Pages的限制，灵活性大大提升
- 首单5折的优惠力度也不错

### 给其他开发者的建议

**选择部署平台的考虑因素：**

1. **项目规模和复杂度**
   - 简单静态站点：GitHub Pages
   - 复杂前端应用：云服务器
   - 资源密集型项目：专业云服务

2. **用户群体和访问模式**
   - 国外用户为主：GitHub Pages / Vercel
   - 国内用户为主：国内云服务商
   - 高并发需求：专业CDN方案

3. **预算和维护成本**
   - 个人项目：优先考虑免费方案
   - 商业项目：选择稳定可靠的付费服务
   - 长期项目：考虑维护和扩展成本

**技术选型建议：**

- 充分调研开源项目的维护状态
- 优先选择活跃维护的项目
- 关注依赖的更新和安全性
- 做好技术债务管理

## 后续计划

虽然GitHub Pages这条路没走通，但项目本身还是很有意思的。接下来的计划：

### 短期目标

1. **使用雨云重新部署**
   - 申请合适配置的云服务器
   - 解决构建和部署问题
   - 优化资源加载性能

2. **项目优化改进**
   - 更新依赖到稳定版本
   - 实现资源懒加载
   - 优化移动端体验
   - 添加PWA支持

### 长期规划

1. **写一篇成功部署教程**
   - 详细记录雨云部署过程
   - 分享性能优化经验
   - 总结最佳实践

2. **技术栈升级**
   - 迁移到Vue 3.x
   - 使用Vite替代Webpack
   - 引入TypeScript支持

3. **开源贡献**
   - 向原项目提交修复PR
   - 分享部署和优化经验
   - 帮助其他开发者避坑

## 写在最后

这次从满怀期待到现实打脸的经历让我深刻体会到了技术选型的重要性。**不是所有项目都适合GitHub Pages，也不是免费的就是最好的。**

作为开发者，我们需要：
- 🎯 **理性评估项目需求**：不盲目追求免费方案
- 🎯 **选择合适的工具**：工具是为项目服务的，不是相反
- 🎯 **重视用户体验**：部署成功只是第一步，用户体验才是关键
- 🎯 **持续学习改进**：技术在发展，我们的认知也要跟上

感谢大家耐心看完这篇"失败总结"。失败并不可怕，关键是要从中学习和成长。希望我的踩坑经历能给大家一些参考和启发。

如果你也有类似的项目部署需求，不妨试试雨云的云服务器，也许会有不一样的体验！