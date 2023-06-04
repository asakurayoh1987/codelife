[原文地址](https://mp.weixin.qq.com/s/4p8ecpcKkK_Wh0nOWxRp7A)

调用config 接口的时候传入参数 debug: true 可以开启debug模式，页面会alert出错误信息。以下为常见错误及解决方法：

1. invalid url domain当前页面所在域名与使用的appid没有绑定，请确认正确填写绑定的域名，如果使用了端口号，则配置的绑定域名也要加上端口号（一个appid可以绑定三个有效域名，见 目录1.1.1）。

2. invalid signature签名错误。建议按如下顺序检查：

   - 确认签名算法正确，可用 http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign 页面工具进行校验。

   - 确认config中nonceStr（js中驼峰标准大写S）, timestamp与用以签名中的对应noncestr, timestamp一致。

   - 确认url是页面完整的url(请在当前页面alert(location.href.split('#')[0])确认)，包括'http(s)://'部分，以及'？'后面的GET参数部分,但不包括'#'hash后面的部分。

   - 确认 config 中的 appid 与用来获取 jsapi_ticket 的 appid 一致。

   - 确保一定缓存access_token和jsapi_ticket。

   - **确保你获取用来签名的url是动态获取的，动态页面可参见实例代码中php的实现方式。如果是html的静态页面在前端通过ajax将url传到后台签名，前端需要用js获取当前页面除去'#'hash部分的链接（可用location.href.split('#')[0]获取），因为页面一旦分享，微信客户端会在你的链接末尾加入其它参数，如果不是动态获取当前链接，将导致分享后的页面签名失败。**

3. the permission value is offline verifying这个错误是因为config没有正确执行，或者是调用的JSAPI没有传入config的jsApiList参数中。建议按如下顺序检查：
   - 确认config正确通过。
   - 如果是在页面加载好时就调用了JSAPI，则必须写在wx.ready的回调中。
   - 确认config的jsApiList参数包含了这个JSAPI。

4. permission denied该公众号没有权限使用这个JSAPI，或者是调用的JSAPI没有传入config的jsApiList参数中（部分接口需要认证之后才能使用）。
5. function not exist当前客户端版本不支持该接口，请升级到新版体验。
6. 为什么6.0.1版本config:ok，但是6.0.2版本之后不ok（因为6.0.2版本之前没有做权限验证，所以config都是ok，但这并不意味着你config中的签名是OK的，请在6.0.2检验是否生成正确的签名以保证config在高版本中也ok。）
7. Android用户已取消分享朋友圈，但仍返回分享成功（微信团队已修复此问题，已在Android6.1版本上线）
8. 在iOS和Android都无法分享（请确认公众号已经认证，只有认证的公众号才具有分享相关接口权限，如果确实已经认证，则要检查监听接口是否在wx.ready回调函数中触发）
9. Android6.0.2部分客户端无法使用监听分享接口（Android6.0.2之前以及6.0.2.58以后的版本都不会有问题，请从官网weixin.qq.com下载最新版本体验）
10. 服务上线之后无法获取jsapi_ticket，自己测试时没问题。（因为access_token和jsapi_ticket必须要在自己的服务器缓存，否则上线后会触发频率限制。请确保一定对token和ticket做缓存以减少2次服务器请求，不仅可以避免触发频率限制，还加快你们自己的服务速度。目前为了方便测试提供了1w的获取量，超过阀值后，服务将不再可用，**请确保在服务上线前一定全局缓存access_token和jsapi_ticket，两者有效期均为7200秒，否则一旦上线触发频率限制，服务将不再可用）**。
11. Android部分版本上传图片接口偶尔卡住（早期的Android6.0.2版本存在此问题，官方已修复，6.0.2.58之后的版本都支持，请从官网下载最新版本体验）
12. uploadImage怎么传多图（目前只支持一次上传一张，多张图片需等前一张图片上传之后再调用该接口）
13. 没法对本地选择的图片进行预览（chooseImage接口本身就支持预览，不需要额外支持）
14. 通过a链接(例如先通过微信授权登录)跳转到b链接，invalid signature签名失败（后台生成签名的链接为使用jssdk的当前链接，也就是跳转后的b链接，请不要用微信登录的授权链接进行签名计算，后台签名的url一定是使用jssdk的当前页面的完整url除去'#'部分）
15. 出现config:fail错误（这是由于传入的config参数不全导致，请确保传入正确的appId、timestamp、nonceStr、signature和需要使用的jsApiList）
16. 如何把jsapi上传到微信的多媒体资源下载到自己的服务器（请参见文档中uploadVoice和uploadImage接口的备注说明）
17. 在Android中通过其他设备上传的serverId，能下载成功但无法播放（微信团队已经确认此问题，已在Android6.1版本中修复）
18. Android通过jssdk上传到微信服务器，第三方再从微信下载到自己的服务器，会出现杂音（微信团队已经修复此问题，目前后台已优化上线）
19. **绑定父级域名，是否其子域名也是可用的**（是的，合法的子域名在绑定父域名之后是完全支持的）
20. 在Android中通过扫一扫打开页面，再调用scanQRCode并返回数据给第三方，会直接退出当前页（微信团队已经确认此问题，已在Android6.1版本中修复）
21. 在iOS微信6.1版本中，分享的图片外链不显示，只能显示公众号页面内链的图片或者微信服务器的图片（微信团队已经确认此问题，会在6.2中修复）
22. 是否需要对低版本自己做兼容（jssdk都是兼容低版本的，不需要第三方自己额外做更多工作，但有的接口是6.0.2新引入的，只有新版才可调用）
23. 该公众号支付签名无效，无法发起该笔交易（请确保你使用的jweixin.js是官方线上版本，不仅可以减少用户流量，还有可能对某些bug进行修复，拷贝到第三方服务器中使用，官方将不对其出现的任何问题提供保障，具体支付签名算法可参考 JSSDK微信支付一栏）
24. 之前微信jssdk语音接口上传的文件类型为speex，第三方在自己的网页无法直接播放（微信团队确认了该问题，并已在本周上线解决了此问题，第三方可以通过多媒体接口下载得到标准的amr音频文件）
25. 目前Android微信客户端不支持pushState的H5新特性，所以使用pushState来实现web app的页面会导致签名失败，此问题会在Android6.2中修复
