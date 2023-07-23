---
title: 單元測試的藝術 - Ch8 好的單元測試的支柱 - Sec. 8.2 撰寫可維護的測試
date: 2021-06-13 13:42:52
tags:
  - Unit Test
  - Test
  - 讀書會報告
categories:
  - 讀書會報告
cover: https://picsum.photos/id/153/800/600
---

# 單元測試的藝術 - Ch8 好的單元測試的支柱 - Sec. 8.2 撰寫可維護的測試

隨著時間推移，測試會變得難以理解及維護，所以可維護性是大多數開發人員撰寫單元測試的核心問題之一。

## 8.2.1 測試私有或保護的方法

在測試時，你應該只考慮公開契約（整體功能），因爲私有方法不會獨立存在，他必定會被某個公開方法或是私有方法呼叫，也就是說，任何私有方法通常都是更大的工作單元的使用案例的一部分。

如果一個私有方法值得被測試，那他也許應該是公開的、靜態的，或是至少是內部的（ internal ），或者應該將它獨立到另一個類別中。

### 讓方法變成公開方法

如果你覺得某個私有方法具有被測試的價值，那可能表示這個方法對於呼叫端來說有某種已知的行為或契約的效用，將這個方法改成公開的，代表正式將契約對外開放

### 把方法抽取到新類別中

如果一個方法中包含了很多獨立職責的邏輯，或是類別的某些狀態只和這個方法有關，把這個方法抽取到另外一個具有特定功能的類別會更好，也符合單一職責原則。

### 把方法改成靜態方法

如果一個方法沒有用到執行物件的任何變數跟狀態，那你可以考慮把它改為靜態方法，同時這也表示了這個方法為某種輔助方法。

### 把方法改為內部（ internal ）方法 - For C#

如果上述的方法都行不通，你也可以考慮將方法設置為 internal ，並且在測試中設定 ```[InternalsVisibleTo("Test- Assembly")]``` ，讓測試專案也可以呼叫到該方法。

使用內部方法並不能提升測試的維護性，但如果這個方法公開為明確的公開契約，你比較可以確保開發人員知道，這個方法有一個真正公開的契約，不能隨意破壞。

## 8.2.2 去除重複的程式碼

DRY 原則也同樣適用於測試程式碼，如果測試中有很多重複的程式碼，那表示當我們需要修改測試時，就得被迫修改更多的測試程式碼。

底下我們來看一個例子

**before**


**LogAnalyzer**
```php=
class LogAnalyzer
{
    public function isValid(string $fileName): bool
    {
        if (strlen($fileName) < 8) {
            return true;
        }
        return false;
    }
}
```

**Test**
```php=
/**
 * @test
 */
public function IsValid_LengthBiggerThan8_IsFalse(): void
{
    //Arrange
    $filename = '123456789';
    $LogAnalyzer = new LogAnalyzer();

    //Actual
    $actual = $LogAnalyzer->isValid($filename);

    //Assert
    $this->assertFalse($actual);
}

/**
 * @test
 */
public function IsValid_LengthSmallerThan8_IsTrue(): void
{    //Arrange
    $filename = '1234567';
    $LogAnalyzer = new LogAnalyzer();

    //Actual
    $actual = $LogAnalyzer->isValid($filename);

    //Assert
    $this->assertTrue($actual);
}
```

如果當今天 ```LogAnalyzer``` 的使用方式發生變化，每個測試方法都得獨立的去進行調整，這樣維護的工作量會變大

**after**

**LogAnalyzer**
```php=
class LogAnalyzer
{
    /**
     * @var bool $initialize 是否初始化
     */
    private $initialize = false;

    public function initialize(): void
    {
        // do something initialize
        $this->initialize = true;
    }

    public function isValid(string $fileName): bool
    {
        if ( ! $this->initialize) {
            throw new Exception('not initialize');
        }

        if (strlen($fileName) < 8) {
            return true;
        }
        return false;
    }
}
```

改動了 ```LogAnalyzer``` 之後，之前寫的兩個測試會全部壞掉，因爲沒有進行過初始化的動作，所以我這邊必須去修改兩個測試重複的程式碼部分，讓他們都有好好的初始化。

