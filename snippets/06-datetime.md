# 06 · 日期时间

> 格式化、相对时间、周数、加减天——Date 实操高频小件。

### 1. 两个日期差几天（getDaysDiffBetweenDates）
- **作用**：两个 ISO 日期之间差多少天。
- **代码**：
```javascript
const getDaysDiffBetweenDates = (dateInitial, dateFinal) =>
  (new Date(dateFinal) - new Date(dateInitial)) / (1000 * 3600 * 24);
getDaysDiffBetweenDates('2024-12-01', '2024-12-10'); // 9
```
- **说明**：活动倒计时、签到连续天数。
- **注意**：跨夏令时地区会有小时级偏差；纯日期 UTC 化更稳。

### 2. HH:mm:ss 格式（getColonTimeFromDate）
- **作用**：拿到本地时间的 HH:mm:ss 串。
- **代码**：
```javascript
const getColonTimeFromDate = date => date.toTimeString().slice(0, 8);
getColonTimeFromDate(new Date()); // '14:30:00'
```
- **说明**：日志时间戳。
- **注意**：本地时区；UTC 用 toUTCString().slice(17,25)。

### 3. 当月天数（getDaysInMonth）
- **作用**：某年某月有几天。
- **代码**：
```javascript
const getDaysInMonth = (year, month) => new Date(year, month, 0).getDate();
getDaysInMonth(2024, 2); // 29（闰年）
```
- **说明**：日历组件渲染。
- **注意**：month 是 0-based（1 月 = 0）。

### 4. 上午下午后缀（getMeridiemSuffix）
- **作用**：把 24 小时数字转成 AM/PM 后缀。
- **代码**：
```javascript
const getMeridiemSuffixOfInteger = num =>
  num === 0 || num === 24
    ? 12 + 'am'
    : num === 12
    ? 12 + 'pm'
    : num < 12
    ? (num % 12) + 'am'
    : (num % 12) + 'pm';
getMeridiemSuffixOfInteger(0);  // '12am'
getMeridiemSuffixOfInteger(13); // '1pm'
```
- **说明**：兼容美区展示。
- **注意**：0 和 24 都当 12am。

### 5. 相对时间（timeAgo）
- **作用**：把 Date 转成「3 分钟前」「2 天前」。
- **代码**：
```javascript
const timeAgo = date => {
  const diff = (Date.now() - new Date(date).getTime()) / 1000;
  const units = [
    ['年', 3600 * 24 * 365],
    ['月', 3600 * 24 * 30],
    ['周', 3600 * 24 * 7],
    ['天', 3600 * 24],
    ['小时', 3600],
    ['分钟', 60],
    ['秒', 1],
  ];
  for (const [name, sec] of units) {
    const v = Math.floor(diff / sec);
    if (v >= 1) return `${v} ${name}前`;
  }
  return '刚刚';
};
timeAgo(Date.now() - 3 * 60 * 1000); // '3 分钟前'
```
- **说明**：列表发布时间。
- **注意**：月份按 30 天近似，精确历法用 date-fns。

### 6. 明天 / 昨天（tomorrow / yesterday）
- **作用**：拿明天/昨天的 Date 对象。
- **代码**：
```javascript
const tomorrow = () => {
  let t = new Date();
  t.setDate(t.getDate() + 1);
  return t;
};
const yesterday = () => {
  let t = new Date();
  t.setDate(t.getDate() - 1);
  return t;
};
```
- **说明**：默认日期范围选择。
- **注意**：setDate 自动跨月跨年。

### 7. 最大/最小日期（maxDate / minDate）
- **作用**：一组日期里找最新/最旧。
- **代码**：
```javascript
const maxDate = dates => new Date(Math.max(...dates.map(d => new Date(d))));
const minDate = dates => new Date(Math.min(...dates.map(d => new Date(d))));
maxDate([new Date('2024-01-01'), new Date('2024-12-31')]);
```
- **说明**：活动时间范围自动取极值。
- **注意**：Math.max 接受时间戳，Date 对象自动 valueOf。

### 8. 加天（addDays）
- **作用**：日期上加 n 天。
- **代码**：
```javascript
const addDays = (date, n) => {
  const d = new Date(date);
  d.setDate(d.getDate() + n);
  return d;
};
addDays(new Date(), 7); // 一周后
```
- **说明**：截止日期计算。
- **注意**：返回新 Date，不改原对象。

