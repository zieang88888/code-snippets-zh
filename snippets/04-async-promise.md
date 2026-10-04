# 04 · 异步与 Promise

> 重试 / 并发限制 / 超时 / 串行并行——接口和异步任务的工程化套路。

### 1. 睡眠（sleep）
- **作用**：Promise 化的 setTimeout，等待 n 毫秒。
- **代码**：
```javascript
const sleep = ms => new Promise(resolve => setTimeout(resolve, ms));
(async () => {
  console.log('hi');
  await sleep(1000);
  console.log('1s later');
})();
```
- **说明**：异步流里做节奏控制。
- **注意**：Node 里也可以 `await new Promise(r => setTimeout(r, ms))`，本函数就是封装。

### 2. 回调转 Promise（promisify）
- **作用**：把 callback 风格函数包装成 Promise。
- **代码**：
```javascript
const promisify = func => (...args) =>
  new Promise((resolve, reject) =>
    func(...args, (err, result) => (err ? reject(err) : resolve(result))
  );
const delay = promisify((d, cb) => setTimeout(cb, d));
delay(200).then(() => console.log('Hi!'));
```
- **说明**：老 Node API 转 async/await。
- **注意**：Node 自带 `util.promisify`，本函数给浏览器。

### 3. 串行执行（runPromisesInSeries）
- **作用**：按顺序执行一组 Promise 工厂，前一个完了才下一个。
- **代码**：
```javascript
const runPromisesInSeries = ps =>
  ps.reduce((p, next) => p.then(next), Promise.resolve());
const delay = d => () => new Promise(r => setTimeout(r, d));
runPromisesInSeries([delay(1000), delay(2000)]);
```
- **说明**：避免并发打爆后端的批量任务。
- **注意**：传「函数返回 Promise」而不是 Promise 本身，否则一开始就并发了。

### 4. 异步 reduce（asyncReduce）
- **作用**：数组里的异步任务依次执行并累积结果。
- **代码**：
```javascript
const asyncReduce = async (arr, fn, acc) => {
  for (const item of arr) {
    acc = await fn(acc, item);
  }
  return acc;
};
await asyncReduce([1, 2, 3], async (a, x) => a + x, 0); // 6
```
- **说明**：顺序处理 + 累加。
- **注意**：和 `Promise.all` 不同，这里一个完了才下一个。

### 5. 并发限制（asyncPool）
- **作用**：最多同时跑 n 个异步任务。
- **代码**：
```javascript
const asyncPool = async (limit, items, worker) => {
  const ret = [];
  const executing = new Set();
  for (const item of items) {
    const p = Promise.resolve().then(() => worker(item));
    ret.push(p);
    executing.add(p);
    const clean = () => executing.delete(p);
    p.then(clean, clean);
    if (executing.size >= limit) await Promise.race(executing);
  }
  return Promise.all(ret);
};
await asyncPool(3, urls, url => fetch(url));
```
- **说明**：爬取、批量上传的并发闸。
- **注意**：超过 limit 就等最快的一个完成再放新的。

### 6. 带超时的 Promise（withTimeout）
- **作用**：Promise 超过 ms 就 reject。
- **代码**：
```javascript
const withTimeout = (ms, promise) =>
  new Promise((resolve, reject) => {
    const timer = setTimeout(() => reject(new Error('timeout')), ms);
    promise
      .then(v => (clearTimeout(timer), resolve(v)))
      .catch(e => (clearTimeout(timer), reject(e)));
  });
await withTimeout(1000, fetch('/api'));
```
- **说明**：接口卡死兜底。
- **注意**：原 promise 本身不会被取消，只是外层不再等。

### 7. 重试（retry）
- **作用**：失败自动重试 n 次。
- **代码**：
```javascript
const retry = async (fn, times = 3, delay = 500) => {
  for (let i = 0; i < times; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === times - 1) throw e;
      await sleep(delay);
    }
  }
};
await retry(() => fetch('/flaky-api'), 3, 800);
```
- **说明**：网络抖动场景。
- **注意**：配合指数退避（delay *= 2）更友好。

