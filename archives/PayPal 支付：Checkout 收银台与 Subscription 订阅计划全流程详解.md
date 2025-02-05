# PayPal 支付：Checkout 收银台与 Subscription 订阅计划全流程详解

本文将详细介绍 PayPal 的 Checkout 收银台和 Subscription 订阅计划的完整流程，帮助开发者更好地理解和实现这两种支付方式。

## 一、支付生命周期

### 1. Checkout - 收银台支付

Checkout 支付的流程类似于支付宝的收银台，具体步骤如下：

![Checkout 支付流程](https://bbtdd.com/img/744289937.webp!large)

1. **组装参数并请求 Checkout 接口**：本地应用组装好参数并请求 Checkout 接口，接口同步返回一个支付 URL。
2. **重定向至支付页面**：本地应用重定向至该 URL，用户登录 PayPal 账户并确认支付，支付成功后跳转至本地应用指定地址。
3. **发起扣款请求**：本地请求 PayPal 执行付款接口发起扣款。
4. **异步通知**：PayPal 发送异步通知至本地应用，本地应用收到数据包后进行验签操作。
5. **处理支付完成后的业务**：验签成功后，执行本地订单状态更新、增加销量、发送邮件等后续操作。

### 2. Subscription - 订阅支付

Subscription 订阅支付的流程如下：

![Subscription 订阅支付流程](https://bbtdd.com/img/2509038747244.webp!large)

1. **创建计划**：首先创建一个订阅计划。
2. **激活计划**：将创建的计划激活。
3. **创建订阅申请**：使用已激活的计划创建一个订阅申请。
4. **用户授权并完成第一期付款**：本地应用跳转至订阅申请链接，用户授权并完成第一期付款，支付成功后携带 token 跳转至本地应用指定地址。
5. **执行订阅**：回跳后请求执行订阅。
6. **处理异步回调**：收到订阅授权和支付结果的异步回调，验证支付异步回调成功后，执行支付完成后的业务。

## 二、具体实现

### Checkout 支付实现

#### 1. 安装 PayPal SDK

使用 Composer 安装 PayPal 官方 SDK：

bash
$ composer require paypal/rest-api-sdk-php:*


#### 2. 创建 PayPal 配置文件

创建 `config/paypal.php` 配置文件，配置沙箱和生产环境的相关信息：

php
<?php

return [
    'sandbox' => [
        'client_id' => env('PAYPAL_SANDBOX_CLIENT_ID', ''),
        'secret' => env('PAYPAL_SANDBOX_SECRET', ''),
        'notify_web_hook_id' => env('PAYPAL_SANDBOX_NOTIFY_WEB_HOOK_ID', ''),
        'checkout_notify_web_hook_id' => env('PAYPAL_SANDBOX_CHECKOUT_NOTIFY_WEB_HOOK_ID', ''),
        'subscription_notify_web_hook_id' => env('PAYPAL_SANDBOX_SUBSCRIPTION_NOTIFY_WEB_HOOK_ID', ''),
    ],

    'live' => [
        'client_id' => env('PAYPAL_CLIENT_ID', ''),
        'secret' => env('PAYPAL_SECRET', ''),
        'notify_web_hook_id' => env('PAYPAL_NOTIFY_WEB_HOOK_ID', ''),
        'checkout_notify_web_hook_id' => env('PAYPAL_CHECKOUT_NOTIFY_WEB_HOOK_ID', ''),
        'subscription_notify_web_hook_id' => env('PAYPAL_SUBSCRIPTION_NOTIFY_WEB_HOOK_ID', ''),
    ],
];


#### 3. 创建 PayPal 服务类

在 `app/Services` 目录下创建 `PayPalService.php` 文件，并编写 Checkout 支付相关的方法，具体实现可参考官方示例。

#### 4. 注册 PayPal 服务类

在 `AppServiceProvider` 中注册 PayPal 服务类：

php
$this->app->singleton('paypal', function () {
    if (app()->environment() !== 'production') {
        $config = [
            'mode' => 'sandbox',
            'client_id' => config('paypal.sandbox.client_id'),
            'secret' => config('paypal.sandbox.secret'),
            'web_hook_id' => config('paypal.sandbox.notify_web_hook_id'),
        ];
    } else {
        $config = [
            'mode' => 'live',
            'client_id' => config('paypal.live.client_id'),
            'secret' => config('paypal.live.secret'),
            'web_hook_id' => config('paypal.live.notify_web_hook_id'),
        ];
    }
    return new PayPalService($config);
});


#### 5. 创建控制器并实现支付逻辑

通过控制器处理 Checkout 支付请求和回调，具体代码可参考官方示例。

### Subscription 订阅支付实现

#### 1. 创建计划并激活计划

在 PayPal 服务类中编写创建计划并激活计划的方法，具体实现可参考官方示例。

#### 2. 创建订阅申请

使用已激活的计划创建订阅申请，并获取用户授权链接。

#### 3. 执行订阅

在用户授权后，执行订阅并处理支付结果。

#### 4. 处理异步回调

处理 PayPal 的异步回调，验证成功后执行本地业务逻辑。

## 三、测试与部署

### 1. 测试 Checkout 支付

在 PayPal 开发者中心的沙箱环境中进行测试，确保支付流程无误。

### 2. 测试 Subscription 订阅支付

同样在沙箱环境中测试订阅支付流程，确保用户授权和支付结果处理无误。

## 四、总结

通过本文的详细讲解，开发者可以轻松实现 PayPal 的 Checkout 收银台和 Subscription 订阅支付功能。无论是单次支付还是订阅支付，PayPal 都提供了完善的 SDK 和文档支持，帮助开发者快速集成。

👉 [WildCard | 一分钟注册，轻松订阅海外线上服务](https://bbtdd.com/WildCard)

希望本文能对您有所帮助，如有任何问题，欢迎在评论区留言讨论。