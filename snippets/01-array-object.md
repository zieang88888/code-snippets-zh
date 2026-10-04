# 01 · 数组与对象操作

> 去重 / 分组 / 深拷贝 / 合并 / 取值 / 拍平等最高频的数据整理套路。

### 1. 取首个元素（head）
- **作用**：拿到数组第一个元素，空数组返回 `undefined`。
- **代码**：
```javascript
const head = arr => (arr && arr.length ? arr[0] : undefined);
head([1, 2, 3]); // 1
```
- **说明**：比直接写 `arr[0]` 多一层空值保护，适合从接口拿可能为 null 的数据。
- **注意**：不会改变原数组；需要「第一个或默认值」可写 `arr?.[0] ?? '默认'`。

### 2. 取末尾元素（last）
- **作用**：拿到数组最后一个元素。
- **代码**：
```javascript
const last = arr => (arr && arr.length ? arr[arr.length - 1] : undefined);
last([1, 2, 3]); // 3
```
- **说明**：替代 `arr.slice(-1)[0]`，语义更直白。
- **注意**：新版 Node/浏览器也支持 `arr.at(-1)`，现代环境优先用它。

### 3. 数组分块（chunk）
- **作用**：把数组按固定大小切成二维块。
- **代码**：
```javascript
const chunk = (arr, size) =>
  Array.from({ length: Math.ceil(arr.length / size) }, (_, i) =>
    arr.slice(i * size, i * size + size)
  );
chunk([1, 2, 3, 4, 5], 2); // [[1,2],[3,4],[5]]
```
- **说明**：分页渲染、批量提交接口常用；最后一块可能不足 size。
- **注意**：`size` 必须 ≥1，否则 `Math.ceil` 会产生大量空块。

### 4. 剔除假值（compact）
- **作用**：去掉数组中的 `false / null / 0 / '' / undefined / NaN`。
- **代码**：
```javascript
const compact = arr => arr.filter(Boolean);
compact([0, 1, false, 2, '', 3, null]); // [1, 2, 3]
```
- **说明**：表单提交前清洗字段非常顺手。
- **注意**：字符串 `'0'`、`'false'` 是真值会被保留，业务里要单独过滤。

### 5. 按条件计数（countBy）
- **作用**：对数组元素做归类并统计每类数量。
- **代码**：
```javascript
const countBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : val => val[fn]).reduce((acc, val) => {
    acc[val] = (acc[val] || 0) + 1;
    return acc;
  }, {});
countBy([6.1, 4.2, 6.3], Math.floor); // { '4': 1, '6': 2 }
countBy(['one', 'two', 'three'], 'length'); // { '3': 2, '5': 1 }
```
- **说明**：传函数做规则，传字符串直接按对象属性归类。
- **注意**：对象 key 永远是字符串，`'6'` 不是数字 `6`。

### 6. 深度拍平（deepFlatten）
- **作用**：把任意嵌套数组拍成一维。
- **代码**：
```javascript
const deepFlatten = arr =>
  [].concat(...arr.map(v => (Array.isArray(v) ? deepFlatten(v) : v)));
deepFlatten([1, [2], [[3], 4], 5]); // [1,2,3,4,5]
```
- **说明**：JSON 里嵌套数组整理成平铺数据时用。
- **注意**：现代环境可直接 `arr.flat(Infinity)`；本写法兼容老环境。

### 7. 数组差集（difference）
- **作用**：返回在 a 但不在 b 里的元素。
- **代码**：
```javascript
const difference = (a, b) => {
  const s = new Set(b);
  return a.filter(x => !s.has(x));
};
difference([1, 2, 3], [1, 2, 4]); // [3]
```
- **说明**：用 Set 把 O(n²) 降成 O(n)。
- **注意**：只比较基本类型；对象要靠 `differenceBy`（按字段比）。

### 8. 头部丢弃（drop / dropRight）
- **作用**：从左/右丢弃 n 个元素。
- **代码**：
```javascript
const drop = (arr, n = 1) => arr.slice(n);
const dropRight = (arr, n = 1) => arr.slice(0, -n);
drop([1, 2, 3]);      // [2, 3]
dropRight([1, 2, 3]); // [1, 2]
```
- **说明**：分页时去掉「当前已加载」那几条很方便。
- **注意**：`n >= length` 时返回空数组，不会报错。

