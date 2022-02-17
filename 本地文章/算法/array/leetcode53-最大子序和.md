# leetcode53-最大子序和

<a href="https://leetcode-cn.com/problems/maximum-subarray/" target="_blank">最大子序和</a>

给定一个整数数组 `nums` ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

**示例1：**

```
输入：nums = [-2,1,-3,4,-1,2,1,-5,4]
输出：6
解释：连续子数组 [4,-1,2,1] 的和最大，为 6 。
```

**示例2：**

```
输入：nums = [1]
输出：1
```



**思路：**

- 类似滚动数组的思想
- 当前项与前一项想加，与当前项对比，较大的一项赋值给pre
  - 如果pre + x 较大，相当于pre对x有增益作用
  - 如果x较大，则说明pre对x无增益作用，将pre舍弃
- pre和max中较大值赋值给max（记录计算过程中产生的最大值）



**题解：**

```js
var maxSubArray = function(nums) {
    let pre = 0, maxAns = nums[0];
    nums.forEach((x) => {
        pre = Math.max(pre + x, x);
        maxAns = Math.max(maxAns, pre);
    });
    return maxAns;
};
```