### 8. 指数退避重试（retryWithBackoff）
- **作用**：每次失败等待时间翻倍。
- **代码**：
```javascript
const retryWithBackoff = async (fn, times = 5, base = 200) => {
  let delay = base;
  for (let i = 0; i < times; i++) {
    try {
      return await fn();
    } catch (e) {
      if (i === times - 1) throw e;
      await sleep(delay);
      delay *= 2;
    }
  }
};
```
- **说明**：被限流时不要猛打，逐步拉长。
- **注意**：加一点随机抖动（jitter）避免多个客户端同步重试。

### 9. Promise 全部 settle（settle）
- **作用**：等所有 promise 都结束，无论成败。
- **代码**：
```javascript
const settle = promises =>
  Promise.all(
    promises.map(p =>
      p.then(value => ({ status: 'fulfilled', value })).catch(reason => ({
        status: 'rejected',
        reason,
      }))
    )
  );
await settle([fetch('/a'), fetch('/b')]);
```
- **说明**：等价于 `Promise.allSettled` 的手写版。
- **注意**：现代环境直接 `Promise.allSettled`。

### 10. Promise 管道（pipeAsyncFunctions）
- **作用**：把多个 async 函数串成管道。
- **代码**：
```javascript
const pipeAsyncFunctions =
  (...fns) =>
  arg =>
    fns.reduce((p, f) => p.then(f), Promise.resolve(arg));
const sum = pipeAsyncFunctions(
  async x => x + 1,
  async x => x * 2,
  async x => x - 3
);
await sum(5); // 11
```
- **说明**：异步版函数组合。
- **注意**：前一个返回值是后一个入参。

### 11. 并发执行（parallel）
- **作用**：一组 promise 工厂并发跑。
- **代码**：
```javascript
const parallel = async tasks => Promise.all(tasks.map(t => t()));
await parallel([() => fetch('/a'), () => fetch('/b')]);
```
- **说明**：能并发就别串行。
- **注意**：任一个失败 Promise.all 就整体 reject；要容错用 settle。

### 12. 串行链式（chainPromises）
- **作用**：把 Promise 工厂数组串成 then 链。
- **代码**：
```javascript
const chainPromises = promiseFactories =>
  promiseFactories.reduce((chain, task) => chain.then(task), Promise.resolve());
```
- **说明**：和 runPromisesInSeries 几乎一样。
- **注意**：传 () => promise，不要传 promise 本体。

### 13. 异步条件等待（awaitCondition）
- **作用**：轮询某个条件直到满足或超时。
- **代码**：
```javascript
const awaitCondition = async (predicate, interval = 100, timeout = 5000) => {
  const start = Date.now();
  while (!predicate()) {
    if (Date.now() - start > timeout) throw new Error('awaitCondition timeout');
    await sleep(interval);
  }
};
await awaitCondition(() => document.readyState === 'complete');
```
- **说明**：等元素出现、等变量就绪。
- **注意**：轮询不是事件，精度取决于 interval。

### 14. 异步每一项（asyncEvery）
- **作用**：所有异步结果都满足条件才 true。
- **代码**：
```javascript
const asyncEvery = async (arr, fn) => (await Promise.all(arr.map(fn))).every(Boolean);
await asyncEvery([1, 2, 3], async n => n > 0); // true
```
- **说明**：并发跑完再统一判断。
- **注意**：会全部并发，不等第一个失败就停。

### 15. 异步存在项（asyncSome）
- **作用**：任一异步结果满足条件即 true。
- **代码**：
```javascript
const asyncSome = async (arr, fn) => (await Promise.all(arr.map(fn))).some(Boolean);
await asyncSome([1, 2, 3], async n => n > 2); // true
```
- **说明**：和 asyncEvery 配对。
- **注意**：不会短路，全部跑完才返回。

