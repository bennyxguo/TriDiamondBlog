---
title: 阿里云GitLab Webhook 自动部署手册
abstract: 手把手教你在阿里云代码托管配置自动部署webhook和PHP脚本
header_image: /assets/images/lumen-banner.png
categories:
  - Gitlab
tags:
  - Gitlab
  - Webhook
---

## 创建与填写部署公钥
### 创建部署公钥
```bash
sudo -Hu www ssh-keygen -t rsa
```
> 如果创建失败首先需要创建/home/www/.ssh这个文件夹

### 查看公钥
```bash
cat /home/www/.ssh/id_rsa.pub
```

### 添加Hook
在阿里云code.aliyun.com上的profile>ssh_key里面添加公钥

## 初始化git项目文件夹
```bash
sudo -Hu www git clone [git地址]
```
> 这里注意, 一定要用www的身份状态要不后期无法自动git pull

## 自动部署脚本 (PHP)

### Shell_exec
在使用这个PHP脚本的时候我们需要用到```shell_exec```php的原生函数, php-fpm是默认屏蔽这个函数的, 所有需要在php.ini里面修改一下配置
> 找到```disable_functions```这个参数, 并且在里面去掉```shell_exec```

### PHP 脚本
```php
$token = 'token';

if (!isset($_GET['token']) && $_GET['token'] != $token) {
	die('access denied');
}

$json = json_decode(file_get_contents('php://input'), true);
$repo = $json['repository']['name'];

// 只在主分支提交时且提交数大于0执行自动部署
if ($json['ref']=='refs/heads/master' && $json['total_commits_count']>0) {
	$pull_result = shell_exec('cd /to/project/path/ && git pull');

	if ($pull_result) {
		$res_log = '----------pull 成功---------------'.PHP_EOL;
	    
		$res_log .= $json['user_name'] . ' 在' . date('Y-m-d H:i:s') . '向' . $json['repository']['name'] . '项目的' . $json['ref'] . '分支push了' . $json['total_commits_count'] . '个commit：' . PHP_EOL;
		$res_log .= $pull_result.PHP_EOL;

		file_put_contents("cityconcierge-webhook-log.txt", $res_log, FILE_APPEND);//追加写入
	} else {
		$res_log = '------------pull 失败-------------'.PHP_EOL;
	    
		$res_log .= $json['user_name'] . ' 在' . date('Y-m-d H:i:s') . '向' . $json['repository']['name'] . '项目的' . $json['ref'] . '分支push了' . $json['total_commits_count'] . '个commit：' . PHP_EOL;
		$res_log .= $pull_result.PHP_EOL;

		file_put_contents("cityconcierge-webhook-log.txt", $res_log, FILE_APPEND);//追加写入
	}
}
```
