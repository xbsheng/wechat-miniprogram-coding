# 微信小程序 AI 编码规则

> 本文件是通用规则源，各工具的规则文件从此生成。
> 如需修改规则，编辑此文件后同步到各工具规则文件。

## 平台约束（严格禁止）

微信小程序运行在双线程架构中（逻辑层 JsCore + 渲染层 WebView），**没有 DOM、没有 BOM**。

### 禁止使用的 H5 API

```
❌ DOM/BOM:
  document.* (getElementById, querySelector, createElement, addEventListener, ...)
  window.* (location, history, navigator, open, alert, confirm, prompt, ...)
  document.cookie

❌ 网络请求:
  fetch() / axios / XMLHttpRequest / $.ajax

❌ 存储:
  localStorage / sessionStorage / IndexedDB

❌ 其他:
  alert() / confirm() / prompt()           → 用 wx.showModal
  new Worker()                              → 无 Web Workers
  WebSocket (原生)                          → 用 wx.connectSocket
  FileReader / Blob / URL.createObjectURL   → 用 wx.getFileSystemManager
  navigator.geolocation                     → 用 wx.getLocation
  history.pushState                         → 用 wx.navigateTo
  CSS: * 选择器、:nth-child()、:hover（有限）
```

## API 映射速查

### 核心 API

| H5 | 原生 (wx) | Taro | uni-app |
|----|-----------|------|---------|
| fetch/axios | `wx.request` | `Taro.request` | `uni.request` |
| localStorage.set | `wx.setStorageSync` | `Taro.setStorageSync` | `uni.setStorageSync` |
| localStorage.get | `wx.getStorageSync` | `Taro.getStorageSync` | `uni.getStorageSync` |
| location.href | `wx.navigateTo` | `Taro.navigateTo` | `uni.navigateTo` |
| history.back | `wx.navigateBack` | `Taro.navigateBack` | `uni.navigateBack` |
| alert() | `wx.showToast` | `Taro.showToast` | `uni.showToast` |
| confirm() | `wx.showModal` | `Taro.showModal` | `uni.showModal` |
| navigator.geolocation | `wx.getLocation` | `Taro.getLocation` | `uni.getLocation` |
| document.title | `wx.setNavigationBarTitle` | `Taro.setNavigationBarTitle` | `uni.setNavigationBarTitle` |
| IntersectionObserver | `wx.createIntersectionObserver` | `Taro.createIntersectionObserver` | `uni.createIntersectionObserver` |

### DOM 查询 → 小程序查询

```javascript
// ❌ H5
const el = document.querySelector('.my-class');
el.getBoundingClientRect();

// ✅ 原生 / Taro / uni-app
wx.createSelectorQuery().select('.my-class').boundingClientRect(rect => {
  console.log(rect);
}).exec();
```

### 事件绑定

```
❌ H5:     element.addEventListener('click', handler)
✅ 原生:   <view bindtap="onTap">  (bind=冒泡, catch=阻止冒泡)
✅ Taro:   <View onClick={onTap}>
✅ uni-app: <view @tap="onTap">
```

## CSS 约束

| CSS 特性 | 状态 | 替代方案 |
|----------|------|---------|
| `*` 选择器 | ❌ 不支持 | 使用 class 选择器 |
| `:nth-child(n)` | ❌ 不支持 | 用 `wx:for` + 条件 class |
| `:hover` | ⚠️ 有限 | 用 `hover-class` 属性 |
| `position: fixed` | ⚠️ 部分 | z-index 在某些组件上有问题 |
| `calc()` | ✅ 支持 | — |
| `rpx` 单位 | ✅ 推荐 | 替代 `rem`/`vw` |
| CSS Grid | ⚠️ 有限 | 优先用 Flexbox |
| `background-image` | ⚠️ 无本地路径 | 用网络 URL 或 base64 |
| `@font-face` | ⚠️ 有限 | 仅支持网络 URL |
| CSS 变量 | ✅ (2.9.2+) | `--color: #f00;` |

### 单位

```css
/* ✅ 推荐：rpx 响应式 */
.box { width: 750rpx; height: 100rpx; font-size: 28rpx; }

/* ⚠️ 可用但不常用 */
.box { width: 50%; padding: 10px; }

/* ❌ 不推荐：vw/vh 支持不一致 */
.box { width: 100vw; height: 100vh; }
```

## 框架代码模板

### 原生小程序

```javascript
Page({
  data: { list: [] },
  onLoad(options) { /* 页面加载 */ },
  onShow() { /* 页面显示 */ },
  onReady() { /* DOM 就绪 */ },
  onHide() { /* 页面隐藏 */ },
  onUnload() { /* 页面销毁 */ },

  // ✅ 事件处理函数必须用普通方法，不能用箭头函数
  onTap(e) {
    const id = e.currentTarget.dataset.id;
    this.setData({ selectedId: id });  // ✅ 必须用 setData
  },

  fetchData() {
    wx.request({
      url: 'https://api.example.com/data',
      method: 'GET',
      success: (res) => {
        this.setData({ list: res.data.list });
      }
    });
  }
});
```