### 9. 按条件丢弃直到（dropWhile）
- **作用**：从头丢掉满足条件的前缀，遇到第一个不满足就停。
- **代码**：
```javascript
const dropWhile = (arr, fn) => {
  while (arr.length && !fn(arr[0])) arr = arr.slice(1);
  return arr;
};
dropWhile([1, 2, 3, 4], n => n < 3); // [3, 4]
```
- **说明**：去掉表头空行、前置占位数据很合适。
- **注意**：是「前缀」丢弃，中间不连续满足的不会被跳过。

### 10. 单层拍平（flatten）
- **作用**：只拍平一层嵌套。
- **代码**：
```javascript
const flatten = arr => arr.reduce((a, b) => a.concat(b), []);
flatten([1, [2, 3], [4, 5]]); // [1,2,3,4,5]
```
- **说明**：`arr.flat()` 的手写版。
- **注意**：深层嵌套要用 deepFlatten。

### 11. 分组（groupBy）
- **作用**：按规则把元素归到不同组。
- **代码**：
```javascript
const groupBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : val => val[fn]).reduce((acc, val, i) => {
    acc[val] = (acc[val] || []).concat(arr[i]);
    return acc;
  }, {});
groupBy([6.1, 4.2, 6.3], Math.floor); // {4:[4.2], 6:[6.1,6.3]}
```
- **说明**：按日期/类别/首字母分组渲染列表时核心套路。
- **注意**：分组 key 是字符串；分组顺序按原数组首次出现顺序。

### 12. 判重（hasDuplicates）
- **作用**：数组里是否有重复值。
- **代码**：
```javascript
const hasDuplicates = arr => new Set(arr).size !== arr.length;
hasDuplicates([1, 2, 2, 3]); // true
```
- **说明**：表单提交前校验 id/手机号是否重复。
- **注意**：只比较基本类型（SameValueZero），对象引用不同即使内容相同也视为不同。

### 13. 交集（intersection）
- **作用**：两个数组共同拥有的元素。
- **代码**：
```javascript
const intersection = (a, b) => {
  const s = new Set(b);
  return a.filter(x => s.has(x));
};
intersection([1, 2, 3], [4, 3, 2]); // [2, 3]
```
- **说明**：求「同时拥有的标签/权限」。
- **注意**：结果顺序按 a 的顺序；重复元素保留（a 里重复就重复出现）。

### 14. 最大 N 个（maxN）
- **作用**：从大到小取前 n 个。
- **代码**：
```javascript
const maxN = (arr, n = 1) => [...arr].sort((a, b) => b - a).slice(0, n);
maxN([1, 2, 3]);    // [3]
maxN([1, 2, 3], 2); // [3, 2]
```
- **说明**：排行榜 Top N 展示。
- **注意**：用 `[...arr]` 拷贝后再 sort，避免污染原数组。

### 15. 最小 N 个（minN）
- **作用**：从小到大取前 n 个。
- **代码**：
```javascript
const minN = (arr, n = 1) => [...arr].sort((a, b) => a - b).slice(0, n);
minN([1, 2, 3], 2); // [1, 2]
```
- **说明**：取最低分/最便宜商品。
- **注意**：同上，排序前先拷贝。

### 16. 键值对转对象（objectFromPairs）
- **作用**：把 `[[k,v], ...]` 变成对象。
- **代码**：
```javascript
const objectFromPairs = arr => arr.reduce((a, [k, v]) => ((a[k] = v), a), {});
objectFromPairs([['a', 1], ['b', 2]]); // { a: 1, b: 2 }
```
- **说明**：URLSearchParams、表格行转对象时常用。
- **注意**：重复 key 后者覆盖前者。

