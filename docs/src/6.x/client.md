# API 调用

与以往版本不同的是，SDK 不再内置具体 API 的逻辑，所有的 API 均交由开发者自行调用，以获取用户列表为例：

```php
$api = $app->getClient();

$response = $api->post('/cgi-bin/user/info/updateremark', ['body' => [
    "openid" => "oDF3iY9ffA-hqb2vVvbr7qxf6A0Q",
    "remark" => "pangzi"
]]);
```

## 语法说明

```php
Symfony\Contracts\HttpClient\ResponseInterface {get/post/patch/put/delete}($uri, $options = [])
```

**参数说明：**

- `$uri` 为需要请求的 `path`；
- `$options` 为请求参数，可以指定 `query` / `body` / `headers` 等等，具体请参考：[Symfony\Contracts\HttpClient\HttpClientInterface::OPTIONS_DEFAULTS](https://github.com/symfony/symfony/blob/5.3/src/Symfony/Contracts/HttpClient/HttpClientInterface.php)

---

## 请求参数

### GET

```php
$users = $api->get('/cgi-bin/user/list'， [
    'query' => [
            'next_openid' => 'OPENID1',
        ]
    ])->toArray();
```

### POST

```php
$response = $api->post('/cgi-bin/user/info/updateremark', [
    'body' => [
            "openid" => "oDF3iY9ffA-hqb2vVvbr7qxf6A0Q",
            "remark" => "pangzi"
        ]
    ]);
```

或者可以简写为：

```php
$response = $api->post('/cgi-bin/user/info/updateremark', [
        "openid" => "oDF3iY9ffA-hqb2vVvbr7qxf6A0Q",
        "remark" => "pangzi"
    ]);
```

或者指定 json 格式：

```php
$response = $api->post('/cgi-bin/user/info/updateremark', [
    'json' => [
            "openid" => "oDF3iY9ffA-hqb2vVvbr7qxf6A0Q",
            "remark" => "pangzi"
        ]
    ]);
```

或者指定 xml 格式：

```php
$response = $api->post('/mmpaymkttransfers/promotion/transfers', [
    'xml' => [
        'mch_appid' => $app->getConfig()['app_id'],
        'mchid' => $app->getConfig()['mch_id'],
        'partner_trade_no' => '202203081646729819743',
        'openid' => 'ogn1H45HCRxVRiEMLbLLuABbxxxx',
        'check_name' => 'FORCE_CHECK',
        're_user_name'=> 'overtrue',
        'amount' => 100,
        'desc' => '理赔',
    ]]);
```

### 请求证书

你可以在请求支付时指定证书，以微信支付 V2 为例：

```php
$response = $api->post('/mmpaymkttransfers/promotion/transfers', [
    'xml' => [
        'mch_appid' => $app->getConfig()['app_id'],
        'mchid' => $app->getConfig()['mch_id'],
        'partner_trade_no' => '202203081646729819743',
        'openid' => 'ogn1H45HCRxVRiEMLbLLuABbxxxx',
        'check_name' => 'FORCE_CHECK',
        're_user_name'=> 'overtrue',
        'amount' => 100,
        'desc' => '理赔',
    ],
    'local_cert' => $app->getConfig()['cert_path'],
    'local_pk' => $app->getConfig()['key_path'],
    ]);
```

> 参考：[symfony/http-client#options](https://symfony.com/doc/current/reference/configuration/framework.html#local-cert)

### 文件上传

你有两种上传文件的方式可以选择：

#### 从指定路径上传

```php
use EasyWeChat\Kernel\Form\File;
use EasyWeChat\Kernel\Form\Form;

$options = Form::create(
    [
        'media' => File::fromPath('/path/to/image.jpg'),
    ]
)->toArray();

$response = $api->post('cgi-bin/media/upload?type=image', $options);
```

#### 从二进制内容上传

```php
use EasyWeChat\Kernel\Form\File;
use EasyWeChat\Kernel\Form\Form;

$options = Form::create(
    [
        'media' => File::withContents($contents, 'image.jpg'), // 注意：请指定文件名
    ]
)->toArray();

$response = $api->post('cgi-bin/media/upload?type=image', $options);
```

---

## 处理响应

API Client 基于 [symfony/http-client](https://github.com/symfony/http-client) 实现，你可以通过以下方式对响应值进行访问：

```php
$response = $api->get('/cgi-bin/user/get', ['query' => ['openid' => '...']]);

// 获取状态码
$statusCode = $response->getStatusCode();

// 获取全部响应头
$headers = $response->getHeaders();

// 获取响应原始内容
$content = $response->getContent();
// 获取响应原始内容（不抛出异常）
$content = $response->getContent(false);

// 获取 json 转换后的数组格式
$content = $response->toArray();
// 获取 json 转换后的数组格式（不抛出异常）
$content = $response->toArray(false);

// 将内容转换成 Stream 返回
$content = $response->toStream();
// 将内容转换成 Stream 返回 (不抛出异常)
$content = $response->toStream(false);

// 获取其他信息，如："response_headers", "redirect_count", "start_time", "redirect_url" 等.
$httpInfo = $response->getInfo();

// 获取指定信息
$startTime = $response->getInfo('start_time');

// 获取请求日志
$httpLogs = $response->getInfo('debug');
```

:book: 更多使用请参考： [HTTP client: Processing Responses](https://symfony.com/doc/current/http_client.html#processing-responses)

---

## 异步请求

所有的请求都是异步的，当你第一次访问 `$response` 时才会真正的请求，比如：

```php
// 这段代码会立即执行，并不会发起网络请求
$response = $api->post('/cgi-bin/user/info/updateremark', ['body' => [
    "openid" => "oDF3iY9ffA-hqb2vVvbr7qxf6A0Q",
    "remark" => "pangzi"
]])

// 当你尝试访问 $response 的信息时，才会发起请求并等待返回
$contentType = $response->getHeaders()['content-type'][0];

// 尝试获取响应内容将阻塞执行，直到接收到完整的响应内容
$content = $response->getContent();
```

## 并行请求

由于请求天然是异步的，那么你可以很简单实现并行请求：

```php
$responses = [
    $api->get('/cgi-bin/user/get'),
    $api->post('/cgi-bin/user/info/updateremark', ['body' => ...]),
    $api->post('/cgi-bin/user/message/custom/send', ['body' => ...]),
];

// 访问任意一个 $response 时将执行并发请求：
foreach ($responses as $response) {
    $content = $response->getContent();
    // ...
}
```

当然你也可以给每个请求分配名字独立访问：

```php
$responses = [
    'users'=> $api->get('/cgi-bin/user/get'),
    'remark' => $api->post('/cgi-bin/user/info/updateremark', ['body' => ...]),
    'message' => $api->post('/cgi-bin/user/message/custom/send', ['body' => ...]),
];

// 访问任意一个 $response 时将执行并发请求：
$responses['users']->toArray();
```

## 失败重试

默认在公众号、小程序开启了重试机制，你可以通过全局配置或者手动开启重试特性。

> 🚨 不建议在支付模块使用重试功能，因为一旦重试导致支付数据异常，可能造成无法挽回的损失。

### 方式一：全局配置

在支持重试的模块里增加如下配置可以完成重试机制的配置

```php
    'http' => [
        'timeout' => 5,

        'retry' => true, // 使用默认配置
        // 'retry' => [
        //     // 仅以下状态码重试
        //     'http_codes' => [429, 500]
        //     'max_retries' => 3
        //     // 请求间隔 (毫秒)
        //     'delay' => 1000,
        //     // 如果设置，每次重试的等待时间都会增加这个系数
        //     // (例如. 首次:1000ms; 第二次: 3 * 1000ms; etc.)
        //     'multiplier' => 0.1
        // ],
    ],
```

### 方式二：手动开启

如果你不想使用基于配置的全局重试机制，你可以使用 `HttpClient::retry()` 方法来开启失败重试能力：

```php
$app->getClient()->retry()->get('/foo/bar');
```

当然，你可以在 `retry` 配置中自定义重试的配置，如下所示：

```php
$app->getClient()->retry([
    'max_retries' => 2,
    //...
])->get('/foo/bar');
```

### 自定义重试策略

如果觉得参数不能满足需求，你还可以自己实现 [`Symfony\Component\HttpClient\RetryStrategyInterface`](https://github.com/symfony/symfony/blob/6.1/src/Symfony/Component/HttpClient/Retry/RetryStrategyInterface.php) 接口来自定义重试策略，然后调用 `retryUsing` 方法来使用它。

> 💡 建议继承基类来拓展，以实现默认重试类的基础功能。

```php
class MyRetryStrategy extends \Symfony\Component\HttpClient\Retry\GenericRetryStrategy
{
    public function shouldRetry(AsyncContext $context, ?string $responseContent, ?TransportExceptionInterface $exception): ?bool
    {
        // 你的自定义逻辑
        // if (...) {
        //     return false;
        // }

        return parent::shouldRetry($context, $responseContent, $exception);
    }
}
```

使用自定义重试策略：

```php
$app->getClient()->retryUsing(new MyRetryStrategy())->get('/foo/bar');
```

## 更多使用方法

:book: 更多使用请参考：[symfony/http-client](https://github.com/symfony/http-client)