```xml
<!-- .wxml 模板 -->
<view class="container">
  <view wx:for="{{list}}" wx:key="id" bindtap="onTap" data-id="{{item.id}}">
    <text>{{item.name}}</text>
  </view>
</view>
```

### Taro（React 语法）

```jsx
import { View, Text } from '@tarojs/components';
import Taro from '@tarojs/taro';
import { useState, useEffect } from 'react';

export default function MyPage() {
  const [list, setList] = useState([]);

  useEffect(() => {
    Taro.request({ url: 'https://api.example.com/data' })
      .then(res => setList(res.data.list));
  }, []);

  const onTap = (id) => {
    Taro.navigateTo({ url: `/pages/detail/index?id=${id}` });
  };

  return (
    <View className="container">
      {list.map(item => (
        <View key={item.id} onClick={() => onTap(item.id)}>
          <Text>{item.name}</Text>
        </View>
      ))}
    </View>
  );
}
```

### uni-app（Vue 语法）

```vue
<template>
  <view class="container">
    <view v-for="item in list" :key="item.id" @tap="onTap(item.id)">
      <text>{{ item.name }}</text>
    </view>
  </view>
</template>

<script setup>
import { ref } from 'vue';

const list = ref([]);

onLoad(() => {
  uni.request({
    url: 'https://api.example.com/data',
    success: (res) => { list.value = res.data.list; }
  });
});

const onTap = (id) => {
  uni.navigateTo({ url: `/pages/detail/index?id=${id}` });
};
</script>
```

## AI 常见错误及修正

### 1. 用 this.state 代替 this.setData

```javascript
// ❌ AI 生成的 React 风格
this.state = { count: 0 };
this.setState({ count: 1 });

// ✅ 原生小程序
this.data = { count: 0 };
this.setData({ count: 1 });  // 必须用 setData 触发视图更新
```

### 2. 箭头函数作为 Page 方法

```javascript
// ❌ 箭头函数的 this 指向错误
Page({
  onTap: () => {
    this.setData({ x: 1 });  // this 指向错误！
  }
});

// ✅ 使用普通方法
Page({
  onTap() {
    this.setData({ x: 1 });
  }
});
```

### 3. 使用 `<img>` 标签

```xml
<!-- ❌ H5 -->
<img src="/static/logo.png" alt="logo" />

<!-- ✅ 小程序 -->
<image src="/static/logo.png" mode="aspectFit" />
```

### 4. 使用 `<a>` 标签导航

```xml
<!-- ❌ H5 -->
<a href="/pages/about/index">关于我们</a>

<!-- ✅ 小程序 -->
<navigator url="/pages/about/index" open-type="navigate">关于我们</navigator>
```

### 5. 背景图使用本地路径

```css
/* ❌ 本地路径无效 */
.box { background-image: url('/images/bg.png'); }

/* ✅ 使用网络 URL */
.box { background-image: url('https://cdn.example.com/bg.png'); }

/* ✅ 或使用 base64 */
.box { background-image: url('data:image/png;base64,...'); }

/* ✅ 或用 <image> 组件替代 */
<view class="wrapper">
  <image src="/images/bg.png" class="bg" />
  <view class="content">...</view>
</view>
```

### 6. 导入不兼容的 npm 包

```javascript
// ❌ 依赖 DOM 的包在小程序中不可用
import moment from 'moment';        // 可能有问题
import $ from 'jquery';             // 完全不可用

// ✅ 使用小程序兼容的库
// dayjs 替代 moment（Taro/uni-app 可用）
import dayjs from 'dayjs';
```

## 项目配置要点

### app.json 关键配置

```json
{
  "pages": ["pages/index/index", "pages/detail/index"],
  "window": {
    "navigationBarTitleText": "我的小程序",
    "navigationBarBackgroundColor": "#ffffff"
  },
  "permission": {
    "scope.userLocation": { "desc": "用于显示附近门店" }
  }
}
```

### 必须配置

- 后台配置服务器域名白名单（`mp.weixin.qq.com` → 开发管理 → 服务器域名）
- `wx.request` 的域名必须在白名单中，且必须是 HTTPS

## 框架识别方法

不确定项目用的哪个框架？看这些文件：

| 文件 | 框架 |
|------|------|
| `project.config.json` + `.wxml` 文件 | **原生小程序** |
| `config/index.js` + `package.json` 含 `@tarojs/*` | **Taro** |
| `manifest.json` + `pages.json` | **uni-app** |

## Prompt 模板

向 AI 提问时，在 prompt 前加上：

```
你正在开发微信小程序（框架：原生/Taro/uni-app）。严格遵守：
1. 禁止使用 document/window/fetch/axios/localStorage
2. 网络请求用 wx.request / Taro.request / uni.request
3. 存储用 wx.setStorageSync 等 wx API
4. 导航用 wx.navigateTo，不用 <a> 标签
5. 图片用 <image>，不用 <img>
6. 样式用 rpx 单位，不用 * 选择器
7. 背景图用网络 URL，不用本地路径
8. 原生小程序 Page 方法用普通函数，不能用箭头函数
9. 数据更新必须用 setData，不能用 setState
```
