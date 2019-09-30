---
layout: post
title: 微支付开发笔记
tags: ['微信']
---
##概述##
越来越多的平台开始接入微信，通过微信这个平台，我们可以快速的接入移动互联网。
购物平台、人工收银系统、等，一个很关键的功能就是支付了，微信开发平台为我们提供一套支付系统。
微信支付平台分为两个版本：

    1:v2 (2014年9月10号之前申请的为v2版)
    2:v3 (之后申请的为v3版)

V3版的微信支付没有paySignKey参数

V2版中的参数有
AppID
AppSecret
支付专用签名串PaySignKey
商户号PartnerID
初始密钥PartnerKey

并且包含一个证书文件: 安全证书

V3版中的参数有
AppID
AppSecret
商户号PartnerID
初始密钥PartnerKey
商户号MCHID
申请编号
商户平台登录帐号
商户平台登录密码

包含5个证书文件（证书pkcs12格式、证书pem格式、证书密钥pem格式、CA证书， 安全证书）

##支付模式：

1.被扫支付
　　被扫支付是用户展示微信上“我的刷卡条码/二维码”给商户系统扫描后直接完成支付的模式。主要应用线下面对面收银的场景。

2.扫码支付
　　扫码支付是商户系统按微信支付协议生成支付二维码，用户再用微信“扫一扫”完成支付的模式。该模式适用于PC网站支付、实体店单品或订单支付、媒体广告支付等场景。

3.微信内网页支付
　　微信内网页支付是用户在微信中打开商户的H5页面，商户在H5页面通过调用微信支付提供的JSAPI接口调起微信支付模块完成支付。应用场景有：
　　　◆　用户在微信公众账号内进入商家公众号，打开某个主页面，完成支付
　　　◆　用户的好友在朋友圈、聊天窗口等分享商家页面连接，用户点击链接打开商家页面，完成支付
　　　◆　将商户页面转换成二维码，用户扫描二维码后在微信浏览器中打开页面后完成支付

4.APP支付
　　APP支付又称移动端支付，是商户通过在移动端应用APP中集成开放SDK调起微信支付模块完成支付的模式。

5.普通浏览器网页支付模式

phper可能以后会遇到的就是 网页支付(公众号支付)

网页支付，通过js 调用微信为我们封装好的支付组件（WeixinJSBridge）

	function onBridgeReady(){
	   WeixinJSBridge.invoke(
	       'getBrandWCPayRequest', {
	           "appId" : "wx2421b1c4370ec43b",     //公众号名称，由商户传入    
	           "timeStamp":" 1395712654",         //时间戳，自1970年以来的秒数    
	           "nonceStr" : "e61463f8efa94090b1f366cccfbbb444", //随机串    
	           "package" : "prepay_id=u802345jgfjsdfgsdg888",    
	           "signType" : "MD5",         //微信签名方式:    
	           "paySign" : "70EA570631E4BB79628FBCA90534C63FF7FADD89" //微信签名
	       },
	       function(res){    
	           if(res.err_msg == "get_brand_wcpay_request:ok" ) {}     // 使用以上方式判断前端返回,微信团队郑重提示：res.err_msg将在用户支付成功后返回    ok，但并不保证它绝对可靠。
	       }
	   );
	}//调启函数
	if (typeof WeixinJSBridge == "undefined"){如果不存在这个微信对象api
	   if( document.addEventListener ){//开始注册这个对象
	       document.addEventListener('WeixinJSBridgeReady', onBridgeReady, false);
	   }else if (document.attachEvent){
	       document.attachEvent('WeixinJSBridgeReady', onBridgeReady);
	       document.attachEvent('onWeixinJSBridgeReady', onBridgeReady);
	   }
	}else{
	   onBridgeReady();
	}


所以调起支付只需要我们组建好我们要传递给getBrandWCPayRequest对象的参数
一般我们可以在视图里用一个变量，表示我们组合好的参数,如

	WeixinJSBridge.invoke({'getBrandWCPayRequest',{$parameter}});

