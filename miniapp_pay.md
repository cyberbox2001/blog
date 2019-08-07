# 小程序支付流程
一个完整的支付，一般情况下都是包含了下面三个主要的点

1. 支付(正常是支付平台提供的h5页面让用户操作，主要是输密码)
1. 通知(用户完成一笔支付了，支付平台要通知商家支付结果，商家收到结果后进行一些相应的处理)
2. 查询(与第二点有点反过来的意思，商家自己主动去支付平台查询支付的结果，然后根据结果做相应的处理)

## 小程序的实现
简单起见，在index.wxml中添加一个输入框和一个button，绑定一下相应的事件，输入框主要是用于输入订单号，按钮用于模拟提交一个订单并发起支付

```html
<!--index.wxml-->
<view class="container">
    <input type="text" bindinput="getOrderCode" style="border:1px solid #ccc;"  />
    <button bindtap="pay">立即支付</button>
</view>
```
然后在index.js中写上一小段代码，主要是处理上面按钮的点击事件

```javascript
Page({
    data: {
        txtOrderCode: ''
    },
    pay: function () {
        var ordercode = this.data.txtOrderCode;
        wx.login({
          success: function (res) {
            if (res.code) {
              wx.request({
                url: 'https://www.yourdomain.com/pay',//登录成功后，把code与ordercode提交到自己服务器
                data: {
                  code: res.code,//要去换取openid的登录凭证
                  ordercode: ordercode
                },
                method: 'GET',
                success: function (res) {
                  console.log(res.data)
                  wx.requestPayment({//自己服务器换取openid成功,返回预付订单信息
                    timeStamp: res.data.timeStamp,发起支付类
                    nonceStr: res.data.nonceStr,
                    package: res.data.package,
                    signType: 'MD5',
                    paySign: res.data.paySign,
                    success: function (res) {
                      // success
                      console.log(res);
                    },
                    fail: function (res) {
                      // fail
                      console.log(res);
                    },
                    complete: function (res) {
                      // complete
                      console.log(res);
                    }
                  })
                }
              })
            } else {
              console.log('获取用户登录态失败！' + res.errMsg)
            }
          }
        });
    },
    getOrderCode: function (event) {
        this.setData({
          txtOrderCode: event.detail.value
        });
    }
})
```

这里Catcher先通过wx.login这个API先取到了登录的凭证code，并把这个凭证code做为请求参数用wx.request这个API发起一个网络请求。

在这个网络请求处理后会返回小程序支付所需要的相关参数。拿到这些参数后，再调用wx.requestPayment这个支付API，***此时才算是真正的发起支付***。

至此，小程序这边的事已经做完了，接下来就是要去处理接口那边的事了，其实接口要做的就是返回小程序需要的几个参数。但是要拿到这几个参数还是需要做不少事情的。

```C#
///这个pay就是wx.request请求的action
public ActionResult Pay(string code, string ordercode)
{
    var paramter = new Parameters();
    paramter.out_trade_no = ordercode;

    //使用登录凭证 code 获取 session_key 和 openid
    var unifiedorderRes = GetOpenIdAndSessionKey(paramter.appid, paramter.secret, code);

    //反序列化session_key 和 openid成ChangeResponseEntity实体
    var tmp = JsonConvert.DeserializeObject<ChangeResponseEntity>(unifiedorderRes);

    //统一下单的url和参数
    var payUrl = "https://api.mch.weixin.qq.com/pay/unifiedorder";
    var param = GetUnifiedOrderParam(tmp.openid, paramter);
               
    //统一下单后拿到的xml结果
    var payResXML = Helper.DoPost(param, payUrl);
    var payRes = XDocument.Parse(payResXML);
    var root = payRes.Element("xml");
    
    //序列化相应参数返回给小程序
    var res = GetPayRequestParam(root, paramter.appid, paramter.key);
    return Json(res, JsonRequestBehavior.AllowGet);            
}
```

由于只是一个演示的过程，不想这些数据经常以字符串的形式频繁出现在代码中，所以把相关的参数全部都放到了一个名为Parameters的类中(放到配置文件中也是可以的)，除了订单号是从小程序传过来的，当然在实际中这是不合理的，毕竟像金额这些东西，不可能每次都是同一个！这点是要注意的。