### 9. 减天（subtractDays）
- **作用**：日期减 n 天。
- **代码**：
```javascript
const subtractDays = (date, n) => addDays(date, -n);
subtractDays(new Date(), 3); // 3 天前
```
- **说明**：和 addDays 对称。
- **注意**：和 addDays 同一实现。

### 10. 加小时（addHours）
- **作用**：加 n 小时。
- **代码**：
```javascript
const addHours = (date, n) => {
  const d = new Date(date);
  d.setTime(d.getTime() + n * 3600 * 1000);
  return d;
};
```
- **说明**：会话过期时间计算。
- **注意**：用 getTime 加减毫秒，避免时区坑。

### 11. 一年中第几周（getWeekOfYear）
- **作用**：Date 是当年第几周。
- **代码**：
```javascript
const getWeekOfYear = date => {
  const start = new Date(date.getFullYear(), 0, 1);
  const diff = date - start;
  return Math.ceil(((diff / 1000 / 60 / 60 / 24) + start.getDay() + 1) / 7);
};
getWeekOfYear(new Date('2024-12-31')); // ~53
```
- **说明**：周报、周维度统计。
- **注意**：ISO 周（周一开始、第一周含周四）是另一套规则，要用 getISOWeek。

### 12. ISO 周数（getISOWeek）
- **作用**：严格按 ISO 8601 算周数。
- **代码**：
```javascript
const getISOWeek = date => {
  const d = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
  const dayNum = d.getUTCDay() || 7;
  d.setUTCDate(d.getUTCDate() + 4 - dayNum);
  const yearStart = new Date(Date.UTC(d.getUTCFullYear(), 0, 1));
  return Math.ceil((((d - yearStart) / 86400000) + 1) / 7);
};
```
- **说明**：和日历 App 显示一致。
- **注意**：跨年边界可能属于上一年的第 52/53 周。

### 13. 是否周末（isWeekend）
- **作用**：Date 是不是周六/周日。
- **代码**：
```javascript
const isWeekend = date => date.getDay() % 6 === 0;
isWeekend(new Date('2026-10-04')); // 今天周日 true
```
- **说明**：排班、工作日过滤。
- **注意**：getDay() 0=周日 6=周六。

### 14. 日期比较（isAfter / isBefore）
- **作用**：判断日期先后。
- **代码**：
```javascript
const isAfter = (d1, d2) => new Date(d1) > new Date(d2);
const isBefore = (d1, d2) => new Date(d1) < new Date(d2);
isAfter('2024-12-31', '2024-01-01'); // true
```
- **说明**：表单日期范围校验。
- **注意**：直接 > 比较 Date 对象自动转时间戳。

### 15. 是否同一天（isSameDate）
- **作用**：两个 Date 是否是同一天（忽略时分秒）。
- **代码**：
```javascript
const isSameDate = (d1, d2) =>
  d1.getFullYear() === d2.getFullYear() &&
  d1.getMonth() === d2.getMonth() &&
  d1.getDate() === d2.getDate();
```
- **说明**：今日订单筛选。
- **注意**：忽略时分秒；时区按本地。

### 16. 一年中第几天（dayOfYear）
- **作用**：Date 是当年第几天。
- **代码**：
```javascript
const dayOfYear = date =>
  Math.floor((date - new Date(date.getFullYear(), 0, 0)) / 1000 / 60 / 60 / 24);
dayOfYear(new Date('2024-12-31')); // 366
```
- **说明**：年度进度条。
- **注意**：闰年 366。

### 17. 月起始 / 月结束（startOfMonth / endOfMonth）
- **作用**：当月 1 号 0 点 / 月末 23:59:59。
- **代码**：
```javascript
const startOfMonth = date => new Date(date.getFullYear(), date.getMonth(), 1);
const endOfMonth = date =>
  new Date(date.getFullYear(), date.getMonth() + 1, 0, 23, 59, 59);
endOfMonth(new Date('2024-02-10')); // 2024-02-29 23:59:59
```
- **说明**：月报时间范围。
- **注意**：用 month+1, day=0 自动得到月末。

### 18. 日起始 / 日结束（startOfDay / endOfDay）
- **作用**：当天 0 点 / 23:59:59.999。
- **代码**：
```javascript
const startOfDay = date => new Date(date.getFullYear(), date.getMonth(), date.getDate());
const endOfDay = date =>
  new Date(date.getFullYear(), date.getMonth(), date.getDate(), 23, 59, 59, 999);
```
- **说明**：按天查数据。
- **注意**：不要用 setHours(0,0,0,0) 之外的方法，容易漏毫秒。

