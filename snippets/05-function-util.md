# 05 · 函数与工具

> 柯里化、记忆化、深比较、类型判断——函数式与日常工具小件。

### 1. 管道（pipe）
- **作用**：从左到右把函数串起来。
- **代码**：
```javascript
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);
const add1 = n => n + 1;
const double = n => n * 2;
pipe(add1, double)(3); // 8
```
- **说明**：数据流从左流到右。
- **注意**：每步输出是下一步输入。

### 2. 组合（compose）
- **作用**：从右到左组合函数（数学里的 f∘g）。
- **代码**：
```javascript
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const roundPlus1 = compose(Math.round, n => n + 1);
roundPlus1(3.4); // 4
```
- **说明**：和 pipe 方向相反。
- **注意**：命名别拼错，compose 不是 compole。

### 3. 柯里化（curry）
- **作用**：把多参函数变成一串单参函数。
- **代码**：
```javascript
const curry = fn => {
  const arity = fn.length;
  const curried = (...args) =>
    args.length >= arity
      ? fn(...args)
      : (...next) => curried(...args, ...next);
  return curried;
};
const add = curry((a, b, c) => a + b + c);
add(1)(2)(3);      // 6
add(1, 2)(3);      // 6
```
- **说明**：复用部分参数，生成专用函数。
- **注意**：基于 fn.length，带默认值的参数不计入。

### 4. 偏函数应用（partial）
- **作用**：预置部分参数，剩下的后补。
- **代码**：
```javascript
const partial = (fn, ...preset) => (...later) => fn(...preset, ...later);
const multiply = (a, b) => a * b;
const double = partial(multiply, 2);
double(4); // 8
```
- **说明**：比 curry 简单，只分两步。
- **注意**：占位符放中间要自己用 `_` 占位方案。

### 5. 只执行一次（once）
- **作用**：函数首次调用后再也不生效。
- **代码**：
```javascript
const once = fn => {
  let called = false;
  let result;
  return (...args) => {
    if (!called) {
      called = true;
      result = fn(...args);
    }
    return result;
  };
};
const init = once(() => console.log('init once'));
init(); init(); // 只打印一次
```
- **说明**：埋点只上报一次、初始化只跑一次。
- **注意**：多次调用返回第一次的结果。

### 6. 记忆化（memoize）
- **作用**：缓存函数计算结果。
- **代码**：
```javascript
const memoize = fn => {
  const cache = new Map();
  return (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const r = fn(...args);
    cache.set(key, r);
    return r;
  };
};
const fib = memoize(n => (n <= 1 ? n : fib(n - 1) + fib(n - 2)));
```
- **说明**：递归斐波那契瞬间变快。
- **注意**：JSON.stringify 做 key，函数/undefined 参数不适用。

### 7. 耗时测量（timeTaken）
- **作用**：测一段同步代码跑多久。
- **代码**：
```javascript
const timeTaken = callback => {
  console.time('timeTaken');
  const r = callback();
  console.timeEnd('timeTaken');
  return r;
};
timeTaken(() => Math.pow(2, 10)); // 1024, logged time
```
- **说明**：性能对比。
- **注意**：用 performance.now() 精度更高。

### 8. 深比较（deepEqual）
- **作用**：两个值是否深度相等。
- **代码**：
```javascript
const deepEqual = (a, b) => {
  if (a === b) return true;
  if (typeof a !== typeof b) return false;
  if (a && b && typeof a === 'object') {
    const keysA = Object.keys(a), keysB = Object.keys(b);
    if (keysA.length !== keysB.length) return false;
    return keysA.every(k => deepEqual(a[k], b[k]));
  }
  return false;\n};
deepEqual({ a: [1, { b: 2 }] }, { a: [1, { b: 2 }] }); // true
```
- **说明**：判断两次提交的数据是否有变化。
- **注意**：不处理 Date/Map/Set/循环引用；复杂场景用 lodash.isEqual。

### 9. 类型判断（type）
- **作用**：比 typeof 更准的类型标签。
- **代码**：
```javascript
const type = v =>
  v === undefined
    ? 'undefined'
    : v === null
    ? 'null'
    : Object.prototype.toString.call(v).slice(8, -1).toLowerCase();
type([]);         // 'array'
type(null);       // 'null'
type(new Date()); // 'date'
```
- **说明**：解决 `typeof null === 'object'` 的坑。
- **注意**：返回小写字符串。

