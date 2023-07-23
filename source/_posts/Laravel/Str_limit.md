---
title: Laravel Helper - Str::limit() 在中文應用場景的大小事
date: 2021-04-22 14:25:44
tags:
  - PHP
  - Laravel
  - Laravel Helper
categories:
  - Laravel
cover: https://picsum.photos/id/660/800/600
---

# Laravel Helper - Str::limit() 在中文應用場景的大小事

最近在開發上遇到了要擷取中文訊息內容預覽的問題，例如今天我有一個句子

> 萊布尼茨和牛頓都被普遍認為是獨立的微積分發明者。牛頓最先將微積分應用到普通物理當中，而萊布尼茨創作了不少今天在微積分所使用的符號。牛頓、萊布尼茨都給出了微分、積分的基本規則，二階與更高階導數，近似多項式級數的記法等。在牛頓的時代，微積分基本定理是已知的事實。

擷取自 [微積分 wiki](https://zh.wikipedia.org/wiki/%E5%BE%AE%E7%A7%AF%E5%88%86%E5%AD%A6)

這很可能是某個文章的內容，而我在顯示頁面上的預覽只有要顯示部分的文字，例如前十個字好了，而且我想讓被隱藏的文字用「...」代替，我想顯示的內容大概如下

> 萊布尼茨和牛頓都被普...

這時候我們就需要用到 [Laraevl Helper](https://laravel.com/docs/8.x/helpers#introduction) 裡面幫我們預先寫好的字串處理函式了

我們要使用的是 ```Str::limit()``` 這個函式，它可以幫助我們擷取某個字串的前幾個字，並且可以自定義超過字數之後的結尾

## [Str::limit](https://laravel.com/docs/8.x/helpers#method-str-limit)

先來看看官網的例子

```php=
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

但是當我們的文字是中文的時候，使用起來結果會如下

```php=
use Illuminate\Support\Str;

$post = '萊布尼茨和牛頓都被普遍認為是獨立的微積分發明者。';
$truncated = Str::limit($post, 10);

// 萊布尼茨和...
```

結果跟我們預想的差蠻多的，明顯比我們想要限制的10的字還要少，所以我們就來研究一下背後到底做了什麼事情

首先，我們先去找到 ```Str::limit()``` 的 [source code](https://github.com/laravel/framework/blob/8.x/src/Illuminate/Support/Str.php#L344)

**Str::limit()**
```php=
/**
 * Limit the number of characters in a string.
 *
 * @param  string  $value
 * @param  int  $limit
 * @param  string  $end
 * @return string
 */
public static function limit($value, $limit = 100, $end = '...')
{
    if (mb_strwidth($value, 'UTF-8') <= $limit) {
        return $value;
    }

    return rtrim(mb_strimwidth($value, 0, $limit, '', 'UTF-8')).$end;
}
```

### $end

首先我們可以注意一下 input 最後一個變數 ```$end``` ，在上面的例子中都沒有賦予 ```$end``` 值，但這邊是可以給他設定值的，給了設定值之後結尾的 "..." 就可以被置換成你想要換的字串

```php=
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, '~~~');

// The quick brown fox~~~
```

### 中文字串字數判定

接著我們看到 [11 行的位置](https://github.com/laravel/framework/blob/8.x/src/Illuminate/Support/Str.php#L346)

這邊的用意是當我給定的字串 ```$value``` 比我想要擷取的字串長度還要來得短的時候，就直接回傳原本的結果（並且不會加上 ```$end```）

這邊值得注意的是用來判斷字串長度的 ```mb_strwidth()``` ，我們來看一下 [php 官方的文件](https://www.php.net/manual/en/function.mb-strwidth.php)

> ```mb_strwidth ( string $string , string|null $encoding = null ) : int```
>
> Returns the width of string string, where halfwidth characters count as 1, and fullwidth characters count as 2. See » http://www.unicode.org/reports/tr11/ for details regarding East Asian character widths.

大致上的意思就是當你輸入的文字為半形的時候，他會回傳 1，而當你輸入的文字是全形的時候，就會回傳 2，也就是說，當我輸入「你好」的時候，我會得到 4 的回傳值

而底下 [15 行的位置](https://github.com/laravel/framework/blob/8.x/src/Illuminate/Support/Str.php#L350) 使用到的 ```mb_strimwidth()``` 也是同理，我們來看一下[文件的描述](https://www.php.net/manual/en/function.mb-strimwidth.php)

> mb_strimwidth ( string $string , int $start , int $width , string $trim_marker = "" , string|null $encoding = null ) : string
>
> Truncates string string to specified width, where halfwidth characters count as 1, and fullwidth characters count as 2. See » http://www.unicode.org/reports/tr11/ for details regarding East Asian character widths.

```mb_strimwidth()``` 在擷取字串的時候所使用的長度判斷就是上方提到的那套依據全形半形去做依據的判斷方式，所以如果我想要擷取的是中文字 10 個字，那麼我應該輸入的值是 20 ，而不是 10

```php=
use Illuminate\Support\Str;

$post = '萊布尼茨和牛頓都被普遍認為是獨立的微積分發明者。';
$truncated = Str::limit($post, 20, '...');

// 萊布尼茨和牛頓都被普...
```