Parameters类的定义
```C#
public class Parameters
{      
    public string appid { get { return "申请的appid"; } }
    public string mchid { get { return "申请的商户号"; } }
    public string nonce { get { return Helper.GetNoncestr(); } }
    public string notify_url { get { return "http://yourdomain.com/notifyurl"; } }
    public string body { get { return "testpay"; } }
    public string out_trade_no { get; set; }
    public string spbill_create_ip { get { return "IP地址"; } }
    public string total_fee { get { return "1"; } }
    public string trade_type { get { return "JSAPI"; } }
    public string key { get { return "在商家后台设置的密钥"; } }
    public string secret { get { return "在配置小程序时的密钥"; } }
}
```

首先是获取到登录凭证后发起的这个网络请求。这个网络请求是决定了这次支付能否成功的第一步！

下面要做的是用登录凭证去换我们要的openid。

```C#
/// <summary>
/// 取openid和session_key
/// </summary>
/// <param name="appid"></param>
/// <param name="secret"></param>
/// <param name="js_code"></param>
/// <returns></returns>
private string GetOpenIdAndSessionKey(string appid, string secret, string js_code)
{
    var url = string.Format("https://api.weixin.qq.com/sns/jscode2session?appid={0}&secret={1}&js_code={2}&grant_type=authorization_code"
        , appid,secret,js_code);
    var request = WebRequest.Create(url) as HttpWebRequest;

    var response = request.GetResponse();
    var respStream = response.GetResponseStream();

    var res = string.Empty;
    using (var reader = new StreamReader(respStream, Encoding.UTF8))
    {
        res = reader.ReadToEnd();
    }
    return res;
}
```

要换取openid，就要向微信提供的地址发起一个网络请求，并在URL带上appid,secret和凭证code这三个参数。

然后就可以拿到一个下面形式的json字符串
```C#
{
  "openid": "OPENID",
  "session_key": "SESSIONKEY"
}
```

拿到之后自然就是要对这个字符串进行json的反序列化，这里用到了json.net这个包。

根据时序图，下面要调用统一下单这个接口了。

上面的代码，在统一下单这一块，又分为下面几个步骤

1. 处理统一下单的参数(签名和组装xml)
1. 发起POST请求
1. 解析请求得到的结果

```C#
/// <summary>
/// 取统一下单的请求参数
/// </summary>
/// <param name="openid"></param>
/// <param name="param"></param>
/// <returns></returns>
private string GetUnifiedOrderParam(string openid, Parameters param)
{            
    //参与统一下单签名的参数，除最后的key外，已经按参数名ASCII码从小到大排序
    var unifiedorderSignParam = string.Format("appid={0}&body={1}&mch_id={2}&nonce_str={3}&notify_url={4}&openid={5}&out_trade_no={6}&spbill_create_ip={7}&total_fee={8}&trade_type={9}&key={10}"
        , param.appid, param.body, param.mchid, param.nonce, param.notify_url
        , openid, param.out_trade_no, param.spbill_create_ip, param.total_fee, param.trade_type, param.key);
    //MD5
    var unifiedorderSign = Helper.GetMD5(unifiedorderSignParam).ToUpper();
    //构造统一下单的请求参数
    
   return string.Format(@"<xml>
                                <appid>{0}</appid>                                              
                                <body>{1}</body>
                                <mch_id>{2}</mch_id>   
                                <nonce_str>{3}</nonce_str>
                                <notify_url>{4}</notify_url>
                                <openid>{5}</openid>
                                <out_trade_no>{6}</out_trade_no>
                                <spbill_create_ip>{7}</spbill_create_ip>
                                <total_fee>{8}</total_fee>
                                <trade_type>{9}</trade_type>
                                <sign>{10}</sign>
                               </xml>
                    ", param.appid, param.body, param.mchid, param.nonce, param.notify_url, openid
                     , param.out_trade_no, param.spbill_create_ip, param.total_fee, param.trade_type, unifiedorderSign);
    
}
```

## 这里要注意一点，由于我们的传的trade_type是JSAPI，所以这里必须是要加上openid进行处理的

然后就是解析统一下单返回的XML了，说是解析，其实也就是要拿到我们需要的数据罢了。这里最后会得到一个小程序支付API需要的参数实体

