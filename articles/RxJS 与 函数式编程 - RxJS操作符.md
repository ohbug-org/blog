# RxJS 与 函数式编程 - RxJS 操作符

## 前言

这是一个新的系列, 记录自己学习 RxJS 以及函数式编程的过程.
这个系列的目的一个是作为笔记记录, 另一个是希望看到这个系列的你能对 RxJS 和函数式编程提起兴趣学习/使用起来.

学习 RxJS 不是因为他是一个多么新的技术（它已经存在多年）也不是因为他是多炫酷的技术, 学习它是因为它确实能帮我们解决许多问题：

- 如何控制大量代码的复杂度
- 如何保持代码的可读性
- 如何处理异步操作

可能有许多人早已听说过 RxJS 以及它令人望而生畏的学习曲线（别跑说你呢）, 别怕这个系列会以最易懂的方式解释给你（复杂的我也不会）.
借用 RxJS 入门手册中的一句话：

> 可以把 RxJS 当做是用来处理事件的 [Lodash](https://lodash.com/) .

如无特殊说明, 从本期开始 RxJS 版本基于 `7.3.0`, 系列内容基于程墨的《深入浅出 RxJS》（好书推荐）, 并做了一定更新与总结, 可能会引用书中原话.

### 前文回顾

- [RxJS 与 函数式编程 - 函数式编程](./RxJS%20与%20函数式编程%20-%20函数式编程.md)
- [RxJS 与 函数式编程 - RxJS 入门](./RxJS%20与%20函数式编程%20-%20RxJS入门.md)

## 操作符基础

> ⼀个操作符是返回⼀个 Observable 对象的函数, 不过, 有的操作符是根据其他 Observable 对象产⽣ 返回的 Observable 对象, 有的操作符则是利⽤其他类型输⼊产⽣返回的 Observable 对象, 还有⼀些操作符不需要输⼊就可以凭空创造⼀个 Observable 对象.
>
> 使⽤和组合操作符是 RxJS 编程的重要部分, 毫不夸张地说, 对操作符 使⽤的熟练程度决定对 RxJS 的掌握程度.

先看一个简单的栗子 🌰:

```javascript
import { of } from "rxjs";

const source$ = of(1, 2, 3);

source$.subscribe(console.log);
```

`of` 将它的参数转换为了可观察序列.

使用 `map` 操作符对序列进行修改, 类似于 js 中的 map, `map` 操作符也是对序列的每一项进行修改并返回修改后的数据.

```javascript
import { of } from "rxjs";
import { map } from "rxjs/operators";

const source$ = of(1, 2, 3);

source$.pipe(map((value) => value * 2)).subscribe(console.log);
```

上面代码引入了两个新概念, `pipe` 以及 `map` 操作符.
` `pipe`的出现替代了之前 import 补丁操作符进行链式操作的做法, 一方面解决了`Observable.prototype`增大影响维护的问题, 一方面在工程化层面支持了`tree-shaking` 可以大幅度减小打包体积.

`pipe` 可以传入任何需要的操作符组合, 例如上述代码每个数字在乘 2 的基础上加 1:

```javascript
import { of } from "rxjs";
import { map } from "rxjs/operators";

const source$ = of(1, 2, 3);

source$
  .pipe(
    map((value) => value * 2),
    map((value) => value + 1)
  )
  .subscribe(console.log);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/mystifying-ptolemy-ys6kb?file=/src/index.js) 直接查看结果

在看一个 `filter` 操作符, 想象一个需求 输出给定序列中所有的偶数:

```javascript
import { of } from "rxjs";
import { map, filter } from "rxjs/operators";

const source$ = of(1, 2, 3);

source$
  .pipe(
    filter((value) => value % 2 === 0),
    map((value) => `偶数为: ${value}`)
  )
  .subscribe(console.log);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/gifted-kate-yndox?file=/src/index.js) 直接查看结果

看一个复杂些的例子:

> 在屏幕中间实现一个可拖动的按钮。当它移向边缘时，背景颜色从白色变为红色。

```html
<div id="overlay"></div>
<div id="button" draggable="true"></div>
```

```css
body {
  overflow: hidden;
  margin: 0;
}

#overlay {
  width: 100vw;
  height: 100vh;
  opacity: 0;
  background: red;
}

#button {
  cursor: grabbing;
  background-color: black;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