### 17. 对象转键值对（objectToPairs）
- **作用**：对象转 `[key, value]` 数组。
- **代码**：
```javascript
const objectToPairs = obj => Object.keys(obj).map(k => [k, obj[k]]);
objectToPairs({ a: 1, b: 2 }); // [['a',1],['b',2]]
```
- **说明**：现代环境等价于 `Object.entries(obj)`。
- **注意**：只枚举自身可枚举属性。

### 18. 剔除指定键（omit）
- **作用**：从对象里删掉若干 key，返回新对象。
- **代码**：
```javascript
const omit = (obj, arr) =>
  Object.keys(obj)
    .filter(k => !arr.includes(k))
    .reduce((acc, key) => ((acc[key] = obj[key]), acc), {});
omit({ a: 1, b: 2, c: 3 }, ['b']); // { a: 1, c: 3 }
```
- **说明**：提交接口前剔除 `password`、`_id` 等字段。
- **注意**：浅拷贝，嵌套对象仍共享引用。

### 19. 挑指定键（pick）
- **作用**：只保留对象里指定的几个 key。
- **代码**：
```javascript
const pick = (obj, arr) =>
  arr.reduce((acc, curr) => (curr in obj && (acc[curr] = obj[curr]), acc), {});
pick({ a: 1, b: 2, c: 3 }, ['a', 'c']); // { a: 1, c: 3 }
```
- **说明**：从大对象里裁出接口需要的最小字段集。
- **注意**：key 不存在于 obj 时不会加入结果。

### 20. 多维数组分片（partition）
- **作用**：按真假把数组分成两组。
- **代码**：
```javascript
const partition = (arr, fn) =>
  arr.reduce(
    (acc, val, i, arr) => (acc[fn(val, i, arr) ? 0 : 1].push(val), acc),
    [[], []]
  );
partition([1, 2, 3, 4], n => n % 2); // [[1,3],[2,4]]
```
- **说明**：通过/未通过、已完成/未完成列表拆分。
- **注意**：一次遍历分两组，比 filter 两次省一遍。

### 21. 随机取样（sample）
- **作用**：随机取一个元素。
- **代码**：
```javascript
const sample = arr => arr[Math.floor(Math.random() * arr.length)];
sample([3, 7, 9, 11]); // 随机一个
```
- **说明**：每日一句、随机推荐。
- **注意**：不保证不重复；要不重复抽用 sampleSize。

### 22. 随机取 N 个（sampleSize）
- **作用**：随机取 n 个（不重复）。
- **代码**：
```javascript
const sampleSize = ([...arr], n = 1) => {
  let m = arr.length;
  while (m) {
    const i = Math.floor(Math.random() * m--);
    [arr[m], arr[i]] = [arr[i], arr[m]];
  }
  return arr.slice(0, n);
};
sampleSize([1, 2, 3], 2); // [3,1] 之类
```
- **说明**：Fisher-Yates 洗牌前 n 个，抽奖常用。
- **注意**：n 超过 length 时返回全部（顺序乱）。

### 23. 洗牌（shuffle）
- **作用**：随机打乱数组。
- **代码**：
```javascript
const shuffle = ([...arr]) => {
  let m = arr.length;
  while (m) {
    const i = Math.floor(Math.random() * m--);
    [arr[m], arr[i]] = [arr[i], arr[m]];
  }
  return arr;
};
shuffle([1, 2, 3]); // 乱序
```
- **说明**：洗牌算法经典实现，副本返回不改原数组。
- **注意**：别用 `arr.sort(() => Math.random() - 0.5)`，那不是真随机。

### 24. 两数组相似度（similarity）
- **作用**：返回 a 中也在 b 里的元素（保持 a 的顺序）。
- **代码**：
```javascript
const similarity = (arr, values) => arr.filter(v => values.includes(v));
similarity([1, 2, 3], [1, 2, 4]); // [1, 2]
```
- **说明**：和 intersection 类似，但 values 不是 Set 时写法更直接。
- **注意**：values 大时建议先 `new Set(values)` 提速。

