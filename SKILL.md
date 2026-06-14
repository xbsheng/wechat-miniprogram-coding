---
name: wechat-miniprogram-coding
description: "Use when developing WeChat Mini Programs. Covers native/Taro/uni-app frameworks, forbidden H5 APIs, platform constraints, and correct Mini Program patterns."
version: 1.0.0
author: xbsheng
license: MIT
metadata:
  hermes:
    tags: [wechat, miniprogram, weapp, taro, uni-app, frontend, mobile]
    related_skills: []
---

# 微信小程序开发

## Overview

微信小程序运行在双线程架构中（逻辑层 JsCore + 渲染层 WebView），**没有 DOM、没有 BOM**。AI 编码助手的训练数据以 Web/H5 代码为主，生成的小程序代码经常包含不支持的 API。本 Skill 提供完整的平台约束和正确代码模式。

## When to Use

- 构建或修改微信小程序代码（原生、Taro 或 uni-app）
- AI 生成的代码包含 `document.*`、`window.*`、`fetch`、`axios`、`localStorage`
- 需要验证 CSS 在小程序环境中的兼容性
- 将 H5/Web 组件转换为小程序等价物

## 框架识别

| 文件 | 框架 |
|------|------|
| `project.config.json` + `.wxml` 文件 | **原生小程序** |
| `config/index.js` + `package.json` 含 `@tarojs/*` | **Taro** |
| `manifest.json` + `pages.json` | **uni-app** |

## 禁止使用的 H5 API（关键）

### DOM & BOM
```javascript
// ❌ 这些对象在小程序中不存在
document.getElementById / querySelector / createElement / ...
window.location / window.history / window.navigator / ...
document.cookie
```

### 网络请求
```javascript
// ❌ 禁止
fetch() / axios / XMLHttpRequest / $.ajax
```

### 存储
```javascript
// ❌ 禁止
localStorage / sessionStorage / IndexedDB
```

### 其他禁止的 API
```javascript
// ❌ 禁止
alert() / confirm() / prompt()          // → wx.showModal
new Worker()                             // 无 Web Workers
WebSocket (原生)                         // → wx.connectSocket
canvas.getContext('2d') (DOM canvas)     // → wx.createCanvasContext
FileReader / Blob / URL.createObjectURL  // → wx.getFileSystemManager
navigator.geolocation                    // → wx.getLocation
history.pushState                        // → wx.navigateTo
```

## API 映射速查

| H5 API | 原生 (wx) | Taro | uni-app |
|--------|-----------|------|---------|
| `fetch/axios` | `wx.request` | `Taro.request` | `uni.request` |
| `localStorage.set` | `wx.setStorageSync` | `Taro.setStorageSync` | `uni.setStorageSync` |
| `localStorage.get` | `wx.getStorageSync` | `Taro.getStorageSync` | `uni.getStorageSync` |
| `location.href` | `wx.navigateTo` | `Taro.navigateTo` | `uni.navigateTo` |
| `history.back` | `wx.navigateBack` | `Taro.navigateBack` | `uni.navigateBack` |
| `alert()` | `wx.showToast` | `Taro.showToast` | `uni.showToast` |
| `confirm()` | `wx.showModal` | `Taro.showModal` | `uni.showModal` |
| `navigator.geolocation` | `wx.getLocation` | `Taro.getLocation` | `uni.getLocation` |
| `document.title` | `wx.setNavigationBarTitle` | `Taro.setNavigationBarTitle` | `uni.setNavigationBarTitle` |

## CSS 约束

| CSS 特性 | 状态 | 替代方案 |
|----------|------|---------|
| `*` 选择器 | ❌ | class 选择器 |
| `:nth-child(n)` | ❌ | `wx:for` + 条件 class |
| `:hover` | ⚠️ | `hover-class` 属性 |
| `rpx` 单位 | ✅ 推荐 | 替代 `rem`/`vw` |
| `background-image` | ⚠️ | 仅网络 URL 或 base64 |

## AI 常见错误及修正

### 1. this.state 代替 this.setData
```javascript
// ❌
this.setState({ count: 1 });
// ✅
this.setData({ count: 1 });
```

### 2. 箭头函数作为 Page 方法
```javascript
// ❌
Page({ onTap: () => { this.setData({ x: 1 }); } });
// ✅
Page({ onTap() { this.setData({ x: 1 }); } });
```

### 3. 使用 `<img>` 标签
```xml
<!-- ❌ -->  <img src="/logo.png" />
<!-- ✅ -->  <image src="/logo.png" mode="aspectFit" />
```

### 4. 背景图使用本地路径
```css
/* ❌ */ .box { background-image: url('/images/bg.png'); }
/* ✅ */ .box { background-image: url('https://cdn.example.com/bg.png'); }
```

## 验证清单

- [ ] 无 `document.*` 或 `window.*` 引用
- [ ] 无 `fetch`/`axios` — 请求使用 `wx.request`
- [ ] 无 `localStorage` — 使用 `wx.setStorageSync`
- [ ] Page 方法为普通函数，非箭头函数
- [ ] 数据更新使用 `setData`
- [ ] 图片使用 `<image>` 标签
- [ ] CSS 使用 `rpx` 单位
