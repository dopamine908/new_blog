---
title: PHP8 - Match Expression
date: 2021-08-13 14:25:44
tags:
  - PHP
  - PHP8
categories:
  - PHP
cover: https://picsum.photos/id/137/800/600
---

# PHP8 - Match Expression

PHP 8 新增了一種叫做 match 的方法，幾乎是可以完全取代我們原本熟知的 switch ，同時帶來了一些好處，以下我們看一段範例比較一下 switch 跟 match

**Switch v.s. Match**

```php=
// switch
switch ($operator) {
    case 'plus':
        $result = $a + $b;
        break;
    case 'minus':
        $result = $a - $b;
        break;
    case 'product':
        $result = $a * $b;
        break;
    case 'division':
        $result = $a / $b;
        break;
}

// match
$result = match ($operator) {
    'plus' => $a + $b,
    'minus' => $a - $b,
    'product' => $a * $b,
    'division' => $a / $b,
};
```

## match 帶來更多元且靈活的判斷

比起使用 switch 的時候，我們可以在 match 的判斷比對中放入更複雜的比對邏輯、函式呼叫，使得判斷過程能夠在同一個 scope 中完成

```php=
$x = 'foo';
$result = match (true) {
    $this->conditionA() => ...,
    $this->conditionB($x) => ...,
    $this->bar => ...,
    default => ...
};
```

## Switch 與 Match 的差異

幾乎所有的 switch 都可以轉化成 match ，但這兩種語法還是存在著一些執行上的差異，底下會一一列舉

### Switch 與 Match 的條件比對

Switch 與 Match 在做判斷的時候，使用的比對運算是不相同的

- switch 使用的是 ``` == ```
- match 使用的是 ``` === ```

match 比起 switch ，但斷上更為嚴格，且比較不會因為 PHP 弱型別的主動轉型造成問題，底下我們看個例子

```php=
// switch
switch ('1') {
    case 1:
        $switchResult = "Oh no!\n";
        break;
    case '1':
        $switchResult = "This is what I expected\n";
        break;
}
echo 'switchResult : ';
echo $switchResult; // output : switchResult : Oh no!


// match 
$matchResult = match ('1') {
    1 => "Oh no!\n",
    '1' => "This is what I expected\n",
};
echo 'matchResult : ';
echo $matchResult; // matchResult : This is what I expected
```

### 當 switch 寫了 case 卻沒有 break 的時候

撰寫 switch 的時候需要搭配 break，不然會繼續往下執行，這樣的特性或許在某些特殊用法中很不錯，但一般來說我們既不期望他繼續往下執行，也難免會忘記寫上 break，而如果使用 match ，他會在判斷成功的那一行就強制跳出去，不會再繼續往下執行，可以參考下面的例子

```php=
// switch
echo "----- switch result ----- \n";
switch ('foo') {
    case 'foo':
        echo "Oh foo!\n";
        // 這邊忘記寫 break 了！
    case 'bar':
        echo "Oh bar!\n";
        break;
}
// optput
----- switch result ----- 
Oh foo!
Oh bar!

// match
echo "----- match result ----- \n";
echo match ('foo') {
    'foo' => "Oh foo!\n",
    'bar' => "Oh bar!\n",
};
// output
----- match result ----- 
Oh foo!
```

### 多條件對應同結果的時候

我們常常會遇到多種比對條件必須合併對應到某個相同結果的時候，這時候在 switch 及 match 的差異如下

```php=
// switch
echo "----- switch result ----- \n";
switch ('bar') {
    case 'foo':
    case 'bar':
        echo "bingo!\n";
        break;
}
// output

// match
----- switch result ----- 
bingo!
echo "----- match result ----- \n";
echo match ('bar') {
    'foo', 'bar' => "bingo!\n",
};
// output
----- match result ----- 
bingo!
```

### 錯誤訊息的比較

有時候在寫 switch 我們會漏掉一些 case 沒有定義，這時候的錯誤訊息與我們將這樣的 switch 轉換為 match 不同，match 的錯誤訊息會比 switch 的更加精確一些，看以下範例

```php=
// switch
echo "----- switch result ----- \n";
switch ('-') {
    case '+':
        $switchResult = "plus\n";
        break;
}
echo $switchResult;
// ErrorException : Undefined variable $switchResult

// match
echo "----- match result ----- \n";
$matchResult = match ('-') {
    '+' => "plus\n",
};
echo $matchResult;
// UnhandledMatchError : Unhandled match value of type string
```

## Reference

[PHP RFC : Match expression v2](https://wiki.php.net/rfc/match_expression_v2)
