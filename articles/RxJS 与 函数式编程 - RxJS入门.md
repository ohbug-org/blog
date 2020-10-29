# RxJS 与 函数式编程 - RxJS入门

## 前言

这是一个新的系列，记录自己学习 RxJS 以及函数式编程的过程。
这个系列的目的一个是作为笔记记录，另一个是希望看到这个系列的你能对 RxJS 和函数式编程提起兴趣学习/使用起来。

学习 RxJS 不是因为他是一个多么新的技术（它已经存在多年）也不是因为他是多炫酷的技术，学习它是因为它确实能帮我们解决许多问题：

- 如何控制大量代码的复杂度
- 如何保持代码的可读性
- 如何处理异步操作

可能有许多人早已听说过 RxJS 以及它令人望而生畏的学习曲线（别跑说你呢），别怕这个系列会以最易懂的方式解释给你（复杂的我也不会）。
借用 RxJS 入门手册中的一句话：

> 可以把 RxJS 当做是用来处理事件的 [Lodash](https://lodash.com/) 。

如果没有特殊说明，这个系列的 RxJS 版本都是 6.6.3。系列内容基于程墨的《深入浅出RxJS》（好书推荐），并做了一定更新与总结，可能会引用书中原话。

## 概念介绍

在 RxJS 中用来解决异步事件管理的的基本概念是：

- **Observable (可观察对象)**: 表示一个概念，这个概念是一个可调用的未来值或事件的集合。
- **Observer (观察者)**: 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。
- **Subscription (订阅)**: 表示 Observable 的执行，主要用于取消 Observable 的执行。
- **Operators (操作符)**: 采用函数式编程风格的纯函数 (pure function)，使用像 map、filter、concat、flatMap 等这样的操作符来处理集合。
- **Subject (主体)**: 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。
- **Schedulers (调度器)**: 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 setTimeout 或 requestAnimationFrame 或其他。

### Observable 和 Observer

学习 RxJS 绝对避不开的概念就是 Observable  和 Observer。

> 可以说 RxJS 的运行就是 Observable 和 Observer 之间的互动游戏。

顾名思义，Observable 就是 “可被观察的东西”，你可以把它理解为数据源，Observer 是 “观察者”，可以理解为对数据源进行操作的角色。二者通过 Observable 对象的 subscribe 函数连接。

#### 第一个 Observable

我们来看一个简单的 Observable：

```javascript
import { Observable } from 'rxjs'

const observable = new Observable((observer) => {
  observer.next(1);
  observer.next(2);
  observer.next(3);
});

const observer = {
  next: (x) => console.log("got value " + x),
};

observable.subscribe(observer);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/floral-framework-ui861?file=/src/index.js) 直接查看结果

这段代码依次输出：

```
got value 1
got value 2
got value 3
```

这是一个典型的观察者模式实现，`observable` 直接由 RxJS 的 **Observable** 实例化得来，而在他的回调参数里面连续调用三次 next 输出了3个值；
下面 `observer` 定义了一个含有 next 函数的对象，next 函数只是 log 了输入值，很明显这里的 next 函数正是上方 `observable` 回调中连续调用三次的 next；
最后，由 `observable` 的 `subscribe` 方法连接起 `observable` 与 `observer`。

只是为了输出几个数，写了一长串代码似乎有些本末倒置，下面就用 RxJS 解决一些复杂的问题。

#### Observable 的 complete

```javascript
import { Observable } from "rxjs";

const observable = new Observable((observer) => {
  let num = 1;
  const handle = setInterval(() => {
    observer.next(num++);

    if (num > 10) {
      clearInterval(handle);
    }
  }, 1000);
});

const observer = {
  next: (x) => console.log("got value " + x)
};

observable.subscribe(observer);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/hidden-dew-ifhjp?file=/src/index.js) 直接查看结果

上面代码将之前的三个 next 调用改为利用 `setInterval` 每隔一秒递增 num，并且当 num 大于 10 的时候取消掉 `setInterval`。

看到这里不知道大家有没有注意两个点：

1. Observable 推送数据可以存在时间间隔，这意味着它可以处理异步操作。
1. Observable 虽然在 num 大于 10 的时候停止了推送数据，但是它本身并不知道会不会有新数据产生，这意味着 Observable 仍在工作。

为了通知 Observable 停止工作，需要对上面的代码进行一些完善：

```javascript
import { Observable } from "rxjs";

const observable = new Observable((observer) => {
  let num = 1;
  const handle = setInterval(() => {
    observer.next(num++);

    if (num > 10) {
      clearInterval(handle);
      observer.complete();
    }
  }, 1000);
});

const observer = {
  next: (x) => console.log("got value " + x),
  complete: () => console.log("complete")
};

observable.subscribe(observer);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/beautiful-leaf-8nyji?file=/src/index.js) 直接查看结果

你可以看到我们修改了两个地方:

- 为 `observer` 对象增加了一个 complete 函数，这个函数描述 Observable 结束时做的事情。
- 为 `observable` 的回调取消 `setInterval` 后调用 complete 函数，告知 Observable 应该如何结束。

#### Observable 的 error

健壮的代码除了 next 与 complete 表示逻辑，非常需要对错误的处理。
拿代码举个栗子：

```javascript
import { Observable } from "rxjs";

