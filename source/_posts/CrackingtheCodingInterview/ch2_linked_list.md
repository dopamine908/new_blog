---
title: 提升程式設計師的面試力 - Ch2 - 鏈結串列(Linked List)
date: 2022-02-10 15:00:38
tags:
  - 提升程式設計師的面試力
  - 資料結構
  - 讀書會報告
categories:
  - 讀書會報告
cover: https://picsum.photos/id/137/800/600
---

# 提升程式設計師的面試力 - Ch2 - 鏈結串列(Linked List)

[鏈結串列(Linked List)](https://www.youtube.com/watch?v=mKOHEZ17PnY)是一種表示節點順序的資料結構。

- 單向串列(Singly Linked)：每個節點都指向鏈結串列中的下一個節點
- 雙向串列(Doubly Linked)：每個節點提供指向下一個節點及前一個節點的指標

如果我們想找到串列中的第 K 的元素的話，我們就必須迭代 K 次才能夠找到

Linked List 的優點是，當你想加入或刪除開頭元素時，可以在嘗試時間內完成

## 建立 Linked List

```php=
class LinkedListNode
{
    public LinkedListNode $next;
    public int $data;

    public function __construct(int $data)
    {
        $this->data = $data;
    }

    public function appendToTail(LinkedListNode $node): void
    {
        $this->next = $node;
    }
}
```

## 從單向鏈結串列中刪除一個節點

```php=
public function deleteNode(LinkedListNode $node, int $data): LinkedListNode
{
    $head = $node;

    if ($head->data == $data) {
        return $head->next;
    }

    while ($head->next !== null) {
        if ($head->next->data === $data) {
            $head->next = $head->next->next;
            return $node;
        }
        $head = $head->next;
    }

    return $node;
}
```

## 「Runner」技巧

「Runner」技巧，或稱第二指標(second pointer)，是指同時用兩個指標同時迭代 Linked List，其中一個指標在另外一個指標之前。

例如如果我有兩個指標，p1、p2，我讓 p1(快指標)在 p2 每移動一格的時候移動兩格，如此一來 p1 到達尾端時 p2 會剛好停留在中間的位置。

## 遞迴問題

許多 Linked List 問題其實都需要依靠遞迴來解決，而遞迴演算法至少都會佔用 $O(n)$ 空間，其中 $n$ 為呼叫的深度

## 面試題目（1~3）

