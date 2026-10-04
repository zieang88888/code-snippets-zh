# 07 · 数字与数学

> 随机、取整、进制、统计——数字处理高频小件。

### 1. 范围内随机数（randomInRange）
- **作用**：[min, max) 内的随机整数。
- **代码**：
```javascript
const randomIntInRange = (min, max) =>
  Math.floor(Math.random() * (max - min)) + min;
randomIntInRange(1, 10); // 1~9
```
- **说明**：测试数据、抽签。
- **注意**：上限不包含；要包含 max 就 max+1。

### 2. 随机浮点（randomFloatInRange）
- **作用**：[min, max) 内随机浮点。
- **代码**：
```javascript
const randomFloatInRange = (min, max) =>
  Math.random() * (max - min) + min;
randomFloatInRange(1.5, 5.5); // 2.342...
```
- **说明**：模拟传感器数据。
- **注意**：Math.random 不是加密级，别用于安全。

### 3. 数组随机（sample）
- **作用**：从数组随机取一个。
- **代码**：
```javascript
const sample = arr => arr[Math.floor(Math.random() * arr.length)];
sample([1, 2, 3, 4]); // 随机一个
```
- **说明**：随机一句话、随机奖励。
- **注意**：空数组返回 undefined。

### 4. 数组随机打乱（shuffle）
- **作用**：洗牌算法。
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
shuffle([1, 2, 3, 4]); // 乱序
```
- **说明**：随机出题。
- **注意**：Fisher-Yates 洗牌，不是 sort(() => Math.random()-0.5)。

### 5. 取整到指定位数（roundTo）
- **作用**：保留 n 位小数四舍五入。
- **代码**：
```javascript
const roundTo = (n, decimals = 0) => {
  const p = 10 ** decimals;
  return Math.round((n + Number.EPSILON) * p) / p;
};
roundTo(1.005, 2);  // 1.01
roundTo(1.23456, 2); // 1.23
```
- **说明**：金额、百分比展示。
- **注意**：加 Number.EPSILON 修正浮点误差（0.1+0.2=0.30000000004）。

### 6. 数值钳制（clampNumber）
- **作用**：和 05 那个 clamp 同义，这里单独给数字场景。
- **代码**：
```javascript
const clampNumber = (num, a, b) =>
  Math.max(Math.min(num, Math.max(a, b)), Math.min(a, b));
