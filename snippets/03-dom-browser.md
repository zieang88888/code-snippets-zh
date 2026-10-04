# 03 · DOM 与浏览器

> 防抖节流、滚动、剪贴板、全屏、事件、复制——前端日常最高频的浏览器侧小件。

### 1. 事件绑定（on / off）
- **作用**：绑定/解绑事件，返回解绑函数方便清理。
- **代码**：
```javascript
const on = (el, evt, fn, opts = {}) => {
  el.addEventListener(evt, fn, opts);
  return () => el.removeEventListener(evt, fn, opts);
};
const off = on(document, 'click', () => {}); // 调用 off() 解绑
```
- **说明**：组件 useEffect 里直接 `return on(...)` 自动清理。
- **注意**：`opts` 必须和绑定时一致，否则 remove 匹配不上。

### 2. 判类名（hasClass）
- **作用**：元素是否带某个 class。
- **代码**：
```javascript
const hasClass = (el, className) => el.classList.contains(className);
hasClass(document.querySelector('p.special'), 'special'); // true
```
- **说明**：现代浏览器全支持。
- **注意**：类名里不能有空格；多类名先 split。

### 3. 增删类（addClass / removeClass / toggleClass）
- **作用**：操作 class 列表。
- **代码**：
```javascript
const addClass = (el, cls) => el.classList.add(cls);
const removeClass = (el, cls) => el.classList.remove(cls);
const toggleClass = (el, cls) => el.classList.toggle(cls);
toggleClass(document.querySelector('p'), 'special');
```
- **说明**：UI 状态切换三件套。
- **注意**：classList 支持多参数 `add('a','b')`。

### 4. 是否在视口内（elementIsVisibleInViewport）
- **作用**：判断元素是否出现在视口。
- **代码**：
```javascript
const elementIsVisibleInViewport = (el, partially = false) => {
  const { top, left, bottom, right } = el.getBoundingClientRect();
  const vw = document.documentElement.clientWidth;
  const vh = document.documentElement.clientHeight;
  return partially
    ? top < vh && bottom > 0 && left < vw && right > 0
    : top >= 0 && left >= 0 && bottom <= vh && right <= vw;
};
```
- **说明**：懒加载、列表曝光埋点。
- **注意**：性能敏感场景优先 IntersectionObserver。

### 5. 滚动位置（getScrollPosition）
- **作用**：拿到当前滚动到的 x/y。
- **代码**：
```javascript
const getScrollPosition = (el = window) => ({
  x: el.pageXOffset !== undefined ? el.pageXOffset : el.scrollLeft,
  y: el.pageYOffset !== undefined ? el.pageYOffset : el.scrollTop,
});
getScrollPosition(); // {x: 0, y: 200}
```
- **说明**：记滚动位置、返回顶部。
- **注意**：传 window 用 pageXOffset，传具体容器用 scrollLeft。

### 6. 平滑滚回顶部（scrollToTop）
- **作用**：平滑滚动到页面顶部。
- **代码**：
```javascript
const scrollToTop = () => {
  const c = document.documentElement.scrollTop || document.body.scrollTop;
  if (c > 0) {
    window.requestAnimationFrame(scrollToTop);
    window.scrollTo(0, c - c / 8);
  }
};
```
- **说明**：老版浏览器兼容的平滑回顶；现代可直接 `window.scrollTo({top:0,behavior:'smooth'})`。
- **注意**：requestAnimationFrame 递归直到归零。

### 7. 复制到剪贴板（copyToClipboard）
- **作用**：把文本复制到系统剪贴板。
- **代码**：
```javascript
const copyToClipboard = str => {
  if (navigator.clipboard) return navigator.clipboard.writeText(str);
  const el = document.createElement('textarea');
  el.value = str;
  el.setAttribute('readonly', '');
  el.style.position = 'absolute';
  el.style.left = '-9999px';
  document.body.appendChild(el);
  el.select();
  document.execCommand('copy');
  document.body.removeChild(el);
};
```
- **说明**：现代浏览器优先 Clipboard API，旧浏览器兜底 execCommand。
- **注意**：必须在用户手势（点击）里调用，否则浏览器拒绝。

### 8. 读取样式（getStyle）
- **作用**：拿到元素计算后的实际样式。
- **代码**：
```javascript
const getStyle = (el, ruleName) => getComputedStyle(el)[ruleName];
getStyle(document.querySelector('p'), 'font-size'); // '16px'
```
- **说明**：`el.style.xxx` 只能拿内联，这个拿真实值。
- **注意**：属性名用驼峰 `backgroundColor` 而非 `background-color`。

