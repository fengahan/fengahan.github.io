---
title: "爬虫京东商品为mysql 生成数据"
date: 2019-07-18T12:31:58+05:30
description: "因为服务器要新增3台从数据库 如此大任，舍我其谁 哈哈，所以我 还是先在本地走一遍流程避免线上操做出现问题，为了尽量的模仿线上的情况，写了这个东东 一直插入数据"
categories: [php]
---

真心不容易啊 下班了自己的时间在本地做测试。
因为服务器要新增3台从数据库 如此大任，舍我其谁 哈哈，所以我 还是先在本地走一遍流程避免出现问题，为了尽量的模仿线上的情况，写了这个东东 一直插入数据

增加从数据库以及使用xtrabackup备份的文章以后再写吧 只是简单记录了笔记没整理 但是代码是现成的 大家拿去用吧

``` php

<?php
$dsn = 'mysql:dbname=ff_items;host=192.168.31.123;port=3306';
$username = 'root';
$password = '123456';
try {
    $db = new PDO($dsn, $username, $password); 
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

} catch(PDOException $e) {
    die("Could not connect to the database:\n" . $e);
}
$sql="INSERT INTO `ff_goods` (`name`, `url`, `price`, `img`, `created_at`) VALUES (?,?,?,?,?)";
$stm=$db->prepare($sql);
//
$sub=[12098,12097,12095,12472,12473,12111];
$url_preg="https://coll.jd.com/list.html?sub={{sub}}&page={{page}}&JL=6_0_0";

$c=count($sub);
for ($i=0;$i<$c;$i++){
    $subUrl=str_replace("{{sub}}",$sub[$i],$url_preg);

    for ($j=1;$j<100;$j++){
        $contentUrl=str_replace("{{page}}",$j,$subUrl);
        $content=get_content_by_curl($contentUrl);
        $arr=getContentVale($content);
        echo "insert：$contentUrl\n";
      for($k=0;$k<count($arr);$k++){
            $stm->bindParam(1,mb_substr($arr[$k]['name'],2));
            $stm->bindParam(2,$arr[$k]['url']);
            $stm->bindParam(3,$arr[$k]['price'],PDO::PARAM_INT);
            $stm->bindParam(4,$arr[$k]['img']);
            $stm->bindParam(5,time(),PDO::PARAM_INT);
            $stm->execute();
            $t=rand(0,2);//此处sleep 可调整
            sleep($t);
        }
    }
}

/**
 *获取指定url的数据
 * @param string $url  地址
 * @return string
 */
function get_content_by_curl($url){
   
    $curl = curl_init();
    curl_setopt($curl, CURLOPT_URL, $url);
    curl_setopt($curl, CURLOPT_HEADER, 1);
    curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1);
    $data = curl_exec($curl);
    curl_close($curl);
    return $data;
}

/**
 * @param string $content 爬到的内容
 * @return array
 */
function getContentVale($content){
    $goods_list=[];
    $rue="/<div\sclass=\"g\w+(.*)>[\s\S]+?<\/div>\s(.*)<\/li>/";
    $rue_name="/<em>\S(.*)<\/em>/";
    $rue_url="/<a target=\"_blank\" href=\"(.*?)\">/";
    $rue_price='';
    $rue_img='/data-lazy-img=(.*?)" alt/';
    $err=preg_match_all($rue,$content,$now_div);
    if($err !==false){
        $div=$now_div[0];
        foreach ($div as $key=>$val){
           preg_match($rue_name,$val,$names);
           preg_match($rue_img, $val, $imgs);
           preg_match($rue_url,$val,$urls);
					 $img=isset($imgs[1])?$imgs[1]:"/img12.360buyimg.com/";
           $goods["name"]= isset($names[1])?$names[1]:"未找到商品名称";
           $goods["price"]=rand(10,100);
           $goods["img"]=$img;
           $goods["url"]=$urls[1];
           $goods_list[]=$goods;
        }
    }
    return $goods_list;
}


```