我们在在控制器里组合这些参数

	 header("Expires : 0");
	 header("Cache-Control : no-store,private, post-check=0, pre-check=0, max-age=0");
	 header("Pragma : no-cache");
	 //微信环境判断
	 $agent = $_SERVER['HTTP_USER_AGENT'];
	 include_once(LIB_PATH."/Class/WxPay/WxPayPubHelper.php");
	 $jsApi = new JsApi_pub();
	 if (!isset($_GET['code']))
	 {
	     //触发微信返回code码
	     $url = $jsApi->createOauthUrlForCode(WxPayConf_pub::JS_API_CALL_URL);//注意带上自己需要回调的参数
	     Header("Location: $url");
	 }else
	 {
	     //获取code码，以获取openid
	     $code = $_GET['code'];
	     $jsApi->setCode($code);
	     $openid = $jsApi->getOpenId();
	 }
	 
	 //=========步骤2：使用统一支付接口，获取prepay_id============
	 //使用统一支付接口
	 $unifiedOrder = new UnifiedOrder_pub();
	  
	 //设置统一支付接口参数
	 //设置必填参数
	 //appid已填,商户无需重复填写
	 //mch_id已填,商户无需重复填写
	 //noncestr已填,商户无需重复填写
	 //spbill_create_ip已填,商户无需重复填写
	 //sign已填,商户无需重复填写
	 $unifiedOrder->setParameter("openid","$openid");//商品描述
	 $unifiedOrder->setParameter("body","微信安全支付");//商品描述
	 //自定义订单号，此处仅作举例
	 $timeStamp = time();
	 //$out_trade_no = WxPayConf_pub::APPID."$timeStamp";
	 $out_trade_no = $orderInfo['sn'];
	 $payamount = $orderInfo['payamount'] * 100;
	 //$payamount = 1;//测试用金额
	 $unifiedOrder->setParameter("out_trade_no","$out_trade_no");//商户订单号
	 //$unifiedOrder->setParameter("total_fee",$orderInfop['payamount']);//总金额
	 $unifiedOrder->setParameter("total_fee","$payamount");//总金额
	 $unifiedOrder->setParameter("notify_url",WxPayConf_pub::NOTIFY_URL);//通知地址
	 $unifiedOrder->setParameter("trade_type","JSAPI");//交易类型
	 //非必填参数，商户可根据实际情况选填
	 //$unifiedOrder->setParameter("sub_mch_id","XXXX");//子商户号 
	 //$unifiedOrder->setParameter("device_info","XXXX");//设备号
	 //$unifiedOrder->setParameter("attach","XXXX");//附加数据
	 //$unifiedOrder->setParameter("time_start","XXXX");//交易起始时间
	 //$unifiedOrder->setParameter("time_expire","XXXX");//交易结束时间
	 //$unifiedOrder->setParameter("goods_tag","XXXX");//商品标记
	 //$unifiedOrder->setParameter("openid","XXXX");//用户标识
	 //$unifiedOrder->setParameter("product_id","XXXX");//商品ID
	 
	 $prepay_id = $unifiedOrder->getPrepayId();
	 //=========步骤3：使用jsapi调起支付============
	 $jsApi->setPrepayId($prepay_id);
	 
	 $jsApiParameters = $jsApi->getParameters();
	 $this->assign('parameter',$jsApiParameters);
	//加载视图
	
	
大致流程为：

- 获取用户openid，通过接口OAuth2.0形式获取
获取code，如果code为空跳转获取，通过code像微信服务器换取accesstoken，在通过accesstoken换取用户信息
我们通过微信的sdk可以很简单的做到这点
pic
- 设置商品基本信息，生成package参数
- 通过支付密钥加密生成签名paySign
生成paySign过程：
拼接所以需要传递的字段
在最后加入key=商家密钥进行md5加密
假设传送的参数如下：
appid： wxd930ea5d5a258f4f
mch_id： 10000100
device_info： 1000
body： test
nonce_str： ibuaiVcKdpRxkhJA

第一步：对参数按照key=value的格式，并按照参数名ASCII字典序排序如下：
stringA="appid=wxd930ea5d5a258f4f&body=test&device_info=1000&mch_id=10000100&nonce_str=ibuaiVcKdpRxkhJA";
第二步：拼接API密钥：
stringSignTemp="stringA&key=192006250b4c09247ec02edce69f6a2d"
sign=MD5(stringSignTemp).toUpperCase()="9A0A8659F005D6984697E2CA0A9CF3B7"
最终得到最终发送的数据：

wxd930ea5d5a258f4f
10000100
1000
test
ibuaiVcKdpRxkhJA
9A0A8659F005D6984697E2CA0A9CF3B7

至此发起支付的工作基本就ok了

其次还有一个重要的步骤就是 对支付结果的处理
我们在发起支付时 会给微信服务器传递一个通知接口，在支持成功后微信服务会将这个接口post支付通知



支付回调:

	public function payNotify(){
		include_once(LIB_PATH."/Class/WxPay/WxPayPubHelper.php");
		$notify = new Notify_pub();
		//存储微信的回调
		$xml = $GLOBALS['HTTP_RAW_POST_DATA']; 
		$notify->saveData($xml);
		$data = $notify->getData();
		if($notify->checkSign() == FALSE){
		    $notify->setReturnParameter("return_code","FAIL");//返回状态码
		    $notify->setReturnParameter("return_msg","签名失败");//返回信息
		    Log::write('签名验证失败');
		}else{
		    //支付成功业务处理
		
		
		    $notify->setReturnParameter("return_code","SUCCESS");//设置返回码
		}
		$returnXml = $notify->returnXml();
		echo $returnXml;
	}


 