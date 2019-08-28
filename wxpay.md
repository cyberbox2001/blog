### 不明所以的坑
- senparc的tenpayv3controller的index是用于html5的授权使用的，与小程序没有关系
- 其中用的OAuthApi.GetAuthorizeUrl 其实是用于页面授权取openid，类似于QQ授权登录 ：（


### 小程序支付
1. 前端 wx.login 成功后取得 code
2. 把code与produce_id提交给后端  
```
///OnLogin(string code)换取openid与session_key 
var jsonResult = SnsApi.JsCode2Json(WxOpenAppId, WxOpenAppSecret, code);
var sessionBag = SessionContainer.UpdateSession(null, jsonResult.openid, jsonResult.session_key, unionId);
```
3. 有了openid,就可以生成订单并统一下单了


### 业务流程
1. 生成订单页面 有产品名称，价格，数量。。。

2. 提交订单----前端是否存有openid,如果没有，先换code，用户确认授权，提交到后端，后端返回用户信息，后端 换取openid,返回后，前端再订单
  
