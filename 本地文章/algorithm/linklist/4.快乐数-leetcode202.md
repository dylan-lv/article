# leetcode202-快乐数

<a href="https://leetcode-cn.com/problems/happy-number/" target="_blank">快乐数</a>

编写一个算法来判断一个数 `n` 是不是快乐数。

「快乐数」定义为：

- 对于一个正整数，每一次将该数替换为它每个位置上的数字的平方和。
- 然后重复这个过程直到这个数变为 1，也可能是 `无限循环` 但始终变不到 1。
- 如果 `可以变为` 1，那么这个数就是快乐数。

如果 `n` 是快乐数就返回 `true` ；不是，则返回 `false` 。

**思路：**

1. 我们举个例子：当输入值为19时，平方和就转换成了：19 --> 82 --> 68 --> 100 --> 1
2. 题目就可以转化为，判断一个链表是否有环。如果遍历某个节点为1，说明没环，就是快乐数。如果遍历到重复的节点值，说明有环，就不是快乐数。

**题解：**

```js
var isHappy = function(n) {
    const getNext = function(n) {
        let sum = 0;
        while(n) {
            sum += (n%10) * (n%10);
            n = Math.floor(n/10);
        }
        return sum;
    }

    let slow = n, fast = getNext(n);
    while(fast !== 1 && slow !== fast) {
        slow = getNext(slow);
        fast = getNext(getNext(fast));
    }
    return fast === 1;
};
```

