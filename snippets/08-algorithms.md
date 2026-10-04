# 08 · 数据结构与轻量算法

> 快排/二分/栈/队列——面试手写常考、日常也用得上的轻量实现。

### 1. 快速排序（quickSort）
- **作用**：经典快排。
- **代码**：
```javascript
const quickSort = arr => {
  if (arr.length <= 1) return arr;
  const [pivot, ...rest] = arr;
  const left = rest.filter(x => x < pivot);
  const right = rest.filter(x => x >= pivot);
  return [...quickSort(left), pivot, ...quickSort(right)];
};
quickSort([3, 1, 4, 1, 5, 9, 2, 6]);
```
- **说明**：面试手写版。
- **注意**：O(n log n) 平均，O(n²) 最坏；空间上非原地。

### 2. 归并排序（mergeSort）
- **作用**：稳定排序。
- **代码**：
```javascript
const mergeSort = arr => {
  if (arr.length <= 1) return arr;
  const mid = Math.floor(arr.length / 2);
  const left = mergeSort(arr.slice(0, mid));
  const right = mergeSort(arr.slice(mid));
  const out = [];
  let i = 0, j = 0;
  while (i < left.length && j < right.length) {
    out.push(left[i] <= right[j] ? left[i++] : right[j++]);
  }
  return out.concat(left.slice(i), right.slice(j));
};
mergeSort([5, 2, 4, 7, 1, 3, 6]);
```
- **说明**：稳定排序 O(n log n)。
- **注意**：空间 O(n)。

### 3. 二分查找（binarySearch）
- **作用**：有序数组里找 target。
- **代码**：
```javascript
const binarySearch = (arr, target) => {
  let lo = 0, hi = arr.length - 1;
  while (lo <= hi) {
    const mid = (lo + hi) >> 1;
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) lo = mid + 1;
    else hi = mid - 1;
  }
  return -1;
};
binarySearch([1, 3, 5, 7, 9], 5); // 2
```
- **说明**：O(log n)。
- **注意**：数组必须有序；(lo+hi)>>1 比 /2 快且防溢出。

### 4. 线性查找（linearSearch）
- **作用**：裸遍历找第一个匹配。
- **代码**：
```javascript
const linearSearch = (arr, x) => {
  for (let i = 0; i < arr.length; i++) if (arr[i] === x) return i;
  return -1;
};
```
- **说明**：就是 indexOf 的手写版。
- **注意**：O(n)，别用在大数组上。

### 5. 栈（Stack）
- **作用**：LIFO。
- **代码**：
```javascript
class Stack {
  #items = [];
  push(x) { this.#items.push(x); }
  pop() { return this.#items.pop(); }
  peek() { return this.#items[this.#items.length - 1]; }
  size() { return this.#items.length; }
  isEmpty() { return this.#items.length === 0; }
}
const s = new Stack();
s.push(1); s.push(2); s.pop(); // 2
```
- **说明**：括号匹配、DFS。
- **注意**：私有字段 #items 避免外部乱改。

### 6. 队列（Queue）
- **作用**：FIFO。
- **代码**：
```javascript
class Queue {
  #items = [];
  enqueue(x) { this.#items.push(x); }
  dequeue() { return this.#items.shift(); }
  front() { return this.#items[0]; }
  size() { return this.#items.length; }
  isEmpty() { return this.#items.length === 0; }
}
```
- **说明**：BFS、任务排队。
- **注意**：shift 是 O(n)，高频场景用环形队列。

### 7. 优先队列（PriorityQueue）
- **作用**：每次取出最小（或最大）的。
- **代码**：
```javascript
class PriorityQueue {
  #heap = [];
  push(x) {
    this.#heap.push(x);
    this.#bubbleUp(this.#heap.length - 1);
  }
  pop() {
    const top = this.#heap[0];
    const end = this.#heap.pop();
    if (this.#heap.length) {
      this.#heap[0] = end;
      this.#bubbleDown(0);
    }
    return top;
  }
  #bubbleUp(i) {
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (this.#heap[p] <= this.#heap[i]) break;
      [this.#heap[p], this.#heap[i]] = [this.#heap[i], this.#heap[p]];
      i = p;
    }
  }
  #bubbleDown(i) {
    const n = this.#heap.length;
    while (true) {
      let l = i * 2 + 1, r = l + 1, s = i;
      if (l < n && this.#heap[l] < this.#heap[s]) s = l;
      if (r < n && this.#heap[r] < this.#heap[s]) s = r;
      if (s === i) break;
      [this.#heap[s], this.#heap[i]] = [this.#heap[i], this.#heap[s]];
      i = s;
    }
  }
  size() { return this.#heap.length; }
}
```
- **说明**：小顶堆。
- **注意**：push/pop 都是 O(log n)。

