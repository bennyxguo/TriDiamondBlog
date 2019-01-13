---
title: Lumen 使用 laravel passport
abstract: Lumen是laravel的简洁版, 把laravel里面深重的依赖都去掉了, 所以直接安装laravel的passport是无法正常使用的. 
          所以如果要在lumen上使用laravel的passport就需要安装另外一个插件.
date: 2019-01-10 01:51:23
header_image: /assets/images/lumen-banner.png
categories:
  - Lumen
tags:
  - Lumen
  - Laravel Passport
---

Lumen是laravel的简洁版, 把laravel里面深重的依赖都去掉了, 所以直接安装laravel的passport是无法正常使用的. 
所以如果要在lumen上使用laravel的passport就需要安装另外一个插件.

## 安装要求

- PHP >= 5.6.3
- Lumen >= 5.3

## Composer安装lumen-passport插件

首先安装 Lumen Passport

```bash
# 进入项目根目录
$ cd lumen-app

# 使用composer安装插件
$ composer require dusterio/lumen-passport
```

## 修改 bootstrap (bootstrap/app.php)

需要引入Laravel Passport的provider和Lumen的一些provider

```php
// 开启 Facades
$app->withFacades();

// 开启 Eloquent
$app->withEloquent();

// 开启 auth 中间件
$app->routeMiddleware([
    'auth' => App\Http\Middleware\Authenticate::class,
]);

// 注册laravel passport的provider和lumen passport的provider
$app->register(Laravel\Passport\PassportServiceProvider::class);
$app->register(Dusterio\LumenPassport\PassportServiceProvider::class);
```

## 数据表移植和安装Laravel Passport

```bash
# 移植passport的数据表
php artisan migrate

# 安装passport需要的配置
php artisan passport:install
```

## Lumen Passport自带的路由

这个lumen-passport包已经引入了一下路由, 但是与web相关的路由因为lumen是没有web的路由的, 只有api的, 所以这个插件已经把web端的路由都去掉了.

Verb | Path | NamedRoute | Controller | Action | Middleware
--- | --- | --- | --- | --- | ---
POST   | /oauth/token                             |            | \Laravel\Passport\Http\Controllers\AccessTokenController           | issueToken | -
GET    | /oauth/tokens                            |            | \Laravel\Passport\Http\Controllers\AuthorizedAccessTokenController | forUser    | auth
DELETE | /oauth/tokens/{token_id}                 |            | \Laravel\Passport\Http\Controllers\AuthorizedAccessTokenController | destroy    | auth
POST   | /oauth/token/refresh                     |            | \Laravel\Passport\Http\Controllers\TransientTokenController        | refresh    | auth
GET    | /oauth/clients                           |            | \Laravel\Passport\Http\Controllers\ClientController                | forUser    | auth
POST   | /oauth/clients                           |            | \Laravel\Passport\Http\Controllers\ClientController                | store      | auth
PUT    | /oauth/clients/{client_id}               |            | \Laravel\Passport\Http\Controllers\ClientController                | update     | auth
DELETE | /oauth/clients/{client_id}               |            | \Laravel\Passport\Http\Controllers\ClientController                | destroy    | auth
GET    | /oauth/scopes                            |            | \Laravel\Passport\Http\Controllers\ScopeController                 | all        | auth
GET    | /oauth/personal-access-tokens            |            | \Laravel\Passport\Http\Controllers\PersonalAccessTokenController   | forUser    | auth
POST   | /oauth/personal-access-tokens            |            | \Laravel\Passport\Http\Controllers\PersonalAccessTokenController   | store      | auth
DELETE | /oauth/personal-access-tokens/{token_id} |            | \Laravel\Passport\Http\Controllers\PersonalAccessTokenController   | destroy    | auth

## 配置

修改 ```config/auth.php``` 里面的配置, 按照项目需要修改. 下面是一个简单的例子

```php
return [
    'defaults' => [
        'guard' => 'api',
        'passwords' => 'users',
    ],

    'guards' => [
        'api' => [
            'driver' => 'passport',
            'provider' => 'users',
        ],
    ],

    'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => \App\User::class
        ]
    ]
];
```

需要在 ```vendor\laravel\lumen-framework\config\auth.php``` 复制到项目根目录下的```config```文件夹里面, 如果没有config文件夹, 需要手动添加一个.

然后在```bootstrap/app.php```最前面加入配置应用, 因为lumen是不自动引入config里面的配置的.
```php
$app->configure('auth');
```

## 注册路由

需要在`Provider\AuthServiceProviders.php`里面的`boot`方法里面注册路由

```php
/**
* Boot the authentication services for the application.
*
* @return void
*/
public function boot()
{
    // Here you may define how you wish users to be authenticated for your Lumen
    // application. The callback which receives the incoming request instance
    // should return either a User instance or null. You're free to obtain
    // the User instance via an API token or any other method necessary.

    LumenPassport::routes($this->app); // 注册路由

    LumenPassport::tokensExpireIn(Carbon::now()->addDays(7));

    LumenPassport::refreshTokensExpireIn(Carbon::now()->addDays(30));
}
```

简单路由注册
```php
Dusterio\LumenPassport\LumenPassport::routes($this->app);
```

通用版本控制的路由
```php
Dusterio\LumenPassport\LumenPassport::routes($this->app, ['prefix' => 'v1/oauth']);
```

## 用户模型

需要在用户模型里面加入```HasApiTokens```的trait, 例子:

```php
class User extends Model implements AuthenticatableContract, AuthorizableContract
{
    use HasApiTokens, Authenticatable, Authorizable;

    /* rest of the model */
}
```

## 其他

其他的详细文档可以查看lumen-passport的插件[github](https://github.com/dusterio/lumen-passport)