這邊有兩種策略可以使用

1. 使用輔助方法
2. 使用 ```setUp()``` 方法

### 使用輔助方法來去除重複的程式碼

我們將創造 ```LogAnalyzer``` 及 ```initialize()``` 的過程封裝為一個工廠方法，並讓所有的測試都去呼叫這個工廠方法來拿取 ```LogAnalyzer```

**Test**
```php=
/**
 * @test
 */
public function IsValid_LengthBiggerThan8_IsFalse(): void
{
    //Arrange
    $filename = '123456789';
    $LogAnalyzer = $this->getNewLogAnalyzer();

    //Actual
    $actual = $LogAnalyzer->isValid($filename);

    //Assert
    $this->assertFalse($actual);
}

/**
 * @test
 */
public function IsValid_LengthSmallerThan8_IsTrue(): void
{    //Arrange
    $filename = '1234567';
    $LogAnalyzer = $this->getNewLogAnalyzer();

    //Actual
    $actual = $LogAnalyzer->isValid($filename);

    //Assert
    $this->assertTrue($actual);
}

/**
 * @return LogAnalyzer
 */
private function getNewLogAnalyzer(): LogAnalyzer
{
    $LogAnalyzer = new LogAnalyzer();
    $LogAnalyzer->initialize();
    return $LogAnalyzer;
}
```

### 使用 ```setUp()``` 方法來去除重複的程式碼

也可以在 ```setUp()``` 方法中做初始化

**Test**
```php=
/**
 * @var LogAnalyzer
 */
private $LogAnalyzer;

public function setUp(): void
{
    parent::setUp();
    $this->LogAnalyzer = new LogAnalyzer();
    $this->LogAnalyzer->initialize();
}

/**
 * @test
 */
public function IsValid_LengthBiggerThan8_IsFalse(): void
{
    //Arrange
    $filename = '123456789';

    //Actual
    $actual = $this->LogAnalyzer->isValid($filename);

    //Assert
    $this->assertFalse($actual);
}

/**
 * @test
 */
public function IsValid_LengthSmallerThan8_IsTrue(): void
{    //Arrange
    $filename = '1234567';

    //Actual
    $actual = $this->LogAnalyzer->isValid($filename);

    //Assert
    $this->assertTrue($actual);
}
```

使用 ```setUp()``` ，不需要在每個測試中都包含一行建立 ```LogAnalyzer``` 的語句，在執行每個測試之前  ```setUp()``` 都會執行。

使用 ```setUp()``` 來去除重複程式碼不是一個好的方式，其原因下一節會做討論。

## 8.2.3 具可維護性的設計來使用 ```setUp()``` 方法

```setUp()``` 方法使用起來相當容易，可是他卻有他的局限性

- ```setUp()``` 方法只用於需要進行初始化的工作時
- ```setUp()``` 方法並不總是去除重複程式碼的最佳方式，有時候重複程式碼的目的並不是初始化
- ```setUp()``` 方法沒有參數或者回傳值
- ```setUp()``` 方法因為沒有回傳值，也就無法當作物件生成的工廠方法使用
- ```setUp()``` 方法應該只包含適用於目前測試類別中所有測試方法中的程式碼，否則會使測試更加難以理解

基於這些局限性，開發者通常會用以下的方式去濫用 ```setUp()``` 方法

- 在 ```setUp()``` 方法中初始化只有某一些測試方法會用到的物件
- ```setUp()``` 方法內的程式碼冗長難懂
- ```setUp()``` 方法中準備模擬物件及假物件

### 初始化只在某些測試會使用到的物件

我們來看一個例子

