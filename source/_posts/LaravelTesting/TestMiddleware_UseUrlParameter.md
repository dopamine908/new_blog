---
title: Testing Laravel - 測試 Middleware - 取用 url 路徑上的變數
date: 2021-07-30 14:25:44
tags:
  - Test
  - Laravel
  - PHP
  - Middleware
categories:
  - Testing Laravel
cover: https://picsum.photos/id/137/800/600
---

# Testing Laravel - 測試 Middleware - 取用 url 路徑上的變數

本文章使用的開發環境為
PHP : 8.0.8
Laravel : 8.51.0

---

這週在開發上遇到了一個情境，我想要對某一個 Route 的多個 Middleware 中的其中一個 Middleware 去寫測試

**帶有多個 Middleware 的 Route**
```php=
Route::get('lets_test_middleware/{message}', function () {})
    ->middleware([
       OtherMiddleware1::class,
       WaitForTestMiddleware::class, // 測試目標
       OtherMiddleware2::class,
    ])
    ->name('lets_test_middleware');
```

而我在 ```WaitForTestMiddleware``` 中則會取用到 ```lets_test_middleware/{message}``` 路徑上的 ```message``` 參數，假設我們會把他放到 ```Session``` 中好了

**WaitForTestMiddleware**
```php=
class WaitForTestMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        $message = $request->route()->parameter('message');
        Session::put('message', $message);
        return $next($request);
    }
}
```

一開始我寫的測試像是這樣

**WaitForTestMiddlewareTest**
```php=
class WaitForTestMiddlewareTest extends TestCase
{
    /**
     * @test
     */
    public function handle()
    {
        // Arrange
        $request = Request::create(route('lets_test_middleware', ['message' => 'hello']));
        $WaitForMiddleware = new WaitForTestMiddleware();

        // Actual
        $response = $WaitForMiddleware->handle($request, function (Request $passRequest) {
        });

        // Assert
        $this->assertEquals('hello', Session::get('message'));
    }
}
```

在執行測試的時候卻遇到一個錯誤

```
Error : Call to a member function parameter() on null
```

這讓人滿納悶的，明明我是依照 ```Request``` 提供的 Api 去生成 ```$request``` ，為什麼會在 ```$request->route()``` 得到 ```null``` 的結果呢？

經過一段時間的搜尋跟深入底層的 code 去釐清，最後參照這篇文章 [Simulate a http request and parse route parameters in Laravel testcase](https://newbedev.com/simulate-a-http-request-and-parse-route-parameters-in-laravel-testcase) ，終於大概理解原由了

其原因在於，當我們呼叫 ```$request->route()``` 的時候

**[src/Illuminate/Http/Request.php#L530](https://github.com/laravel/framework/blob/8.x/src/Illuminate/Http/Request.php#L530)**
```php=
 public function route($param = null, $default = null)
{
    $route = call_user_func($this->getRouteResolver());

    if (is_null($route) || is_null($param)) {
        return $route;
    }

    return $route->parameter($param, $default);
}
```

由上我們可以看到在第 3 行處，我們會先去呼叫 ```getRouteResolver()```

所以這邊我們繼續往下追，看一下 ```getRouteResolver()```

**[src/Illuminate/Http/Request.php#L601](https://github.com/laravel/framework/blob/8.x/src/Illuminate/Http/Request.php#L601)**
```php=
public function getRouteResolver()
{
    return $this->routeResolver ?: function () {
        //
    };
}
```

由此可知， ```$this->routeResolver``` 是未設定過的，這是因為當在 Laravel 接真實的 HTTP Request 的時候，在生成 Request 的生命週期中會去設定 ```routeResolver``` ，但我們在測試中手動去生成的時候卻沒有經過設定的環節

所以，其實我們只要補上 ```routeResolver``` 的設定就可以了，也正好 Request 物件有提供我們一個 ```setRouteResolver()``` 方法可以使用，底下我將測試改寫為有設置 ```routeResolver``` 的版本

**WaitForTestMiddlewareTest**
```php=
class WaitForTestMiddlewareTest extends TestCase
{
    /**
     * @test
     */
    public function handle()
    {
        // Arrange
        $request = Request::create(route('lets_test_middleware', ['message' => 'hello']));
        $request->setRouteResolver(
            function () use ($request) {
                return (new Route('GET', 'lets_test_middleware/{message}', []))
                    ->bind($request);
            }
        );
        $WaitForMiddleware = new WaitForTestMiddleware();

        // Actual
        $response = $WaitForMiddleware->handle($request, function (Request $passRequest) {
        });

        // Assert
        $this->assertEquals('hello', Session::get('message'));
    }
}
```

設定完之後我們就可以如期的驗證了

## Reference

- [Simulate a http request and parse route parameters in Laravel testcase](https://newbedev.com/simulate-a-http-request-and-parse-route-parameters-in-laravel-testcase)
- [Testing Middleware in Laravel with PHPUnit](https://semaphoreci.com/community/tutorials/testing-middleware-in-laravel-with-phpunit)