### 16. 错误重试包装（withRetry）
- **作用**：给任意 async 函数加重试能力。
- **代码**：
```javascript
const withRetry = (fn, times = 3, delay = 300) =>
  async (...args) => {
    let lastErr;
    for (let i = 0; i < times; i++) {
      try {
        return await fn(...args);
      } catch (e) {
        lastErr = e;
        await sleep(delay);
      }
    }
    throw lastErr;
  };
const safeFetch = withRetry(fetch, 3);
```
- **说明**：装饰器思路，把重试逻辑从业务里抽离。
- **注意**：fn 必须是 async（或返回 Promise）。

### 17. 并发限制 fetch（fetchWithConcurrency）
- **作用**：批量抓 URL 但限制并发。
- **代码**：
```javascript
const fetchWithConcurrency = async (urls, limit = 4) => {
  const results = [];
  const executing = new Set();
  for (const url of urls) {
    const p = fetch(url).then(r => r.json());
    results.push(p);
    executing.add(p);
    p.finally(() => executing.delete(p));
    if (executing.size >= limit) await Promise.race(executing);
  }
  return Promise.all(results);
};
```
- **说明**：批量拉接口不触发浏览器同域并发上限。
- **注意**：浏览器默认同域并发 6，limit 一般设 4-6。

### 18. 超时 fetch（timeoutFetch）
- **作用**：fetch 自带超时。
- **代码**：
```javascript
const timeoutFetch = (url, ms, opts = {}) => {
  const ctrl = new AbortController();
  const timer = setTimeout(() => ctrl.abort(), ms);
  return fetch(url, { ...opts, signal: ctrl.signal }).finally(() =>
    clearTimeout(timer)
  );
};
await timeoutFetch('/api', 3000);
```
- **说明**：比 withTimeout 更彻底，真的取消请求。
- **注意**：AbortController 现代浏览器全支持。

### 19. 异步记忆化（memoizeAsync）
- **作用**：缓存 async 函数结果。
- **代码**：
```javascript
const memoizeAsync = fn => {
  const cache = new Map();
  return async (...args) => {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const v = await fn(...args);
    cache.set(key, v);
    return v;
  };
};
```
- **说明**：同一接口多次调用只打一次。
- **注意**：Map 无上限，长驻页面要加 LRU。

### 20. 异步串行 map（asyncMapSeries）
- **作用**：map 里是 async 函数时，串行执行并收集结果。
- **代码**：
```javascript
const asyncMapSeries = async (arr, fn) => {
  const out = [];
  for (const x of arr) out.push(await fn(x));
  return out;
};
await asyncMapSeries([1, 2, 3], async n => n * 2); // [2,4,6]
```
- **说明**：顺序敏感的批量处理。
- **注意**：和 `arr.map(fn)` 再 Promise.all 不同，这里一个完了才下一个。

### 21. 异步任务队列（asyncQueue）
- **作用**：一个 FIFO 队列，往里面塞任务自动按并发跑。
- **代码**：
```javascript
const createQueue = (concurrency = 2) => {
  const queue = [];
  let active = 0;
  const next = () => {
    active--;
    drain();
  };
  const drain = () => {
    while (active < concurrency && queue.length) {
      const task = queue.shift();
      active++;
      Promise.resolve().then(task).then(next, next);
    }
  };
  return task => {
    queue.push(task);
    drain();
  };
};
const enqueue = createQueue(2);
enqueue(() => fetch('/a'));
```
- **说明**：通用任务池。
- **注意**：返回的是 enqueue 函数，没有任务完成回调钩子，要加得自己扩。

### 22. Promise 状态查询（promiseState）
- **作用**：探测一个 promise 当前状态。
- **代码**：
```javascript
const promiseState = p =>
  Promise.race([
    p.then(() => 'fulfilled').catch(() => 'rejected'),
    Promise.resolve().then(() => 'pending'),
  ]);
await promiseState(Promise.resolve(1)); // 'fulfilled'
```
- **说明**：调试用。
- **注意**：本质是竞态，不保证 100% 准确。