**Test**
```php=
/**
 * @var LogAnalyzer
 */
private $LogAnalyzer;

private $file_info;

public function setUp(): void
{
    parent::setUp();
    $this->LogAnalyzer = new LogAnalyzer();
    $this->LogAnalyzer->initialize();
    $this->file_info = 'c://someFile.txt';
}

/**
 * @test
 */
public function IsValid_LengthBiggerThan8_IsFalse(): void
{
    //Arrange
    $filename = '123456789';

    //Actual
    $actual = $this->LogAnalyzer->isValid($filename);

    //Assert
    $this->assertFalse($actual);
}

/**
 * @test
 */
public function IsValid_LengthSmallerThan8_IsTrue(): void
{    //Arrange
    $filename = '1234567';

    //Actual
    $actual = $this->LogAnalyzer->isValid($filename);

    //Assert
    $this->assertTrue($actual);
}

/**
 * @test
 */
public function IsValid_BadFileInfoInput_ReturnsFalse(): void
{    //Arrange
    $filename = $this->file_info;

    //Actual
    $actual = $this->LogAnalyzer->isValid($filename);

    //Assert
    $this->assertFalse($actual);
}
```

上述測試程式碼為什麼不好維護呢？因為我們在維護的時候大概會經過以下的過程

1. 讀一遍 ```setUp()``` 方法，理解初始化了什麼
2. 假設所有的測試都會用到 ```setUp()``` 方法
3. 在維護的過程發現自己的假設錯誤，進而花時間去找誰沒有使用到 ```setUp()``` 方法的內容
4. 花更多時間來理解測試的意圖跟內容

### 冗長難懂的 ```setUp()``` 方法

開發者往往會在 ```setUp()``` 方法中初始化很多東西，導致程式可讀性變差。要解決這個問題，可以把初始化的過程都提取整輔助方法，並用方法名稱去做語意化的命名，讓人容易去理解初始化的過程。

### 在 ```setUp()``` 方法中準備模擬物件或假物件

在 ```setUp()``` 方法中準備模擬物件或假物件會使可讀性變差，也不好維護。

作者建議可以把假物件的生成寫成輔助方法配合語意化的命名，這樣在呼叫的時候也能夠確切知道發生什麼事情。

**作者呼籲大家停止使用 ```setUp()``` 方法**

## 8.2.4 實作測試隔離

如果沒有將測試隔離，他們就會互相影響，當出現問題時，往往需要花費更大量的時間去定位真正的問題所在。如果你的測試執行起來有下面這些跡象，很有可能表示你的測試有隔離上的問題。

- 強制的測試順序：測試需要以某種特定的順序去執行
- 存在呼叫其他測試的隱藏動作：在測試中呼叫其他的測試
- 共享狀態損毀：測試共享記憶體裡面的某個狀態，卻沒有將它重置
- 外部共享狀態損毀：整合測試共享資源，卻沒有重置資源

底下我們一一來討論這幾種反模式

### 強制的測試順序

大多數的測試平台，並不能夠保證測試按某種特定的順序執行，因此很有可能這次會過的測試，在下一次執行的時候因為順序不同就變成不通過的，也會進而產生許多問題

- 使用新版本的測試框架，導致執行順序改變，進而讓測試執行失敗
- 只執行一部分的測試，其測試結果可能與執行所有測試，或者執行其他部分的測試結果不同
- 測試維護成本提升，因為你必須考量到測試之間的前後因果關係
- 測試可能因為錯誤的原因導致失敗或通過
- 刪除或修改測試可能影響到其他的測試結果
- 很難替測試命名，因為他測試了很多東西

有幾種常見的模式，會導致測試隔離的問題

- 流程測試或整合測試
    - 不要在單元測試撰寫流程測試的相關程式碼，可以考慮使用整合測試框架去處理流程測試或整合測試
- 開發人員因為偷懶，而沒有將測試會異動的狀態回復初始值
    - 作者建議您不要繼續從事程式開發的工作

### 隱藏的測試呼叫

測試直接呼叫同類測試或者是其他測試的測試方法，造成測試之間互相依賴，這種依賴可能導致一些問題

- 執行部分測試的結果可能會與執行全部的測試，或執行另外某些部分的測試結果不相同
- 維護測試的工作變複雜，因為必須去考慮測試之間的關係及互動
- 有可能因為某個相依的測試失敗而導致目標測試一並失敗
- 對於呼叫其他測試的測試，命名困難

問題產生的原因及解法如下

- 流程或整合測試
    - 同前面提到的，使用整合框架
