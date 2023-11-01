---
layout: article
title: JavaScript 包管理器
date: 2023-10-10
key: P2023-10-10
tags: 前端
show_author_profile: true
comment: true
sharing: true
aside:
  toc: true
---

JavaScript 包管理器

<!--more-->

# Node.js 和 JavaScript 之间的联系和区别

Node.js 使用的是 JavaScript 语言。换句话说，Node.js 不是一种新的编程语言，而是一个运行时环境，它使用 JavaScript 作为其编程语言。

- 运行环境：传统上，JavaScript 主要在浏览器中运行，用于为网站提供交互性和动态功能。Node.js它是一个允许在服务器端运行 JavaScript 的平台。
- API 和对象: 在浏览器环境中，JavaScript 提供了许多专门针对网页交互的 API，如 `document`, `window` 等。Node.js提供了许多服务器端功能的 API，例如文件系统操作、网络请求、流等。它没有浏览器提供的 DOM API。

- 用途：JavaScript主要用于浏览器中，为网页添加交互性。Node.js用于构建服务器端应用程序，但也可以用于构建工具、脚本等。
- 工具和库：
  - JavaScript：有很多库和框架，如 jQuery, React, Angular, Vue 等，专为浏览器环境设计。
  - Node.js：拥有一个庞大的包生态系统（npm），其中包含大量为服务器端开发和其他任务设计的库和工具。

# NVM

Node Version Manager: Node.js 版本管理器

NVM for Windows Github: https://github.com/coreybutler/nvm-windows

```
nvm current: Display active version.
nvm list installed
```

NVM安装的每个 Node.js 版本都有其独立的全局 npm 包，这样可以避免版本冲突。

# npm

NPM (Node Package Manager) 是 Node.js 的默认包管理器，用于安装、管理和发布 JavaScript 库。但是对某些项目，安装速度可能较慢。

# npx

npx 是 npm 包运行器工具，它随 npm 5.2.0 而来。它允许用户运行局部安装的命令行工具，或者直接从 npm 上运行代码，而不需要全局安装。

NPX可以直接运行包而无需全局/局部安装。也可以轻松运行不同版本的工具。npx会在执行后自动清理，不会保留全局安装。

NPX主要是为了解决某些特定的使用场景，不是一个完整的包管理器。

# yarn

yarn 是一个 JavaScript 包管理器，由 Facebook 推出，作为 npm 的替代方案。由于 Node.js 是 JavaScript 运行时环境，所以我们也会说它是 Node.js 的包管理器。

`yarn.lock` 文件确保了安装的依赖版本的一致性。

# PNPM

PNPM 是另一个 JavaScript 包管理器，其目标是更高效地存储项目的依赖，节省空间。

- 使用硬链接和符号链接存储 `node_modules`，从而节省空间。
- 严格的 `node_modules` 结构，确保所有项目共享相同的依赖版本。

但是，相对于 npm 和 yarn，PNPM 社区规模较小。同时一些工具或库可能不完全支持 PNPM 的 `node_modules` 结构。