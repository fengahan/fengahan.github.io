---
title: "如何判断字符串ABCDE中是否 '单独' 存在BCD"
date: 2021-08-10T20:44:58+06:48
description: "具体的业务场景不便透露。如何从ABCDE中判断是否存在BCD 字串 如A BCD E 为存在,A BCDE为不存在"
categories: [php,算法]
draft: false
---
需求:如何从ABCDE中判断是否存在BCD为单独存在

>分析

存在:
1. A BCD E 
2. AE BCD
3. BCD AE
4. BCD
5. 空格+BCD

不存在:
1. ABC DE
2. ABCDE
3. A BCED
4. 更多。。。。

经上述分析可以得到  BCD 前后仅能同时存在一个空格或者无空格。这种需求最简单最直接最有效的方式就是正则匹配
它更容易让人理解。下面是同事写的正则匹配。

```php

<? php
$str="ABCDE";
if(preg_match('/^(BCD)$|^((BCD)\s+)|(\s+(BCD)\s+)|\s+(BCD)$/',$str)){
   echo "单独存在BCD"
}else{
    echo "XXXXXX";
}

```

如果仅仅满足业务以上代码足以,但是我没有用正则的习惯，所以当我遇到问题的时候对正则的方案也没有
多加考虑,便花了点时间写了个字符串处理版本,在看下面的代码的之前 我建议呢 先熟悉下php自带的几个
非常常用的函数 它们非常有用而且高效。请多了解了解它们，也比较建议了解各个函数的实现方法,你一定
大有收获。

1. stripos
2. stristr
3. str_replace
4. mb_strlen

上面的函数你可能用过一些,但是呢也可能你所用到的只是他们用法的一部分 比如str_replace 就有两种
用法。同时在手册上你查看某个函数 它还会推荐给你 一些相关或相似的函数，所以多看手册是一个php开发
的基本素养0_0

废话不多说 show me code

=======================

```php
<?php

function find($chars,$value)

{
    $_chars=$chars;//字符串 的子串
    $number=mb_strlen($chars);//字符串的总长度
    $value_count=mb_strlen($value);//当前验证的字符长度
    while (mb_strlen($_chars)>$value_count && $number>0){
        $number--;//避免特殊情况或者算法bug 造成无限循环
        $pos=stripos($_chars,$value);
        if ($pos !==false){
            $pre=isset($_chars[$pos-1])?$_chars[$pos-1]:'';
            $end=isset($_chars[$pos+$value_count])?$_chars[$pos+$value_count]:'';
            $pre=str_replace(" ",'',$pre);
            $end=str_replace(" ",'',$end);
            if (empty($pre) && empty($end)){
                $res=false;
                break;
            }
        }
        $_chars=stristr($_chars ,$value);
        $_chars=mb_substr($_chars,0-(mb_strlen($_chars)-$value_count));
    }
    if ($res==false){
        break;
    }

}
var_dump(find(ABCDE,BCD));//output:true
?>
```
下面这张图是两个方法运行100万次的时间对比:仅供娱乐。


![ftrJdx.png](https://z3.ax1x.com/2021/08/10/ftrJdx.png)


两者时间相差还是可观的,但是在实际的业务场景中我还是推荐正则，至于为什么。在文章开头已经说过了，不再重复,
两者在单次执行的时间差不大 没必要搞一个字符串处理 搞得其他同学一时半会看不懂 `正则的处理方式 会受到字符串长度的影响 比如
ABCDEBCCCESD BCD 和 A BCD`的时间是不一样的。而字符串处理的方式 则基本不会受影响 (其实影响非常小而已)

`最后送上go 圣经中一句经典语录:`


> 毫无疑问，对效率的片面追求会导致各种滥用。程序员会浪费大量的时间在非关键程序的速度上，实际上这些尝试提升效率的行为反倒可能产生很大的负面影响，特别是当调试和维护的时候。我们不应该过度纠结于细节的优化
，应该说约97%的场景：过早的优化是万恶之源。当然我们也不应该放弃对那关键3%的优化。一个好的程序员不会因为这个比例小就裹足不前，他们会明智地观察和识别哪些是关键的代码；
但是仅当关键代码已经被确认的前提下才会进行优化。对于很多程序员来说，判断哪部分是关键的性能瓶颈，是很容易犯经验上的错误的
，因此一般应该借助测量工具来证明.


  