### 19. Unix 时间戳（getUnixTimestamp）
- **作用**：当前秒级时间戳。
- **代码**：
```javascript
const getUnixTimestamp = () => Math.floor(Date.now() / 1000);
```
- **说明**：后端接口要秒级时。
- **注意**：JS 原生是毫秒，记得除 1000。

### 20. 格式化日期（formatDate）
- **作用**：把 Date 格式化成 YYYY-MM-DD。
- **代码**：
```javascript
const formatDate = date => {
  const d = new Date(date);
  const y = d.getFullYear();
  const m = String(d.getMonth() + 1).padStart(2, '0');
  const day = String(d.getDate()).padStart(2, '0');
  return `${y}-${m}-${day}`;
};
formatDate(new Date()); // '2026-10-04'
```
- **说明**：通用日期展示。
- **注意**：本地时区；要 UTC 用 getUTCFullYear 等。

### 21. 格式化日期时间（formatDateTime）
- **作用**：YYYY-MM-DD HH:mm:ss。
- **代码**：
```javascript
const formatDateTime = date => {
  const d = new Date(date);
  const p = n => String(n).padStart(2, '0');
  return `${d.getFullYear()}-${p(d.getMonth() + 1)}-${p(d.getDate())} ` +
         `${p(d.getHours())}:${p(d.getMinutes())}:${p(d.getSeconds())}`;
};
```
- **说明**：日志、列表时间戳。
- **注意**：本地时区。

### 22. 时长格式化（formatDuration）
- **作用**：毫秒数转「3分20秒」。
- **代码**：
```javascript
const formatDuration = ms => {
  const s = Math.floor(ms / 1000);
  const h = Math.floor(s / 3600);
  const m = Math.floor((s % 3600) / 60);
  const sec = s % 60;
  return [h && `${h} 小时`, m && `${m} 分`, `${sec} 秒`].filter(Boolean).join(' ');
};
formatDuration(3 * 60 * 1000 + 20 * 1000); // '3 分 20 秒'
```
- **说明**：视频时长、任务耗时。
- **注意**：小时为 0 时自动省略。

### 23. 相对时间（futureTimeAgo）
- **作用**：未来时间转「3 天后」。
- **代码**：
```javascript
const futureTime = date => {
  const diff = (new Date(date).getTime() - Date.now()) / 1000;
  if (diff < 0) return timeAgo(date);
  const units = [
    ['天', 86400], ['小时', 3600], ['分钟', 60], ['秒', 1],
  ];
  for (const [n, s] of units) {
    const v = Math.floor(diff / s);
    if (v >= 1) return `${v} ${n}后`;
  }
  return '即将';
};
futureTime(Date.now() + 2 * 86400 * 1000); // '2 天后'
```
- **说明**：活动倒计时文案。
- **注意**：依赖前面定义的 timeAgo。

### 24. 星期几（getWeekday）
- **作用**：Date 是星期几（中文）。
- **代码**：
```javascript
const getWeekday = date =>
  ['周日', '周一', '周二', '周三', '周四', '周五', '周六'][new Date(date).getDay()];
getWeekday(new Date('2026-10-04')); // '周日'
```
- **说明**：日历组件头。
- **注意**：getDay() 0=周日。

### 25. 月底星期几（lastDayOfWeek）
- **作用**：当月最后一天是周几。
- **代码**：
```javascript
const lastDayOfWeek = date => {
  const d = new Date(date.getFullYear(), date.getMonth() + 1, 0);
  return d.getDay();
};
```
- **说明**：日历格子对齐。
- **注意**：和 endOfMonth 配合用。

### 26. 日期合法性（isValidDate）
- **作用**：值是不是合法日期。
- **代码**：
```javascript
const isValidDate = date =>
  date instanceof Date && !Number.isNaN(date.valueOf());
isValidDate(new Date('2024-01-01')); // true
isValidDate(new Date('bad'));        // false
```
- **说明**：表单校验。
- **注意**：`new Date('bad')` 不抛错，只是 Invalid Date。

### 27. 两个日期是否同天（isSameDay）
- **作用**：和 isSameDate 同义，命名更短。
- **代码**：
```javascript
const isSameDay = (a, b) =>
  a.toDateString() === b.toDateString();
isSameDay(new Date(), new Date()); // true
```
- **说明**：简洁实现。
- **注意**：toDateString 是英文格式，但比较无所谓。

