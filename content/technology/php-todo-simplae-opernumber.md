---
title: "PHP利用stack(栈)写了个简单都计算器"
date: 2020-08-14T20:11:19+06:59
description: "本身想用go写个微信小程序都template Dom解析的 但是无奈能力有限,不过让我了解到类似html的dom解析无法通过简单的正则实现。而是需要stack(栈)"
categories: [php,算法]
---

废话少说先上代码才是好汉,其核心部分为将人们平常用的中缀表达式(运算符在中间)转换为后缀表达式。计算的时候只需先

依次读出符号再读出该符号前两个数字即可,以下一个简单的示列:代码写的很丑 请务嘲笑😊

```php
<?php
/**
 *
 * php 实现基本的计算器 支持+ - * / 以及优先运算符()
 * 仅支持(个位数)不支持小数点和多位数
 * 仅需要简单修改即可支持小数点和多位数(思路:scan中缀表达式的时候进行正则匹配数字)。
 */

const OADD = 1;
const OSUB = 1;
const OMUL = 2;
const ODIV = 2;

$oper_stack = array();
$number_stack = array();
/**
 转逆波兰表达式
*/
$sym = "1+(2+3)*4-8/4+3*(2+1)";
$sym_len = mb_strlen($sym);
for ($i = 0; $i < $sym_len; $i++) {
    $char = $sym[$i];
    if (ctype_digit($char)) {//数字
        array_push($number_stack, $char);
    } else {//操作符号
            if ((empty($oper_stack) || $char == "(") && $char != ')') {
                array_push($oper_stack, $char);
            } else if ($char != ")") {
                $pre=end($oper_stack);
                if($pre!="("){
                    $pc=count($oper_stack);
                    for ($e = 0; $e < $pc+1; $e++) {
                        if (empty($oper_stack) ) {
                            array_push($oper_stack, $char);
                            break;
                        }
                        if (OperToInt($char) > OperToint($pre)) {
                            array_push($oper_stack, $char);
                            break;
                        } else {
                            $pop = array_pop($oper_stack);
                            array_push($number_stack, $pop);
                        }
                    }
                }else{
                    array_push($oper_stack,$char);
                }
            } else if ($char == ')') {
                for ($k = 0; $k < count($oper_stack); $k++) {
                    $pop = array_pop($oper_stack);
                    if ($pop != "(") {
                        array_push($number_stack, $pop);
                    }
                }
            }
     }
}

function OperToInt($oper)
{
    if ($oper == "+" || $oper == "-") {
        $flag = 1;
    } elseif ($oper == "*" || $oper == "/") {
        $flag = 2;
    }
    return $flag;
}
/***
计算过程
***/

for ($i = 0; $i <= count($oper_stack); $i++) {

    $pop = array_pop($oper_stack);
    if (!empty($pop)) {
        array_push($number_stack, $pop);
    }
}
for ($i = 0; $i < count($number_stack) + 1; $i++) {
    $pop = $number_stack[$i];
    $res .= $pop . ' ';
}
$number_exp = preg_split("/[\s]+/", trim($res), -1, PREG_SPLIT_NO_EMPTY);
$res_stack = [];
echo  $res."\n\r";
for ($i = 0; $i < count($number_exp); $i++) {
    if (ctype_digit($number_exp[$i])) {
        array_push($res_stack, $number_exp[$i]);
    } else {
        $num1 = array_pop($res_stack);
        $num2 = array_pop($res_stack);
        if ($number_exp[$i] == "+") {
            $res = $num2 + $num1;
        }
        if ($number_exp[$i] == "-") {
            $res = $num2 - $num1;
        }
        if ($number_exp[$i] == "*") {
            $res = $num2 * $num1;
        }
        if ($number_exp[$i] == "/") {
            $res = $num2 / $num1;
        }
        array_push($res_stack, $res);
    }
}
echo sprintf("%s的结果为%d",$sym,$res_stack[0]);

```

网上盗了一张图:
![阿大博客](https://s1.ax1x.com/2020/08/23/d0BnjH.png)
