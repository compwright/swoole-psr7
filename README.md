# SwoolePsr7

[![Build Status](https://travis-ci.org/compwright/swoole-psr7.svg?branch=master)](https://travis-ci.org/compwright/swoole-psr7)
[![Coverage Status](https://coveralls.io/repos/github/compwright/swoole-psr7/badge.svg?branch=master)](https://coveralls.io/github/compwright/swoole-psr7?branch=master)

Use any PSR 17 Factory to convert to PSR 7 Response/Request.

## Install

Via Composer

``` bash
$ composer require compwright/swoole-psr7
```

## Usage

``` php
<?php declare(strict_types=1);

use Slim\App;
use Nyholm\Psr7\Factory\Psr17Factory;
use Psr\Http\Message\ResponseInterface;
use Compwright\SwoolePsr7\SwooleServerRequestConverter;
use Compwright\SwoolePsr7\SwooleResponseConverter;
use Swoole\Http\Response;
use Swoole\Http\Request;

include 'vendor/autoload.php';

$serverRequestFactory = new SwooleServerRequestConverter(
    new Psr17Factory(),
    new Psr17Factory(),
    new Psr17Factory(),
    new Psr17Factory()
);

$app = new App(new Psr17Factory());
$app->get('/hello/{name}', function ($request, ResponseInterface $response, $args) {
    $response->getBody()->write("Hello, " . $args['name']);
    return $response->withHeader('X-Powered-By','compwright');
});

$http = new swoole_http_server('0.0.0.0', 9501);
$http->on('start', function ($server) {
    echo "Swoole server started at http://127.0.0.1:9501\n";
});

$http->on('request', function (Request $request, Response $response) use ($serverRequestFactory, $app) {
    $psr7Request = $serverRequestFactory->createFromSwoole($request);
    $psr7Response = $app->handle($psr7Request);
    $converter = new SwooleResponseConverter($response);
    $converter->send($psr7Response);
});

$http->start();
```

## Reference
- https://github.com/slimphp/Slim/tree/4.x
- https://github.com/slimphp/Slim-Psr7
- https://github.com/swoft-cloud/swoft-http-message
- https://github.com/zendframework/zend-expressive-swoole

