

使用堆实现

```js
class Heap {
  constructor(fn) {
    if (typeof fn === 'function') {
      this.fn = fn;
    } else if (fn === '>') { // 大顶堆
      this.fn = (a, b) => a < b; // 父节点小于子节点，则交换
    } else if (fn === '<') { // 小顶堆
      this.fn = (a, b) => a > b;
    }
    this.arr = [];
    this.cnt = 0;
  }
  shift_up(ind) { // 向上调整（插入使用）
    while (Math.floor((ind - 1) / 2) >= 0 && this.fn(this.arr[Math.floor((ind - 1) / 2)], this.arr[ind])) {
      [this.arr[Math.floor((ind - 1) / 2)], this.arr[ind]] = [this.arr[ind], this.arr[Math.floor((ind - 1) / 2)]];
      ind = Math.floor((ind - 1) / 2);
    }
  }
  shift_down(ind) { // 向下调整（pop使用）
    const n = this.cnt - 1; // n：最后一个子节点的下标
    while (ind * 2 + 1 <= n) { // 左子树的下标小于等于最后一个子节点下标（此时一定有左孩子）
      let temp = ind; // temp保存最大值的下标
      if (this.fn(this.arr[temp], this.arr[ind * 2 + 1])) temp = ind * 2 + 1;
      if (ind * 2 + 2 <= n && this.fn(this.arr[temp], this.arr[ind * 2 + 2])) temp = ind * 2 + 2;
      if (temp === ind) break;
      [this.arr[temp], this.arr[ind]] = [this.arr[ind], this.arr[temp], this.arr[ind]];
      ind = temp;
    }
  }
  push(x) {
    this.arr[this.cnt++] = x;
    this.shift_up(this.cnt - 1);
    return;
  }
  pop() {
    if (this.cnt === 0) return;
    const pop_val = this.top();
    [this.arr[0], this.arr[this.cnt - 1]] = [this.arr[this.cnt - 1], this.arr[0]]; // 最后一位和头部交换，为了排序做准备
    this.cnt--;
    this.shift_down(0);
    // this.print(); // 输出
    return pop_val;
  }
  top() {
    return this.arr[0];
  }
  size() {
    return this.cnt;
  }
  empty() {
    return this.cnt === 0;
  }
  getData() {
    const list = [];
    for (let i = 0; i < this.cnt; i++) list.push(this.arr[i]);
    return list;
  }
  print() {
    console.log(this.getData().join(' '));
  }
}
```



```js
var mergeKLists = function(lists) {
    const h = new Heap((p, q) => {
        return p.val > q.val; // 小顶堆
    })
    // 依次将K个链表的头部元素，压入到优先队列中去
    for(let x of lists) {
        if(!x) continue;
        h.push(x);
    }
    let ret = new ListNode(), p = ret; // 合并链表的虚拟头节点，和合并链表的末尾指针

    while(!h.empty()) {
        // 当优先队列还有元素的时候，每次弹出一个元素
        let cur = h.pop();
        // 然后将最小值接入到末尾
        p.next = cur;
        p = p.next;
        // 将cur的下一个节点，压入到优先队列中去
        if(cur.next) h.push(cur.next);
    }
    return ret.next;
};
```