### 25. 求和（sum / sumBy）
- **作用**：数组求和，或按字段求和。
- **代码**：
```javascript
const sum = arr => arr.reduce((acc, v) => acc + v, 0);
const sumBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : v => v[fn]).reduce((acc, v) => acc + v, 0);
sum([1, 2, 3]); // 6
sumBy([{ n: 4 }, { n: 2 }], o => o.n); // 6
```
- **说明**：订单金额合计、统计页核心计算。
- **注意**：空数组必须带初值 0，否则 reduce 报错。

### 26. 对称差（symmetricDifference）
- **作用**：只在其中一个数组出现的元素。
- **代码**：
```javascript
const symmetricDifference = (a, b) => {
  const sA = new Set(a), sB = new Set(b);
  return [...a.filter(x => !sB.has(x)), ...b.filter(x => !sA.has(x))];
};
symmetricDifference([1, 2, 3], [1, 2, 4]); // [3, 4]
```
- **说明**：对比两份数据「差异项」时用。
- **注意**：结果按 a 后接 b 的顺序排列。

### 27. 取前 n 个（take / takeRight）
- **作用**：从头/尾取 n 个。
- **代码**：
```javascript
const take = (arr, n = 1) => arr.slice(0, n);
const takeRight = (arr, n = 1) => arr.slice(-n);
take([1, 2, 3], 2);      // [1, 2]
takeRight([1, 2, 3], 2); // [2, 3]
```
- **说明**：列表只预览前几条，「查看更多」展开。
- **注意**：n 超过 length 时返回整个数组。

### 28. 并集（union）
- **作用**：合并并去重。
- **代码**：
```javascript
const union = (a, b) => Array.from(new Set([...a, ...b]));
union([1, 2, 3], [4, 3, 2]); // [1, 2, 3, 4]
```
- **说明**：标签合并、权限合并。
- **注意**：只对基本类型生效。

### 29. 去重（uniqueElements）
- **作用**：数组去重。
- **代码**：
```javascript
const uniqueElements = arr => [...new Set(arr)];
uniqueElements([1, 2, 2, 3, 4, 4]); // [1, 2, 3, 4]
```
- **说明**：最简洁的一维去重。
- **注意**：对象数组要按字段去重（见 uniqueBy）。

### 30. 按字段去重（uniqueBy）
- **作用**：对象数组按某 key 去重。
- **代码**：
```javascript
const uniqueBy = (arr, key) => {
  const seen = new Set();
  return arr.filter(x => {
    const k = typeof key === 'function' ? key(x) : x[key];
    if (seen.has(k)) return false;
    seen.add(k);
    return true;
  });
};
uniqueBy([{ id: 1 }, { id: 2 }, { id: 1 }], 'id'); // [{id:1},{id:2}]
```
- **说明**：接口返回的重复数据清洗。
- **注意**：保留「最先出现」的那条。

### 31. 反向排除（without）
- **作用**：去掉指定值后的新数组。
- **代码**：
```javascript
const without = (arr, ...args) => arr.filter(v => !args.includes(v));
without([2, 1, 2, 3], 1, 2); // [3]
```
- **说明**：批量剔除某些枚举值。
- **注意**：`args.includes` 复杂度 O(n)，黑名单大时换 Set。

### 32. 拉链合并（zip）
- **作用**：把多个数组按下标拼成二维。
- **代码**：
```javascript
const zip = (...arrays) =>
  Array.from({ length: Math.max(...arrays.map(a => a.length)) }, (_, i) =>
    arrays.map(a => a[i])
  );
zip(['a', 'b'], [1, 2], [true, false]); // [['a',1,true],['b',2,false]]
```
- **说明**：表头和数据行合并成行。
- **注意**：短数组缺项为 `undefined`。

### 33. 反拉链（unzip）
- **作用**：zip 的反向操作。
- **代码**：
```javascript
const unzip = arr =>
  arr.reduce(
    (acc, val) => (val.forEach((v, i) => acc[i].push(v)), acc),
    Array.from({ length: Math.max(...arr.map(a => a.length)) }, () => [])
  );
unzip([['a', 1, true], ['b', 2, false]]); // [['a','b'],[1,2],[true,false]]
```
- **说明**：把转置后的表再转回来。
- **注意**：行列数取决于最长那一行。

