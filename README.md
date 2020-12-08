# 推啊小程序媒体对接

<!-- ## 产品介绍
![产品介绍](./img/public.png) -->

## 类型一：使用WebView组件打开推啊互动广告

### 业务域名添加

推啊业务域名：
- https://engine.aoclia.com
- https://tui.yiyhua.cn

为保证WebView组件能够打开推啊域名，须添加小程序信任文件到推啊业务服务器，以及将推啊业务域名添加进小程序业务域名中。

![信任文件下载](./img/img1.png)

![添加业务域名](./img/img2.png)

### 基础实现代码样例
- wxml
  ```html
  <view>
    <web-view src="{{url}}" binderror="loadError" bindload="loadSuccess" />
  </view>
  ```
- 方式一：js（通过appKey和adslotId方式打开）
  ```javascript
  Page({
    data: {
      url: '',
    },
    onLoad(options) {
      wx.showLoading({
        title: '页面加载中...'
      })

      const params = {
        appKey: '', // 必传 your appKey
        adslotId: '', // 非必传 your adslotId
        device_id: '', // 非必传 用户设备ID Andriod:imei;iOS:idfa
        userId: '', // 非必传 用户唯一标识（涉及虚拟奖品发放时需要传）
      }
      function serialize(obj) {
        return Object.keys(obj)
          .map((key) =>
            obj[key] === null || obj[key] === undefined
              ? ''
              : key + '=' + obj[key]
          )
          .join('&');
      }

      this.setData({
        url: `https://engine.aoclia.com/index/activity?${serialize(params)}`
      })
    },
    loadError(e) {
      console.error(e)
      wx.hideLoading()
    },
    loadSuccess(e) {
      console.log(e)
      wx.hideLoading()
    }
  })
  ```
- 方式二：js（通过媒体后台获取URL方式）
URL类似如下：
https://engine.aoclia.com/index/activity?appKey=appKey&adslotId=adslotId
```javascript
  Page({
    data: {
      url: '',
    },
    onLoad(options) {
      wx.showLoading({
        title: '页面加载中...'
      })

      this.setData({
        url: `https://engine.aoclia.com/index/activity?appKey=appKey&adslotId=adslotId`
      })
    },
    loadError(e) {
      console.error(e)
      wx.hideLoading()
    },
    loadSuccess(e) {
      console.log(e)
      wx.hideLoading()
    }
  })
  ```

### 测试

对接完成后请体验整个广告流程（WebView 打开推啊活动 -> 参加活动 -> 点击各类券 -> 进入落地页），如反复检验后仍有问题请联系推啊开发


## 类型二：跳转到推啊小程序

### 对接流程

- 合作方媒体在推啊媒体平台 (https://ssp.tuia.cn) 注册账号
- 创建广告位，在后台获取 appKey、adslotId
- 在广告入口处进行小程序跳转

### 跳转推啊小程序参数说明

- 推啊小程序appId为

  `wx4645f4f355c0521a`

- 推啊小程序跳转的path为

  `pages/activity/index`

- 跳转时需要携带的参数 **<font color="red">具体参考《推啊媒体API对接文档2.1.4版 》</font>**

  |  参数名称   | 参数定义  |  是否必传  |
  |  ----  | ----  |  ----  |
  | appKey  | 媒体的Key（从推啊媒体平台获取） |  必传  |
  | adslotId  | 广告位id（从推啊媒体平台获取） |  必传  |
  | userId  | 当前用户在媒体系统的唯一标识符（不能含特殊字符如<,%） |  非必传（涉及虚拟奖品发放时需要传）  |
  | device_id  | 当前用户设备号，Andriod: imei, Ios: idfa(不能含特殊字符如<,%） |  保留字段，非必传  |

  > 注意：

  > appKey 或 adslotId 值传错，进入空白页面。


### 基础实现代码样例
  ```javascript
  const params = {
    appKey: '', // 必传 your appKey
    adslotId: '', // 必传 your adslotId
    device_id: '', // 非必传 用户设备ID Andriod:imei;iOS:idfa
    userId: '', // 非必传 用户唯一标识（涉及虚拟奖品发放时需要传）
  }
  function serialize(obj) {
    return Object.keys(obj)
      .map((key) =>
        obj[key] === null || obj[key] === undefined
          ? ''
          : key + '=' + obj[key]
      )
      .join('&');
  }
  const appConfig = {
    appId: 'wx4645f4f355c0521a',
    path: 'pages/activity/index?' + serialize(params),
    extraData: {},
    envVersion: 'release',
  }
  if (wx.navigateToMiniProgram) { // 兼容性处理
    wx.navigateToMiniProgram({
      ...appConfig,
      success(res) {
        console.log(res)
      },
      fail(res) {
        console.error(res)
      },
      complete(res) {
        console.log(res)
      }
    })
  } else {
    wx.showModal({
      title: '提示',
      content: '当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。'
    })
  }
  ```
  具体用法可以参见[wx.navigateToMiniProgram](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/miniprogram-navigate/wx.navigateToMiniProgram.html)


# 广告主对接文档

> 面向小程序广告主，上报数据使用

## 接口

### 1、微信卡券核销接口

> 用于用户核销微信卡券时广告主回调使用

请求url：https://activity.tuia.cn/weixin/consumeCard/callback

请求方式：GET或者POST

请求参数：

| 字段名称   | 数据类型 | 是否必传 | 说明         |
| ---------- | -------- | -------- | ------------ |
| advertKey  | string   | 是       | 推啊广告秘钥 |
| wechatCode | string   | 是       | 微信卡券code |
| timestamp  | string   | 是       | 时间戳（ms） |

响应参数：

| 字段名称 | 数据类型 | 是否必传 | 说明                              |
| -------- | -------- | -------- | --------------------------------- |
| data     | boolean  | 是       | 响应数据                          |
| success  | boolean  | 是       | 响应状态，true：成功，false：失败 |

示例：

```json
{"data":true,"success":true}
```


### 2、落地页转化数据回传接口

> 用于广告主回传落地页转化数据埋点

请求url：https://activity.tuia.cn/log/effect/wechatCard

请求方式：GET或者POST

请求参数：

| 字段名称   | 数据类型 | 是否必传 | 说明                                                |
| ---------- | -------- | -------- | --------------------------------------------------- |
| advertKey  | string   | 是       | 推啊广告秘钥                                        |
| wechatCode | string   | 是       | 微信卡券code                                        |
| type       | int      | 是       | 转化数据类型，1、落地页曝光 2、填表状态 3、支付状态 |

响应参数：

| 字段名称 | 数据类型 | 是否必传 | 说明       |
| -------- | -------- | -------- | ---------- |
| redesc   | string   | 是       | 错误描述   |
| record   | string   | 是       | 错误码     |
| a_old    | String   | 是       | 推啊订单id |

示例：

```json
{"redesc":"成功","record":"0000000","a_oId":"taw-123"}
```

注意：只有在 a_oId 有效的情况下才会返回 a_oId 字段，测试情况或者 a_oId 无效时则不返回。 

## Demo

> 示例代码用于小程序前端进行数据上报，后端上报同理

```javascript
// 在小程序的onShow生命周期发起核销操作
onShow(params) {
  // 该参数就是wechatCode
  const encrypt_code = params.encrypt_code
  if (encrypt_code) {
    // 发起核销
    this.consumeCard(encrypt_code)
    this.exposure1(encrypt_code)
  }
},
// 核销
consumeCard(wechatCode) {
  this.ajax('https://activity.tuia.cn/weixin/consumeCard/callback', {
    wechatCode
  })
},
// 页面曝光
exposure1(wechatCode) {
  this.ajax('https://activity.tuia.cn/log/effect/wechatCard', {
    wechatCode,
    type: 1
  })
},
// 填表完成
exposure2(wechatCode) {
  this.ajax('https://activity.tuia.cn/log/effect/wechatCard', {
    wechatCode,
    type: 2
  })
},
// 支付完成
exposure3(wechatCode) {
  this.ajax('https://activity.tuia.cn/log/effect/wechatCard', {
    wechatCode,
    type: 3
  })
},
// 上报数据的工具函数封装，开发者自行处理
ajax(url, data) {
  // 如果使用post请求，需要表单提交
  http.get(url, {
    params: {
      advertKey: '推啊广告秘钥，自行申请',
      timestamp: Date.now(),
      ...data
    }
  })
}
```