- 試圖避免撰寫相同的程式碼
    - 你應該把重複的程式碼移動到新的方法，並讓兩個測試去共用這個新方法
- 懶得分隔測試
    - 把手頭上的測試當作系統中唯一的測試

### 共享狀態損毀、外部共享狀態損毀

這種反模式主要有兩種情況，且這兩種情況通常獨立存在

- 測試共享資源（如：記憶體、資料庫、檔案系統），但沒有清理或者復原成乾淨的狀態
- 測試開始前沒有設定測試所需的初始狀態

這些情況可能會有以下問題

- 執行部分測試的結果可能會與執行全部的測試，或執行另外某些部分的測試結果不相同
- 測試維護變困難，因為你的修改可能破壞其他測試所需要的狀態，而導致其他測試失敗
- 測試可能會因為錯誤的原因失敗或是通過
- 修改測試會影響測試成果，而且看起來會像是隨機發生的

問題產生的原因跟解法如下

- 沒有在每個測試執行前設定狀態
    - 善用 ```setUp()``` 或者輔助方法來協助你得到正確的狀態
- 使用共享狀態
    - 每個測試都使用一個單獨的物件執行個體
- 測試使用了靜態的物件執行個體
    - 小心留意狀態管理，可以使用 ```setUp()``` 或 ```tearDown()``` 來協助你清理狀態。若是在測試單例(Singleton)物件，可以為他新增一個方法來協助體將單利重置為乾淨的物件。

## 8.2.5 避免對不同的關注點進行多次驗證

多個關注點請看以下範例

**Operator**
```php=
class Operator
{
    public function sum(int $a, int $b): int
    {
        return $a + $b;
    }
}
```

**Test**
```php=
/**
 * @test
 */
public function Sum()
{
    $Operator = new Operator();

    // Assert
    $this->assertEquals(0, $Operator->sum(0, 0));
    $this->assertEquals(1, $Operator->sum(1, 0));
    $this->assertEquals(2, $Operator->sum(1, 1));
}
```

這個測試方法中含有多個測試，我們可以認為他測試了三種不同的狀況，而這會導致如果第一個 assert 錯誤，這個測試就會直接被宣告為失敗，這是我們不期望看到的，我們希望可以單獨觀察每一個案例的執行結果

以下幾點建議去改善這樣的模式

- 每個驗證建立一個單獨的測試
- 使用參數化測試
- 把驗證碼放在一個 try-catch 區塊中

### 參數化測試

也就是使用 ```phpunit``` 的 ```@dataProvider``` 特性

**Test**
```php=
/**
 * @test
 * @dataProvider ResultDataProvider
 */
public function Sum($expected, $a, $b)
{
    $Operator = new Operator();

    // Assert
    $this->assertEquals($expected, $Operator->sum($a, $b));
}

public function ResultDataProvider()
{
    return [
        [0, 0, 0],
        [1, 1, 0],
        [2, 1, 1],
    ];
}
```

```@dataProvider``` 會依序將你的參數及期望值帶入，並執行許多次的同個驗證，如果你的某一個 case 錯誤了，測試框架依然可以繼續執行其他 case ，並告知你錯誤的 case 的錯誤訊息

### 使用 try-catch

有人會認為 try-catch 是一個好主意，他可以攔截例外，並把例外接收到的錯誤資訊印在介面上，也可以繼續執行下一個測試斷言。但使用這種方式在可讀性上就顯得比較差勁一點，而可讀性不好就連帶的會提升維護成本，作者是建議大家用參數化測試取代 try-catch

## 8.2.6 物件比較

這邊的範例也是使用多個驗證，但他不是要驗證多種案例，而是對於某種結果進行多方面的驗證

**Log**
```php=
class Log
{
    public $driver;
    public $content;
    public $createdTime;

    public function __construct(string $driver, string $content, string $createdTime)
    {
        $this->driver = $driver;
        $this->content = $content;
        $this->createdTime = $createdTime;
    }
}
```

**LogAnalyzer**
```php=
class LogAnalyzer
{
    public function output(): Log
    {
        return new Log('file', 'this is output content', '2021-05-29');
    }
}
```

