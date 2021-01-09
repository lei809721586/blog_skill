---
title: 阿里支付宝app支付接口开发（php版）
date: 2020-01-05 14:22:22
tags:
	- php
categories: php
---

```
最近本人刚做了手机APP支付宝支付功能，在此跟大家分享一下
```

#### 第一步：创建应用并获取APPID（应用唯一标识）

![image](https://img.qichuanqing.cn/img/ali/1.png)

![image](https://img.qichuanqing.cn/img/ali/2.png)


#### 第二步:签约功能

![image](https://img.qichuanqing.cn/img/ali/3.jpg)


#### 第三步:配置秘钥

1. 生成RSA密钥
2. 上传应用公钥
3. 配置公钥与私钥
4. 使用支付宝公钥验签 （这里是支付宝公钥，不是应用公钥）


传送门：[支付宝开放平台开发助手](https://docs.open.alipay.com/291/105971/)


#### 第四步:实战，引入SDK，封装支付接口


```
这里以tp5框架为例，简单易懂
```

1.将官方sdk引入到extend目录下

![image](https://img.qichuanqing.cn/img/ali/3.png)

2.设置命名空间


```
目录结构：

extend/AliCloud/AliPay/AopClient.php 


namespace AliCloud\AliPay;


然后在 extend/AliCloud/AliPay/request 目录下找到 AlipayTradeAppPayRequest.php

设置命名空间 ：namespace AliCloud\AliPay\Request;

```
3.设置公钥（否则回调签名不通过）

![image](https://img.qichuanqing.cn/img/ali/4.png)

4.封装公用下单接口

extend/AliCloud下新建AliPay.php


```
<?php
namespace AliCloud;

use AliCloud\AliPay\AopClient;

class AliPay
{
    public $client;

    public $config = [
        'notify_url' => '默认回调地址接口',
        'alipayrsaPublicKey' => '你的支付宝公钥',
];

    public function __construct()
    {
        //调用阿里官方的接口
        $this->client = new \AliCloud\AliPay\AopClient();
        $this->client->gatewayUrl = "https://openapi.alipay.com/gateway.do";
        $this->client->appId = "你的appid";
        $this->client->rsaPrivateKey = '你的私钥';
        $this->client->format = "json";
        $this->client->charset = "UTF-8";
        $this->client->signType = "RSA2";
        $this->client->alipayrsaPublicKey = '你的支付宝公钥';

    }

    /**
     * 创建APP订单
     *
     * @param [type] $trade_no
     * @param [type] $reason
     * @param [type] $openid
     * @param [type] $money
     * @param [type] $callback_url
     * @return void
     */
    public function CreateAppOrder(
        $trade_no, 
        $reason,
        $money,
        $callback_url = null
    ) :array
    {
        try{

            if ($callback_url == null){
                $this->config['notify_url'] = '你的回调地址接口';
            }else{
                $this->config['notify_url']  = $callback_url;
            }
            //调用阿里官方的app接口
            $request = new \AliCloud\AliPay\Request\AlipayTradeAppPayRequest();

            $bizcontent = [
                'body' => 'APP支付宝充值',
                'subject' => $reason,
                'out_trade_no' => $trade_no,
                'total_amount' => $money,
                'product_code' => 'QUICK_MSECURITY_PAY'
            ];

            $bizcontent = json_encode($bizcontent,true);
            $request->setNotifyUrl($this->config['notify_url']);
            $request->setBizContent($bizcontent);
            //$response = $aop->sdkExecute($request);
            $response = $this->client->sdkExecute($request);


            if (empty($response)){
                return Result(0,'创建订单失败');
            }

            return Result(200,'',[
                'result' => $response
            ]);
        } catch(\Exception $e) {
            //记录日志
            trace('错误码：' . $e->getCode() . '，错误信息：' . $e->getMessage(), 'info');
            return Result($e->getCode(), $e->getMessage());
        }

    }
}
```

#### 第五步:准备下单调用接口，测试

伪代码如下：


```
/**
     * 创建订单
     * @param [type] $openid
     * @param [type] $money
     * @param [type] $check_login
     * @param [type] $userid
     * @return void
     */
    public function create_order($openid = '', $money, $check_login = 1, $userid = null, $target = 'aliApp')
    {
        try {
            // 数据准备
            if ($check_login == 1) {
                if (IsLogin() == false) return Result(0, '请登录后再进行操作');

                if ($userid === null) {
                    $userid = $GLOBALS['user_id'];
                }
            } else {
                if ($userid === null || $userid === '') {
                    return Result(0, '未找到该用户');
                }
            }

            $money = intval($money);
            if ($money <= 0) return Result(0, '金额错误');

            if ($target == 'aliApp' ) {
                return logic('Bill')->CreateAppOrder($money, $userid, $target);
            }else{
                return logic('Bill')->CreateWeixinOrder($openid, $money, $userid,$target);
            }

        } catch (Exception $e) {
            return Result(0, '系统错误，请稍后再试');
        }
    }
```

业务层伪代码：


```
/**
     * 创建微信APP支付订单
     * @param int $money
     * @param $userid
     * @param $target
     * @return mixed
     */
    public function CreateAppOrder(int $money, $userid, $target)
    {

        switch ($target) {
            case 'app':
                $callback = '回调地址';
                break;
            case 'weiChat':
               $callback = '回调地址';
                break;
            case 'pay':
               $callback = '回调地址';
                break;
            default:
                return Result(0, '充值失败');
        }

        // $user 查询用户信息
        if ($user == null) {
            return Result(0, '社员不存在');
        }


        $trade_no = rand(000000, 999999) . date('YmdHisu') . rand(000000, 999999);

        if ($target == 'aliApp'){
            $response = (new \AliCloud\AiPay())->CreateAppOrder($trade_no, $reason, $money / 100, $callback);
        }else{
           //其他类型下单
        }

        // 如果出现敏感词 可能是因为姓名引起的 则省略姓名后重试
        if ($response['code'] == 0 && $response['error'] == '参数包含敏感词') {
            $realName = mb_substr($user['name'], 0, 1) . '*';
            $reason = "为{$realName}充值";
            // 重试创建订单
            if ($target == 'aliApp'){
                $response = (new \AliCloud\AiPay())->CreateAppOrder($trade_no, $reason, $money, $callback);
            }

        }

        // 下单成功 记录入库
        if ($response['code'] == 200) {
         
            // 判断充值类型
            if ($target == 'aliApp') {
                $order_reason = 'APP支付宝充';
            } else if ($target == 'weiChat') {
                $order_reason = '微信充值';
            } else if ($target == 'pay') {
                $order_reason = '其他类型充值';
            } 
            
            //写入Mysql订单表记录
            $insertCount = db('user_order')->insert([
                'orderId' => $trade_no,
                'memberId' => $userid,
                'payStats' => 0,
                'totalFeeE2' => $money,
                'transactionId' => $trade_no,
                'createTime' => time(),
                'updateTime' => 0,
                'reason' => $order_reason,
                'NowTotalamte2' => $user['totalAmtE2']
            ]);

            if ($insertCount == 0) {
                return Result(0, '创建订单失败');
            }

            $response['orderId'] = $trade_no;
        }

        return $response;
    }
```

ps:App下单返回给前端调用即成功（可以用阿里官方沙箱测试服务端是否正常下单可调起支付宝客户端）


```
alipay_sdk=alipay-sdk-php-easyalipay-20190926&app_id=您的appid&biz_content=%7B%22body%22%3A%22%5Cu6d4b%5Cu8bd5%5Cu6570%5Cu636e%22%2C%22subject%22%3A%22App%5Cu652f%5Cu4ed8%5Cu4e0b%5Cu5355%5Cu6d4b%5Cu8bd5%22%2C%22out_trade_no%22%3A%2220170125test01%22%2C%22timeout_express%22%3A%2230m%22%2C%22total_amount%22%3A%22100%22%2C%22product_code%22%3A%22QUICK_MSECURITY_PAY%22%7D&charset=UTF-8&format=json&method=alipay.trade.app.pay&notify_url=https%3A%2F%2F你的回调接口域名%2Fv1%2Fcharge%2Fcallback_app&sign_type=RSA2&timestamp=2019-12-25+16%3A21%3A37&version=1.0&sign=签名%2BpPhkn4KghvzJHQxdB6QVcELGntamAci1X5Nk5emb4o8%2FK6zDteWM%2FZo2HR%2FZuFIP%2BpfzcW5ONX1H14pk1%2BrvZ1Q5MC3CEg3VS7mdtmcjHrGgkPdnkmYui7CIUONXg%2FsC6jHxqeg24ceycMKF4V2k43Tsa%2BfmX2oZxzFLj2oMn3JPeaXS5IrcezX10kaTyXaknQsIUCKyCsVREszv6enObXPSxj2DABA6XuYuPUStvbScYoWqomKPExV7ZIEuP3miVMsGDttHQ1OhDrMj%2B9TfzRvlPSAnsI74E6zFAMHdwlwoK8ndOYW1m6CThpGnalYc3g%3D%3D

```

---

#### 第六步：回调


```
    /**
     * app支付宝充值回调
     * @return [type] [description]
     */
    public function callback_app($target = 'aliApp')
    {

        $printError = function ($msg = 'FAIL', $code = 500) {
            return response('<xml><return_code><![CDATA[FAIL]]></return_code><return_msg><![CDATA[' . $msg . ']]></return_msg></xml>', $code);
        };
        $printSuccess = function () {
            echo '<xml><return_code><![CDATA[SUCCESS]]></return_code><return_msg><![CDATA[OK]]></return_msg></xml>';
            return;
        };

        try {

            $parmas = input('post.');//tp的助手函数获取参数
            trace(json_encode($parmas,true), 'debug');

            if ($parmas['trade_status'] == 'TRADE_FINISHED' || $parmas['trade_status'] == 'TRADE_SUCCESS') {
                //1. 认证签名
                $AliPay =  new \AliCloud\AiPay();
                $AopClient =  new \AliCloud\AliPay\AopClient();
                $result = $AopClient->rsaCheckV1($parmas,$AliPay->config['alipayrsaPublicKey'],$parmas['sign_type']);


                if ($result == false) return $printError('签名错误');

                //2.检查订单是否已被处理
                $orderId = $parmas['out_trade_no'];
                $cacheKey = "kagsAli:order:$orderId";
                
                //写入redis防止重复回调
                if (cache($cacheKey) !== false) {
                    if (cache($cacheKey) == 2) {
                        return $printSuccess();
                    }else{
                        return $printError('订单正在处理中');
                    }
                }else{
                    // 标记订单为处理中
                    cache($cacheKey, 1, 600);
                }

                //3.查询mysql中的订单数据
                $db =  db('user_order');
                $order = $db->where('orderId', $orderId)->find();
                if (empty($order)) return $printError('订单不存在');
                if ($order['payStats'] != 0) {
                    return $printSuccess;
                }
                $userid = $order['memberId'];
                //查询用户信息
                $user = model("User")->where('id',$userid)->find();

                $db->where('id', $order['id'])->update([
                    'updateTime' => time(),
                    'payTime' => time(),
                    'payStats' => 1,
                    'AfterTotalamte2' => $user['totalAmtE2'],
                    'transactionId' => $parmas['trade_no']
                ]);

                //支付宝app下单充值金额为实际金额需要乘以100 　
                model('User')->where('id', $userid)->setInc('totalAmtE2', $parmas['total_amount'] * 100);
                echo 'success';
                cache($key, 2, 600);
                return $printSuccess();
            }else{
                return $printError('订单未支付成功');
            }
        }catch (\Throwable $th) {
            if (isset($cacheKey)) {
                cache::rm($cacheKey);
            }
            echo 'fail';
            return $printError($th->getMessage());
        }
    }
```

ok,完成！
