---
title: PHP单点登录
categories:
- 小练习
- PHP
tags:
- PHP
- 登录
src: //eatboooo.gitee.io/img/background/PHP_login_h.png
description: 使用redis+cookie完成登录状态的保持  #次标题
---
# PHP单点登录
## 结构概览
![项目组成](http://111.231.250.175/img/thumb/PHP_login_makeup.jpg)
### 连接数据库
首先建立数据库连接，用于登录校验以及注册添加。
> db_config.php
```php
header("Content-type: text/html; charset=utf-8");
$conn = new mysqli('eatboooo.cn','root','*****','develop_test');
if ($conn->connect_error) {
    echo '数据库连接失败！';
    exit(0);
}
```
### 连接redis
连接redis，用于储存md5随机产生的cookieId，用于多台服务器之前的登录校验，用于保持登录状态。
> re_config.php
```php
header("Content-type: text/html; charset=utf-8");
$redis = new Redis();
$redis->connect('eatboooo.cn', 6379); //连接Redis
$redis->auth('********'); //密码验证
$redis->select(1);//选择数据库1
```
### 登录实现
首先是简单的前端From表单。
> login.html
```html
<form action="../Interface/LoginInterface.php" method="post">
    <div>
        <p>用户名：</p>
        <input type="text" name="userId" placeholder="手机号/邮箱" />
        <p>密码：</p>
        <input type="password" name="userPassword" placeholder="密码可见" />
        <br />
        <button class="slig-btn">登录</button>
        <p>
            没有账号？请
            <a href="regist.html">注册</a>
            <a href="#" class="wj">忘记密码？</a>
        </p>
    </div>
</form>
```
前端的表单提交到后端验证,同时记得对明文密码进行加密处理。
> LoginInterface.php
```php
include('../config/db_config.php');
include('../config/re_config.php');
$userId = $_POST['userId'];
$userPassword = md5($_POST['userPassword']); //md5对明文密码进行加密
if ($userId == '') {
    echo '<script>alert("请输入用户名！");history.go(-1);</script>';
    exit(0);
}
if ($userPassword == '') {
    echo '<script>alert("请输入密码！");history.go(-1);</script>';
    exit(0);
}
$sql
    = "select * from user where userId = '$userId' and userPassword = '$userPassword'";
$result = $conn->query($sql);
$number = mysqli_num_rows($result);
$msg = $result->fetch_assoc(); //拿出用户信息
$cid = md5(
    uniqid(microtime(true), true)
); //随机生成cookieID 存储在redis和cookie中 用于保持登录状态
if ($number) {
    setcookie('user', $cid, time() + 3000);
//    array_push($msg, $cid); 数组追加cid操作
    $redis->setex($cid, 3600, json_encode($msg));
//    $redis->setex($cid, 3600, urldecode($msg));  //因为数组中有中文 所以使用urldecode方法把数组转json
    echo '<script>window.location="../web/index.html";</script>';
} else {
    echo '<script>alert("用户名或密码错误！");history.go(-1);</script>';
}
```
### 注册实现
编写前端简单的注册表单。
> regist.html
```html
<form action="../Interface/RegistInterface.php" method="post">
    <div class="psw">
        <p class="psw-p1">用户名</p>
        <input type="text" name="userId" placeholder="请输入用户名"/>
        <span class="dui"></span>
    </div>
    <div class="psw">
        <p class="psw-p1">输入密码</p>
        <input type="password" name="userPassword" placeholder="请输入密码"/>
        <span class="cuo">密码由6-16的字母、数字、符号组成</span>
    </div>
    <div class="psw psw2">
        <p class="psw-p1">姓名</p>
        <input type="text" name="userName" placeholder="请输入姓名"/>
    </div>
    <div class="psw psw2">
        <p class="psw-p1">简介</p>
        <input type="text" name="userDescript" placeholder="请输入简介"/>
    </div>

    <button type="submit" value="注册" class="psw-btn">立即注册</button>
    <p class="sign-in">
        已有账号？请
        <a href="login.html">登录</a>
    </p>
</form>
```
后端操作mysql实现注册,注意插入的数据类型是否与数据库中的表中的数据类型一致。
```php
include('../config/db_config.php');
$userName = $_POST['userName'];
$userPassword = md5($_POST['userPassword']); //md5对明文密码进行加密
$userDescript = $_POST['userDescript'];
$userId = $_POST['userId'];

$sql = "select userId from user where userId = '$_POST[userId]'";
$result = $conn->query($sql);
$number = mysqli_num_rows($result);
if ($number) {
    echo '<script>alert("用户名已经存在");history.go(-1);</script>';
} else {
    $sql_insert = "INSERT INTO user() VALUES('$userId','$userName','$userPassword','$userDescript')";
    $res_insert = $conn->query($sql_insert);
    if ($res_insert) {
        echo '<script>window.location="../web/login.html";</script>';
    } else {
        echo "<script>alert('系统繁忙，请稍候！');</script>";
    }
}
```
### 校验实现
拿到cookie中的cookieID，查看服务器上redis中是否存在。
> check.php
```php
include('../config/re_config.php');
if ($redis->get($_COOKIE["user"])) {
    $msg = json_decode($redis->get($_COOKIE["user"]));
    var_dump($msg);
} else {
    echo "您害妹登录";
}
```