### 28. 时间戳格式化（formatTimestamp）
- **作用**：秒级时间戳转 YYYY-MM-DD HH:mm。
- **代码**：
```javascript
const formatTimestamp = ts => {
  const d = new Date(ts * 1000);
  const p = n => String(n).padStart(2, '0');
  return `${d.getFullYear()}-${p(d.getMonth() + 1)}-${p(d.getDate())} ` +
         `${p(d.getHours())}:${p(d.getMinutes())}`;
};
formatTimestamp(1735689600); // '2025-01-01 08:00' 之类
```
- **说明**：后端返回秒级时间戳。
- **注意**：乘 1000 转毫秒。

### 29. 工作日加（addBusinessDays）
- **作用**：跳过周末加 n 个工作日。
- **代码**：
```javascript
const addBusinessDays = (date, n) => {
  const d = new Date(date);
  let added = 0;
  while (added < n) {
    d.setDate(d.getDate() + 1);
    if (d.getDay() % 6 !== 0) added++;
  }
  return d;
};
addBusinessDays(new Date('2026-10-02'), 3); // 跳过周末到 10-07
```
- **说明**：SLA、交货日期计算。
- **注意**：不考虑法定节假日，那些要节假日表。

### 30. 一年内工作日数（businessDaysInYear）
- **作用**：某年有多少工作日。
- **代码**：
```javascript
const businessDaysInYear = year => {
  let count = 0;
  const d = new Date(year, 0, 1);
  while (d.getFullYear() === year) {
    if (d.getDay() % 6 !== 0) count++;
    d.setDate(d.getDate() + 1);
  }
  return count;
};
businessDaysInYear(2026); // ~261
```
- **说明**：年度人力测算。
- **注意**：不含节假日。

### 31. 时区缩写（timezoneAbbr）
- **作用**：拿到当前时区缩写（CST 等）。
- **代码**：
```javascript
const timezoneAbbr = () => {
  const s = new Date().toString();
  const m = s.match(/\(([^)]+)\)$/);
  return m ? m[1] : s;
};
timezoneAbbr(); // '中国标准时间' 之类
```
- **说明**：调试时区问题。
- **注意**：浏览器字符串格式不一，别用于关键业务。

### 32. 季度（getQuarter）
- **作用**：Date 是当年第几季度。
- **代码**：
```javascript
const getQuarter = date => Math.floor(new Date(date).getMonth() / 3) + 1;
getQuarter(new Date('2026-10-04')); // 4
```
- **说明**：季度报表。
- **注意**：Q1=1-3 月。

### 33. 半年标识（getHalfYear）
- **作用**：上半年还是下半年。
- **代码**：
```javascript
const getHalfYear = date => (new Date(date).getMonth() < 6 ? 'H1' : 'H2');
getHalfYear(new Date()); // 'H2'
```
- **说明**：半年总结。
- **注意**：H1=1-6 月。

### 34. 月份中文名（monthNameZh）
- **作用**：Date 转中文月份。
- **代码**：
```javascript
const monthNameZh = date =>
  `${new Date(date).getMonth() + 1} 月`;
monthNameZh(new Date()); // '10 月'
```
- **说明**：图表 X 轴。
- **注意**：要「十月」这种无数字就自己映射。

### 35. 周范围（weekRange）
- **作用**：拿到当前周的周一到周日日期。
- **代码**：
```javascript
const weekRange = date => {
  const d = new Date(date);
  const day = d.getDay() || 7;
  const monday = new Date(d);
  monday.setDate(d.getDate() - day + 1);
  const sunday = new Date(monday);
  sunday.setDate(monday.getDate() + 6);
  return [monday, sunday].map(formatDate);
};
weekRange(new Date()); // ['2026-09-28', '2026-10-04']
```
- **说明**：周报标题。
- **注意**：周一为一周开始。

### 36. 两时刻之间分钟数（minutesBetween）
- **作用**：两个 Date 之间差多少分钟。
- **代码**：
```javascript
const minutesBetween = (d1, d2) =>
  Math.abs(new Date(d2) - new Date(d1)) / 60000;
minutesBetween('2026-10-04T10:00', '2026-10-04T10:30'); // 30
```
- **说明**：会议时长、通话时长。
- **注意**：Math.abs 取绝对值。
