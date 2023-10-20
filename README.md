# Hyperf JWT 组件
来源于 hyperf-ext/jwt 主要是 hyperf3.0 不可用，故修改自己使用
该组件基于 [`tymon/jwt-auth`](https://github.com/tymondesigns/jwt-auth )，实现了完整用于 JWT 认证的能力。

该组件并不直接提供身份认证的能力，你可以基于该组件提供的功能特性来实现自己的身份认证。

如果你不想自己动手，可以同时安装 [`hyperf-ext/auth`](https://github.com/hyperf-ext/auth) 组件来获得接近开箱即用的身份认证和授权功能。 

## 安装

```shell script
composer composer require hyperf-yfd/jwt
```

## 发布配置

```shell script
php bin/hyperf.php vendor:publish hyperf-yfd/jwt
```

> 文件位于 `config/autoload/jwt.php`。

## 配置

```php
[
    /*
    |--------------------------------------------------------------------------
    | JWT 密钥
    |--------------------------------------------------------------------------
    |
    | 该密钥用于签名你的令牌，切记要在 .env 文件中设置。组件提供了一个辅助命令来完成
    | 这步操作：
    | `php bin/hyperf.php gen:jwt-secret`
    |
    | 注意：该密钥仅用于对称算法（HMAC），RSA 和 ECDSA 使用公私钥体系（见下方）。
    |
    | 注意：该值必须使用 BASE64 编码。
    |
    */

    'secret' => env('JWT_SECRET'),

    /*
    |--------------------------------------------------------------------------
    | JWT 公私钥
    |--------------------------------------------------------------------------
    |
    | 你使用的算法将决定你的令牌是使用随机字符串（在 `JWT_SECRET` 中定设置）还是
    | 使用以下公钥和私钥来签名。组件提供了一个辅助命令来完成这步操作：
    | `php bin/hyperf.php gen:jwt-keypair`
    |
    | 对称算法：
    | HS256、HS384 和 HS512 使用 `JWT_SECRET`。
    |
    | 非对称算法：
    | RS256、RS384 和 RS512 / ES256、ES384 和 ES512 使用下面的公私钥。
    |
    */

    'keys' => [
        /*
        |--------------------------------------------------------------------------
        | 公钥
        |--------------------------------------------------------------------------
        |
        | 你的公钥内容。
        |
        */

        'public' => env('JWT_PUBLIC_KEY'),

        /*
        |--------------------------------------------------------------------------
        | 私钥
        |-------------------------------0-------------------------------------------
        |
        | 你的私钥内容。
        |
        */

        'private' => env('JWT_PRIVATE_KEY'),

        /*
        |--------------------------------------------------------------------------
        | 密码
        |--------------------------------------------------------------------------
        |
        | 你的私钥的密码。不需要密码可设置为 `null`。
        |
        | 注意：该值必须使用 BASE64 编码。
        |
        */

        'passphrase' => env('JWT_PASSPHRASE'),
    ],

    /*
    |--------------------------------------------------------------------------
    | JWT 生存时间
    |--------------------------------------------------------------------------
    |
    | 指定令牌有效的时长（以秒为单位）。默认为 1 小时。
    |
    | 你可以将其设置为 `null`，以产生永不过期的令牌。某些场景下有人可能想要这种行为，
    | 例如在用于手机应用的情况下。
    | 不太推荐这样做，因此请确保你有适当的体系来在必要时可以撤消令牌。
    | 注意：如果将其设置为 `null`，则应从 `required_claims` 列表中删除 `exp` 元素。
    |
    */

    'ttl' => env('JWT_TTL', 3600),

    /*
    |--------------------------------------------------------------------------
    | 刷新生存时间
    |--------------------------------------------------------------------------
    |
    | 指定一个时长以在其有效期内可刷新令牌（以秒为单位）。 例如，用户可以
    | 在创建原始令牌后的 2 周内刷新该令牌，直到他们必须重新进行身份验证为止。
    | 默认为 2 周。
    |
    | 你可以将其设置为 `null`，以提供无限的刷新时间。某些场景下有人可能想要这种行为，
    | 而不是永不过期的令牌，例如在用于手机应用的情况下。
    | 不太推荐这样做，因此请确保你有适当的体系来在必要时可以撤消令牌。
    |
    */

    'refresh_ttl' => env('JWT_REFRESH_TTL', 3600 * 24 * 14),

    /*
    |--------------------------------------------------------------------------
    | JWT 哈希算法
    |--------------------------------------------------------------------------
    |
    | 用于签名你的令牌的哈希算法。
    |
    | 关于算法的详细描述可参阅 https://tools.ietf.org/html/rfc7518。
    |
    | 可能的值：HS256, HS384, HS512, RS256, RS384, RS512, ES256, ES384, ES512
    |
    */

    'algo' => env('JWT_ALGO', 'HS512'),

    /*
    |--------------------------------------------------------------------------
    | 必要声明
    |--------------------------------------------------------------------------
    |
    | 指定在任一令牌中必须存在的必要声明。如果在有效载荷中不存在这些声明中的任意一个，
    | 则将抛出 `TokenInvalidException` 异常。
    |
    */

    'required_claims' => [
        'iss',
        'iat',
        'exp',
        'nbf',
        'sub',
        'jti',
    ],

    /*
    |--------------------------------------------------------------------------
    | 保留声明
    |--------------------------------------------------------------------------
    |
    | 指定在刷新令牌时要保留的声明的键名。
    | 除了这些声明之外，`sub`、`iat` 和 `prv`（如果有）声明也将自动保留。
    |
    | 注意：如果有声明不存在，则会将其忽略。
    |
    */

    'persistent_claims' => [
        // 'foo',
        // 'bar',
    ],

    /*
    |--------------------------------------------------------------------------
    | 锁定主题声明
    |--------------------------------------------------------------------------
    |
    | 这将决定是否将一个 `prv` 声明自动添加到令牌中。
    | 此目的是确保在你拥有多个身份验证模型时，例如 `App\User` 和 `App\OtherPerson`，
    | 如果两个令牌在两个不同的模型中碰巧具有相同的 ID（`sub` 声明），则我们应当防止
    | 一个身份验证请求冒充另一个身份验证请求。
    |
    | 在特定情况下，你可能需要禁用该行为，例如你只有一个身份验证模型的情况下，
    | 这可以减少一些令牌大小。
    |
    */

    'lock_subject' => true,

    /*
    |--------------------------------------------------------------------------
    | 时间容差
    |--------------------------------------------------------------------------
    |
    | 该属性为 JWT 的时间戳类声明提供了一些时间上的容差。
    | 这意味着，如果你的某些服务器上不可避免地存在轻微的时钟偏差，
    | 那么这将可以为此提供一定程度的缓冲。
    |
    | 该设置适用于 `iat`、`nbf` 和 `exp`声明。
    | 以秒为单位设置该值，仅在你了解你真正需要它时才指定。
    |
    */

    'leeway' => env('JWT_LEEWAY', 0),

    /*
    |--------------------------------------------------------------------------
    | 启用黑名单
    |--------------------------------------------------------------------------
    |
    | 为使令牌无效，你必须启用黑名单。
    | 如果你不想或不需要此功能，请将其设置为 `false`。
    |
    */

    'blacklist_enabled' => env('JWT_BLACKLIST_ENABLED', true),

    /*
    | -------------------------------------------------------------------------
    | 黑名单宽限期
    | -------------------------------------------------------------------------
    |
    | 当使用同一个 JWT 发送多个并发请求时，由于每次请求都会重新生成令牌，
    | 因此其中一些可能会失败。
    |
    | 设置宽限期（以秒为单位）以防止并发请求失败。
    |
    */

    'blacklist_grace_period' => env('JWT_BLACKLIST_GRACE_PERIOD', 0),

    /*
    |--------------------------------------------------------------------------
    | 黑名单存储
    |--------------------------------------------------------------------------
    |
    | 指定用于实现在黑名单中存储令牌行为的类。
    |
    | 自定义存储类需要实现 `HyperfExt\Jwt\Contracts\StorageInterface` 接口。
    |
    */

    'blacklist_storage' => HyperfExt\Jwt\Storage\HyperfCache::class,
];
```

## 使用

如果你使用 [`hyperf-ext/auth`](https://github.com/hyperf-ext/auth) 组件，则可以忽略该部分。

```php
<?php

declare(strict_types=1);

use HyperfExt\Jwt\Contracts\JwtFactoryInterface;
use HyperfExt\Jwt\Contracts\ManagerInterface;

class SomeClass
{
    /**
     * 提供了对 JWT 编解码、刷新和失活的能力。
     *
     * @var \HyperfExt\Jwt\Contracts\ManagerInterface
     */
    protected $manager;

    /**
     * 提供了从请求解析 JWT 及对 JWT 进行一系列相关操作的能力。
     *
     * @var \HyperfExt\Jwt\Jwt
     */
    protected $jwt;

    public function __construct(
        ManagerInterface $manager,
        JwtFactoryInterface $jwtFactory
    ) {
        $this->manager = $manager;
        $this->jwt = $jwtFactory->make();
    }
}
```

可阅读上述两个类来详细了解如何使用。