**Test**
```php=
/**
 * @test
 */
public function Output()
{
    //Arrange
    $LogAnalyzer = new LogAnalyzer();
    $expected_driver = 'file';
    $expected_content = 'this is output content';
    $expected_createdTime = '2021-05-29';

    //Actual
    $actual = $LogAnalyzer->output();

    //Assert
    $this->assertEquals($expected_driver, $actual->driver);
    $this->assertEquals($expected_content, $actual->content);
    $this->assertEquals($expected_createdTime, $actual->createdTime);
}
```

### 提高測試的可維護性

與錯驗證不同，這裡建立了一個新的物件，為他設定欄位，我們比較期望的物件是否相等，這樣寫的可讀性會優於多方面去做驗證

**Test**

```php=
/**
 * @test
 */
public function Output()
{
    //Arrange
    $LogAnalyzer = new LogAnalyzer();
    $expected_log = new Log('file', 'this is output content', '2021-05-29');

    //Actual
    $actual = $LogAnalyzer->output();

    //Assert
    $this->assertEquals($expected_log, $actual);
}
```

### 覆寫 ToString()

書上在這邊的描述是說，去覆寫 ```ToString()``` 這類的方法，使測試輸出錯誤結果的時候可以更清楚的表示錯誤，但有鑒於現在測試框架已經很進步了，像是 phpunit ，即便不做這樣的複寫我們在錯誤發生時也有很清楚的錯誤訊息可以讓我們了解錯誤發的地方，也很好去 trace code ，所以我想至少在 php+phpunit 的範疇內，應該用不上這小節所傳授的技術

## 8.2.7 避免過度指定

過度指定的測試對一個單元測試如何完成內部行為進行了假設，而不是只對於測試結果去做驗證

單元測試中過度指定主要有以下幾種情況

- 測試對於被測試物件的純內部狀態進行驗證
- 測試中使用了多個模擬物件
- 測試在需要虛設常式時，使用模擬物件
- 測試在不必要的狀況下，指定了順序或是使用了精準的參數匹配器

### 驗證被測試物件的純內部行為

單元測試應該只測試物件的公開契約和公開功能

### 在需要虛設常式時，使用模擬物件

在需要使用虛設常式時使用模擬物件，是一種常見的過度指定，請看以下例子

**Payment**
```php=
class Payment
{
    private $pay;

    public function __construct(Pay $pay)
    {
        $this->pay = $pay;
    }

    public function payMoney(string $way, int $money): bool
    {
        $result = $this->pay->pay($way, $money);
        if ($result == true) {
            return true;
        }
        return false;
    }
}
```

**Pay**
```php=
class Pay
{
    public function pay(string $way, int $money): bool
    {
        echo 'pay money : ' . $money . PHP_EOL;
        switch ($way) {
            case 'credit_card':
                return true;
                break;
            case 'real_money':
                return true;
                break;
        }
        return false;
    }
}
```

**Test**
```php=
/**
 * @test
 */
public function PayMoney()
{
    //Arrange
    $mockPay = $this->createMock(Pay::class);
    $Payment = new Payment($mockPay);
    $mockPay->expects($this->once())
        ->method('pay')
        ->with('credit_card', 100)
        ->willReturn(true);

    //Actual
    $actual = $Payment->payMoney('credit_card', 100);

    //Assert
    $this->assertTrue($actual);
}
```

我關注的測試目標是驗證回傳，那就不應該再去關注內部方法呼叫，下面是正確的例子

**Test**
```php=
/**
 * @test
 */
public function PayMoney()
{
    //Arrange
    $stubPay = $this->createStub(Pay::class);
    $Payment = new Payment($stubPay);
    $stubPay->method('pay')
        ->willReturn(true);

    //Actual
    $actual = $Payment->payMoney('credit_card', 100);

    //Assert
    $this->assertTrue($actual);
}
```
### 不必要的順序，或過於精準的參數匹配器

對於測試的回傳值，我們常常想要很精準地去比對是否正確，例如字串，實質上的意義可能只需要去驗證字串是否含有某一部分即可，又例如 array 或者 collection 之類的，我們可能只需要驗證是否含有某些欄位就可以了
