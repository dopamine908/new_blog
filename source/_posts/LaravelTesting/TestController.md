---
title: Testing Laravel - 測試 Controller
date: 2021-07-08 14:25:44
tags:
  - Test
  - Laravel
  - PHP
  - Controller
categories:
  - Testing Laravel
cover: https://picsum.photos/id/660/800/600
---

# Testing Laravel - 測試 Controller

本文章使用的開發環境為
PHP : 8.0.8
Laravel : 8.51.0

--- 

在 Laravel 的開發過程中我們想要對 response 寫測試的時候，比較常見的是使用官方 [HTTP Tests](https://laravel.com/docs/http-tests) 章節內提到[對於 api 回傳的 json 去做驗證](https://laravel.com/docs/http-tests#testing-json-apis)

但我們也很常遇到前後端沒有拆分的那麼乾淨的專案，這樣的專案會使用 SSR(Server Side Render) 形式去回傳 response ，這時候我們通常會在 laravel 的 controller 回傳一個 view 然後夾帶一些需要使用的參數

假設我們今天的 ```web.php``` 有一個 route

**web.php**
```php=
Route::get('demo_controller_test', DemoTestController::class);
```

對應到我們的 ```DemoTestController```

**DemoTestController**
```php=
class DemoTestController extends Controller
{
    public function __invoke()
    {
        return view(
            'demo_controller_test',
            [
                'title' => 'test title',
                'content' => 'test content'
            ]
        );
    }
}
```

如何對於回傳的 view 以及挾帶的參數做測試呢？以下會介紹幾個好用的斷言

## assertViewIs

如果我想要驗證我回傳的 view 是不是對應到我期望的 blade 檔，我們可以用 ```assertViewIs()```

**DemoTestControllerTest**
```php=
class DemoTestControllerTest extends TestCase
{
    /**
     * @test
     */
    public function invoke()
    {
        /**Actual*/
        $response = $this->get('demo_controller_test');

        /**Assert*/
        // 驗證回傳的 view 是否正確
        $response->assertViewIs('demo_controller_test');
    }
}
```

## assertViewHas

如果我們想要驗證挾帶的參數及其值是否正確，可以使用 ```assertViewHas()```

**DemoTestControllerTest**
```php=
class DemoTestControllerTest extends TestCase
{
    /**
     * @test
     */
    public function invoke()
    {
        /**Actual*/
        $response = $this->get('demo_controller_test');

        /**Assert*/
        // 驗證與 view 一起回傳的參數是否存在且值為正確（個別驗證）
        $response->assertViewHas('title', 'test title');
        $response->assertViewHas('content', 'test content');
    }
}
```

```assertViewHas()``` 可以只放一個參數，如 ```$response->assertViewHas('title')``` ，這樣就只會驗證有沒有傳遞 ```title``` 的參數而不會驗證他的值是否為期望的

## assertViewHasAll

如果覺得 ```assertViewHas()``` 要逐個去驗證太麻煩，你也可以使用 ```assertViewHasAll()``` 一次性的驗證所有的參數跟他的值

**DemoTestControllerTest**
```php=
class DemoTestControllerTest extends TestCase
{
    /**
     * @test
     */
    public function invoke()
    {
        /**Actual*/
        $response = $this->get('demo_controller_test');

        /**Assert*/
        // 驗證與 view 一起回傳的參數是否存在且值為正確（全部驗證）
        $response->assertViewHasAll(
            [
                'title' => 'test title',
                'content' => 'test content'
            ]
        );
    }
}
```

## assertViewMissing

當然除了驗證我們期望挾帶的參數外，我們有些參數是無論如何都不希望他被不小心夾帶到 blade 去了，例如使用者的密碼之類的。這時候就可以用 ```assertViewMissing()``` 去確保不夾帶某個參數

**DemoTestControllerTest**
```php=
class DemoTestControllerTest extends TestCase
{
    /**
     * @test
     */
    public function invoke()
    {
        /**Actual*/
        $response = $this->get('demo_controller_test');

        /**Assert*/
        // 驗證與 view 一起回傳的參數不帶有某個參數
        $response->assertViewMissing('password');
    }
}
```
