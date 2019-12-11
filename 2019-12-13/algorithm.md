# 算法

#### 两数相加

给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

> https://leetcode-cn.com/problems/add-two-numbers/

示例:

```
输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807
```

#### 答案

**JS解题**
 
```js
/**
 * Definition for singly-linked list.
 * function ListNode(val) {
 *     this.val = val;
 *     this.next = null;
 * }
 */
/**
 * @param {ListNode} l1
 * @param {ListNode} l2
 * @return {ListNode}
 */
var addTwoNumbers = function (l1, l2) {
    let result = new ListNode(0);
    let currResult = result;
    let isAdd = false;

    while (l1 != null || l2 != null) {
        let x = l1 ? l1.val : 0;
        let y = l2 ? l2.val : 0

        // 每个对应位相加
        let val = x + y;
        if (isAdd) {
            val++;
        }

        // 两数相加需要考虑进位问题
        isAdd = false;
        if (val >= 10) {
            val = val - 10;
            isAdd = true;
        }

        let next = new ListNode(val);
        currResult.next = next;
        currResult = next;

        l1 = l1 ? l1.next : null;
        l2 = l2 ? l2.next : null;
    }

    if (isAdd) {
        currResult.next = new ListNode(1);
    }

    return result.next;
};
```