### 23. 异步等待一组（awaitAll）
- **作用**：等待多个命名异步任务并按 key 取结果。
- **代码**：
```javascript
const awaitAll = async obj => {
  const entries = await Promise.all(
    Object.entries(obj).map(async ([k, p]) => [k, await p()])
  );
  return Object.fromEntries(entries);
};
const { user, orders } = await awaitAll({
  user: fetch('/user'),
  orders: fetch('/orders'),
});
```
- **说明**：并行拿多接口、按名取。
- **注意**：任一失败整组 reject。

### 24. 异步节流（throttleAsync）
- **作用**：高频调用 async 函数，保证执行期间不重入。
- **代码**：
```javascript
const throttleAsync = fn => {
  let inFlight = false;
  return async (...args) => {
    if (inFlight) return;
    inFlight = true;
    try {
      return await fn(...args);
    } finally {
      inFlight = false;
    }
  };
};
```
- **说明**：防止双击按钮重复提交。
- **注意**：和时间节流不同，这里是「进行中就忽略」。

### 25. 异步并发 map（asyncMapLimit）
- **作用**：map + 并发上限。
- **代码**：
```javascript
const asyncMapLimit = async (arr, limit, fn) => {
  const out = new Array(arr.length);
  let i = 0;
  const workers = Array.from({ length: Math.min(limit, arr.length) }, async () => {
    while (i < arr.length) {
      const idx = i++;
      out[idx] = await fn(arr[idx], idx);
    }
  });
  await Promise.all(workers);
  return out;
};
```
- **说明**：保持顺序 + 限流。
- **注意**：out 按下标写，最终结果顺序和原数组一致。

### 26. 等待所有 fetch 收尾（drainFetches）
- **作用**：页面卸载前把未完成的 fetch 都完成。
- **代码**：
```javascript
const pending = new Set();
const trackedFetch = (...args) => {
  const p = fetch(...args);
  pending.add(p);
  p.finally(() => pending.delete(p));
  return p;
};
window.addEventListener('beforeunload', () =>
  Promise.allSettled([...pending])
);
```
- **说明**：表单提交兜底。
- **注意**：beforeunload 里真正发请求要用 `navigator.sendBeacon`。

### 27. 间隔轮询（poll）
- **作用**：定时跑一个异步函数，直到条件满足。
- **代码**：
```javascript
const poll = async (fn, { interval = 1000, timeout = 30000 }) => {
  const start = Date.now();
  while (true) {
    const r = await fn();
    if (r) return r;
    if (Date.now() - start > timeout) throw new Error('poll timeout');
    await sleep(interval);
  }
};
await poll(async () => (await fetch('/status')).ok);
```
- **说明**：轮询任务状态。
- **注意**：长轮询用 SSE/WebSocket 更省。

### 28. 异步降级（asyncFallback）
- **作用**：一组异步函数依次尝试，第一个成功就返回。
- **代码**：
```javascript
const asyncFallback = async (...fns) => {
  let lastErr;
  for (const fn of fns) {
    try {
      return await fn();
    } catch (e) {
      lastErr = e;
    }
  }
  throw lastErr;
};
await asyncFallback(
  () => fetch('/cdn1/lib.js'),
  () => fetch('/cdn2/lib.js')
);
```
- **说明**：多 CDN 兜底。
- **注意**：和 retry 区别是这里换「不同源」，retry 是同一个源重打。

### 29. 延迟执行（delayPromise）
- **作用**：等 n 毫秒后 resolve 一个值。
- **代码**：
```javascript
const delayPromise = (ms, value) =>
  new Promise(resolve => setTimeout(() => resolve(value), ms));
await delayPromise(500, 'ready'); // 500ms 后 'ready'
```
- **说明**：测试里模拟慢接口。
- **注意**：和 sleep 区别是这个能带值。