### 9. 插入元素（insertAfter / insertBefore）
- **作用**：在参考节点后/前插入新节点。
- **代码**：
```javascript
const insertAfter = (el, htmlString) =>
  el.insertAdjacentHTML('afterend', htmlString);
const insertBefore = (el, htmlString) =>
  el.insertAdjacentHTML('beforebegin', htmlString);
insertAfter(document.getElementById('myId'), '<p>after</p>');
```
- **说明**：DOM 插入比 appendChild 灵活，支持 HTML 字符串。
- **注意**：XSS 场景下 htmlString 必须是可信内容。

### 10. 兄弟节点（siblings）
- **作用**：拿到元素的所有兄弟节点。
- **代码**：
```javascript
const siblings = el =>
  [...el.parentNode.children].filter(node => node !== el);
```
- **说明**：tab 切换时把其他兄弟取消选中。
- **注意**：用 children 排除文本节点；要含文本节点用 childNodes。

### 11. 表单对象化（formToObject）
- **作用**：把 form 表单序列化成语法对象。
- **代码**：
```javascript
const formToObject = form =>
  Array.from(new FormData(form)).reduce((acc, [key, value]) => {
    acc[key] = value;
    return acc;
  }, {});
formToObject(document.querySelector('#form'));
```
- **说明**：提交前一键收集所有字段。
- **注意**：同名字段后者覆盖前者；多选框要自己处理数组。

### 12. URL 参数提取（getURLParameters）
- **作用**：解析 location.search 为对象。
- **代码**：
```javascript
const getURLParameters = url =>
  (url.match(/([^?=&]+)(=([^&]*))/g) || []).reduce((acc, kv) => {
    const [k, v] = kv.split('=');
    acc[k] = decodeURIComponent(v);
    return acc;
  }, {});
getURLParameters('http://url.com/page?name=Adam&surname=Smith');
// {name: 'Adam', surname: 'Smith'}
```
- **说明**：落地页拿渠道参数。
- **注意**：现代环境直接 `Object.fromEntries(new URLSearchParams(location.search))`。

### 13. 跳页（redirect）
- **作用**：跳转到指定 URL。
- **代码**：
```javascript
const redirect = (url, asLink = true) =>
  asLink ? (window.location.href = url) : (window.location.replace(url));
redirect('https://example.com');
```
- **说明**：登录后跳回原页。
- **注意**：`replace` 不留历史记录，用户不能点返回。

### 14. 是否触底（bottomVisible）
- **作用**：页面是否滚动到底部。
- **代码**：
```javascript
const bottomVisible = () =>
  document.documentElement.clientHeight + window.scrollY >=
  (document.documentElement.scrollHeight || document.documentElement.scrollHeight);
bottomVisible(); // true
```
- **说明**：无限滚动加载下一页。
- **注意**：用 IntersectionObserver 监听一个哨兵元素更省性能。

### 15. 元素坐标（getCoordinates）
- **作用**：拿到元素在页面上的绝对位置。
- **代码**：
```javascript
const getCoordinates = el => {
  const rect = el.getBoundingClientRect();
  return {
    left: rect.left + window.scrollX,
    top: rect.top + window.scrollY,
    width: rect.width,
    height: rect.height,
  };
};
getCoordinates(document.querySelector('#myId'));
```
- **说明**：做悬浮跟随、tooltip 定位。
- **注意**：`getBoundingClientRect` 是相对视口，必须加滚动偏移才是文档坐标。

### 16. NodeList 转数组（nodeListToArray）
- **作用**：`document.querySelectorAll` 结果转真正数组。
- **代码**：
```javascript
const nodeListToArray = nodeList => [...nodeList];
nodeListToArray(document.querySelectorAll('p'));
```
- **说明**：NodeList 没有 forEach 之外的数组方法。
- **注意**：现代浏览器 NodeList 已支持 forEach，但 filter/map 仍需转数组。

### 17. 创建元素（createElement）
- **作用**：从 HTML 字符串创建 DOM 节点。
- **代码**：
```javascript
const createElement = str => {
  const el = document.createElement('div');
  el.innerHTML = str;
  return el.firstElementChild;
};
const el = createElement('<div class="container">Hello</div>');
```
- **说明**：模板字符串快速造节点。
- **注意**：innerHTML 注入用户内容前必须转义，否则 XSS。

### 18. 清空元素（emptyElement）
- **作用**：把节点里所有子节点删光。
- **代码**：
```javascript
const emptyElement = el => (el.innerHTML = '');
emptyElement(document.querySelector('#container'));
```
- **说明**：重渲染前清容器。
- **注意**：直接 `el.textContent = ''` 更安全（不会触发解析）。

### 19. 视口宽高（viewport）
- **作用**：拿浏览器视口尺寸。
- **代码**：
```javascript
const viewport = () => ({
  width: document.documentElement.clientWidth,
  height: document.documentElement.clientHeight,
});
```
- **说明**：响应式断点判断。
- **注意**：用 documentElement 而非 window.innerWidth，后者包含滚动条宽度。

