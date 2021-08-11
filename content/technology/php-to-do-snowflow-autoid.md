---
title: "Twitter的分布式自增ID算法snowflake (PHP版)"
date: 2019-10-26T11:05:58+05:30
description: "PHP 实现Twitter的分布式自增ID算法snowflake生成唯一ID, 唯一Id 的使用场景非常多,实现方式各不相同，比如用redis的用数据库自增的 但是这两种效率都普遍低下特别是数据库自增 虽然可以为表设置自增步长 增加多台数据库，但是真的很浪费。redis 就不用说了,内存的东西还是不可靠的。下面这个算法来自twitter 自行研究"
categories: [算法, php]
---

### ！！！注意 ！注意！注意  重要的事情说三遍  在Php-fpm 的运行方式下并不安全 可以用在workman 和swoole 等常驻内存的应用程序上
```php 

<?php
/**
 * Created by PhpStorm.
 * User: hanfeng
 * Date: 2018/8/24
 * Time: 11:38
 */

class Snow{

    /** 开始时间截 (2015-01-01) */
    private $twepoch = 1420041600000;
    private $workerIdBits = 5;
    private $dataCenterIdBits=5;
    private $maxWorkerId;
    private $maxDataCenterId ;
    private $sequenceBits =12;
    private $workerIdShift;
    private $dataCenterIdShift;
    private $timestampLeftShift;
    private $sequenceMask;
    private $workerId;
    private $dataCenterId;
    private $sequence=0;
    private $lastTimestamp = -1;
    public function __construct($workerId,$dataCenterId)
    {

       if ($workerId>$this->maxWorkerId ||$workerId<0){
           throw new \Exception("workerId 有误");
       }
       if ($dataCenterId>$this->maxDataCenterId || $dataCenterId<0){
           throw new \Exception("dataCenterId 有误");
       }
       $this->workerId=$workerId;
       $this->dataCenterId=$dataCenterId;
       $this->maxWorkerId=-1 ^ (-1 << $this->workerIdBits);
       $this->workerIdShift=$this->sequenceBits;
       $this->dataCenterIdShift=$this->sequenceBits + $this->workerIdBits;
       $this->timestampLeftShift=$this->dataCenterIdShift+$this->dataCenterIdBits;
       $this->sequenceMask=-1 ^ (-1 << $this->sequenceBits);

    }

    public function nextId()
    {

        $timestamp=$this->getTimeMicro();

        //如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
        if ($timestamp < $this->lastTimestamp){
            throw new \Exception("时间有误。请重新调整系统时间");
        }

        if ($this->lastTimestamp == $timestamp) {
           $this->sequence = ($this->sequence + 1) & $this->sequenceMask;
            //毫秒内序列溢出
            if ($this->sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                $timestamp= $this->tilNextMillis($this->lastTimestamp);
            }
        }
        //时间戳改变，毫秒内序列重置
        else {
            $this->sequence = 0;
        }
        //上次生成ID的时间截
       $this->lastTimestamp = $timestamp;

        //移位并通过或运算拼到一起组成64位的ID
        return (($timestamp - $this->twepoch) << $this->timestampLeftShift) //
            | ($this->dataCenterId << $this->dataCenterIdShift) //
            | ($this->workerId << $this->workerIdShift) //
            | $this->sequence;
    }

    public function tilNextMillis($stamp){
        $timestamp = $this->getTimeMicro();
        while ($timestamp <= $stamp) {
            $timestamp =  $this->getTimeMicro();
        }
        return $timestamp;
    }
    /**
     * 获取毫秒
     */
    public function getTimeMicro()
    {
        list($mic,$at)=explode(" ",microtime());//
        return (float)sprintf('%.0f',(floatval($mic)+floatval($at))*1000);
    }

}
//5位dataCenterId和5位workerId
//最大十进制数字为31 最小为0
$s=new Snow(0,0);
for ($i=0;$i<1000000;$i++){
    $id= $s->nextId();
    $arr[$id]=$id;
}
echo count($arr);

```
