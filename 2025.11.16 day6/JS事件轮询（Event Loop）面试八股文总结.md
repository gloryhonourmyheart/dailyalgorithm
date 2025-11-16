# JS 事件轮询（Event Loop）面试八股文总结

## 一、核心基础概念（先懂定义，再讲原理）

### 1. 单线程



* 定义：JS 同一时间只能执行一段代码（像 “一只手做事”）

* 原因：避免 DOM 操作冲突（比如同时修改和删除同一个节点）

* 问题：遇到耗时任务（如网络请求、定时器）会阻塞页面，需异步编程解决

### 2. 异步编程



* 定义：不阻塞主线程，先执行其他任务，待耗时任务有结果后再回头处理

* 目的：保证页面流畅响应（比如加载图片时，可继续点击按钮）

* 核心实现：事件轮询（协调同步 / 异步任务执行顺序）

### 3. 关键角色



| 角色               | 作用                           |
| ---------------- | ---------------------------- |
| 调用栈（Call Stack）  | 执行同步代码的 “操作台”，遵循「后进先出」（LIFO） |
| 宏任务队列            | 存放低优先级异步任务，一次执行 1 个          |
| 微任务队列            | 存放高优先级异步任务，一次执行所有（清空前不执行宏任务） |
| 事件轮询（Event Loop） | 协调三者的执行顺序，循环执行 “同步→微任务→宏任务”  |

### 4. 宏任务 vs 微任务（必背分类）

#### 宏任务（Macro Task）



* 常见类型：`setTimeout`、`setInterval`、DOM 事件回调（点击 / 输入）、`fetch` 网络请求、`script` 整体代码

* 特点：耗时较长、优先级低，执行完一个后需重新检查微任务队列

#### 微任务（Micro Task）



* 常见类型：`Promise.then/catch/finally`、`async/await`（Promise 语法糖）、`queueMicrotask`、`MutationObserver`

* 特点：耗时短、优先级高，当前宏任务执行完后立即清空所有微任务

### 5. 其他关联概念



* 事件：用户 / 浏览器的动作（如点击、图片加载完成）

* 回调函数：异步任务的 “后续操作”（如 “点击后执行的函数”）

* Promise：处理异步的工具，`Promise.resolve()` 表示异步成功完成

* `then()` 方法：指定 Promise 成功后的回调（微任务）

* JS 引擎：执行 JS 代码的程序（如浏览器 V8 引擎）

## 二、事件轮询执行流程（核心，面试必讲）

### 1. 简化流程（一句话记）

`同步代码执行 → 清空所有微任务 → 浏览器UI渲染（可选） → 执行一个宏任务 → 重复上述步骤`

### 2. 详细步骤（分点背诵）



1. 主线程先执行调用栈中的所有同步代码，直到栈空；

2. 检查微任务队列，将队列中所有任务按顺序推入调用栈执行，直到微任务队列为空；

3. 浏览器判断是否需要 UI 渲染（如 DOM 更新），若需要则执行渲染；

4. 从宏任务队列中取出**第一个**任务，推入调用栈执行；

5. 执行完当前宏任务后，回到步骤 2，形成循环（事件轮询）。

### 3. 生活类比（辅助记忆）



* 你（JS 引擎）炒菜（同步代码）→ 烤箱叮了（微任务）→ 电话响了（宏任务）

* 流程：炒完菜（同步）→ 先拿烤鸡（清微任务）→ 再接电话（执行一个宏任务）→ 循环

## 三、经典代码示例（面试常考，会算顺序）

### 示例 1：基础题（必背）



```
console.log('1'); // 同步任务

setTimeout(() => console.log('2'), 0); // 宏任务

Promise.resolve().then(() => console.log('3')); // 微任务

console.log('4'); // 同步任务
```



* 输出顺序：`1 → 4 → 3 → 2`

* 解析：同步先执行（1、4）→ 清微任务（3）→ 执行宏任务（2）

### 示例 2：嵌套题（高频）



```
console.log('1');

setTimeout(() => {

&#x20; console.log('2');

&#x20; Promise.resolve().then(() => console.log('3')); // 宏任务内的微任务

}, 0);

Promise.resolve().then(() => {

&#x20; console.log('4');

&#x20; setTimeout(() => console.log('5'), 0); // 微任务内的宏任务

});

console.log('6');
```



* 输出顺序：`1 → 6 → 4 → 2 → 3 → 5`

