# 02 · 字符串与格式化

> 大小写、截断、命名风格、千分位、转义——文本处理的高频小件。

### 1. 首字母大写（capitalize）
- **作用**：字符串首字母转大写，其余保持原样。
- **代码**：
```javascript
const capitalize = ([first, ...rest]) =>
  first.toUpperCase() + rest.join('');
capitalize('fooBar'); // 'FooBar'
```
- **说明**：标题、用户名展示。
- **注意**：全大写其余字母不会变小写；要「首字母大写其余小写」用 `capitalize(str.toLowerCase())`。

### 2. 每个单词首字母大写（capitalizeEveryWord）
- **作用**：句中每个单词首字母都大写。
- **代码**：
```javascript
const capitalizeEveryWord = str =>
  str.replace(/\b[a-z]/g, char => char.toUpperCase());
capitalizeEveryWord('hello world'); // 'Hello World'
```
- **说明**：文章标题、英文人名格式。
- **注意**：依赖 `\b` 词边界，对含连字符/数字的文本表现一般。

### 3. 正则转义（escapeRegExp）
- **作用**：把字符串里的正则特殊字符转义，安全拼进 RegExp。
- **代码**：
```javascript
const escapeRegExp = str => str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
new RegExp(escapeRegExp('(test)')); // 精确匹配 "(test)"
```
- **说明**：用户搜索框拼正则时必做，否则 `( )` 会被当分组。
- **注意**：现代 JS 也有 `RegExp.escape(str)`（部分环境支持）。

### 4. 驼峰转普通（fromCamelCase）
- **作用**：camelCase 拆成用分隔符连接的小写串。
- **代码**：
```javascript
const fromCamelCase = (str, separator = '_') =>
  str
    .replace(/([a-z\d])([A-Z])/g, '$1' + separator + '$2')
    .replace(/([A-Z]+)([A-Z][a-z\d]+)/g, '$1' + separator + '$2')
    .toLowerCase();
fromCamelCase('someDatabaseFieldName', '-'); // 'some-database-field-name'
```
- **说明**：变量名转 URL slug / CSS 类名。
- **注意**：连续大写缩写（HTMLParser）也能正确切开。

### 5. 转驼峰（toCamelCase）
- **作用**：横杠/下划线/空格分隔的串转 camelCase。
- **代码**：
```javascript
const toCamelCase = str => {
  const s =
    str &&
    str
      .match(/[A-Z]{2,}(?=[A-Z][a-z]+[0-9]*|\b)|[A-Z]?[a-z]+[0-9]*|[A-Z]|[0-9]+/g)
      .map(x => x.slice(0, 1).toUpperCase() + x.slice(1).toLowerCase())
      .join('');
  return s.slice(0, 1).toLowerCase() + s.slice(1);
};
toCamelCase('some_database_field_name'); // 'someDatabaseFieldName'
```
- **说明**：后端 snake_case 字段转前端变量名。
- **注意**：输入非法时返回空串，先判空。

### 6. 截断字符串（truncate）
- **作用**：超过长度就截断并加省略号。
- **代码**：
```javascript
const truncate = (str, length = 100) =>
  str.length > length ? str.slice(0, length).trim() + '…' : str;
truncate('boomerang', 7); // 'boomera…'
```
- **说明**：列表卡片预览长文本。
- **注意**：截断后 `trim()` 避免省略号前多出空格。

### 7. 按词截断（truncateAtWords）
- **作用**：尽量在词边界截断，不把单词切一半。
- **代码**：
```javascript
const truncateAtWords = (str, n) =>
  str.length > n ? str.slice(0, str.slice(0, n).lastIndexOf(' ')) + '…' : str;
truncateAtWords('Introducing a great developers tool', 20);
// 'Introducing a great…'
```
- **说明**：摘要展示更美观。
- **注意**：长单词（无空格）时会直接硬切到 n。

