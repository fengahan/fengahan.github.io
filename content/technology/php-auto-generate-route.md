---
title: "还在手动添加路由进行rabc权限管理,那太out了"
date: 2020-08-24 21:36:46.178 +0800
description: "每次开发完某个模块添加权限很是繁琐,而且增加了出错的几率,目前看到还是有部分项目在
用十年前的(手动)添加方式..."
categories: [php]
---

后台rabc权限是很普遍的一个应该应用了,目前后台基础框架,一大堆也很少有人去从头开始实现后台基础项目的部署.

但是如果你找到到基础后台项目还没有实现
通过reflect实现路由到自动到添加和整合,就真的是太out了。。。

老规矩 闲话少叙,言归正传。比如我现在写了一个member Controller 内容如下

```php
<?php

Class Member{
	public function __construct(){}
	/**
    * @return false|string
    */
	public function index(){
		return $this->out(['A','B','C','D']);
	}
	/**
    * @return false|string
    */
	public function create(){
		return $this->out(['success']);
	}
	/**
    * @return false|string
    */
	public function update(){
		return $this->out(['success']);
	}
   /**
    * @return array
    */
    public static function view(){
        return [];
    }
    /**
    * @return array
    */
	protected function findModel(){
		return [];
	}
     /**
     * @return false|string
     */
	private function out(array $data){
		return json_encode($data);
	}
	public function __destruct() {
        return;
    }
}

```

>十年前的方式:

  1.登录后台 
  
  2.打开路由添加页面
  
  3.把{Controller}/{Method},规则名称添加到数据库
  
  4.关闭页面
  
  5.重复操作2-4(三个)步骤 n次
   
  6.同步数据库到正式服
    
>现在的方式:我们或许只需要一个命令就行了 下面是我们的新类

```php
<?php

Class Member{
	public function __construct(){
	    echo 'so easy';
	}
    /**
     * @router("列表","查看会员列表")
     * @return string
     */
	public function index(){
		return $this->out(['A','B','C','D']);
	}
    /**
     *描述可以没有
     * @router("创建")
     * @return string
     */
	public function create(){
		return $this->out(['success']);
	}
     /**
     *名字也可以没有
     * @router("","更新信息")
     * @return false|string
     */
	public function update(){
		return $this->out(['success']);
	}
    /**
     *static
     * @router("","预览信息")
     * @return array
     */
    public static function view(){
        return [];
    }
    /**
     * 该方法不会被放到路由中因为是受保护的
     * @router("获取模型","方法的描述")
     * @return array
     */
	protected function findModel(){
		return [];
	}
    /**
     * 该方法不会被放到路由中因为是私有的
     * @router("输出","方法的描述")
     * @param array $data
     * @return false|string
     */
	private function out(array $data){
		return json_encode($data);
	}
	/**
     * 该方法不会被放到路由中因为是析构函数
     * @router("输出","方法的描述")
     * @return false|string
     */
	public function __destruct() {
	   return;
	}
}

```

很容易的就能看出我们在注释里面添加了@router("AAA","ZZZ")的注释(不是注解,注解是能够提供元数据的)字样
其中 `AAA` 代表名称 `ZZZ` 代表描述。

>下面我们开始一系列骚操作
 
```php
<?php 
 
 //Member 当前类名
 $class = new \ReflectionClass('Member');
 $className=$class->getShortName();
 $methods=$class->getMethods();
 $arr=[];
 foreach($methods as &$method)
 {
     $ret=[];
     //过滤掉无法通过route 访问对方法类型
     if ( $method->isConstructor() || $method->isStatic() || $method->isDestructor()){
         continue;
     }
     if($method->isPublic())
     {
         $docConTent=$method->getDocComment();////获取注释
         preg_match('/\@router(?:\()(.*)(?:\))/i', $docConTent, $matches);// 匹配
         $methodName=$method->getName();
         if(isset($matches[1]) && !empty($matches[1])){
             $routeArg=preg_split("/[,]+/", trim($matches[1]), -1, PREG_SPLIT_NO_EMPTY);
             $ret['method']=$methodName;
             $ret['route_name']=$routeArg[0]??'';//路由名称对应@router的第一个参数
             $ret['route_description']=$routeArg[1]??'';//路由描述对应@router的第二个参数
             // your code ... 如插入数据库
             //通过更改@router('AA','ZZZ')添加更多的内容
             //如@router('A','BB','CCC')可以在某次更新的时候只将含有CCC的加入到数据库
             $arr[$className][]=$ret;
         }else{
            echo sprintf("Error:Class %s 的 %s方法 注释路由解析失败\r\n",$className,$methodName);
         }
 
     }
 }
 print_r($arr);

```

这里是的$arr一定是你特别想要的东东:

结果:

Array
(
    [Member] => Array
        (
        
            [0] => Array
                (
                    [method] => index
                    [route_name] => "列表"
                    [route_description] => "查看会员列表"
                )

            [1] => Array
                (
                    [method] => create
                    [route_name] => "创建"
                    [route_description] =>
                )

            [2] => Array
                (
                    [method] => update
                    [route_name] => ""
                    [route_description] => "更新信息"
                )

        )

)


#### 既然你看到这里类了,说明你还是对这个小技巧很感兴趣滴,偷偷告诉你😘,可以配置你的编辑器 添加输入自动完成@router功能哦(很简单的哦,具体步骤百度下就ok了)