clampNumber(5, 0, 10);  // 5
clampNumber(-3, 0, 10); // 0
```
- **说明**：滑块值保护。
- **注意**：自动处理 a > b 的情况。

### 7. 数字补零（padNumber）
- **作用**：把数字左侧补零到固定长度。
- **代码**：
```javascript
const padNumber = (n, l) => String(n).padStart(l, '0');
padNumber(123, 6); // '000123'
```
- **说明**：学号、订单号。
- **注意**：负数会把符号当一位。

### 8. 百分位计算（toPercentage）
- **作用**：小数转百分比字符串。
- **代码**：
```javascript
const toPercentage = (num, digits = 2) => `${(num * 100).toFixed(digits)}%`;
toPercentage(0.12345); // '12.35%'
```
- **说明**：进度、转化率。
- **注意**：toFixed 会四舍五入。

### 9. 进制转换（toDecimal / fromDecimal）
- **作用**：n 进制数和十进制互转。
- **代码**：
```javascript
const toDecimal = (n, base) => parseInt(n, base);
const fromDecimal = (n, base) => n.toString(base);
toDecimal('ff', 16);    // 255
fromDecimal(255, 16);  // 'ff'
```
- **说明**：颜色 hex 和十进制互转。
- **注意**：toString(base) base 范围 2-36。

### 10. 数字求和（sum）
- **作用**：数组求和。
- **代码**：
```javascript
const sum = arr => arr.reduce((acc, n) => acc + n, 0);
sum([1, 2, 3, 4]); // 10
```
- **说明**：统计合计。
- **注意**：空数组得 0。

### 11. 数字乘积（product）
- **作用**：数组乘积。
- **代码**：
```javascript
const product = arr => arr.reduce((acc, n) => acc * n, 1);
product([1, 2, 3, 4]); // 24
```
- **说明**：阶乘就是 product([1..n])。
- **注意**：初始值必须 1，不能 0。

### 12. 中位数（median）
- **作用**：数组中位数。
- **代码**：
```javascript
const median = arr => {
  const s = [...arr].sort((a, b) => a - b);
  const mid = Math.floor(s.length / 2);
  return s.length % 2 ? s[mid] : (s[mid - 1] + s[mid]) / 2;
};
median([1, 3, 2, 5]); // 2.5
```
- **说明**：比平均值抗极端值。
- **注意**：拷贝后再 sort，别原地改。

### 13. 众数（mode）
- **作用**：数组里出现最多的值。
- **代码**：
```javascript
const mode = arr => {
  const c = new Map();
  arr.forEach(x => c.set(x, (c.get(x) || 0) + 1));
  return [...c.entries()].sort((a, b) => b[1] - a[1])[0][0];
};
mode([1, 2, 2, 3, 3, 3]); // 3
```
- **说明**：投票统计。
- **注意**：并列时取先出现的。

### 14. 标准差（standardDeviation）
- **作用**：数组标准差。
- **代码**：
```javascript
const standardDeviation = arr => {
  const n = arr.length;
  const mean = arr.reduce((a, b) => a + b, 0) / n;
  return Math.sqrt(arr.reduce((a, b) => a + (b - mean) ** 2, 0) / n);
};
standardDeviation([1, 2, 3, 4, 5]); // ~1.414
```
- **说明**：数据离散程度。
- **注意**：这是总体标准差（除以 n），样本标准差除以 n-1。

### 15. 阶乘（factorial）
- **作用**：n!。
- **代码**：
```javascript
const factorial = n => n < 0 ? (() => { throw new Error('negative') })() : n <= 1 ? 1 : n * factorial(n - 1);
factorial(5); // 120
```
- **说明**：组合数学。
- **注意**：n 别太大，会爆栈。

### 16. 斐波那契（fibonacci）
- **作用**：生成前 n 个斐波那契数。
- **代码**：
```javascript
const fibonacci = n => {
  const fib = [0, 1];
  for (let i = 2; i < n; i++) fib.push(fib[i - 1] + fib[i - 2]);
  return fib.slice(0, n);
};
fibonacci(8); // [0, 1, 1, 2, 3, 5, 8, 13]
```
- **说明**：递归 + memoize 之前先看这个迭代版。
- **注意**：迭代 O(n)，别用纯递归。

### 17. 最大公约数（gcd）
- **作用**：两个数最大公约数。
- **代码**：
```javascript
const gcd = (...arr) => {
  const _gcd = (x, y) => (!y ? x : _gcd(y, x % y));
  return [...arr].reduce((a, b) => _gcd(a, b));
};
gcd(8, 36); // 4
```
- **说明**：约分。
- **注意**：欧几里得算法。

### 18. 最小公倍数（lcm）
- **作用**：两个数最小公倍数。
- **代码**：
```javascript
const lcm = (...arr) => {
  const g = (x, y) => (!y ? x : g(y, x % y));
  return [...arr].reduce((a, b) => (a * b) / g(a, b));
};
lcm(12, 7); // 84
```
- **说明**：周期对齐。
- **注意**：a*b/gcd(a,b)。

### 19. 数字转罗马（toRomanNumeral）
- **作用**：1-3999 转罗马数字。
- **代码**：
```javascript
const toRomanNumeral = num => {
  const map = [[1000,'M'],[900,'CM'],[500,'D'],[400,'CD'],[100,'C'],[90,'XC'],[50,'L'],[40,'XL'],[10,'X'],[9,'IX'],[5,'V'],[4,'IV'],[1,'I']];
  return map.reduce((r, [v, s]) => r + s.repeat(Math.floor(num / v)) && (num -= v * Math.floor(num / v), r + s.repeat(Math.floor(num / v))), '');
};
toRomanNumeral(2026); // 'MMXXVI'
```
- **说明**：电影年份、章节编号。
- **注意**：上面 reduce 写法绕，更直观见下方注意。

### 20. 罗马数字直观版（toRomanSimple）
- **作用**：和上同，更易读。
- **代码**：
```javascript
const toRomanSimple = num => {
  const map = [[1000,'M'],[900,'CM'],[500,'D'],[400,'CD'],[100,'C'],[90,'XC'],[50,'L'],[40,'XL'],[10,'X'],[9,'IX'],[5,'V'],[4,'IV'],[1,'I']];
  let out = '';
  for (const [v, s] of map) {
    while (num >= v) {
      out += s;
      num -= v;
    }
  }
  return out;
};
toRomanSimple(42); // 'XLII'
```
- **说明**：推荐用这版。
- **注意**：num 必须 1-3999。

### 21. 数字转中文金额（numberToChinese）
- **作用**：整数金额转中文大写（简版）。
- **代码**：
```javascript
const numberToChinese = n => {
  const digits = '零一二三四五六七八九';
  const units = ['', '十', '百', '千'];
  const bigUnits = ['', '万', '亿'];
  let out = '';
  let s = String(n);
  for (let i = 0; i < s.length; i++) {
    const d = +s[i];
    const pos = s.length - 1 - i;
    out += digits[d];
    if (d !== 0) out += units[pos % 4];
    if (pos % 4 === 0) out += bigUnits[pos / 4];
  }
  return out.replace(/零+/g, '零').replace(/零$/g, '');
};
numberToChinese(2026); // '二千零二十六'
```
- **说明**：发票、合同金额。
- **注意**：简化版，不含小数和「壹贰」大写。

### 22. 数字千分位（toThousands）
- **作用**：1234567 → 1,234,567。
- **代码**：
```javascript
const toThousands = n => n.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ',');
toThousands(1234567); // '1,234,567'
```
- **说明**：金额展示。
- **注意**：现代环境用 `n.toLocaleString('zh-CN')`。

### 23. 字节数人类可读（bytesToHuman）
- **作用**：和 05 那个 prettyBytes 同源，这里 1024 进制。
- **代码**：
```javascript
const bytesToHuman = bytes => {
  const u = ['B', 'KB', 'MB', 'GB', 'TB'];
  const i = Math.floor(Math.log(bytes) / Math.log(1024));
  return (bytes / 1024 ** i).toFixed(2) + ' ' + u[i];
};
bytesToHuman(1024 * 1024); // '1.00 MB'
```
- **说明**：文件大小。
- **注意**：log(0) 是 -Infinity，先判 0。

### 24. 范围随机取 N 个（randomIntList）
- **作用**：生成 n 个 [min, max) 随机整数。
- **代码**：
```javascript
const randomIntList = (n, min, max) =>
  Array.from({ length: n }, () => Math.floor(Math.random() * (max - min)) + min);
