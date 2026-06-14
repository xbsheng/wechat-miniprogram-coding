# 微信小程序开发规范

本项目为微信小程序项目。你生成的所有代码必须严格遵守微信小程序的平台约束。

## 严格禁止

禁止生成以下任何 H5/Web API：

- `document.*`（getElementById、querySelector、createElement、addEventListener 等）
- `window.*`（location、history、navigator、open、alert、confirm、prompt 等）
- `fetch()` / `axios` / `XMLHttpRequest` / `$.ajax`
- `localStorage` / `sessionStorage` / `IndexedDB`
- `document.cookie`
- `new Worker()`、原生 `WebSocket`、`FileReader`、`Blob`
- CSS：`*` 选择器、`:nth-child()`、`:hover`（用 `hover-class` 替代）

## 必须使用的替代 API

| 禁止 | 必须使用 |
|------|---------|
| fetch/axios | `wx.request` |
| localStorage | `wx.setStorageSync` / `wx.getStorageSync` |
| location.href | `wx.navigateTo` / `wx.redirectTo` |
| history.back | `wx.navigateBack` |
| alert() | `wx.showToast` / `wx.showModal` |
| `<img>` | `<image>` |
| `<a href>` | `<navigator url>` |
| `this.setState` | `this.setData` |

## 代码规范

1. Page 方法使用普通函数，**不能用箭头函数**（this 指向会错误）
2. 数据更新必须用 `this.setData({})`，不能直接赋值 `this.data.x = 1`
3. 样式单位用 `rpx`，不用 `rem` / `vw` / `vh`
4. `background-image` 只能用网络 URL 或 base64，不能用本地路径
5. 图片组件用 `<image src="..." mode="aspectFit" />`
6. 导航用 `wx.navigateTo({ url: '/pages/xxx/index' })`

## 生命周期

```javascript
Page({
  data: {},
  onLoad(options) {},   // 页面加载，接收路由参数
  onShow() {},          // 页面显示
  onReady() {},         // 页面初次渲染完成
  onHide() {},          // 页面隐藏
  onUnload() {},        // 页面销毁
});
```

## 网络请求模板

```javascript
wx.request({
  url: 'https://api.example.com/data',
  method: 'GET',
  header: { 'Authorization': 'Bearer xxx' },
  success: (res) => {
    this.setData({ list: res.data.list });
  },
  fail: (err) => {
    wx.showToast({ title: '请求失败', icon: 'error' });
  }
});
```

## 框架说明

如项目使用 Taro 或 uni-app：
- Taro：用 `Taro.request`、`Taro.navigateTo`、`Taro.setStorageSync` 等
- uni-app：用 `uni.request`、`uni.navigateTo`、`uni.setStorageSync` 等
- 组件语法相应调整（Taro 用 JSX，uni-app 用 Vue SFC）
