[![Serverless PHP Laravel Tencent Cloud](https://img.serverlesscloud.cn/20191226/1577347087676-website_%E9%95%BF.png)](http://serverless.com)

# 腾讯云 Laravel Serverless Component

[![npm](https://img.shields.io/npm/v/%40serverless%2Ftencent-laravel)](http://www.npmtrends.com/%40serverless%2Ftencent-laravel)
[![NPM downloads](http://img.shields.io/npm/dm/%40serverless%2Ftencent-laravel.svg?style=flat-square)](http://www.npmtrends.com/%40serverless%2Ftencent-laravel)

简体中文 | [English](https://github.com/serverless-components/tencent-thinkphp/blob/master/README.en.md)

## 简介

腾讯云 [Laravel](https://github.com/laravel/laravel) Serverless Component, 支持 Restful API 服务的部署。

## 目录

0. [准备](#0-准备)
1. [安装](#1-安装)
1. [配置](#2-配置)
1. [部署](#3-部署)
1. [移除](#4-移除)

### 0. 准备

#### 初始化 Laravel 项目

在使用此组件之前，你需要先自己初始化一个 `laravel` 项目

```bash
composer create-project --prefer-dist laravel/laravel serverless-laravel
```

> 注意：Laravel 使用 Coposer 管理依赖的，所以你需要先自行安装 Composer，请参考 [官方安装文档](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos)

#### 修改 Laravel 项目

由于云函数在执行时，只有 `/tmp` 可读写的，所以我们需要将 `laravel` 框架运行时的 `storage` 目录写到该目录下，为此需要修改 `bootstrap/app.php` 文件，在 `$app = new Illuminate\Foundation\Application` 后添加：

```php
$app->useStoragePath($_ENV['APP_STORAGE'] ?? $app->storagePath());
```

然后在跟目录下的 `.env` 文件中新增如下配置:

```dotenv
# 视图文件编译路径
VIEW_COMPILED_PATH=/tmp/storage/framework/views

# 由于是无服务函数，所以没法存储 session 在硬盘上，如果不需要 sessions，可以使用 array
# 如果需要你可以将 session 存储到 cookie 或者数据库中
SESSION_DRIVER=array

# 建议将错误日志输出到控制台，方便云端去查看
LOG_CHANNEL=stderr

# 应用的 storage 目录必须为 /tmp
APP_STORAGE=/tmp
```

### 1. 安装

通过 npm 全局安装 [serverless cli](https://github.com/serverless/serverless)

```bash
$ npm install -g serverless
```

### 2. 配置

在项目根目录，创建 `serverless.yml` 文件，在其中进行如下配置

```bash
$ touch serverless.yml
```

```yml
# serverless.yml

MyComponent:
  component: '@serverless/tencent-laravel'
  inputs:
    region: ap-guangzhou
    functionName: laravel-function
    code: ./
    functionConf:
      timeout: 10
      memorySize: 128
      environment:
        variables:
          TEST: vale
      vpcConfig:
        subnetId: ''
        vpcId: ''
    apigatewayConf:
      protocols:
        - https
      environment: release
```

- [更多配置](https://github.com/serverless-components/tencent-laravel/tree/master/docs/configure.md)

### 3. 部署

> 注意：**在部署前，你需要先清理本地运行的配置缓存，执行 `php artisan config:clear` 即可。**

如您的账号未 [登陆](https://cloud.tencent.com/login) 或 [注册](https://cloud.tencent.com/register) 腾讯云，您可以直接通过 `微信` 扫描命令行中的二维码进行授权登陆和注册。

通过 `sls` 命令进行部署，并可以添加 `--debug` 参数查看部署过程中的信息

```bash
$ sls --debug

  DEBUG ─ Resolving the template's static variables.
  DEBUG ─ Collecting components from the template.
  DEBUG ─ Downloading any NPM components found in the template.
  DEBUG ─ Analyzing the template's components dependencies.
  DEBUG ─ Creating the template's components graph.
  DEBUG ─ Syncing template state.
  DEBUG ─ Executing the template's components graph.
  DEBUG ─ Compressing function laravel-function file to /Users/yugasun/Desktop/Develop/serverless/tencent-laravel/example/.serverless/laravel-function.zip.
  DEBUG ─ Compressed function laravel-function file successful
  DEBUG ─ Uploading service package to cos[sls-cloudfunction-ap-guangzhou-code]. sls-cloudfunction-default-laravel-function-1584409722.zip
  DEBUG ─ Uploaded package successful /Users/yugasun/Desktop/Develop/serverless/tencent-laravel/example/.serverless/laravel-function.zip
  DEBUG ─ Creating function laravel-function
  laravel-function [████████████████████████████████████████] 100% | ETA: 0s | Speed: 437.95k/s
  DEBUG ─ Created function laravel-function successful
  DEBUG ─ Setting tags for function laravel-function
  DEBUG ─ Creating trigger for function laravel-function
  DEBUG ─ Deployed function laravel-function successful
  DEBUG ─ Starting API-Gateway deployment with name ap-guangzhou-apigateway in the ap-guangzhou region
  DEBUG ─ Service with ID service-em7sgz40 created.
  DEBUG ─ API with id api-lln5145m created.
  DEBUG ─ Deploying service with id service-em7sgz40.
  DEBUG ─ Deployment successful for the api named ap-guangzhou-apigateway in the ap-guangzhou region.

  MyLaravel:
    functionName:        laravel-function
    functionOutputs:
      ap-guangzhou:
        Name:        laravel-function
        Runtime:     Php7
        Handler:     serverless-handler.handler
        MemorySize:  128
        Timeout:     10
        Region:      ap-guangzhou
        Namespace:   default
        Description: This is a template function
    region:              ap-guangzhou
    apiGatewayServiceId: service-em7sgz40
    url:                 https://service-em7sgz40-1251556596.gz.apigw.tencentcs.com/release/
    cns:                 (empty array)

  51s › MyLaravel › done
```

> 注意: `sls` 是 `serverless` 命令的简写。

### 4. 移除

通过以下命令移除部署的 Laravel 服务。

```bash
$ sls remove --debug

  DEBUG ─ Flushing template state and removing all components.
  DEBUG ─ Removed function laravel-function successful
  DEBUG ─ Removing any previously deployed API. api-lln5145m
  DEBUG ─ Removing any previously deployed service. service-em7sgz40

  14s › MyLaravel › done
```

### 账号配置（可选）

当前默认支持 CLI 扫描二维码登录，如您希望配置持久的环境变量/秘钥信息，也可以本地创建 `.env` 文件

在 `.env` 文件中配置腾讯云的 SecretId 和 SecretKey 信息并保存

如果没有腾讯云账号，可以在此 [注册新账号](https://cloud.tencent.com/register)。

如果已有腾讯云账号，可以在 [API 密钥管理](https://console.cloud.tencent.com/cam/capi) 中获取 `SecretId` 和`SecretKey`.

```text
# .env
TENCENT_SECRET_ID=123
TENCENT_SECRET_KEY=123
```

### 更多组件

可以在 [Serverless Components](https://github.com/serverless/components) repo 中查询更多组件的信息。