### 10. 首函数调用（attempt）
- **作用**：执行可能抛错的函数，返回结果或错误对象。
- **代码**：
```javascript
const attempt = fn => {
  try {
    return fn();
  } catch (e) {
    return e;
  }
};
attempt(() => JSON.parse('{"a":1}')); // {a:1}
```
- **说明**：不用 try/catch 包裹业务代码。
- **注意**：判断返回值是 `instanceof Error`。

### 11. 反转参数（flip）
- **作用**：把函数参数顺序反过来。
- **代码**：
```javascript
const flip = fn => (first, ...rest) => fn(...rest, first);
const divide = (a, b) => a / b;
const safeDiv = flip(divide);
safeDiv(10, 2); // 0.2
```
- **说明**：函数式小工具。
- **注意**：这里是把第一个挪到最后。

### 12. 否定谓词（negate）
- **作用**：把返回布尔的函数取反。
- **代码**：
```javascript
const negate = fn => (...args) => !fn(...args);
const isEven = n => n % 2 === 0;
const isOdd = negate(isEven);
isOdd(3); // true
```
- **说明**：filter 里复用现有谓词。
- **注意**：只是布尔取反，不影响原函数。

### 13. 一次执行多个函数（over）
- **作用**：对同一组参数跑多个函数，返回结果数组。
- **代码**：
```javascript
const over = (...fns) => (...args) => fns.map(fn => fn(...args));
const minMax = over(Math.min, Math.max);
minMax(1, 2, 3, 4); // [1, 4]
```
- **说明**：同时取 min 和 max。
- **注意**：fns 都要能接受同样的参数。

### 14. 防抖（debounceFn）
- **作用**：函数工具版防抖（和 DOM 那个同思想，这里不绑事件）。
- **代码**：
```javascript
const debounceFn = (fn, ms = 300) => {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), ms);
  };
};
```
- **说明**：和 DOM 里的 debounce 一致，摘出来做通用工具。
- **注意**：需要 cancel 就再加 timer 暴露。

### 15. 节流（throttleFn）
- **作用**：通用版节流。
- **代码**：
```javascript
const throttleFn = (fn, ms = 300) => {
  let waiting = false;
  return (...args) => {
    if (waiting) return;
    waiting = true;
    fn(...args);
    setTimeout(() => (waiting = false), ms);
  };
};
```
- **说明**：和 DOM 节流通用版。
- **注意**：这版是 leading-only，最后 trailing 那次不补。

### 16. 深冻结（deepFreeze）
- **作用**：递归冻结对象，禁止修改。
- **代码**：
```javascript
const deepFreeze = obj => {
  Object.keys(obj).forEach(prop => {
    if (typeof obj[prop] === 'object' && obj[prop] !== null) deepFreeze(obj[prop]);
  });
  return Object.freeze(obj);
};
const config = deepFreeze({ api: { host: 'x' } });
```
- **说明**：常量配置防止误改。
- **注意**：严格模式下修改会报错，非严格模式静默失败。

### 17. 深映射（deepMap）
- **作用**：递归遍历嵌套对象/数组并对每个叶子值做变换。
- **代码**：
```javascript
const deepMap = (obj, fn) =>
  Object.keys(obj).reduce((acc, k) => {
    acc[k] =
      typeof obj[k] === 'object' && obj[k] !== null
        ? deepMap(obj[k], fn)
        : fn(obj[k]);
    return acc;
  }, Array.isArray(obj) ? [] : {});
deepMap({ a: 1, b: { c: 2 } }, v => v * 2);
```
- **说明**：把整个对象里的数字翻倍。
- **注意**：不处理循环引用。

### 18. 默认值合并（defaults）
- **作用**：给对象填默认值（不覆盖已有）。
- **代码**：
```javascript
const defaults = (obj, defaults) =>
  Object.keys(defaults).reduce(
    (acc, k) => (acc[k] = obj[k] ?? defaults[k], acc),
    { ...obj }
  );
defaults({ a: 1 }, { a: 0, b: 0 }); // { a: 1, b: 0 }
```
- **说明**：配置合并。
- **注意**：只浅合并；嵌套对象用 deepMerge。

### 19. 类型判断：是函数（isFunction）
- **作用**：值是不是函数。
- **代码**：
```javascript
const isFunction = fn => typeof fn === 'function';
isFunction(() => {}); // true
```
- **说明**：回调可选参数判断。
- **注意**：class 也是 function。