### 8. 链表（LinkedList）
- **作用**：单向链表基础。
- **代码**：
```javascript
class ListNode {
  constructor(val, next = null) { this.val = val; this.next = next; }
}
class LinkedList {
  #head = null;
  append(v) {
    const node = new ListNode(v);
    if (!this.#head) return (this.#head = node);
    let cur = this.#head;
    while (cur.next) cur = cur.next;
    cur.next = node;
  }
  toArray() {
    const out = [];
    let cur = this.#head;
    while (cur) { out.push(cur.val); cur = cur.next; }
    return out;
  }
}
```
- **说明**：面试手写题常客。
- **注意**：查找是 O(n)。

### 9. 哈希表（HashTable）
- **作用**：简易哈希表。
- **代码**：
```javascript
class HashTable {
  #buckets = Array.from({ length: 16 }, () => []);
  #hash(k) { return String(k).length % this.#buckets.length; }
  set(k, v) {
    const b = this.#buckets[this.#hash(k)];
    const entry = b.find(([key]) => key === k);
    entry ? (entry[1] = v) : b.push([k, v]);
  }
  get(k) {
    return this.#buckets[this.#hash(k)].find(([key]) => key === k)?.[1];
  }
}
```
- **说明**：理解 Map 底层。
- **注意**：冲突用链表解决；扩容要 rehash。

### 10. 二叉树遍历（treeTraversal）
- **作用**：DFS 前中后序 + BFS。
- **代码**：
```javascript
const preorder = (root, out = []) => {
  if (!root) return out;
  out.push(root.value);
  preorder(root.left, out);
  preorder(root.right, out);
  return out;
};
const bfs = root => {
  const out = [], q = [root];
  while (q.length) {
    const n = q.shift();
    if (!n) continue;
    out.push(n.value);
    q.push(n.left, n.right);
  }
  return out;
};
```
- **说明**：树操作基础。
- **注意**：BFS 用队列，DFS 用栈/递归。

### 11. 二叉搜索树插入（bstInsert）
- **作用**：往 BST 插节点。
- **代码**：
```javascript
class BST {
  constructor() { this.root = null; }
  insert(v) {
    const node = { v, left: null, right: null };
    if (!this.root) return (this.root = node);
    let cur = this.root;
    while (true) {
      if (v < cur.v) {
        if (!cur.left) return (cur.left = node);
        cur = cur.left;
      } else {
        if (!cur.right) return (cur.right = node);
        cur = cur.right;
      }
    }
  }
}
```
- **说明**：有序字典。
- **注意**：插入顺序影响树高，极端情况退化成链表。

### 12. 冒泡排序（bubbleSort）
- **作用**：最直观的排序。
- **代码**：
```javascript
const bubbleSort = arr => {
  const a = [...arr];
  for (let i = 0; i < a.length; i++) {
    for (let j = 0; j < a.length - i - 1; j++) {
      if (a[j] > a[j + 1]) [a[j], a[j + 1]] = [a[j + 1], a[j]];
    }
  }
  return a;
};
```
- **说明**：教学用，实际别用。
- **注意**：O(n²)。

### 13. 选择排序（selectionSort）
- **作用**：每次选最小。
- **代码**：
```javascript
const selectionSort = arr => {
  const a = [...arr];
  for (let i = 0; i < a.length; i++) {
    let min = i;
    for (let j = i + 1; j < a.length; j++) if (a[j] < a[min]) min = j;
    [a[i], a[min]] = [a[min], a[i]];
  }
  return a;
};
```
- **说明**：O(n²)。
- **注意**：交换次数少，写入密集场景可考虑。