### 8. 补白（pad / padStart / padEnd）
- **作用**：在字符串两侧/一侧补字符到固定长度。
- **代码**：
```javascript
const pad = (str, length, char = ' ') =>
  str.padStart((str.length + length) / 2, char).padEnd(length, char);
pad('cat', 8);           // '  cat   '
pad(String(42), 6, '0'); // '004200'
```
- **说明**：账单数字对齐、等宽打印。
- **注意**：`length` 是目标总长度；字符宽度 1。

### 9. 拆行（splitLines）
- **作用**：把多行字符串按行拆成数组，兼容 \r\n / \n。
- **代码**：
```javascript
const splitLines = str => str.split(/\r?\n/);
splitLines('line1\nline2\nline3'); // ['line1','line2','line3']
```
- **说明**：粘贴文本框逐行处理。
- **注意**：空行保留为空字符串元素。

### 10. 去 HTML 标签（stripHTMLTags）
- **作用**：把 HTML 串里的标签剥掉。
- **代码**：
```javascript
const stripHTMLTags = str => str.replace(/<[^>]*>/g, '');
stripHTMLTags('<p><em>lorem</em> <strong>ipsum</strong></p>'); // 'lorem ipsum'
```
- **说明**：富文本摘要转纯文本。
- **注意**：正则方案对恶意嵌套/属性里含 `>` 的 edge case 不完美；严格场景用 DOMParser。

### 11. 转短横线（toKebabCase）
- **作用**：转 kebab-case（URL slug 标准）。
- **代码**：
```javascript
const toKebabCase = str =>
  str
    .match(/[A-Z]{2,}(?=[A-Z][a-z]+[0-9]*|\b)|[A-Z]?[a-z]+[0-9]*|[A-Z]|[0-9]+/g)
    .map(x => x.toLowerCase())
    .join('-');
toKebabCase('camelCase'); // 'camel-case'
```
- **说明**：文章 URL、CSS 类名。
- **注意**：和 toCamelCase 用同一套分词规则。

### 12. 转下划线（toSnakeCase）
- **作用**：转 snake_case（后端字段名常用）。
- **代码**：
```javascript
const toSnakeCase = str =>
  str &&
  str
    .match(/[A-Z]{2,}(?=[A-Z][a-z]+[0-9]*|\b)|[A-Z]?[a-z]+[0-9]*|[A-Z]|[0-9]+/g)
    .map(x => x.toLowerCase())
    .join('_');
toSnakeCase('camelCase'); // 'camel_case'
```
- **说明**：前端对象 key 转后端约定。
- **注意**：null/空串先判空。

### 13. 取单词（words）
- **作用**：把字符串拆成单词数组。
- **代码**：
```javascript
const words = (str, pattern = /[^a-zA-Z-]+/) => str.split(pattern).filter(Boolean);
words('I love javaScript!!');      // ['I', 'love', 'javaScript']
words('python, javaScript & coffee'); // ['python', 'javaScript', 'coffee']
```
- **说明**：词频统计、标签提取。
- **注意**：pattern 默认只认英文，中文要换 `/[\u4e00-\u9fa5]+/g`。

### 14. 字节大小（byteSize）
- **作用**：返回字符串占用的字节数。
- **代码**：
```javascript
const byteSize = str => new Blob([str]).size;
byteSize('😀');   // 4
byteSize('Hello'); // 5
```
- **说明**：表单前端预校验输入大小。
- **注意**：Emoji/中文多字节，不能用 `length`。

### 15. 判断绝对 URL（isAbsoluteURL）
- **作用**：字符串是不是以协议开头的绝对 URL。
- **代码**：
```javascript
const isAbsoluteURL = str => /^[a-z][a-z0-9+.-]*:\/\//i.test(str);
isAbsoluteURL('https://example.com'); // true
isAbsoluteURL('/foo');                // false
```
- **说明**：路由跳转前决定 `a.href` 还是 `router.push`。
- **注意**：不校验域名合法性，只看 scheme 形态。

