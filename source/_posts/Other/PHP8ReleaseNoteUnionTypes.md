---
title: PHP8 - Union Types
date: 2021-08-08 14:25:44
tags:
  - PHP
  - PHP8
categories:
  - PHP
cover: https://picsum.photos/id/137/800/600
---

# PHP8 - Union Types

在 PHP 8 之前，雖然已經有支援 type hint ，但如果我的 function 回傳有可能不只一種型別的時候，我們只能透過 phpdoc 去描述這件事情，而且同時我可能要把原本設置的單一型別 type hint 撤除，這樣的方式在實務層面缺乏了主動的檢查機制

**PHP 8 以前回傳多 type 的寫法**
```php=
/**
 * @param int|float $number
 * @return int|float
 */
 public function test($number) {
     return $number;
 }
```

不過有一個特例，在 PHP 8 之前有支持一種多形別的 type hint ，算是我們常常用到的情形，當今天我們的變數是可以指定爲某種 type 或是 null 的時候，我們可以這樣指定

**變數接受 array 或是 null**
```php=
/**
 * @param array|null $arr
 * @return array|null
 */
public function test(?array $arr): ?array
{
    return $arr;
}
```

但是在更廣泛的狀況下就別無選擇只能單靠 phpdoc 去指定多型別的回傳了，不過 PHP 8 新增了一個 **Union Types** 的 feature ，這使得這個窘境開始有了解套

## Union Types

如果們習慣在 phpdoc 裡面描述的多型別參數或回傳的寫法，在 PHP 8 中你可以直接使用這樣的寫法套用在你的 function 上

**Union Types 範例**
```php=
/**
 * @param array|string $var
 * @return array|string
 */
public function test(array|string $var): array|string
{
    return $var;
}
```

在上面的例子中，我設定我的 ```$var``` 參數只接納 ```array``` 、 ```string``` 而且也只能回傳 ```array``` 、 ```string``` 

這樣一來，程式碼會變得更明確，更容易在開發的時候對於會遇到的型別能夠一併考量，減少錯誤。而且這樣的型別限制對 IDE 來說相當友善，IDE 對於你取用變數裡面的 field 、 function 的提示會更加精確。

## 限制

但 Union Types 也是有些限制的，下面有幾個例子

### 重複宣告

```php=
function foo(): int|INT {} // Disallowed
function foo(): bool|false {} // Disallowed
 ```
 
 ### 繼承型別衝突
 
```php=
use A as B;
function foo(): A|B {} // Disallowed ("use" is part of name resolution)
```

## Reference

- [PHP RFC : Union Types 2.0](https://wiki.php.net/rfc/union_types_v2)