### 14. 插入排序（insertionSort）
- **作用**：小规模数据很快。
- **代码**：
```javascript
const insertionSort = arr => {
  const a = [...arr];
  for (let i = 1; i < a.length; i++) {
    const cur = a[i];
    let j = i - 1;
    while (j >= 0 && a[j] > cur) {
      a[j + 1] = a[j];
      j--;
    }
    a[j + 1] = cur;
  }
  return a;
};
```
- **说明**：近乎有序的数组上 O(n)。
- **注意**：V8 对短数组的 sort 就是它。

### 15. 两数之和（twoSum）
- **作用**：数组里找两个数加起来等于 target。
- **代码**：
```javascript
const twoSum = (arr, target) => {
  const seen = new Map();
  for (let i = 0; i < arr.length; i++) {
    const need = target - arr[i];
    if (seen.has(need)) return [seen.get(need), i];
    seen.set(arr[i], i);
  }
  return [];
};
twoSum([2, 7, 11, 15], 9); // [0, 1]
```
- **说明**：LeetCode 第 1 题。
- **注意**：O(n) 用 hash，别双重循环。

### 16. 最长公共前缀（longestCommonPrefix）
- **作用**：一组字符串最长公共前缀。
- **代码**：
```javascript
const longestCommonPrefix = strs => {
  if (!strs.length) return '';
  for (let i = 0; i < strs[0].length; i++) {
    const c = strs[0][i];
    if (!strs.every(s => s[i] === c)) return strs[0].slice(0, i);
  }
  return strs[0];
};
longestCommonPrefix(['flower', 'flow', 'flight']); // 'fl'
```
- **说明**：自动补全。
- **注意**：以第一个串为基准逐列比。

### 17. 回文判断（isPalindrome）
- **作用**：字符串是否回文。
- **代码**：
```javascript
const isPalindrome = s => {
  const clean = s.toLowerCase().replace(/[^a-z0-9]/g, '');
  return clean === [...clean].reverse().join('');
};
isPalindrome('A man, a plan: Panama'); // true
```
- **说明**：忽略大小写和非字母数字。
- **注意**：先清洗再比。

### 18. 括号匹配（isValidBrackets）
- **作用**：括号是否成对合法。
- **代码**：
```javascript
const isValidBrackets = s => {
  const map = { '(': ')', '[': ']', '{': '}' };
  const stack = [];
  for (const c of s) {
    if (map[c]) stack.push(map[c]);
    else if (stack.pop() !== c) return false;
  }
  return stack.length === 0;
};
isValidBrackets('([]){}'); // true
```
- **说明**：栈的经典应用。
- **注意**：右括号来时栈顶必须是对应左括号。

### 19. 爬楼梯（climbStairs）
- **作用**：一次 1 或 2 步，到 n 阶几种走法。
- **代码**：
```javascript
const climbStairs = n => {
  let a = 1, b = 1;
  for (let i = 2; i <= n; i++) [a, b] = [b, a + b];
  return b;
};
climbStairs(5); // 8
```
- **说明**：斐波那契 DP。
- **注意**：滚动变量省空间。

### 20. 最大子数组和（maxSubArray）
- **作用**：Kadane 算法。
- **代码**：
```javascript
const maxSubArray = arr => {
  let cur = arr[0], best = arr[0];
  for (let i = 1; i < arr.length; i++) {
    cur = Math.max(arr[i], cur + arr[i]);
    best = Math.max(best, cur);
  }
  return best;
};
maxSubArray([-2, 1, -3, 4, -1, 2, 1, -5, 4]); // 6
```
- **说明**：LeetCode 53。
- **注意**：O(n) 一次过。

### 21. 合并两个有序数组（mergeSorted）
- **作用**：两个有序数组合并。
- **代码**：
```javascript
const mergeSorted = (a, b) => {
  const out = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    out.push(a[i] <= b[j] ? a[i++] : b[j++]);
  }
  return out.concat(a.slice(i), b.slice(j));
};
mergeSorted([1, 3, 5], [2, 4, 6]); // [1,2,3,4,5,6]
```
- **说明**：归并排序的合并步。
- **注意**：双指针。