### 34. 深拷贝（deepClone）
- **作用**：递归拷贝嵌套对象/数组。
- **代码**：
```javascript
const deepClone = obj => {
  if (obj === null || typeof obj !== 'object') return obj;
  const copy = Array.isArray(obj) ? [] : {};
  Object.keys(obj).forEach(key => (copy[key] = deepClone(obj[key])));
  return copy;
};
deepClone({ a: { b: 1 }, c: [1, 2] });
```
- **说明**：不依赖 JSON 的手写深拷贝，保留 undefined 之外的常见结构。
- **注意**：不处理 Date/Map/Set/循环引用；那些场景用 `structuredClone()`。

### 35. 对象深合并（deepMerge）
- **作用**：把两个对象递归合并，后写的覆盖先写的。
- **代码**：
```javascript
const deepMerge = (a, b) => {
  const isObj = o => o && typeof o === 'object' && !Array.isArray(o);
  return Object.keys({ ...a, ...b }).reduce((acc, k) => {
    acc[k] = isObj(a[k]) && isObj(b[k]) ? deepMerge(a[k], b[k]) : b[k] ?? a[k];
    return acc;
  }, {});
};
deepMerge({ a: { x: 1, y: 2 } }, { a: { y: 3, z: 4 } }); // { a: { x:1, y:3, z:4 } }
```
- **说明**：默认配置 + 用户配置合并。
- **注意**：数组直接整体替换，不做逐元素合并。

### 36. 拍平后 map（flatMap）
- **作用**：map 之后再拍平一层。
- **代码**：
```javascript
const flatMap = (arr, fn) => arr.reduce((acc, x) => acc.concat(fn(x)), []);
flatMap([1, 2, 3], x => [x, x * 2]); // [1,2,2,4,3,6]
```
- **说明**：现代环境等价于 `arr.flatMap(fn)`。
- **注意**：只拍平一层。

### 37. 出现次数（countOccurrences）
- **作用**：统计每个元素出现次数。
- **代码**：
```javascript
const countOccurrences = (arr, val) =>
  arr.reduce((a, v) => (v === val ? a + 1 : a), 0);
countOccurrences([1, 1, 2, 1, 3], 1); // 3
```
- **说明**：列表里数一下「待处理」状态有几条。
- **注意**：用 `===` 比较，NaN 统计不到（用 Object.is 版可修）。

### 38. 找最后一个满足项（findLast）
- **作用**：从后往前找第一个满足条件的值。
- **代码**：
```javascript
const findLast = (arr, fn) => arr.filter(fn).pop();
findLast([1, 2, 3, 4], n => n % 2 === 1); // 3
```
- **说明**：找「最后一次提交」「最新一条」。
- **注意**：现代环境有原生 `arr.findLast(fn)`；本写法兼容老环境但会全量过滤。

### 39. 取值安全访问（dig）
- **作用**：安全地取嵌套路径，缺层不报错。
- **代码**：
```javascript
const dig = (obj, path) =>
  path.split('.').reduce((o, k) => (o == null ? o : o[k]), obj);
dig({ a: { b: { c: 1 } } }, 'a.b.c'); // 1
dig({ a: {} }, 'a.b.c');             // undefined
```
- **说明**：等价于手写可选链 `obj?.a?.b?.c`，但路径是字符串。
- **注意**：现代环境优先 `obj?.a?.b?.c`；动态路径字符串才用本函数。

### 40. 对象按值排序（orderBy）
- **作用**：对象数组按字段多键排序。
- **代码**：
```javascript
const orderBy = (arr, props, orders = ['asc']) =>
  [...arr].sort((a, b) =>
    props.reduce((acc, prop, i) => {
      if (acc !== 0) return acc;
      const d = a[prop] > b[prop] ? 1 : a[prop] < b[prop] ? -1 : 0;
      return orders[i] === 'desc' ? -d : d;
    }, 0)
  );
orderBy([{ age: 2, name: 'Z' }, { age: 1, name: 'A' }], ['age'], ['asc']);
```
- **说明**：表格多列排序。
- **注意**：返回新数组，原数组顺序不动。