### 16. 回文判断（isPalindrome）
- **作用**：字符串正反读是否一样。
- **代码**：
```javascript
const isPalindrome = str => {
  const s = str.toLowerCase().replace(/[\W_]/g, '');
  return s === [...s].reverse().join('');
};
isPalindrome('taco cat'); // true
```
- **说明**：面试经典题，也用于文案趣味校验。
- **注意**：先统一小写并去掉标点空格再比。

### 17. 变位词判断（isAnagram）
- **作用**：两个字符串是否字母组成相同（顺序不同）。
- **代码**：
```javascript
const isAnagram = (str1, str2) => {
  const normalize = str =>
    str.toLowerCase().replace(/[^a-z0-9]/gi, '').split('').sort().join('');
  return normalize(str1) === normalize(str2);
};
isAnagram('listen', 'silent'); // true
```
- **说明**：单词游戏、密码强度辅助。
- **注意**：O(n log n)，超大字符串用计数桶更省。

### 18. 打码（mask）
- **作用**：把字符串中间打码，保留头尾。
- **代码**：
```javascript
const mask = (str, num = 4) =>
  str.slice(-num).padStart(str.length, '*');
mask('1234567890');      // '******7890'
mask('1234567890', 3);   // '*******890'
```
- **说明**：银行卡号、手机号展示。
- **注意**：保留位数由 num 控制，默认 4 位。

### 19. 重复字符串（repeatString）
- **作用**：把字符串重复 n 次。
- **代码**：
```javascript
const repeatString = (str, n) => (n > 0 ? [...Array(n)].join(str) : '');
repeatString('ha', 3); // 'hahaha'
```
- **说明**：老环境 `str.repeat(n)` 的替代。
- **注意**：现代环境直接用 `str.repeat(n)`。

### 20. 折叠空白（collapseWhitespace）
- **作用**：把连续空白压成单个空格。
- **代码**：
```javascript
const collapseWhitespace = str =>
  str.replace(/\s+/g, ' ').trim();
collapseWhitespace('  a   b  c\n'); // 'a b c'
```
- **说明**：用户复制粘贴进来的脏文本清洗。
- **注意**：会把换行也吃掉，需要保留换行时别用。

### 21. 千分位（toThousands）
- **作用**：数字加千分位逗号。
- **代码**：
```javascript
const toThousands = n =>
  String(n).replace(/\B(?=(\d{3})+(?!\d))/g, ',');
toThousands(1234567.89); // '1,234,567.89'
```
- **说明**：金额、统计数字展示。
- **注意**：更规范的做法是 `n.toLocaleString('zh-CN')`，本函数用于需要严格无语言环境时。

### 22. 去除音调符号（deburr）
- **作用**：把带重音的拉丁字符转成基础字母。
- **代码**：
```javascript
const deburr = str =>
  str.normalize('NFD').replace(/[\u0300-\u036f]/g, '');
deburr('ÀÁÂÃÄÅ'); // 'AAAAAA'
```
- **说明**：搜索时把「café」和「cafe」归一。
- **注意**：对中文无效，中文要单独做拼音映射。

### 23. 字符串包裹（wrapString）
- **作用**：用指定前后缀把字符串包起来。
- **代码**：
```javascript
const wrapString = (str, wrap) => wrap + str + wrap;
wrapString('hello', '*'); // '*hello*'
```
- **说明**：加引号、加 markdown 标记。
- **注意**：前后缀要不同就自己拼，别硬套。

### 24. 是否含空白（containsWhitespace）
- **作用**：字符串里是否有任意空白字符。
- **代码**：
```javascript
const containsWhitespace = str => /\s/.test(str);
containsWhitespace('lorem ipsum'); // true
```
- **说明**：用户名/密码规则校验。
- **注意**：`\s` 匹配空格、Tab、换行等。