```C#
/// <summary>
/// 获取返回给小程序的支付参数
/// </summary>
/// <param name="root"></param>
/// <param name="appid"></param>
/// <param name="key"></param>
/// <returns></returns>
private PayRequesEntity GetPayRequestParam(XElement root,string appid,string key)
{              
    //当return_code 和result_code都为SUCCESS时才有我们要的prepay_id
    if (root.Element("return_code").Value == "SUCCESS" && root.Element("result_code").Value == "SUCCESS")
    {
        var package = "prepay_id=" + root.Element("prepay_id").Value;
        var nonceStr = Helper.GetNoncestr();
        var signType = "MD5";
        var timeStamp = Convert.ToInt64((DateTime.Now - new DateTime(1970, 1, 1)).TotalSeconds).ToString();

        var paySignParam = string.Format("appId={0}&nonceStr={1}&package={2}&signType={3}&timeStamp={4}&key={5}",
             appid, nonceStr, package, signType, timeStamp, key);

        var paySign = Helper.GetMD5(paySignParam).ToUpper();

        var payEntity = new PayRequesEntity
        {
            package = package,
            nonceStr = nonceStr,
            paySign = paySign,
            signType = signType,
            timeStamp = timeStamp
        };
        return payEntity;
    }

    return new PayRequesEntity();
}
```
## 支付参数实体对应的内容如下
```C#
/// <summary>
/// 小程序支付需要的参数
/// </summary>
public class PayRequesEntity
{
    /// <summary>
    /// 时间戳从1970年1月1日00:00:00至今的秒数,即当前的时间
    /// </summary>
    public string timeStamp { get; set; }

    /// <summary>
    /// 随机字符串，长度为32个字符以下。
    /// </summary>
    public string nonceStr { get; set; }

    /// <summary>
    /// 统一下单接口返回的 prepay_id 参数值
    /// </summary>
    public string package { get; set; }

    /// <summary>
    /// 签名算法
    /// </summary>
    public string signType { get; set; }

    /// <summary>
    /// 签名
    /// </summary>
    public string paySign { get; set; }
}
```

需要注意的是，这里的签名操作，一定是要配合appId，这也是Catcher在支付这一块踩的唯一的一个坑，所以提醒一下各位读者，希望能避开这个坑

## 通知的简单说明

前面也提到了，通知是用户支付成功后，微信的服务器会向我们统一下单指定的notify_url发起一个异步的回调。

下面用伪代码来表示这一过程
```C#
public ActionResult Notify()
{
    //1.获取微信通知的参数

    //2.更新订单的相关状态

    //3.返回一个xml格式的结果给微信服务器
    var res = @"<xml>
          <return_code><![CDATA[SUCCESS]]></return_code>
          <return_msg><![CDATA[OK]]></return_msg>
        </xml>";

    return Content(res);
}
```

这里需要注意的是要处理好微信重复通知的情况！

通知和查询本质上都是想知道订单是否支付成功了。

它们的区别是：通知是微信主动通知商家； 查询是商家主动向微信发起查询；

这两个动作的主体是不一样的。

当微信能正常发起推送并且商家接收这个推送的服务器又没有挂的时候，查询的作用是微乎其微的。

当然，不可避免的会出现，微信不能正常发起推送或者商家的服务器挂了，这个时候查询的作用就变得很重要了！！

这个时候我们就要建交起一个定时作业来专门处理这种情况了，可以选择Quartz.Net，Hangfire等！

这个作业的内容具体如下

```C#
public void QueryJob()
{ 
    //1.找到要查询的订单号

    //2.根据订单号和appId等内容向https://api.mch.weixin.qq.com/pay/orderquery这个地址发起网络请求

    //3.拿到微信返回的结果

    //4.根据结果进行相应的处理
}
```

至于多久执行一次这个作业，可能就要根据使用小程序进行购物的数量多不多来做一个大致的估计

1. 生成订单页---   订单类型（1、城市2、市场 3、付费）、金额、时间（按月）
2. 前端在订单页，wx.login，成功后得code，提交code 后端方法requestPay()
3. requestPay()内容:
- 使用登录凭证 code 获取 session_key 和 openid，成功取得openid，连接注册信息（用户ID）等，执行订单生成流程，订单类；
- 执行wx统一下单流程GetUnifiedOrderParam(tmp.openid, paramter) 统一下单类 paramter带有金额等信息***（就是微信方同不同意支付）***，成功返回统一下订单返回类 反相应参数返回给前端小程序，即OnPay完成
4. wx.requestPayment 前端利用上面参数***(有统一下单的返回)***，调起支付
  
5. 微信支付通知接口