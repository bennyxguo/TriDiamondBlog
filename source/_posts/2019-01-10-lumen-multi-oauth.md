---
title: Lumen passport实现多个用户体系下的oauth验证
date: 2019-01-10 01:40:41
abstract: 手把手教你在Lumen的Passport实现多个用户体系下使用多个Oauth验证
header_image: /assets/images/lumen-banner.png
categories:
  - Lumen
tags:
  - Lumen
  - Laravel Passport
---

这个教程是基于, lumen里面已经安装好了lumen-passport的插件, 如果还没有的话可以先到[lumen使用laravel passport教程](/2019/01/10/lumen-passport-usage/)先安装.

## 改写Laravel Passport里面的`UserRepository`

> 文件路径 `vendor\laravel\passport\src\Bridge\UserRepository.php`

- 首先需要改写`userRepositroy`里面的`getUserEntityByUserCredentials`方法

- 复制`userRepositroy`里面的`getUserEntityByUserCredentials`方法, 改名为`getEntityByUserCredentials`

- 在新建的方法里面找到一下代码

```php
$provider = config('auth.guards.api.provider');
```

改成一下样子

```php
$provider = config('auth.guards.'.$provider.'.provider');
```

- 然后在新的方法`getEntityByUserCredentials`的参数里面添加新的参数`$provider`

```php
public function getEntityByUserCredentials($username, $password, $grantType, 
  ClientEntityInterface $clientEntity, $provider) {
      //...
}
```

## 修改oauth2-server里面的PasswordGrand

> 文件路径 `vendor\league\oauth2-server\src\Grant\PasswordGrant.php`

- 修改`validateUser`方法里面的这一串代码:

```php
$user = $this->userRepository->getEntityByUserCredentials(
    $username,
    $password,
    $this->getIdentifier(),
    $client,
    $provider // 新加的provider字段
);
```

- 在同一个方法里面加入新参数的获取

```php
 $provider = $this->getRequestParameter('provider', $request);

 if (is_null($provider)) {
 throw OAuthServerException::invalidRequest('provider');
 }
```

## 在auth.php配置里面加入新的guard

首先需要加入新的guard配置

```php
'guards' => [
    // 原有的api guard
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
    // 新加的admin-api guard
    'admin-api' => [
        'driver' => 'passport',
        'provider' => 'admins',
    ],
],
```

添加新`admin-api` guard的provider

```php
'providers' => [
    'users' => [
        'driver' => 'eloquent',
        'model' => \App\Models\General\MemberLogin::class
    ],
    // 新加的admins provider对应不用的用户模型
    'admins' => [
        'driver' => 'eloquent',
        'model' => \App\Models\Backend\Manager::class
    ]
],
```

## 路由middleware使用

区别在于middleware, 上文加入的新`admin-api`guard, 在新的路由里面就可以使用`auth:admin-api`的权限验证中间件理实现权限控制了!

```php
/*
|--------------------------------------------------------------------------
| Admin API版本 v1 路由
|--------------------------------------------------------------------------.
|
| prefix admin/api/api版本号
| namespace Api\api版本号
|
*/
$app->group(['prefix' => 'admin/api/v1', 'namespace' => 'AdminApi\V1'], function ($app) {

    // ================ 不受登录权限控制的接口路由 ================ //
    //测试
    $app->get('test', 'ExampleController@test');

    // ================ 受登录权限控制的接口路由 ================ //
    $app->group(['middleware' => 'auth:admin-api'], function ($app) {
        //测试
        $app->get('test2', function(){
            return 'oauth test';
        });
        //测试
        $app->get('test3', 'ExampleController@test');
    });

});
```

## 注意事项

使用了多个guard的时候, 在使用laravel默认的`$request->user()`, 这个方法默认是使用`api`guard的, 可以在`auth.php`配置里面看到默认guard的配置.

```php
/*
|--------------------------------------------------------------------------
| Authentication Defaults
|--------------------------------------------------------------------------
|
| This option controls the default authentication "guard" and password
| reset options for your application. You may change these defaults
| as required, but they're a perfect start for most applications.
|
*/

'defaults' => [
    'guard' => env('AUTH_GUARD', 'api'),
    'passwords' => 'users',
],
```

所以在使用新的`admin-api` guard的时候在使用`$request->user()`时需要加入对应的guard. 例子:

```php
namespace App\Http\Controllers\AdminApi\V1;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;

class ExampleController extends Controller
{
    public function test(Request $request)
    {
        $request->user('admin-api')->toArray(); // 获取到admin-api下的用户信息
    }
}
```