### 25. 是否全小写（isLowerCase / isUpperCase）
- **作用**：字符串是否全部小写/大写。
- **代码**：
```javascript
const isLowerCase = str => str === str.toLowerCase();
const isUpperCase = str => str === str.toUpperCase();
isLowerCase('a');   // true
isUpperCase('A');   // true
```
- **说明**：表单格式提示。
- **注意**：没有字母时（纯数字）两者都为 true，需自行判空。

### 26. 字符串哈希（hashString）
- **作用**：把字符串散列成一个 32 位整数。
- **代码**：
```javascript
const hashString = str =>
  [].reduce.call(
    str,
    (hash, char) => (hash << 5) - hash + char.charCodeAt(0),
    0
  );
hashString('hello world'); // 285857635
```
- **说明**：做轻量缓存 key、分桶。
- **注意**：非加密哈希，不要用于安全场景。

### 27. 数字补零（padNumber）
- **作用**：数字前面补零到固定位数。
- **代码**：
```javascript
const padNumber = (n, len) => `${n}`.padStart(len, '0');
padNumber(12, 5); // '00012'
```
- **说明**：订单号、日期 `01` 月。
- **注意**：负数会把符号也算进位数里。

### 28. 转序数后缀（toOrdinalSuffix）
- **作用**：英文序数词 1st / 2nd / 3rd / 4th…
- **代码**：
```javascript
const toOrdinalSuffix = num => {
  const int = parseInt(num, 10);
  const digits = [int % 10, int % 100];
  const ordinals = ['st', 'nd', 'rd', 'th'];
  const o =
    digits[0] === 1 && digits[1] !== 11
      ? ordinals[0]
      : digits[0] === 2 && digits[1] !== 12
      ? ordinals[1]
      : digits[0] === 3 && digits[1] !== 13
      ? ordinals[2]
      : ordinals[3];
  return int + o;
};
toOrdinalSuffix(1);  // '1st'
toOrdinalSuffix(11); // '11th'
```
- **说明**：排行榜名次展示。
- **注意**：11/12/13 是 th，不是 st/nd/rd。

### 29. 缩写词（acronym）
- **作用**：从短语里取首字母缩写。
- **代码**：
```javascript
const acronym = str =>
  str.match(/\b(\w)/g).join('').toUpperCase();
acronym('JavaScript Object Notation'); // 'JSON'
```
- **说明**：标签自动生成。
- **注意**：`\b` 对中文不生效，中文要自己 split。

### 30. 字符串反转（reverseString）
- **作用**：反转字符串。
- **代码**：
```javascript
const reverseString = str => [...str].reverse().join('');
reverseString('foobar'); // 'raboof'
```
- **说明**：面试手写题常客。
- **注意**：用扩展运算符而非 `str.split('')`，才能正确处理 emoji / 代理对。

### 31. 模板填充（template）
- **作用**：用数据对象替换 `{{key}}` 占位。
- **代码**：
```javascript
const template = (str, data) =>
  str.replace(/{{(\w+)}}/g, (_, k) => (k in data ? data[k] : ''));
template('你好，{{name}}，今年 {{age}} 岁', { name: '小明', age: 18 });
// '你好，小明，今年 18 岁'
```
- **说明**：邮件、通知文案模板。
- **注意**：没匹配到的 key 替换为空串；需要保留原文就把第三参改成 `$&`。

### 32. 字符串插入（insertSubstr）
- **作用**：在指定位置插入子串。
- **代码**：
```javascript
const insertSubstr = (str, sub, pos) => str.slice(0, pos) + sub + str.slice(pos);
insertSubstr('123456', '***', 3); // '123***456'
```
- **说明**：卡号分段显示 1234 5678 9012。
- **注意**：pos 越界时插到末尾或开头，不报错。

