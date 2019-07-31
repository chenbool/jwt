[![Build Status](https://travis-ci.org/firebase/php-jwt.png?branch=master)](https://travis-ci.org/firebase/php-jwt)
[![Latest Stable Version](https://poser.pugx.org/firebase/php-jwt/v/stable)](https://packagist.org/packages/firebase/php-jwt)
[![Total Downloads](https://poser.pugx.org/firebase/php-jwt/downloads)](https://packagist.org/packages/firebase/php-jwt)
[![License](https://poser.pugx.org/firebase/php-jwt/license)](https://packagist.org/packages/firebase/php-jwt)

PHP-JWT
=======
>
一个简单的 PHP JSON Web Tokens (JWT) 加密/解密插件 [RFC 7519](https://tools.ietf.org/html/rfc7519).

## 一：JWT介绍：
    全称JSON Web Token,基于JSON的开放标准(RFC 7519),以token的方式代替传统的Cookie-Session模式,用于各服务器、客户端传递信息签名验证

## 二：JWT优点：
    1：服务端不需要保存传统会话信息，没有跨域传输问题，减小服务器开销。
    2：jwt构成简单，占用很少的字节，便于传输。
    3：json格式通用，不同语言之间都可以使用。
--------------------- 



安装
------------

使用 composer 管理依赖并下载:

```bash
composer require chenbool/jwt
```

案例
-------
```php
<?php
use \chenbool\JWT\JWT;

$key = "example_key";
$token = array(
    "iss" => "http://example.org", //签发者 可选
    "aud" => "http://example.com", //接收该JWT的一方 可选
    "iat" => 1356999524, //签发时间
    "nbf" => time()+30, //(Not Before):某个时间点后才能访问,比如设置time()+30,表示当前时间30秒后才能使用
    "exp" => time()=7200,  //过期时间,这里设置2个小时
);

/**
 * 加密
 * https://tools.ietf.org/html/draft-ietf-jose-json-web-algorithms-40
 */
$jwt = JWT::encode($token, $key);
$decoded = JWT::decode($jwt, $key, array('HS256'));
$decoded_array = (array) $decoded;
print_r($decoded);



/**
 * 解密/验证
 * Source: http://self-issued.info/docs/draft-ietf-oauth-json-web-token.html#nbfDef
 */
try {
    JWT::$leeway = 60; //当前时间减去60,把时间留点余地
    $decoded = JWT::decode($jwt, $key, array('HS256')); // /key要和签发一致 HS256方式要和签发一致
    $arr = (array)$decoded;
    print_r($arr);
}catch(Exception $e) {  
    echo $e->getMessage();  //错误提示
}

/**
 * 提示: 在src目录下的JWT.php中的方法 decode中，可以看到是通过多个 自定义 throw new 抛出异常的
 * 所以我们可以根据不同的throw new设置多个catch来捕获
 */
try {
    JWT::$leeway = 60; 
    $decoded = JWT::decode($jwt, $key, array('HS256')); 
} catch(\chenbool\JWT\SignatureInvalidException $e) {  
    echo $e->getMessage(); //签名不正确
}catch(\chenbool\JWT\BeforeValidException $e) {  
    echo $e->getMessage(); // 签名在某个时间点之后才能用
}catch(\chenbool\JWT\ExpiredException $e) {  
    echo $e->getMessage();  // token过期
}catch(Exception $e) {  
    //其他错误
    echo $e->getMessage();
}


?>
```
案例 RS256 (openssl)
----------------------------
```php
<?php
use \chenbool\JWT\JWT;

$privateKey = <<<EOD
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQC8kGa1pSjbSYZVebtTRBLxBz5H4i2p/llLCrEeQhta5kaQu/Rn
vuER4W8oDH3+3iuIYW4VQAzyqFpwuzjkDI+17t5t0tyazyZ8JXw+KgXTxldMPEL9
5+qVhgXvwtihXC1c5oGbRlEDvDF6Sa53rcFVsYJ4ehde/zUxo6UvS7UrBQIDAQAB
AoGAb/MXV46XxCFRxNuB8LyAtmLDgi/xRnTAlMHjSACddwkyKem8//8eZtw9fzxz
bWZ/1/doQOuHBGYZU8aDzzj59FZ78dyzNFoF91hbvZKkg+6wGyd/LrGVEB+Xre0J
Nil0GReM2AHDNZUYRv+HYJPIOrB0CRczLQsgFJ8K6aAD6F0CQQDzbpjYdx10qgK1
cP59UHiHjPZYC0loEsk7s+hUmT3QHerAQJMZWC11Qrn2N+ybwwNblDKv+s5qgMQ5
5tNoQ9IfAkEAxkyffU6ythpg/H0Ixe1I2rd0GbF05biIzO/i77Det3n4YsJVlDck
ZkcvY3SK2iRIL4c9yY6hlIhs+K9wXTtGWwJBAO9Dskl48mO7woPR9uD22jDpNSwe
k90OMepTjzSvlhjbfuPN1IdhqvSJTDychRwn1kIJ7LQZgQ8fVz9OCFZ/6qMCQGOb
qaGwHmUK6xzpUbbacnYrIM6nLSkXgOAwv7XXCojvY614ILTK3iXiLBOxPu5Eu13k
eUz9sHyD6vkgZzjtxXECQAkp4Xerf5TGfQXGXhxIX52yH+N2LtujCdkQZjXAsGdm
B2zNzvrlgRmgBrklMTrMYgm1NPcW+bRLGcwgW2PTvNM=
-----END RSA PRIVATE KEY-----
EOD;

$publicKey = <<<EOD
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC8kGa1pSjbSYZVebtTRBLxBz5H
4i2p/llLCrEeQhta5kaQu/RnvuER4W8oDH3+3iuIYW4VQAzyqFpwuzjkDI+17t5t
0tyazyZ8JXw+KgXTxldMPEL95+qVhgXvwtihXC1c5oGbRlEDvDF6Sa53rcFVsYJ4
ehde/zUxo6UvS7UrBQIDAQAB
-----END PUBLIC KEY-----
EOD;

$token = array(
    "iss" => "example.org", //签发者 可选
    "aud" => "example.com", //接收该JWT的一方 可选
    "iat" => 1356999524,    //签发时间
    "nbf" => 1357000000 //(Not Before):某个时间点后才能访问,比如设置time()+30,表示当前时间30秒后才能使用
);

$jwt = JWT::encode($token, $privateKey, 'RS256');
echo "加密:\n" . print_r($jwt, true) . "\n";

$decoded = JWT::decode($jwt, $publicKey, array('RS256'));
$decoded_array = (array) $decoded;
echo "解密:\n" . print_r($decoded_array, true) . "\n";

?>
```


