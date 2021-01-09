---
title: redis键过期回调
date: 2019-12-21 21:38:33
tags:
	- redis
categories: redis
---

### redis键过期回调


##### 在 Redis 的 2.8.0 版本之后，推出了一个新的特性——键空间消息（Redis Keyspace Notifications），它配合 2.0.0 版本之后的 SUBSCRIBE 就能完成这个定时任务的操作了，定时的单位是秒。


###### 应用场景： 
在我们程序中经常会有需要定时执行的程序，比如：商品下单后半小时内不支付自动撤单等等。 
最简单粗暴的办法，就是写一个程序，让它定时执行，但是这样对服务器压力比较大，本篇将用Redis去实现类似功能。


1.为了节约cup资源，事件通知默认是没有开启的 notify-keyspace-events ""，首先需要修改配置文件redis.conf

```
 cd /usr/local/redis/etc
 vi redis.conf
```



<html>
 
 <div style="color:red">
   修改： notify-keyspace-events ""改为notify-keyspace-events "Ex"
 </div>    
 <br/>
</html>

如图所示：

![image](https://img.qichuanqing.cn/img/redis/redis.png)


配置详解:

![image](https://img.qichuanqing.cn/img/redis/redis2.png)


##### 配置完毕废话不多说上代码：

以下代码以tp5框架为例测试，redis开启订阅操作，等待接收消息，然后写入一条记录到Mysql中

<html>
 
 <div style="color:red">
  ps:此处__keyevent@0代表db0
 </div>    
 <br/>
</html>


```
<?php
namespace app\contrab\controller;

use \Firebase\JWT\JWT
use app\fund\model\SiteTask;

public function redisPsubscribe()
{
    
    $redis = devredis();
    $redis->setOption($redis::OPT_READ_TIMEOUT, -1);
    $redis->psubscribe(['__keyevent@0__:expired'], function (
        $redis,
        $pattern,
        $chan,
        $key
    ) {
       echo $key;
       return  model('SiteTask')->CreateTask('redis_psub',1,1);
    });

}
    
```

1. ssh方式连接服务器：ssh root@你的服务器地址 ，切换到你的web目录public下
2. 开启tp命令行监听：php index.php   contrab/apply/redisPsubscribe 
3. 设置过期key，查看是否回调




```
<?php

namespace app\index\controller;

use think\{Controller,Request};

class Index
{
    public function index()
    {
        $key = 'redis_key';
        //设置10秒后过期
        $result =  devredis()->Setex($key, 10,'redis');

        return Result(1,'设置成功',['result' => $result, 'time' => date('Y-m-d H:i:s')]);
    }
}    
```


![image](https://img.qichuanqing.cn/img/redis/11.png)

再来看看是否回调，mysql数据库是否写入了一条记录


![image](https://img.qichuanqing.cn/img/redis/12.png)

### 刚好10秒，同时控制台也打印出了redis的过期key

![image](https://img.qichuanqing.cn/img/redis/13.png)


##### 结论：显然是可以利用redis的key过期回调事件,在实际开发中经常会有需要定时执行的程序，比如：商品下单后半小时内不支付自动撤单等等！


<html>
 
 <div style="color:red">
  ps:最后补充一点，若在实际生产环境中，应在Linux上开启守护进行监听redis回调事件。
  tp框架为例：nohup  命令行 如：php index.php  模块/控制器/方法名  &
 </div>    
 <br/>
</html>