### 33. 字符串删空格（trim / trimEnd）
- **作用**：去首尾空白（手写版）。
- **代码**：
```javascript
const trim = str => str.replace(/^\s+|\s+$/g, '');
trim('  x  '); // 'x'
```
- **说明**：现代环境直接用原生 `str.trim()`；本函数给老环境。
- **注意**：`\s` 包含换行和 Tab。

### 34. 首行首字大写其余原样（pascalCase）
- **作用**：转 PascalCase（大驼峰）。
- **代码**：
```javascript
const pascalCase = str =>
  str
    .replace(/[-_\s](.)/g, (_, c) => c.toUpperCase())
    .replace(/^./, c => c.toUpperCase());
pascalCase('some_database_field_name'); // 'SomeDatabaseFieldName'
```
- **说明**：类名、组件名。
- **注意**：比 toCamelCase 多首字母大写。

### 35. 比较版本号（compareVersion）
- **作用**：按 `1.2.3` 形式比较两个版本大小。
- **代码**：
```javascript
const compareVersion = (a, b) => {
  const pa = a.split('.'), pb = b.split('.');
  for (let i = 0; i < Math.max(pa.length, pb.length); i++) {
    const x = Number(pa[i] || 0), y = Number(pb[i] || 0);
    if (x > y) return 1;
    if (x < y) return -1;
  }
  return 0;
};
compareVersion('1.10.0', '1.9.1'); // 1
```
- **说明**：判断浏览器/SDK 版本兼容性。
- **注意**：不处理预发布标签（-beta），那种用 semver 库。

### 36. 安全 JSON 字符串化（safeStringify）
- **作用**：JSON.stringify 失败时兜底返回占位串，不抛异常。
- **代码**：
```javascript
const safeStringify = (obj, fallback = '[Circular]') => {
  try {
    return JSON.stringify(obj);
  } catch (_) {
    return fallback;
  }
};
safeStringify({ a: 1 });          // '{"a":1}'
const circ = {}; circ.self = circ;
safeStringify(circ);              // '[Circular]'
```
- **说明**：日志打印大对象防循环引用导致栈溢出。
- **注意**：fallback 只是兜底文案，不会尝试挑出可序列化的部分。

### 37. 提取 URL 参数（stringToURLSearch）
- **作用**：把 query string 转对象。
- **代码**：
```javascript
const stringToURLSearch = str =>
  [...new URLSearchParams(str)].reduce((acc, [k, v]) => ((acc[k] = v), acc), {});
stringToURLSearch('?foo=bar&baz=1'); // { foo: 'bar', baz: '1' }
```
- **说明**：解析 location.search。
- **注意**：重复 key 后面覆盖前面；需要数组用 URLSearchParams.getAll。

### 38. 数字格式化千分位 locale（formatNumber）
- **作用**：按语言环境格式化数字。
- **代码**：
```javascript
const formatNumber = (n, locale = 'zh-CN') => n.toLocaleString(locale);
formatNumber(1234567.89); // '1,234,567.89'
```
- **说明**：比手写正则稳，自动处理小数位和分组。
- **注意**：不同 locale 分组符不同（欧洲用点）。

### 39. 去重字符（uniqueChars）
- **作用**：字符串里出现过的字符去重。
- **代码**：
```javascript
const uniqueChars = str => [...new Set(str)].join('');
uniqueChars('abracadabra'); // 'abrcd'
```
- **说明**：统计用到了哪些符号。
- **注意**：扩展运算符能正确拆 Unicode 字符。

### 40. 字符串截断加后缀（truncateEllipsis）
- **作用**：截断时预留后缀长度。
- **代码**：
```javascript
const truncateEllipsis = (str, maxLen, suffix = '...') =>
  str.length > maxLen ? str.slice(0, maxLen - suffix.length) + suffix : str;
truncateEllipsis('hello world', 9); // 'hello...'
```
- **说明**：卡片标题定长展示。
- **注意**：maxLen 是「总长度」，suffix 长度已扣掉。