### 20. 类型判断：是原始值（isPrimitive）
- **作用**：值是不是 string/number/boolean/symbol/null/undefined。
- **代码**：
```javascript
const isPrimitive = val =>
  val === null || (typeof val !== 'object' && typeof val !== 'function');
isPrimitive(1);    // true
isPrimitive({});   // false
```
- **说明**：拷贝前判断要不要递归。
- **注意**：function 不算 primitive。

### 21. 范围映射（mapRange）
- **作用**：把数值从一个区间线性映射到另一个区间。
- **代码**：
```javascript
const mapRange = (n, inMin, inMax, outMin, outMax) =>
  ((n - inMin) * (outMax - outMin)) / (inMax - inMin) + outMin;
mapRange(0.5, 0, 1, 0, 100); // 50
```
- **说明**：进度条、滑块值换算。
- **注意**：不会自动 clamp，超出范围会外推。

### 22. 范围钳制（clamp）
- **作用**：把数限制在 [min, max] 之间。
- **代码**：
```javascript
const clamp = (num, min, max) => Math.min(Math.max(num, min), max);
clamp(5, 0, 10); // 5
clamp(-3, 0, 10); // 0
```
- **说明**：防止滑块/缩放越界。
- **注意**：min > max 时结果不可预期，先保证顺序。

### 23. 取 min/max（minOf / maxOf）
- **作用**：直接取一组数的极值。
- **代码**：
```javascript
const minOf = arr => Math.min(...arr);
const maxOf = arr => Math.max(...arr);
minOf([1, 2, 3]); // 1
```
- **说明**：比 reduce 写起来短。
- **注意**：空数组返回 Infinity/-Infinity。

### 24. 平均值（average）
- **作用**：数组平均值。
- **代码**：
```javascript
const average = arr => arr.reduce((a, b) => a + b, 0) / arr.length;
average([1, 2, 3, 4]); // 2.5
```
- **说明**：统计均值。
- **注意**：空数组除零得 NaN，先判 length。

### 25. 按字段平均（averageBy）
- **作用**：对象数组按字段求平均。
- **代码**：
```javascript
const averageBy = (arr, fn) =>
  arr.map(typeof fn === 'function' ? fn : x => x[fn]).reduce((a, b) => a + b, 0) /
  arr.length;
averageBy([{ n: 4 }, { n: 2 }], o => o.n); // 3
```
- **说明**：班级平均分。
- **注意**：空数组除零。

### 26. 格式化字节（prettyBytes）
- **作用**：字节数转可读的 KB/MB/GB。
- **代码**：
```javascript
const prettyBytes = num => {
  const units = ['B', 'KB', 'MB', 'GB', 'TB', 'PB'];
  if (num === 0) return '0 B';
  const i = Math.floor(Math.log10(Math.abs(num)) / 3);
  return (num / 10 ** (3 * i)).toFixed(2) + ' ' + units[i];
};
prettyBytes(1000);    // '1000.00 B'
prettyBytes(1024*1024); // '1.05 MB'
```
- **说明**：文件大小展示。
- **注意**：按 1000 还是 1024？这里 log10/3 是 1000 进制。

### 27. 数字补零工具（padNumberFn）
- **作用**：和字符串那个 padNumber 同源，接受 number。
- **代码**：
```javascript
const padNumberFn = (n, len) => String(n).padStart(len, '0');
padNumberFn(3, 4); // '0003'
```
- **说明**：订单号、序号。
- **注意**：负数会把符号占位数。

### 28. 一次调用多函数（attemptAll）
- **作用**：对一组函数全部调用，收集结果或错误。
- **代码**：
```javascript
const attemptAll = fns => fns.map(attempt);
attemptAll([() => 1, () => JSON.parse('bad')]);
// [1, SyntaxError]
```
- **说明**：批量初始化，别一个崩全崩。
- **注意**：和 settle 类似，但是同步版。

### 29. 函数名获取（functionName）
- **作用**：拿到函数的名字。
- **代码**：
```javascript
const functionName = fn => (console.debug(fn.name), fn);
functionName(function myNamed() {}); // 打印 'myNamed'
```
- **说明**：调试日志。
- **注意**：匿名函数 name 是 ''。

### 30. 类型判断：是日期（isDate）
- **作用**：值是不是 Date 对象。
- **代码**：
```javascript
const isDate = val =>
  val instanceof Date && !Number.isNaN(val.valueOf());
isDate(new Date()); // true
isDate(new Date('bad')); // false
```
- **说明**：Invalid Date 也排除掉。
- **注意**：跨 iframe 时 instanceof 不可靠，用 toString 判。