```javascript
import "./styles.css";

import { fromEvent } from "rxjs";
import { map, tap } from "rxjs/operators";

const button = document.querySelector("#button");
const overlay = document.querySelector("#overlay");
const maxY = window.innerHeight / 2;
const maxX = window.innerWidth / 2;

fromEvent(button, "drag")
  .pipe(
    // 计算 overlay 的 opacity
    map((event) => {
      if (event.clientY === 0 && event.clientX === 0) {
        return 0;
      }

      const y = Math.abs(event.clientY - maxY);
      const pY = y / maxY;
      const x = Math.abs(event.clientX - maxX);
      const pX = x / maxX;
      return Math.max(pY, pX);
    }),
    // tap 操作符 通常用于对数据执行副作用
    tap((opacity) => {
      overlay.style.opacity = opacity;
    })
  )
  .subscribe(console.log);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/bold-mclaren-pt957?file=/src/index.js) 直接查看结果

## 操作符的分类

> 掌握操作符最困难之处是当遇到⼀个实际问题的时候，该选择哪⼀个或者哪⼀些操作符来解决问题，所以，⾸先要对这些操作符分门别类，知道各类操作符的特点。

根据功能，操作符可以分为以下类别：

- 创建类（creation）
- 转化类（transformation）
- 过滤类（filtering）
- 合并类（combination）
- 多播类（multicasting）
- 错误处理类（error Handling）
- 辅助⼯具类（utility）
- 条件分⽀类（conditional&boolean）
- 数学和合计类（mathmatical&aggregate）

> 接下来介绍操作符是如何实现的。虽然对于应⽤开发者⽽⾔⼯作的重 点是如何使⽤ RxJS 中的操作符，但是，了解操作符的实现⽅式会加深对 RxJS 的理解。另外，虽然不是每个⼈都会给 RxJS 的代码库中添加操作符， 但是在每个具体的应⽤项⽬中，却很有可能会⽤上⼀些可以重复使⽤的逻 辑，这些逻辑可以封装在⾃定义的操作符中，这时候就需要知道如何定制 ⼀个新的操作符了。

## 创建操作符

每个操作符都是⼀个函数，不管实现什么功能，都必须考虑下⾯这些功能要点：

- 返回⼀个全新的 Observable 对象。
- 对上游和下游的订阅及退订处理。
- 处理异常情况。
- 及时释放资源。

以最简单的 map 操作符实现来说明上⾯的要点。

### 返回⼀个全新的 Observable 对象

```javascript
function map(project) {
  return (source) =>
    new Observable((observer) =>
      source.subscribe({
        next: (value) => observer.next(project(value)),
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      })
    );
}
```

这样一个基本的 `map` 操作符就完成了, 但这并不是一个完备的操作符

### 对上游和下游的订阅及退订处理

上面的实现中对 `source` 进行了订阅却没有处理退订, 如果相关资源得不到释放, 就有可能造成资源泄露. 我们对代码进行改进:

```javascript
function map(project) {
  return (source) =>
    new Observable((observer) => {
      const sub = source.subscribe({
        next: (value) => observer.next(project(value)),
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      });
      return {
        unsubscribe: sub.unsubscribe,
      };
    });
}
```

### 处理异常情况

> 对于 map 这个操作符，参数 project 是⼀种输⼊，不受 map ⾃⾝控制，换句话说，project 可能是有问题的代码，这超出了 map 函数的控制范围，⽽我们能做的，就是考虑到 project 的调⽤可能会出错。

改进上⾯ map 实现的部分，如下所⽰:

```javascript
function map(project) {
  return (source) =>
    new Observable((observer) => {
      const sub = source.subscribe({
        next: (value) => {
          try {
            observer.next(project(value));
          } catch (err) {
            observer.error(err);
          }
        },
        error: (err) => observer.error(err),
        complete: () => observer.complete(),
      });
      return {
        unsubscribe: sub.unsubscribe,
      };
    });
}
```

project 在调用时利用 try catch 捕获可能的错误, 如果出现错误就调用下游的 error 函数.

所以，map 有两种可能向下游传递 error 消息的⽅式，⼀种是上游的 error 直接转⼿给下游，另⼀种是 project 函数执⾏过程中产⽣的 error 也交给下游。

### 及时释放资源

> map 并不占⽤什么资源，但有的操作符则不是这样，尤其是和浏览器资源直接打交道的操作符。⽐如，从 DOM 中获取⽤户操作事件的操作符， 产⽣的 Observable 对象被订阅时，肯定会在 DOM 中添加事件处理函数，如果事件处理函数只被添加⽽不删除，那就有产⽣资源泄露的危险，所以， ⼀定要在退订的时候去掉挂在 DOM 上的这些事件处理函数。 有的操作符还会和 WebSocket 资源关联从中获取推送消息，这些操作符⼀定要在相关 Observable 对象被退订时释放 WebSocket 资源。

现在我们就得到了与 rxjs 官方提供的功能相同的 map 操作符.

## 结语

这一篇文章介绍了 RxJS 中操作符的用法与自定义操作符的实现方式.

下一章聊一下创建数据流的几个操作符

---

<div align="center">

![wechat](../images/wechat.jpg)

</div>
