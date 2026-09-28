---
title: "Reverse Nodes in k-Group"
number: 25
difficulty: Hard
tags: [Linked List, Recursion]
link: https://leetcode.com/problems/reverse-nodes-in-k-group/
time: O(n)
space: O(1)
---

## 題目重點

給一個鏈結串列，每 `k` 個節點為一組做反轉；最後不足 `k` 個的節點維持原順序。只能改節點的指向，不能改節點的值。進階要求：只用 `O(1)` 額外空間。

```
輸入：1 → 2 → 3 → 4 → 5, k = 2
輸出：2 → 1 → 4 → 3 → 5

輸入：1 → 2 → 3 → 4 → 5, k = 3
輸出：3 → 2 → 1 → 4 → 5
```

## 解題歷程

### 第一版：先放進陣列

最直覺的做法是把節點全部放進 `List`，每 `k` 個一段倒過來，再重新串起來。能過，但用了 `O(n)` 空間，不符合進階要求；而且這題真正要練的是「在串列上直接改指標」，繞過去等於沒練到。

### 第二版：遞迴

換個角度看：反轉第一組之後，第一組原本的頭會變成尾，它的 `next` 要接到「後面剩下的串列處理完的結果」。這正好是同一個問題的子問題。

1. 先往前走 `k` 步，確認這一組湊得滿 `k` 個；湊不滿就原樣回傳。
2. 遞迴處理第 `k+1` 個節點之後的串列，拿到新的頭。
3. 把這一組反轉，並讓反轉後的尾巴接到步驟 2 的結果。

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode node = head;
        for (int i = 0; i < k; i++) {
            if (node == null) return head; // 不足 k 個，不動
            node = node.next;
        }
        // prev 從「後面處理完的結果」開始，反轉時第一個節點自然接上去
        ListNode prev = reverseKGroup(node, k);
        ListNode cur = head;
        for (int i = 0; i < k; i++) {
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        return prev;
    }
}
```

邏輯很乾淨，但遞迴深度是 `n / k`，堆疊空間是 `O(n / k)`，還是不算 `O(1)`。

### 第三版：迭代 + dummy 節點

把遞迴攤平成迴圈。關鍵是每一組反轉時要同時抓住四個位置：

| 變數 | 意義 |
|---|---|
| `prevTail` | 上一組反轉後的尾巴（一開始是 `dummy`） |
| `groupHead` | 這一組原本的頭，反轉後會變成尾 |
| `kth` | 這一組第 `k` 個節點，反轉後會變成頭 |
| `nextGroup` | 下一組的第一個節點 |

反轉時讓 `prev` 從 `nextGroup` 開始，這一組反轉完，尾巴就已經接到下一組了；最後再把 `prevTail.next` 接到 `kth`，並把 `prevTail` 移到 `groupHead`。

```java
class Solution {
    public ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0, head);
        ListNode prevTail = dummy;

        while (true) {
            // 從上一組的尾巴往前走 k 步，找到這一組的第 k 個節點
            ListNode kth = prevTail;
            for (int i = 0; i < k && kth != null; i++) {
                kth = kth.next;
            }
            if (kth == null) break; // 剩下不足 k 個

            ListNode groupHead = prevTail.next;
            ListNode nextGroup = kth.next;

            // 反轉 [groupHead, kth]，prev 從 nextGroup 開始，尾巴直接接上下一組
            ListNode prev = nextGroup;
            ListNode cur = groupHead;
            while (cur != nextGroup) {
                ListNode next = cur.next;
                cur.next = prev;
                prev = cur;
                cur = next;
            }

            prevTail.next = kth;   // 上一組接到這一組的新頭
            prevTail = groupHead;  // 原本的頭變成這一組的尾
        }
        return dummy.next;
    }
}
```

以 `1 → 2 → 3 → 4 → 5, k = 2` 走一次：

| 回合 | 反轉的組 | 反轉後整條串列 | 下一輪的 `prevTail` |
|---|---|---|---|
| 1 | `1 → 2` | `2 → 1 → 3 → 4 → 5` | `1` |
| 2 | `3 → 4` | `2 → 1 → 4 → 3 → 5` | `3` |
| 3 | 從 `3` 往後只剩 `5`，不足 2 個 | 結束 | — |

## 踩到的坑

- **反轉時 `prev` 從 `null` 開始**：這是單純反轉整條串列的習慣寫法，但放在這題，反轉完的尾巴會斷掉，後面的節點全部遺失。改成從 `nextGroup` 開始就不用事後再補接。
- **忘了更新 `prevTail`**：反轉完 `groupHead` 已經變成尾巴，下一組要接在它後面，而不是接在 `kth` 後面。
- **先反轉再檢查長度**：如果邊走邊反轉，走到一半才發現不足 `k` 個，就得再反轉回去。先數完再動手最省事。
- **頭節點會換人**：第一組反轉後，整條串列的頭就不是原本的 `head` 了。用 `dummy` 節點就不必對第一組做特別處理。

## 心得

鏈結串列的題目，難的通常不是演算法，而是指標接回去的順序。畫出 `prevTail / groupHead / kth / nextGroup` 四個位置之後，每一行程式都只是在回答「這個節點最後要指向誰」。
