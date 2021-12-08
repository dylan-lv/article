# leetcode141-环形链表（链表判环）

<a href="https://leetcode-cn.com/problems/linked-list-cycle" target="_blank">环形链表</a>

给你一个链表的头节点 `head` ，判断链表中是否有环。

如果链表中有某个节点，可以通过连续跟踪 next 指针再次到达，则链表中存在环。 为了表示给定链表中的环，评测系统内部使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。如果 pos 是 -1，则在该链表中没有环。注意：pos 不作为参数进行传递，仅仅是为了标识链表的实际情况。

如果链表中存在环，则返回 true 。 否则，返回 false 。

**思路：**

使用快慢指针

- 快慢指针首先都指向head
- 慢指针每次走一步，快指针每次走两步
- 如果快慢指针相遇，则一定存在环，否则不存在环



**题解：**

```js
var hasCycle = function(head) {
    if(!head) return false;
    let slow = head, fast = head;
    while(fast && fast.next) {
        slow = slow.next;
        fast = fast.next;
        if(slow === fast) return true;
    }
    return false;
};
```

