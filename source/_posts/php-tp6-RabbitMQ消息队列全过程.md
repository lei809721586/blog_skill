---
title: php+tp6+RabbitMQ消息队列全过程
date: 2020-12-29 20:16:11
tags:
	- RabbitMQ
categories: RabbitMQ
---

## 前言：

这篇笔记记录了php连接rabbitmq实现消息的生产和消费的整个过程，包括声明持久的直连交换机，声明持久的队列，以及生产持久的消息，消费者处理消息等

#### 1.众所周知所谓消息队列肯定有生产者与消费者

初始化时候，还没有任何通道，交换机，队列 ，此处名称为lwf_queue_1 是我测试的demo，请忽略

![image](http://img.mymealwell.cn/demo/mq/231.png)

1.1 先上消费者代码片段

path：app\command\rabbitmq\Work.php
****

Work.php

```
<?php
declare (strict_types = 1);

namespace app\command\rabbitmq;

use think\console\{Command,Input,Output};
use think\console\input\{Argument,Option};

class Work extends Command
{

    /**
     * rabbitMq配置
     * @var string[]
     */
    protected $config = [];

    /**
     * rabbmitMq连接实例
     * @var \AMQPConnection
     */
    private $rabbitMqConnection;

    /**
     * 设置交换机类型
     * @var array
    */
    private $amqpExType = [
        'direct'  =>  AMQP_EX_TYPE_DIRECT, //直连交换机
        'fanout'  =>  AMQP_EX_TYPE_FANOUT, //扇形交换机
        'headers' => AMQP_EX_TYPE_HEADERS, //头交换机
        'topic'   => AMQP_EX_TYPE_TOPIC //主题交换机
    ];

    /**
     * @var int
     */
    protected $durable = AMQP_DURABLE;

    /**
     * 在连接内创建一个通道
     * @return \AMQPChannel
     * @throws \AMQPConnectionException
     */
    protected function connectionAmqpCHannel()
    {
        $ch = new \AMQPChannel($this->rabbitMqConnection );
        return $ch;
    }

    /**
     * 在通道中创建一个交换机
     * @param $channel
     * @return \AMQPExchange
     * @throws \AMQPConnectionException
     * @throws \AMQPExchangeException
     */
    protected function connectionAmqpExchange($channel)
    {
        $exchange = new \AMQPExchange($channel);
        return $exchange;
    }

    /**
     * command连接初始化
     * return void
     */
    protected function configure()
    {
//        $config =  [
//        'host' => '你的host地址（如136.144.12.11）',
//        'vhost' => '/',
//        'port' => '5672',
//        'login' => '你的账户',
//        'password' => '你的密码'
//        ]
        $this->config = config('app.mq'); //mq配置
        $this->init();
    }

    /**
     * rabbitmq连接初始化
     * return void
     */
    protected function init()
    {
        $this->rabbitMqConnection = new \AMQPConnection($this->config);
        if (!$this->rabbitMqConnection->connect()) {
            echo "Cannot connect to the broker";
            exit();
        }
        //在连接内创建一个通道
        $channel = $this->connectionAmqpCHannel();
        //创建一个交换机
        $exchange = $this->connectionAmqpExchange($channel);

        //声明路由键
        $routingKey = 'lwf_key_2';
        //声明交换机名称
        $exchangeName = 'lwf_exchange_2';
        //设置交换机名称
        $exchange->setName($exchangeName);
        //交换机类型
        $exchange->setType($this->amqpExType['direct']);
        //设置交换机持久
        $exchange->setFlags($this->durable);
        //声明交换机
        $exchange->declareExchange();

        return $this->setAmqpQueue($channel,$exchange,$routingKey);
    }

    /**
     * 创建消息队列
     * return void
     */
    protected function setAmqpQueue($channel,$exchange,$routingKey,$queueName = 'lwf_queue_2')
    {
        try {
            $queue = new \AMQPQueue($channel);
            $queue->setName($queueName);
            //设置队列持久
            $queue->setFlags($this->durable);
            //声明消息队列
            $queue->declareQueue();
            //交换机和队列通过$routingKey进行绑定
            $queue->bind($exchange->getName(), $routingKey);
            $queue->consume(function ($envelope, $queue){
                //休眠两秒，
                sleep(2);
                //echo消息内容
                echo $envelope->getBody()."\n";
                //显式确认，队列收到消费者显式确认后，会删除该消息
                $queue->ack($envelope->getDeliveryTag());
            });
            return Result(200,'success');
        }catch (\Exception $e){
            return 'create queue error';
        }
    }
}

```


1.2 生产者代码片段

path：app\job\controller\Job.php
****

Job.php


```
<?php
declare (strict_types = 1);

namespace app\job\controller;

use app\BaseController;
use app\job\service\JobService;

class Job extends BaseController
{
    /**
     * @var Service
     */
    private $service;

    /**
     * rabbmitMq连接实例
     * @var \AMQPConnection
     */
    private $rabbitMqConnection;

    /**
     * rabbitMq配置
     * @var string[]
     */
    protected $config = [];

    /**
     * @var string
     */
    protected $amqp_ex_type = 'direct';


    /**
     * @var int
     */
    protected $durable = AMQP_DURABLE;

    public function __construct(?JobService $jobService)
    {
        parent::__construct(app('app'));
        $this->service = $jobService;
        $this->config = config('app.mq');
        $this->rabbitMqConnection = new \AMQPConnection($this->config);
    }



    /**
     * 在连接内创建一个通道
     * @return \AMQPChannel
     * @throws \AMQPConnectionException
     */
    protected function connectionAmqpCHannel()
    {
        $ch = new \AMQPChannel($this->rabbitMqConnection );
        return $ch;
    }

    /**
     * 在通道中创建一个交换机
     * @param $channel
     * @return \AMQPExchange
     * @throws \AMQPConnectionException
     * @throws \AMQPExchangeException
     */
    protected function connectionAmqpExchange($channel)
    {
        $exchange = new \AMQPExchange($channel);
        return $exchange;
    }

    /**
     * job push
     * @return mixed
     */
    public function push()
    {
        $this->rabbitMqConnection = new \AMQPConnection($this->config);
        if (!$this->rabbitMqConnection->connect()) {
            return Result(0,'Cannot connect to the broker!');
        }

        try {
            $channel = $this->connectionAmqpCHannel();
            $exchange = $this->connectionAmqpExchange($channel);
        }catch (\Exception $e){
            return Result(0,$e->getMessage());
        }

        //消息的路由键，一定要和消费者端一致
        $routingKey = 'lwf_key_2';
        //交换机名称，一定要和消费者端一致，
        $exchangeName = 'lwf_exchange_2';
        $exchange->setName($exchangeName);
        $exchange->setType($this->amqp_ex_type);
        $exchange->setFlags($this->durable);
        $exchange->declareExchange();
        //创建10个消息
        for ($i=1;$i<=10;$i++){
            //消息内容
            $msg = array(
                'data'  => 'message_'.$i,
                'hello' => 'world',
                'name' => 'mr.lei_'.$i
            );
            //发送消息到交换机，并返回发送结果
            //delivery_mode:2声明消息持久，持久的队列+持久的消息在RabbitMQ重启后才不会丢失
            echo "Send Message:".$exchange->publish(json_encode($msg), $routingKey, AMQP_NOPARAM, array('delivery_mode' => 2))."\n";
            //代码执行完毕后进程会自动退出
        }
        return Result(200,'启动消费者队列',[
            'data' => $msg
        ]);
    }
}

```

ok，以上生产与消费者代码已编码完，后面可以测试消息队列，让我们来启动2个命令行/终端查看

#### 2.验证消息队列结果

开启2个终端：

2.1 第一个终端
![image](http://img.mymealwell.cn/demo/queue/a-1.png)


2.1 第一个终端
![image](http://img.mymealwell.cn/demo/queue/a-2.png)


此时rabbitmq web管理页面已经出现了新的通道连接（如下图）

![image](http://img.mymealwell.cn/demo/queue/a-3.png)



![image](http://img.mymealwell.cn/demo/queue/WechatIMG1384.png)

![image](http://img.mymealwell.cn/demo/queue/a-4.png)

产生了新的消息队列 如（lwf_queue_2）

![image](http://img.mymealwell.cn/demo/queue/a-5.png)



好的，我们开始生产，利用postman 调用生产者接口  job

![image](http://img.mymealwell.cn/demo/queue/a-6.png)



此时查看我们2个终端的显示

![image](http://img.mymealwell.cn/demo/queue/a-7.png)

![image](http://img.mymealwell.cn/demo/queue/a-8.png)


end