### 22. 区间合并（mergeIntervals）
- **作用**：合并重叠区间。
- **代码**：
```javascript
const mergeIntervals = intervals => {
  intervals.sort((a, b) => a[0] - b[0]);
  const out = [intervals[0]];
  for (const [s, e] of intervals.slice(1)) {
    const last = out[out.length - 1];
    if (s <= last[1]) last[1] = Math.max(last[1], e);
    else out.push([s, e]);
  }
  return out;
};
mergeIntervals([[1,3],[2,6],[8,10],[15,18]]); // [[1,6],[8,10],[15,18]]
```
- **说明**：会议时间冲突检测。
- **注意**：先按 start 排序。

### 23. 去重（uniqueSorted）
- **作用**：有序数组原地去重。
- **代码**：
```javascript
const uniqueSorted = arr => {
  let i = 0;
  for (let j = 0; j < arr.length; j++) {
    if (arr[j] !== arr[i]) arr[++i] = arr[j];
  }
  return arr.slice(0, i + 1);
};
uniqueSorted([1, 1, 2, 3, 3, 4]); // [1,2,3,4]
```
- **说明**：双指针原地。
- **注意**：必须先排序。

### 24. 缺失数字（missingNumber）
- **作用**：0..n 数组里缺了谁。
- **代码**：
```javascript
const missingNumber = arr => {
  const n = arr.length;
  const sum = (n * (n + 1)) / 2;
  return sum - arr.reduce((a, b) => a + b, 0);
};
missingNumber([3, 0, 1]); // 2
```
- **说明**：高斯求和法。
- **注意**：O(n) 时间 O(1) 空间。

### 25. 旋转数组（rotateArray）
- **作用**：数组右移 k 位。
- **代码**：
```javascript
const rotateArray = (arr, k) => {
  k = k % arr.length;
  return [...arr.slice(-k), ...arr.slice(0, -k)];
};
rotateArray([1, 2, 3, 4, 5], 2); // [4,5,1,2,3]
```
- **说明**：切片法简洁。
- **注意**：原地旋转要三次翻转。

### 26. 斐波那契迭代（fibIter）
- **作用**：O(n) O(1) 空间求第 n 个斐波那契。
- **代码**：
```javascript
const fibIter = n => {
  let a = 0, b = 1;
  for (let i = 0; i < n; i++) [a, b] = [b, a + b];
  return a;
};
fibIter(10); // 55
```
- **说明**：别用递归。
- **注意**：滚动变量。

### 27. 链表反转（reverseLinkedList）
- **作用**：反转单链表。
- **代码**：
```javascript
const reverseLinkedList = head => {
  let prev = null, cur = head;
  while (cur) {
    const next = cur.next;
    cur.next = prev;
    prev = cur;
    cur = next;
  }
  return prev;
};
```
- **说明**：面试高频。
- **注意**：三个指针 prev/cur/next。

### 28. 环形链表检测（hasCycle）
- **作用**：快慢指针。
- **代码**：
```javascript
const hasCycle = head => {
  let slow = head, fast = head;
  while (fast && fast.next) {
    slow = slow.next;
    fast = fast.next.next;
    if (slow === fast) return true;
  }
  return false;
};
```
- **说明**：Floyd 判圈。
- **注意**：空间 O(1)。

### 29. 合并 K 个有序链表（mergeKLists）
- **作用**：两两合并。
- **代码**：
```javascript
const mergeKLists = lists => {
  if (!lists.length) return null;
  while (lists.length > 1) {
    const a = lists.shift(), b = lists.shift();
    lists.push(mergeSortedList(a, b));
  }
  return lists[0];
};
const mergeSortedList = (a, b) => {
  if (!a) return b;
  if (!b) return a;
  if (a.val < b.val) { a.next = mergeSortedList(a.next, b); return a; }
  b.next = mergeSortedList(a, b.next); return b;
};
```
- **说明**：分治。
- **注意**：上面 mergeSortedList 是链表版本。

### 30. 岛屿数量（numIslands）
- **作用**：DFS 数 1 的连通块。
- **代码**：
```javascript
const numIslands = grid => {
  let count = 0;
  const dfs = (i, j) => {
    if (i < 0 || j < 0 || i >= grid.length || j >= grid[0].length) return;
    if (grid[i][j] !== '1') return;
    grid[i][j] = '0';
    dfs(i + 1, j); dfs(i - 1, j); dfs(i, j + 1); dfs(i, j - 1);
  };
  for (let i = 0; i < grid.length; i++)
    for (let j = 0; j < grid[0].length; j++)
      if (grid[i][j] === '1') { count++; dfs(i, j); }
  return count;
};
```
- **说明**：DFS 经典。
- **注意**：会改原 grid；不改就加 visited。

