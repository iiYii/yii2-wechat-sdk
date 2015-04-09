yii2-wechat-sdk
===============

感谢选择 yii2-wechat-sdk 扩展, 该扩展是基于[Yii2](https://github.com/yiisoft/yii2)框架基础开发,借助Yii2的强劲特性可以定制开发属于您自己的微信公众号

说明
------------
本扩展是基于[callmez/yii2-wechat-sdk](https://github.com/callmez/yii2-wechat-sdk)，感谢callmez。，我在他的基础上做了简单的修改，添加了扫码支付，和刷卡支付，后期会添加JS支付。


环境条件
------------
- >= php5.4
- Yii2

安装
------------

您可以使用composer来安装, 添加下列代码在您的``composer.json``文件中并执行``composer update``操作

```json
{
    "require": {
       "iiyii/yii2-wechat-sdk": "dev-master"
    }
}
```

使用示例
------------
在使用前,请先参考微信公众平台的[开发文档](http://mp.weixin.qq.com/wiki/index.php?title=%E9%A6%96%E9%A1%B5)

Wechat定义方式
```php
//在config/web.php配置文件中定义component配置信息
'components' => [
  .....
  'wechat' => [
    'class' => 'iiyii\wechat\sdk\Wechat',
    'appId' => '微信公众平台中的appid',
    'appSecret' => '微信公众平台中的secret',
    'token' => '微信服务器对接您的服务器验证token'
  ]
  ....
]
// 全局公众号sdk使用
$wechat = Yii::$app->wechat; 


//多公众号使用方式
$wechat = Yii::createObject([
    'class' => 'iiyii\wechat\sdk\Wechat',
    'appId' => '微信公众平台中的appid',
    'appSecret' => '微信公众平台中的secret',
    'token' => '微信服务器对接您的服务器验证token'
]);
```

Wechat方法使用(部分示例)
```php
//获取access_token
var_dump($wechat->accessToken);

//获取二维码ticket
$qrcode = $wechat->createQrCode(123);
var_dump($qrcode);

//获取二维码
$imgRawData = $wechat->getQrCodeUrl($qrcode['ticket']);

//获取群组列表
var_dump($wechat->getGroups());


//创建分组
$group = $wechat->createGroup('测试分组');
echo $group ? '测试分组创建成功' : '测试分组创建失败';

//修改分组
echo $wechat->updateGroupName($group['id'], '修改测试分组') ? '修改测试分组成功' : '测试分组创建失败';


//根据关注者ID获取关注者所在分组ID
$openID = 'oiNHQjh-8k4DrQgY5H7xofx_ayfQ'; //此处应填写公众号关注者的唯一openId

//修改关注者所在分组
echo $wechat->updateMemberGroup($openID, 1) ? '修改关注者分组成功' : '修改关注者分组失败';

//获取关注者所在分组
echo $wechat->getGroupId($openID);

//修改关注者备注
echo $wechat->updateMemberRemark($openID, '测试更改备注') ? '关注者备注修改成功' : '关注者备注修改失败';

//获取关注者基本信息
var_dump($wechat->getMemberInfo($openID));

//获取关注者列表
var_dump($wechat->getMemberList());

//获取关注者的客服聊天记录, 
var_dump($wechat->getCustomerServiceRecords($openID, mktime(0, 0, 0, 1, 1, date('Y')), time())); //获取今年的聊天数据(可能获取不到数据)

//上传媒体文件
$filePath = '图片绝对路径'; //目前微信只开发jpg上传
var_dump($media = $wechat->uploadMedia(realpath($filePath), 'image'));

//下载媒体文件
echo $wechat->getMedia($media['media_id']) ? 'media下载成功' : 'media下载失败';

```

支付使用示例
```
//在config/web.php配置文件中定义component配置信息(这种方式可以使用微信所有的接口)
'components' => [
  .....
  'wechat' => [
    'class'     => 'iiyii\wechat\sdk\Wechat',
    'appId'     => '微信公众平台中的appid',
    'appSecret' => '微信公众平台中的secret',
    'token'     => '微信服务器对接您的服务器验证token',
    'mchid'     => '受理商ID',
    'key'       => '商户支付密钥Key',
  ]
  ....
]
// 全局公众号sdk使用
$wechat = Yii::$app->wechat; 


//多公众号使用方式(这种方式只使用微信支付)
$wechat = Yii::createObject([
    'class'     => 'iiyii\wechat\sdk\WechatPay',
    'appId'     => '微信公众平台中的appid',
    'appSecret' => '微信公众平台中的secret',
    'mchid'     => '受理商ID',
    'key'       => '商户支付密钥Key',
]);


// 扫码支付
$wechat->setParameter("body", "贡献一分钱"); //商品描述
//自定义订单号，此处仅作举例
$timeStamp = time();
$out_trade_no = "forecho" . "$timeStamp";
$wechat->setParameter("out_trade_no", "$out_trade_no"); //商户订单号
$wechat->setParameter("total_fee", "1"); //总金额 分为单位
$wechat->setParameter("notify_url", "http://www.xxx.com");//通知地址
$wechat->setParameter("trade_type", "NATIVE");//交易类型
var_dump($wechat->getpay());


// 刷卡支付
$wechat->setParameter("body", "贡献一分钱"); //商品描述
//自定义订单号，此处仅作举例
$timeStamp = time();
$out_trade_no = "forecho" . "$timeStamp";
$wechat->setParameter("out_trade_no", "$out_trade_no"); //商户订单号
$wechat->setParameter("total_fee", "1"); //总金额
//非必填参数，商户可根据实际情况选填
//$wechat->setParameter("sub_mch_id","XXXX");//子商户号
$wechat->setParameter("device_info", "013467007045764"); //设备号
$wechat->setParameter("auth_code", "130735422771146307"); //扫码支付授权码，设备读取用户微信中的条码或者二维码信息
//$wechat->setParameter("attach","XXXX");//附加数据
//$wechat->setParameter("time_start","XXXX");//交易起始时间
//$wechat->setParameter("time_expire","XXXX");//交易结束时间
//$wechat->setParameter("goods_tag","XXXX");//商品标记
//$wechat->setParameter("openid","XXXX");//用户标识
//$wechat->setParameter("product_id","XXXX");//商品ID
print_r($wechat->getMicropay());
```


反馈或贡献代码
------------
您可以在[这里](https://github.com/iiyii/yii2-wechat-sdk/issues)给我提出在使用中碰到的问题或Bug.
我会在第一时间回复您并修复.

您也可以 发送邮件caizhenghai@qq.com给我并且说明您的问题.

如果你有更好代码实现,请fork项目并发起您的pull request.我会及时处理. 感谢!
