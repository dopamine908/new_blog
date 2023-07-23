---
title:  PHP8 - Named Arguments
date: 2021-07-29 14:25:44
tags:
  - PHP
  - PHP8
categories:
  - PHP
cover: https://picsum.photos/id/137/800/600
---

# PHP8 - Named Arguments

## 介紹

今天來介紹一個 PHP8 新增的功能，叫做 「Named Arguments」，如字面上的意思，我們可以為一個參數做命名的動作。

簡單的示範一下

```php=
public function demo($foo, $bar)
{
}

// php 8 以前
demo('fooooooo', 'baaaaaar');

// php 8 之後，我們可以在輸入的值前面去指定我要丟的參數
demo(foo: 'fooooooo', bar: 'baaaaaar');
```

## 優點

讓執行函數的時候可以指定參數名稱有幾個好處

### 更好的可讀性

```php=
public function demo($foo, $bar)
{
}

demo(foo: 'fooooooo', bar: 'baaaaaar');
```

在呼叫函式的時候，我們可以很清楚的知道我們的每一個值是要給哪一個函式內的變數使用，以往我們可能會因為命名上的差異導致沒能那麼容易的聯想被呼叫的函式變數的原始命名意義，但是一但可以指定之後，我們從呼叫的地方就可以看出每個變數的原始命名意義。

### 更靈活的函式呼叫

以往在呼叫函式的時候變數都是用順序去做對應，但是在可以指定名稱之後，我們的變數不一定要按照順序，如下

```php=
public function demo($foo, $bar)
{
}

// 我們會這樣呼叫
demo(foo: 'fooooooo', bar: 'baaaaaar');
// 但順序不同也是會成功對應
demo(bar: 'baaaaaar', foo: 'fooooooo');

```

雖然這樣有時候看起來會比較混亂，但在我有指定的狀況下他仍然可以被成功呼叫

也就是說，我在重構或修改程式碼的過程若中即使是調換參數順序，原本的程式碼也不會出錯，只要我不對參數重新命名的話就不會喪失參數的指定對應

我們比較需要擔心的情況是：如果我們將參數重新命名怎麼辦？這樣不是會需要去每一個有呼叫該函式的地方修改對應的指定嗎？
沒錯，但如果有搭配好的重構工具使用就可以幫助你將全部使用到該變數的地方一並重新命名（據筆者所知 phpstorm 原生可以辦到，至於其他編輯器可能需要找一下支援套件）

下面來看另外一種情況，我們有時候會給函式的變數一些預設值，當該函式有多個預設值的時候，常常我們在呼叫函式的時候就必須手動去指定某些參數的預設值

底下的範例中我想讓 ```$param2``` 為預設值，但 ```$param3``` 要輸入我另外指定的值，我們來看看差異

```php=
public function demo($param1, $param2 = 'default', $param3 = 'default')
{
}

// php 8 以前，我們要手動把 $param2 的預設值丟進去
demo('value1', 'default', 'value3');

// php 8 之後，我們可以跳過 $param2 直接指定我要給 $param3 什麼值
// 這時候 $param2 就會帶入預設值
demo(param1: 'value1', param3: 'value3');

```

使用了命名參數指定之後我們可以更簡潔的去呼叫有多個預設值值的函式，不用硬給一堆看起來有點突兀的預設值


## 限制

當然的，看起來如此便利優秀的功能自然是有一些限制，以下會稍微列出 Named Arguments 在使用上的限制

### 不可重複指定

已經指定過一次的參數，不能重複指定

```php=
public function demo($foo)
{
}

// Error: Named parameter $foo overwrites previous argument
demo(foo: 'fooooooo1', foo: 'foooooooo2');
```

### 沒有命名過的則不可以指定

如標題的意思，沒有被命名過的參數，在被指定的時候想當然爾會對應不到

```php=
function demo($foo) { ... }
 
// Error: Unknown named parameter $bar
test(bar: 'baaaaaar');
```

### 注意沒有使用的參數會參照預設行為

PHP 8 雖然新增了 Named Arguments 的功能，但是默認上並不是強制使用，那麼在我們混用的狀況下就要特別注意一點，當我們的參數未指定的時候， PHP 在執行的時候就會使用以往那種以參數的順序去對應參數的方式去做呼叫，所以當我們是混用的狀況下需要特別注意命名參數指定的順序

```php=
public function demo($foo, $bar)
{
}

// Error : Named parameter $foo overwrites previous argument
demo('foo', foo: 'fooo');

// PHP Fatal error:  Cannot use positional argument after named argument
demo(foo: 'foo', 'fooo');
```

不過比較好的方式是儘量不要混用啦，比較不會因為順序的問題出現不必要的錯誤

### 繼承或是實作介面的時候有可能改變 Named Arguments

有一種狀況也常常會遇到，就是當我們要去實作一個介面，或是我們想要去繼承某個 class 並覆寫他的方法時，有時候我們會去修改函式的變數名稱，在這種時候 Named Arguments 的判定會以覆寫掉的為主，我們可以看以下的兩個例子

**implement interface**
```php=
interface FooContract
{
    public function demo($foo);
}

class Foo implements FooContract
{
    // 這邊複寫變數名稱，從 $foo 改為 $bar
    public function demo($bar)
    {
    }
}

$Foo = new Foo();

// Error : Unknown named parameter $foo
$Foo->demo(foo: 'foo');

// pass
$Foo->demo(bar: 'foo');
```

**extend class and overwrite**
```php=
abstract class Father
{
    abstract public function say(string $father_message);
}

class Son extends Father
{
    // 這邊複寫變數名稱，從 $father_message 改為 $son_message
    public function say(string $son_message)
    {
    }
}

$Son = new Son();

// Error : Unknown named parameter $father_message
$Son->say(father_message: 'hi');

// pass
$Son->say(son_message: 'hi');
```

## Reference
- [PHP RFC : Named Arguments](https://wiki.php.net/rfc/named_params)
