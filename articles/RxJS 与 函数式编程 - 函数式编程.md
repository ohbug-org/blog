# RxJS 与 函数式编程 - 函数式编程

## 前言

这是一个新的系列，记录自己学习 RxJS 以及函数式编程的过程。
这个系列的目的一个是作为笔记记录，另一个是希望看到这个系列的你能对 RxJS 和函数式编程提起兴趣学习/使用起来。

系列的核心是记录自己学习 RxJS 的过程，但是学习它不可避免的要学习函数式编程，本篇文章将着重记录函数式编程。

## 函数式编程

曾经很长一段时间我都认为函数式编程就是使用函数解决问题的编程方式，学习了 RxJS 之后这个观点发生了翻天覆地的变化。

首先改变我观念的是函数式编程固有的特点：

- 声明式 (Declarative)

- 纯函数 (Pure Function)

- 数据不可变性 (Immutability)

### 声明式

往往解释一个概念最简单的方式就是解释它的对立概念。
声明式编程的对立概念应该是命令式编程，举一个生活中的例子 打车：

- 命令式：
  - 上车
  - 路口右转
  - 直行500米
  - 路口左转
  - 阿巴阿巴...
  - 到站下车

- 声明式：
  - 上车
  - 司机师傅我要去XXX广场，然后开始玩手机
  - 到站下车

然后举一个代码的例子 实现一个函数参数为一个数组要求函数将数组的每一项数值都乘以2：

```javascript
function double(arr) {
  const result = []
  for (let i = 0; i < arr.length; i++) {
    const item = arr[i]
    result.push(item * 2)
  }
  return result
}
```

上面的实现属于典型的命令式编程风格，其本质就是告诉计算机整个功能的实现所需要的计算逻辑，而他的对立面声明式编程的代码是这样：

```javascript
function double(arr) {
  return arr.map(item => item * 2)
}
```

如果使用箭头函数代码可以更加简洁：

```javascript
const double = arr => arr.map(item => item * 2)
```

### 纯函数

讲纯函数我认为必须先抛出许多人写代码可能出现的“问题” 还是以 double 函数举例：

```javascript
function double(arr) {
  for (let i = 0; i < arr.length; i++) {
    const item = arr[i]
    arr[i] = item * 2
  }
  return arr
}

const originalArray = [1, 2, 3]
const result = double(originalArray)
```

请问上述代码 result 值是什么。

这时候应该要脱口而出 `[2, 4, 6]`，但是对应的 originalArray 值是什么呢？

这里 originalArray 的值是 `[2, 4, 6]`，单从 “功能是否实现” 的角度看，这里 double 的实现没有任何问题，可是作为开发者的你是否会选择使用这种实现方式的函数呢？或者作为开发者的你是否经常写这种函数呢？

这里的这种函数称之为 “不纯函数 (Impure Function)” 相反前一小节写的那两个 double 函数都是纯函数。不纯函数的特点：

- 改变全局变量
- 改变传入参数
- 抛出异常
- 操作 DOM
- 读取用户输入
- 网络的输入/输出，比如AJAX

扯了这么多就是想表达一句话：
纯函数做的事就是给我相同的输入参数，我始终返给你相同的输出，并且这个过程没有副作用（让我想起了 `React.useEffect`）产生。

最后多说一句，纯函数非常容易些单元测试。

### 数据不可变性

数据不可变的重要性其实在上面 **[纯函数](#纯函数)** 的例子中已经有了体现，double 函数修改了传入参数，导致 originalArray 发生了变化产生副作用，这种类似的问题引发了数不清的 bug。

怎么实现数据的不可变性也是一个重要的内容，可能涉及到深浅拷贝、immutable.js 等，先不展开了以后聊（我懒）。

> 多嘴提一句，JavaScript 中的 `const` 关键字只是规定一个变量的 **引用** 不可改变，所以 `const` 定义的变量严格讲不全属于不可变数据。

## 结语

关于函数式编程的内容本篇介绍只是冰山一角，但考虑到这个系列的核心是讲 RxJS 就不再深挖了（多的不会了）。

下一篇开始讲 RxJS。

---

<div align="center">

  ![wechat](../images/wechat.jpg)

</div>