randomIntList(5, 1, 100);
```
- **说明**：造测试数据。
- **注意**：可能重复，要去重用 shuffle + slice。

### 25. 范围随机不重复（sampleSize）
- **作用**：从数组随机抽 n 个不重复。
- **代码**：
```javascript
const sampleSize = ([...arr], n = 1) => shuffle(arr).slice(0, n);
sampleSize([1, 2, 3, 4, 5], 2);
```
- **说明**：随机抽题。
- **注意**：内部 shuffle 后取前 n。

### 26. 近似比较（approximatelyEqual）
- **作用**：两个浮点是否近似相等。
- **代码**：
```javascript
const approximatelyEqual = (v1, v2, epsilon = 1e-10) =>
  Math.abs(v1 - v2) < epsilon;
approximatelyEqual(0.1 + 0.2, 0.3); // true
```
- **说明**：避开 0.1+0.2≠0.3 坑。
- **注意**：别用 === 比浮点。

### 27. 数字符号（signed）
- **作用**：加正负号展示。
- **代码**：
```javascript
const signed = n => (n > 0 ? `+${n}` : `${n}`);
signed(5);  // '+5'
signed(-3); // '-3'
```
- **说明**：涨跌展示。
- **注意**：0 不带号。

### 28. 货币格式化（formatCurrency）
- **作用**：金额加 ¥/$ 和千分位。
- **代码**：
```javascript
const formatCurrency = (n, symbol = '¥') =>
  `${symbol}${Number(n).toLocaleString('zh-CN', { minimumFractionDigits: 2 })}`;