### 31. 滑动窗口最大值（maxSlidingWindow）
- **作用**：窗口滑过求每步最大。
- **代码**：
```javascript
const maxSlidingWindow = (nums, k) => {
  const out = [], deque = [];
  for (let i = 0; i < nums.length; i++) {
    while (deque.length && deque[0] <= i - k) deque.shift();
    while (deque.length && nums[deque[deque.length - 1]] <= nums[i]) deque.pop();
    deque.push(i);
    if (i >= k - 1) out.push(nums[deque[0]]);
  }
  return out;
};
maxSlidingWindow([1,3,-1,-3,5,3,6,7], 3); // [3,3,5,5,6,7]
```
- **说明**：单调队列。
- **注意**：O(n)。

### 32. 单词拆分（wordBreak）
- **作用**：字符串能否用字典拼出。
- **代码**：
```javascript
const wordBreak = (s, wordDict) => {
  const set = new Set(wordDict);
  const dp = new Array(s.length + 1).fill(false);
  dp[0] = true;
  for (let i = 1; i <= s.length; i++) {
    for (let j = 0; j < i; j++) {
      if (dp[j] && set.has(s.slice(j, i))) { dp[i] = true; break; }
    }
  }
  return dp[s.length];
};
wordBreak('leetcode', ['leet', 'code']); // true
```
- **说明**：DP。
- **注意**：O(n²)。

### 33. 零钱兑换（coinChange）
- **作用**：最少几枚硬币凑 amount。
- **代码**：
```javascript
const coinChange = (coins, amount) => {
  const dp = Array(amount + 1).fill(Infinity);
  dp[0] = 0;
  for (let a = 1; a <= amount; a++) {
    for (const c of coins) if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
  }
  return dp[amount] === Infinity ? -1 : dp[amount];
};
coinChange([1, 2, 5], 11); // 3 (5+5+1)
```
- **说明**：完全背包。
- **注意**：dp 初始 Infinity。

### 34. 最长递增子序列（lengthOfLIS）
- **作用**：LIS 长度。
- **代码**：
```javascript
const lengthOfLIS = nums => {
  const tails = [];
  for (const x of nums) {
    let lo = 0, hi = tails.length;
    while (lo < hi) {
      const mid = (lo + hi) >> 1;
      if (tails[mid] < x) lo = mid + 1;
      else hi = mid;
    }
    tails[lo] = x;
  }
  return tails.length;
};
lengthOfLIS([10, 9, 2, 5, 3, 7, 101, 18]); // 4
```
- **说明**：O(n log n) 贪心 + 二分。
- **注意**：tails[i] 不是真实子序列。

### 35. 并查集（UnionFind）
- **作用**：动态连通性。
- **代码**：
```javascript
class UnionFind {
  constructor(n) { this.parent = Array.from({ length: n }, (_, i) => i); }
  find(x) {
    while (this.parent[x] !== x) {
      this.parent[x] = this.parent[this.parent[x]];
      x = this.parent[x];
    }
    return x;
  }
  union(a, b) { this.parent[this.find(a)] = this.find(b); }
  connected(a, b) { return this.find(a) === this.find(b); }
}
```
- **说明**：路径压缩。
- **注意**：初始化时 parent[i]=i。

### 36. 拓扑排序（topoSort）
- **作用**：课程安排。
- **代码**：
```javascript
const topoSort = (n, edges) => {
  const indeg = Array(n).fill(0);
  const adj = Array.from({ length: n }, () => []);
  edges.forEach(([u, v]) => { adj[u].push(v); indeg[v]++; });
  const q = indeg.map((d, i) => d === 0 && i).filter(Number.isInteger);
  const out = [];
  while (q.length) {
    const u = q.shift();
    out.push(u);
    adj[u].forEach(v => --indeg[v] === 0 && q.push(v));
  }
  return out.length === n ? out : [];
};
topoSort(4, [[1,0],[2,0],[3,1],[3,2]]); // 可能 [0,1,2,3]
```
- **说明**：Kahn 算法。
- **注意**：有环返回空数组。
