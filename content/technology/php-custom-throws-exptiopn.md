---
title: "关于php中自定义错误处理函数遇到的问题"
date: 2019-09-01T20:44:58+06:48
description: "这两天事情少,所以继续基于swoole扩展的web框架开发,在写自定义错误的处理方式遇到了一些问题特此总结"
categories: [swoole, php]
---
其实遇到问题的原因，是因为打算要用walker 提供的针对mysql 8小时因无数据传输而断开链接进行优化的mysql类，实现思路是在进行sql执行的时候 进行try catch ，然而在第一次执行的
时候 加入已经断开链接，就输出一条Warning 异常 就是 mysql gone away的错误，因为 文章是隔天写的，所以这里不再展示具体的错误信息,因为我之前自定义了错误处理方式,所以 这条
warning 就会阻止程序运行而直接给客户端返回500，这样就无法进入catch 于是我想用"@" 符号去 屏蔽该错误但是还是会输出，下面是之前的错误处理代码

```php
<?php
class testError{
   public function handleError($errno, $errstr, $errfile, $errline)
       {
           $this->context->sendCode(500);
           $this->context->send("Internals Server Error");
           $errName=$this->getErrorName($errno);
           $message= "$errName $errstr on line $errline in file $errfile".PHP_EOL;
           $err= "\033[0;33m  {$message}\e \033[0m".PHP_EOL;
           $this->renderError($err);
   
           return true;
       } 
}


```

下面是改进后的代码

```php
<?php
class testError{
     public function handleError($errno, $errstr, $errfile, $errline)
        {
            if (error_reporting() == 0) {//如果不加这个将会怎么呢怎么呢 @符号将无法抑制错误输出
                return true;
            }
            $this->context->sendCode(500);
            $this->context->send("Internals Server Error");
            $errName=$this->getErrorName($errno);
            $message= "$errName $errstr on line $errline in file $errfile";
            $err= "\033[0;33m  {$message}\e \033[0m".PHP_EOL;
            $this->renderError($err);
    
            return true;
        }
        
}
```  
没错就是这个

```php
<?php
//这段代码在http://php.net/manual/en/function.set-error-handler.php 下面的示例中可以找到搜索error_reporting() == 0
 if (error_reporting() == 0) {
            return true;
        }
```	
			



