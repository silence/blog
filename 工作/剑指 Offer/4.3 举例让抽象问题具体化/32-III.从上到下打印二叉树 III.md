请实现一个函数按照之字形顺序打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右到左的顺序打印，第三行再按照从左到右的顺序打印，其他行以此类推。

 

例如:
给定二叉树: [3,9,20,null,null,15,7],

        3
       / \
      9  20
        /  \
       15   7

返回其层次遍历结果：

```
[
  [3],
  [20,9],
  [15,7]
]
```


提示：

节点总数 <= 1000

---

方法：BFS + 单双层从数组前面入值或者后面入值。

```javascript
/**
 * Definition for a binary tree node.
 * function TreeNode(val) {
 *     this.val = val;
 *     this.left = this.right = null;
 * }
 */
/**
 * @param {TreeNode} root
 * @return {number[][]}
 */
var levelOrder = function(root) {
  let queue = [root];
  let res = []
  while(queue.length !== 0 && root !== null) {
    let temp = [];
    let length = queue.length;
    for (let i = 0; i < length; i++) {
      var top = queue.shift();
      res.length % 2 === 0 ? temp.push(top.val) : temp.unshift(top.val)
      if (top.left) queue.push(top.left)
      if (top.right) queue.push(top.right)
    }
    res.push(temp)
  }
  return res;
};
```