### 31. 类型判断：是 Promise（isPromise）
- **作用**：值是不是 thenable。
- **代码**：
```javascript
const isPromise = x =>
  !!x && (typeof x === 'object' || typeof x === 'function') && typeof x.then === 'function';
isPromise(Promise.resolve(1)); // true
```
- **说明**：await 前先判一下。
- **注意**：thenable 不严格等于 Promise。

### 32. 深度遍历（deepWalk）
- **作用**：递归遍历对象所有叶子。
- **代码**：
```javascript
const deepWalk = (obj, fn, path = '') => {
  Object.entries(obj).forEach(([k, v]) => {
    const p = path ? `${path}.${k}` : k;
    if (v && typeof v === 'object') deepWalk(v, fn, p);
    else fn(v, p);
  });
};
deepWalk({ a: { b: 1 } }, (v, p) => console.log(v, p));
// 1 'a.b'
```
- **说明**：把整个 JSON 拍平成路径。
- **注意**：不处理数组索引。

### 33. 默认参数兜底（coalesce）
- **作用**：返回第一个非 null/undefined 的值。
- **代码**：
```javascript
const coalesce = (...args) => args.find(v => v != null);
coalesce(null, undefined, 0, 'a'); // 0
```
- **说明**：和 `??` 类似但支持多候选。
- **注意**：0、''、false 都会被保留。

### 34. 链式异步（chainAsync）
- **作用**：依次跑 async 函数，把结果串起来。
- **代码**：
```javascript
const chainAsync = fns => {
  let curr = 0;
  const next = () => fns[curr++](next);
  next();
};
chainAsync([
  n => { console.log(1); n(); },
  n => { console.log(2); n(); },
]);
```
- **说明**：老版异步流程控制。
- **注意**：现代用 async/await 或 promisePool。

### 35. 等待一帧（nextFrame）
- **作用**：等到下一帧再执行。
- **代码**：
```javascript
const nextFrame = () => new Promise(r => requestAnimationFrame(r));
await nextFrame();
el.classList.add('show'); // 强制触发过渡
```
- **说明**：加 class 前强制浏览器先渲染一帧。
- **注意**：SSR 下没有 requestAnimationFrame，先判 window。

### 36. 执行 N 次（times）
- **作用**：对每个 index 执行函数并收集结果。
- **代码**：
```javascript
const times = (n, fn) => Array.from({ length: n }, (_, i) => fn(i));
times(5, i => i * i); // [0, 1, 4, 9, 16]
```
- **说明**：造测试数据。
- **注意**：fn 接收 index。

### 37. 数值校验（validateNumber）
- **作用**：值是不是有效数字。
- **代码**：
```javascript
const validateNumber = n => !isNaN(parseFloat(n)) && isFinite(Number(n));
validateNumber('10'); // true
validateNumber('10a'); // false
```
- **说明**：表单数字校验。
- **注意**：parseFloat('10a') 会得到 10，所以要再 Number(n) 对比。

### 38. 浅比较 shallowEqual
- **作用**：对象一级 key/value 是否相等。
- **代码**：
```javascript
const shallowEqual = (a, b) => {
  const ka = Object.keys(a), kb = Object.keys(b);
  if (ka.length !== kb.length) return false;
  return ka.every(k => a[k] === b[k]);
};
shallowEqual({ a: 1 }, { a: 1 }); // true
```
- **说明**：React.memo 自定义比较。
- **注意**：嵌套对象是引用比较。

### 39. 防抖取最后（debounceLast）
- **作用**：和 debounce 一样，但返回一个 promise 等到最后一次执行。
- **代码**：
```javascript
const debounceLast = (fn, ms = 300) => {
  let timer;
  return (...args) =>
    new Promise(resolve => {
      clearTimeout(timer);
      timer = setTimeout(() => resolve(fn(...args)), ms);
    });
};
```
- **说明**：搜索框输入完才拿结果。
- **注意**：中间触发会丢弃之前的 promise resolve 时机。

### 40. 组合式并行（parallelAll）
- **作用**：把一组函数并行调用，按 key 收集结果。
- **代码**：
```javascript
const parallelAll = async obj =>
  Object.fromEntries(
    await Promise.all(
      Object.entries(obj).map(async ([k, p]) => [k, await p()])
    )
  );
const { u, o } = await parallelAll({
  u: () => fetch('/user').then(r => r.json()),
  o: () => fetch('/orders').then(r => r.json()),
});
```
- **说明**：和 asyncIf/awaitAll 一组，命名取语义。
- **注意**：传「函数返回 promise」而不是 promise 本体。