* 解析：同步（1、6）→ 清微任务（4，同时加宏任务 5）→ 执行宏任务 2（输出 2，加微任务 3）→ 清微任务 3 → 执行宏任务 5

### 示例 3：async/await 题（重点）



```
console.log('start');

async function fn() {

&#x20; console.log('fn内同步'); // 同步代码

&#x20; await Promise.resolve(); // await后是微任务

&#x20; console.log('fn内异步');

}

fn();

setTimeout(() => console.log('setTimeout'), 0); // 宏任务

console.log('end'); // 同步代码
```



* 输出顺序：`start → fn内同步 → end → fn内异步 → setTimeout`

* 解析：async 函数调用时同步代码先执行，await 后代码入微任务队列

## 四、面试回答模板（直接套用，逻辑清晰）

### 回答框架：背景→角色→流程→例子→坑

面试官问 “请讲一下 JS 事件轮询机制”，按以下话术回答：

“JS 是单线程语言，同一时间只能执行一段代码，但实际开发中需要处理点击、网络请求等多个任务，所以需要事件轮询（Event Loop）来实现异步非阻塞。

事件轮询的核心是协调三个角色：调用栈（执行同步代码）、宏任务队列（存放 setTimeout、DOM 事件等）、微任务队列（存放 Promise.then、async/await 等）。

它的执行流程是：



1. 先执行调用栈中的所有同步代码，直到栈空；

2. 清空微任务队列的所有任务；

3. 浏览器可能进行 UI 渲染；

4. 从宏任务队列取第一个任务执行；

5. 重复上述步骤，形成循环。

举个例子，一段代码里有同步打印、setTimeout 和 Promise.then，输出顺序是同步先执行，再执行 Promise.then（微任务），最后执行 setTimeout（宏任务）。

这里有两个面试常考的坑：



1. setTimeout (fn, 0) 不是立刻执行，而是等同步和微任务执行完后才执行；

2. 微任务优先级高于宏任务，必须清空所有微任务再执行宏任务，且微任务中产生的新微任务会继续执行，直到队列空。”

## 五、面试追问应对（进阶加分）

### 追问 1：Node 环境和浏览器环境的事件轮询区别？



* 浏览器：只有一个宏任务队列、一个微任务队列，流程是「同步→微任务→宏任务」；

* Node：有 6 个阶段的宏任务队列（如 timers、poll），微任务分两类：`process.nextTick`（优先级更高）和其他微任务（如 Promise.then），执行完一个阶段的宏任务后，清空所有微任务。

### 追问 2：微任务中产生新的微任务，会怎么处理？



* 新微任务会加入当前微任务队列尾部，继续执行，直到整个微任务队列为空（不会切换到宏任务）。

### 追问 3：为什么要区分宏任务和微任务？



* 为了让 “紧急异步任务”（如 Promise 回调）更快执行，避免被耗时宏任务（如网络请求）阻塞，提升页面响应速度。

### 追问 4：setTimeout 的延迟时间不准的原因？



* 延迟时间是 “任务加入队列的时间”，而非 “执行时间”；

* 需等待当前同步代码、所有微任务执行完，且宏任务队列前无其他任务，才会执行。

## 六、学习资源（巩固提升用）

### 1. 图文教程



* 掘金《这一次，彻底弄懂 JavaScript 执行机制》：[https://juejin.cn/post/6844903512845860872](https://juejin.cn/post/6844903512845860872)

* MDN 官方文档：[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/EventLoop)

* 可视化漫画：[https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif](https://dev.to/lydiahallie/javascript-visualized-event-loop-3dif)（英文，浏览器翻译即可）

### 2. 视频教程



* B 站尚硅谷（第 23 集）：[https://www.bilibili.com/video/BV1Kt411w7Ep/?p=23](https://www.bilibili.com/video/BV1Kt411w7Ep/?p=23)

* B 站李立超（20 分钟起）：[https://www.bilibili.com/video/BV1Sy4y1C7ha/?t=1200](https://www.bilibili.com/video/BV1Sy4y1C7ha/?t=1200)

### 3. 刷题工具



* 在线执行：JS Bin（[https://jsbin.com/](https://jsbin.com/)）

* 可视化执行：JavaScript Tutor（[https://pythontutor.com/javascript.html](https://pythontutor.com/javascript.html)）

* 面试真题：力扣 JavaScript 专题（筛选 “事件循环”）、牛客网 JS 题库

> （注：文档部分内容可能由 AI 生成）