### 30. Promise 超时取消（cancelableDelay）
- **作用**：一个可手动取消的延时。
- **代码**：
```javascript
const cancelableDelay = ms => {
  let t;
  const promise = new Promise(resolve => {
    t = setTimeout(resolve, ms);
  });
  return { promise, cancel: () => clearTimeout(t) };
};
const d = cancelableDelay(1000);
d.cancel();
```
- **说明**：组件卸载时清定时器。
- **注意**：cancel 后 promise 永远 pending，不会 resolve。

### 31. 并发失败取最快（raceWithTimeout）
- **作用**：多请求竞速，谁先成用谁。
- **代码**：
```javascript
const raceWithTimeout = async (promises, ms) => {
  const t = new Promise((_, rej) => setTimeout(() => rej(new Error('race timeout')), ms));
  return Promise.race([...promises, t]);
};
await raceWithTimeout([fetch('/a'), fetch('/b')], 3000);
```
- **说明**：就近选节点。
- **注意**：慢的那个 promise 不会被取消，只是结果被丢。

### 32. 异步条件分支（asyncIf）
- **作用**：await 一个条件，再决定走哪个分支。
- **代码**：
```javascript
const asyncIf = async (cond, thenFn, elseFn) => {
  const c = await cond;
  return c ? thenFn && thenFn() : elseFn && elseFn();
};
await asyncIf(
  Promise.resolve(user),
  () => renderHome(),
  () => renderLogin()
);
```
- **说明**：把 if 表达式也 promise 化。
- **注意**：不是常用模式，看懂即可。

### 33. 异步数组去重（asyncUnique）
- **作用**：并发 map 之后去重。
- **代码**：
```javascript
const asyncUnique = async (arr, keyFn) => {
  const keys = await Promise.all(arr.map(keyFn));
  const seen = new Set();
  return arr.filter((_, i) => {
    if (seen.has(keys[i])) return false;
    seen.add(keys[i]);
    return true;
  });
};
```
- **说明**：先用异步 key 判重，再过滤原数组。
- **注意**：keyFn 是 async，需要等所有 key 出来。

### 34. 并发数自动降级（adaptiveConcurrency）
- **作用**：失败率高就自动降并发，成功就回升。
- **代码**：
```javascript
const adaptiveConcurrency = async (urls, start = 4) => {
  let limit = start;
  let i = 0;
  const worker = async () => {
    while (i < urls.length) {
      const idx = i++;
      try {
        await fetch(urls[idx]);
        limit = Math.min(limit + 0.5, 8);
      } catch {
        limit = Math.max(limit - 1, 1);
        i--;
        await sleep(500);
      }
    }
  };
  await Promise.all(Array.from({ length: start }, worker));
};
```
- **说明**：自己实现简易自适应并发。
- **注意**：示意代码，生产可用 p-throttle + 自动调参。

### 35. Promise 池（promisePool）
- **作用**：经典并发池，全部结果按顺序返回。
- **代码**：
```javascript
const promisePool = async (tasks, limit = 2) => {
  const results = [];
  const executing = new Set();
  for (const task of tasks) {
    const p = Promise.resolve().then(task);
    results.push(p);
    executing.add(p);
    const clean = () => executing.delete(p);
    p.then(clean, clean);
    if (executing.size >= limit) await Promise.race(executing);
  }
  return Promise.all(results);
};
```
- **说明**：和 asyncPool 本质同构。
- **注意**：results 顺序和 tasks 顺序一致。

### 36. 异步 finally 链（asyncFinally）
- **作用**：无论成败都执行清理。
- **代码**：
```javascript
const asyncFinally = async (fn, cleanup) => {
  try {
    return await fn();
  } finally {
    await cleanup();
  }
};
await asyncFinally(
  () => fetch('/api'),
  () => closeDialog()
);
```
- **说明**：loading 关闭、锁释放。
- **注意**：Promise.prototype.finally 现代环境原生就有，本函数主要给老环境或需要 await cleanup 的场景。