formatCurrency(1234567.8); // '¥1,234,567.80'
```
- **说明**：价格展示。
- **注意**：toLocaleString 自动千分位。

### 29. 整数交换（swapTwo）
- **作用**：交换两个变量。
- **代码**：
```javascript
let a = 1, b = 2;
[a, b] = [b, a];
// a=2, b=1
```
- **说明**：解构一行搞定。
- **注意**：经典 ES6。

### 30. 数字取整到步长（snapToStep）
- **作用**：吸附到步长倍数。
- **代码**：
```javascript
const snapToStep = (n, step) => Math.round(n / step) * step;
snapToStep(3.4, 1);   // 3
snapToStep(3.6, 0.5); // 3.5
```
- **说明**：画布吸附对齐。
- **注意**：round 换 floor/ceil 是向下/向上吸附。

### 31. 数字哈希（hashCode）
- **作用**：字符串转数字哈希。
- **代码**：
```javascript
const hashCode = s =>
  s.split('').reduce((h, c) => (h = (h << 5) - h + c.charCodeAt(0)) | 0, 0);
hashCode('hello'); // 99162322
```
- **说明**：把字符串当 key 分桶。
- **注意**：不是加密哈希，别用于安全。

### 32. 角度弧度互换（degToRad / radToDeg）
- **作用**：几何计算。
- **代码**：
```javascript
const degToRad = deg => (deg * Math.PI) / 180.0;
const radToDeg = rad => (rad * 180.0) / Math.PI;
degToRad(180); // 3.14159...
```
- **说明**：canvas 绘图。
- **注意**：JS Math.sin 吃弧度。

### 33. 两点距离（distance）
- **作用**：平面两点欧氏距离。
- **代码**：
```javascript
const distance = (x0, y0, x1, y1) => Math.hypot(x1 - x0, y1 - y0);
distance(0, 0, 3, 4); // 5
```
- **说明**：canvas 碰撞检测。
- **注意**：Math.hypot 防溢出。

### 34. 角度差（angleDiff）
- **作用**：两个角度最小差（-180~180）。
- **代码**：
```javascript
const angleDiff = (a, b) => {
  let d = (b - a) % 360;
  if (d > 180) d -= 360;
  if (d < -180) d += 360;
  return d;
};
angleDiff(350, 10); // 20
```
- **说明**：旋转动画最短路径。
- **注意**：环形问题。

### 35. 数字校验（isNaturalNumber）
- **作用**：n 是否自然数（≥0 整数）。
- **代码**：
```javascript
const isNaturalNumber = n => typeof n === 'number' && n >= 0 && Number.isInteger(n);
isNaturalNumber(3); // true
isNaturalNumber(-1); // false
```
- **说明**：表单校验。
- **注意**：Number.isInteger 排除小数。

### 36. 数字缩写（abbreviateNumber）
- **作用**：12345 → 1.2万 / 12.3K。
- **代码**：
```javascript
const abbreviateNumber = (num, digits = 1) => {
  const si = [
    { v: 1e12, s: 'T' }, { v: 1e9, s: 'B' }, { v: 1e6, s: 'M' },
    { v: 1e3, s: 'K' },
  ];
  for (const { v, s } of si) {
    if (num >= v) return (num / v).toFixed(digits) + s;
  }
  return String(num);
};
abbreviateNumber(12345);    // '12.3K'
abbreviateNumber(1500000);  // '1.5M'
```
- **说明**：粉丝数、点赞数。
- **注意**：要中文「万/亿」自己加 map。