### 20. 全屏切换（fullscreen / exitFullscreen）
- **作用**：进入/退出全屏。
- **代码**：
```javascript
const fullscreen = el => el.requestFullscreen();
const exitFullscreen = () => document.exitFullscreen();
fullscreen(document.documentElement);
```
- **说明**：视频、图片灯箱。
- **注意**：必须用户手势触发；各浏览器前缀已基本统一。

### 21. 滚动到元素（scrollToElement）
- **作用**：平滑滚动到某个元素。
- **代码**：
```javascript
const scrollToElement = el => el.scrollIntoView({ behavior: 'smooth' });
scrollToElement(document.querySelector('#section-2'));
```
- **说明**：目录锚点点击。
- **注意**：`behavior:'smooth'` 不支持时瞬间跳。

### 22. 设备类型（detectDeviceType）
- **作用**：判断是移动还是桌面。
- **代码**：
```javascript
const detectDeviceType = () =>
  /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
    navigator.userAgent
  )
    ? 'Mobile'
    : 'Desktop';
```
- **说明**：按 UA 分流加载不同资源。
- **注意**：UA 可伪造；现代项目优先用 CSS 媒体查询 + 触摸事件探测。

### 23. 操作系统（detectOS）
- **作用**：粗略判断用户操作系统。
- **代码**：
```javascript
const detectOS = () => {
  const ua = navigator.userAgent;
  if (ua.includes('Win')) return 'Windows';
  if (ua.includes('Mac')) return 'MacOS';
  if (ua.includes('Android')) return 'Android';
  if (ua.includes('iPhone') || ua.includes('iPad')) return 'iOS';
  if (ua.includes('Linux')) return 'Linux';
  return 'Unknown';
};
```
- **说明**：给不同 OS 显示不同下载按钮。
- **注意**：只是字符串粗判，别用于安全判断。

### 24. 复制回调（copyTextAsync）
- **作用**：异步复制并成功/失败回调。
- **代码**：
```javascript
const copyTextAsync = async text => {
  try {
    await navigator.clipboard.writeText(text);
    return true;
  } catch (_) {
    return false;
  }
};
```
- **说明**：分享链接按钮。
- **注意**：需要 HTTPS + 用户手势；file:// 或 http 下 Clipboard API 可能不可用。

### 25. 防抖（debounce）
- **作用**：高频事件停止触发后 n 毫秒才执行一次。
- **代码**：
```javascript
const debounce = (fn, ms = 300) => {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(null, args), ms);
  };
};
window.addEventListener('resize', debounce(() => console.log('resized'), 200));
```
- **说明**：搜索框输入、resize、滚动。
- **注意**：组件卸载记得 cancel，可用 `debounced.cancel?.()`。

### 26. 节流（throttle）
- **作用**：高频事件每 n 毫秒最多执行一次。
- **代码**：
```javascript
const throttle = (fn, ms = 300) => {
  let last = 0;
  let timer = null;
  return (...args) => {
    const now = Date.now();
    const remain = ms - (now - last);
    if (remain <= 0) {
      last = now;
      fn.apply(null, args);
    } else if (!timer) {
      timer = setTimeout(() => {
        last = Date.now();
        timer = null;
        fn.apply(null, args);
      }, remain);
    }
  };
};
```
- **说明**：滚动、mousemove 监听，比 debounce 更实时。
- **注意**：保证最后一次也会执行（leading + trailing）。

### 27. 粘贴板读（readClipboard）
- **作用**：读取剪贴板文本。
- **代码**：
```javascript
const readClipboard = async () => {
  try {
    return await navigator.clipboard.readText();
  } catch (_) {
    return '';
  }
};
```
- **说明**：粘贴监听。
- **注意**：浏览器要求明确授权，且必须在可见 focus 状态。

### 28. 选中文本（selectText）
- **作用**：选中某元素里的全部文本。
- **代码**：
```javascript
const selectText = el => {
  const range = document.createRange();
  range.selectNodeContents(el);
  const sel = window.getSelection();
  sel.removeAllRanges();
  sel.addRange(range);
};
selectText(document.querySelector('#code'));
```
- **说明**：点代码块自动全选方便复制。
- **注意**：对 input 用 `el.select()` 即可。

### 29. 暗黑模式偏好（prefersDarkColorScheme）
- **作用**：读系统是否偏好深色。
- **代码**：
```javascript
const prefersDarkColorScheme = () =>
  window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches;
```
- **说明**：首次加载时决定主题。
- **注意**：要监听变化用 `matchMedia(...).addEventListener('change', cb)`。