const observable = new Observable((observer) => {
  observer.next(1);
  observer.error("something wrong");
  observer.complete();
});

const observer = {
  next: (x) => console.log("got value " + x),
  error: (err) => console.log("something seems to be wrong: " + err),
  complete: () => console.log("complete")
};

observable.subscribe(observer);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/fancy-darkness-50hjz?file=/src/index.js) 直接查看结果

执行结果：

```
got value 1
something seems to be wrong: something wrong
```

看代码注意到了调用 `observer.error` 后紧跟着调用了 `observer.complete`，但是结果却没有输出 complete，这是为什么？

> 在 RxJS 中，一个 Observable 对象只有一种终结状态，要么是完结（complete），要么是出错（error）。一旦进入完结状态，不论是 error 还是 complete 都将不再调用 observer 的其他函数。

#### 退订 Observable

有合就有分，我们已经了解了 Observable 和 Observer 如何建立关系了，是时候知道怎么分开它们了。

```javascript
import { Observable } from "rxjs";

const observable = new Observable((observer) => {
  let num = 1;
  const handle = setInterval(() => {
    observer.next(num++);

    if (num > 10) {
      clearInterval(handle);
      observer.complete();
    }
  }, 1000);

  return {
    unsubscribe: () => {
      console.log("unsubscribe");
    }
  };
});

const observer = {
  next: (x) => console.log("got value " + x),
  error: (err) => console.log("something seems to be wrong: " + err),
  complete: () => console.log("complete")
};

const subscription = observable.subscribe(observer);
setTimeout(() => {
  subscription.unsubscribe();
}, 3500);
```

> 你可以打开 [codesandbox](https://codesandbox.io/s/wonderful-resonance-ds6op?file=/src/index.js) 直接查看结果

看这段代码，我们将 subscribe 的返回结果赋值给变量 `subscription`，并且在完成订阅 3.5 秒后调用 `subscription.unsubscribe`，也就是订阅完成后 3.5 秒退订。

运行后输出：

```
got value 1
got value 2
got value 3
unsubscribe
```

如果不进行主动退订，上面的代码应该会输出到 10 并且调用 complete 后结束。主动退订后，我们看到只输出到 3 并且没有调用 complete，也就是在 complete 之前就断开了连接。

> 这是 RxJS 中很重要的一点：Observable 产生的事件，只有 Observer 通过 subscribe 订阅之后才会收到，在 unsubscribe 之后就不会再收到。

其实读到这里我产生过一个疑问，想要停止代码运行应该调用 complete 还是 unsubscribe？我进行了一个实验：

```javascript
import { Observable } from "rxjs";

const observable = new Observable((observer) => {
  let num = 1;
  const handle = setInterval(() => {
    observer.next(num++);

    if (num > 10) {
      clearInterval(handle);
      observer.complete();
    }
  }, 1000);

  return {
    unsubscribe: () => {
      console.log("unsubscribe");
    }
  };
});

const observer = {
  next: (x) => console.log("got value " + x),
  error: (err) => console.log("something seems to be wrong: " + err),
  complete: () => console.log("complete")
};

const subscription = observable.subscribe(observer);
// setTimeout(() => {
//   subscription.unsubscribe();
// }, 3500);
```

首先我注释掉主动断开连接的部分，让程序通过 complete 调用结束，输出结果：

```
got value 1
got value 2
got value 3
got value 4
got value 5
got value 6
got value 7
got value 8
got value 9
got value 10
complete
unsubscribe
```

可以看到 complete 后仍然调用了 unsubscribe，那么把 `observer.complete` 换成 `observer.error` 呢？
感兴趣的可以去试一下，这里直接说我得到的结论吧：

> 每个 Observable 仅会被退订一次，complete 和 error 这两个代表结束的函数调用后紧接会着调用 unsubscribe。另一种情况，如果在 complete 或 error 前主动进行 `subscription.unsubscribe` 退订，那么 complete 或 error 都不会执行并且退订。

## 结语

这一篇文章介绍了 Observable 和 Observer 这两大 RxJS 的主角，二者通过 Observable 对象的 subscribe 函数连接，让 Observer 对象订阅 Observable 对象推送的内容，并且可以通过 unsubscribe 退订。

下一篇讲一下 RxJS 的操作符。

---

<div align="center">

  ![|}}wechat](../images/wechat.jpg)

</div>