### 30. 网络状态（isOnline）
- **作用**：当前是否联网。
- **代码**：
```javascript
const isOnline = () => navigator.onLine;
window.addEventListener('online', () => console.log('back'));
window.addEventListener('offline', () => console.log('gone'));
```
- **说明**：弱网提示。
- **注意**：navigator.onLine 只表示「是否连到局域网」，不代表真能上外网。

### 31. 脚本动态加载（loadScript）
- **作用**：动态插入 script 标签。
- **代码**：
```javascript
const loadScript = url =>
  new Promise((resolve, reject) => {
    const s = document.createElement('script');
    s.src = url;
    s.onload = resolve;
    s.onerror = reject;
    document.head.appendChild(s);
  });
await loadScript('https://cdn.example.com/lib.js');
```
- **说明**：按需加载第三方 SDK。
- **注意**：跨域脚本走 CORS；重复加载要做缓存判断。

### 32. 图片预加载（preloadImage）
- **作用**：提前加载图片到缓存。
- **代码**：
```javascript
const preloadImage = imgs =>
  imgs.forEach(src => {
    const img = new Image();
    img.src = src;
  });
preloadImage(['img1.jpg', 'img2.jpg']);
```
- **说明**：hover 换图、下一页图提前加载。
- **注意**：浏览器并发有限（一般 6 个），别一次性几百张。

### 33. 元素是否包含（elementContains）
- **作用**：父节点是否包含子节点。
- **代码**：
```javascript
const elementContains = (parent, child) =>
  parent !== child && parent.contains(child);
elementContains(document.querySelector('head'), document.querySelector('title')); // true
```
- **说明**：点击外部关闭弹窗的判断。
- **注意**：`parent.contains(child)` 原生就有，这里加了排除自身。

### 34. 触发事件（triggerEvent）
- **作用**：程序化触发一个事件。
- **代码**：
```javascript
const triggerEvent = (el, eventType, detail) =>
  el.dispatchEvent(new CustomEvent(eventType, { detail }));
triggerEvent(document.querySelector('#myId'), 'click');
```
- **说明**：组件内部对外 emit。
- **注意**：CustomEvent 的 detail 是自定义数据载荷。

### 35. 打开新标签页（openInNewTab）
- **作用**：安全打开新标签。
- **代码**：
```javascript
const openInNewTab = (url, noopener = true) => {
  const win = window.open(url, '_blank', noopener ? 'noopener,noreferrer' : '');
  if (win) win.opener = null;
};
```
- **说明**：防新页通过 window.opener 反向操作原页。
- **注意**：noopener 已是现代浏览器 window.open 默认行为。

### 36. 元素高亮描边（outlineElement）
- **作用**：临时给元素画红框用于调试。
- **代码**：
```javascript
const outlineElement = el => {
  el.style.outline = '1px solid red';
  el.style.outlineOffset = '-1px';
};
document.querySelectorAll('*').forEach(outlineElement);
```
- **说明**：快速调试布局。
- **注意**：调试用，别进生产。

### 37. 剪贴板权限查询（hasClipboardPermission）
- **作用**：查当前是否有剪贴板读写权限。
- **代码**：
```javascript
const hasClipboardPermission = async () => {
  const n = navigator;
  if (!n.permissions || !n.clipboard) return false;
  const r = await n.permissions.query({ name: 'clipboard-read' });
  return r.state === 'granted';
};
```
- **说明**：复制按钮失败时给用户提示。
- **注意**：Firefox 对 permissions API 支持有限。

### 38. 监听点击外部（onClickOutside）
- **作用**：点击元素外部时触发回调。
- **代码**：
```javascript
const onClickOutside = (el, callback) => {
  const handler = e => !el.contains(e.target) && callback(e);
  document.addEventListener('click', handler);
  return () => document.removeEventListener('click', handler);
};
const off = onClickOutside(popup, () => popup.classList.remove('open'));
```
- **说明**：下拉菜单、Modal 点击外部关闭。
- **注意**：返回的解绑函数记得在卸载时调用。

### 39. 页面可见性（onVisibilityChange）
- **作用**：切到后台/前台时回调。
- **代码**：
```javascript
const onVisibilityChange = onChanged => {
  document.addEventListener('visibilitychange', () =>
    onChanged(document.visibilityState)
  );
};
onVisibilityChange(state => console.log(state)); // 'visible' | 'hidden'
```
- **说明**：切后台暂停定时器/视频。
- **注意**：后台标签页的 setTimeout 会被浏览器限流到 1s 以上。

### 40. 平滑滚动到顶部（scrollToTopSmooth）
- **作用**：一行式平滑回顶。
- **代码**：
```javascript
const scrollToTopSmooth = () =>
  window.scrollTo({ top: 0, behavior: 'smooth' });
```
- **说明**：回顶按钮最简实现。
- **注意**：behavior:'smooth' 不支持时瞬间跳；要兼容加 polyfill。
