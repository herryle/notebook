> > 导读：MobX 是一个优秀的响应式状态管理库，在流行的状态管理库 Redux 之外为我们提供了其他选择。如果你还没有尝试过 MobX，我强烈建议你继续阅读本文，并跟着示例动手实践体验一下。本文是 MobX 的入门教程，文章包含 MobX 的设计哲学、状态派生模型、核心API以及与 React 的集成。
>
> [MobX](https://github.com/mobxjs/mobx) 是一个简单、可伸缩的响应式状态管理库。通过 MobX 你可以用最直观的方式修改状态，其他的一切 MobX 都会为你处理好（如自动更新UI），并且具有非常高的性能。
>
> MobX 文档的概念和 API 比较多，初次接触时可能会感觉无从下手或者抓不住重点，本文尝试提供一份 MobX 核心的知识的简明教程，作为阅读官方文档之前的热身。
>
> > 本文不包含任何跟装饰器(decorator)相关的内容，因为装饰器并不是 MobX 的核心内容，它只是语法糖，没有它照样可以使用 MobX。
>
> **目录**
>
> * [MobX 的设计哲学](#MobX%E7%9A%84%E8%AE%BE%E8%AE%A1%E5%93%B2%E5%AD%A6)
>* [MobX 状态响应模型](#MobX%E7%8A%B6%E6%80%81%E5%93%8D%E5%BA%94%E6%A8%A1%E5%9E%8B)
> * [MobX 状态派生模型](#MobX%E7%8A%B6%E6%80%81%E6%B4%BE%E7%94%9F%E6%A8%A1%E5%9E%8B)
> * [MobX 核心 API 解析](#MobX%E6%A0%B8%E5%BF%83API%E8%A7%A3%E6%9E%90)
> 
>   * [ `observable`](#%60observable%60)
>  * [ `autorun`](#%60autorun%60)
>   * [ `computed`](#%60computed%60)
>   * [ `action`](#%60action%60)
> * [MobX 与 React 集成](#MobX%E4%B8%8EReact%E9%9B%86%E6%88%90)
> * [小结](#%E5%B0%8F%E7%BB%93)
> 
> ## MobX 的设计哲学
>在学习使用 MobX API 之前，我们首先要了解 MobX 的设计哲学，它是我们在思考 MobX 应用时的心智模型，可以帮助我们更好的使用 MobX API。
> 
> MobX 的设计哲学概括起来就一句话：**Anything that can be derived from the application state, should be derived. Automatically. (任何可以从应用状态中派生的内容，都应当自动地被派生。)**
>
> 理解这句话的关键是搞清楚【派生】的含义是什么？在 MobX 中【派生】的含义比较广泛，包括：
>
> * 用户接口(UI)，如组件、页面、图表
>* 派生数据(computed data)，如从数组中计算得到的数组长度
> * 副作用(side effect)，如发送网络请求、设置定时任务、打印日志等
> 
> 我们平时常看到的状态响应模型，其中的响应就可以看做是状态的一种派生，MobX 将这种模型进行泛化，形成更通用的状态派生模型，接下来会详细介绍。
>
> ## MobX 状态响应模型
>状态响应模型概括起来主要包含三个要素：定义状态、响应状态、修改状态（如下图所示）。
> 
> ![](https://camo.githubusercontent.com/ce0f18cefc8619d5a002829bce90153cab5b67d8adff72312dcb4742b7450894/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f7768696e632f7069632d6265642f323032302d31312d32302f313630353838373733363431352d6d6f62785f706963312e706e67)
>
> MobX 中通过`observable`来定义可观察状态， 它接受任意 JS 值（包括`Object`、`Array`、`Map`、`Set`），返回原始数据的代理对象，代理对象与原始数据具有相同的接口，你可以把代理对象当做原始数据使用。
>
> ```js
>// 定义状态
> const store = observable({
>   count: 0
> });
> ```
> 
> MobX 中通过`autorun`来定义状态变化时要执行的响应操作，它接受一个函数。此后，每当`observable`中定义的状态发生变化时，MobX 都会立即执行该函数。
>
> ```js
>// 响应状态
> autorun(() => {
>   console.log("count:", store.count);
> });
> ```
> 
> MobX 中修改状态和修改原始数据的方式没什么区别，这也是 MobX 的优点——符合直觉的操作方式。
>
> ```js
>// 修改状态
> store.count += 1;
> ```
> 
> 把上面的部分串起来就是一个最简单的 MobX 示例了（如下），示例中每次修改 count 的值时会自动打印一条日志，并且日志包含最新的 count 值。这个示例揭示了 MobX 中最核心的功能。
>
> ```js
>import { observable, autorun } from "mobx";
> 
> // 1. 定义状态
> const store = observable({
>   count: 0
> });
> 
> // 2. 响应状态
> autorun(() => {
>   console.log("count:", store.count);
> });
> // count: 0
> 
> // 3. 修改状态
> store.count += 1;
> // count: 1
> ```
> 
> 上面例子中，首先通过 MobX 提供的`observable`函数定义状态，例子的状态是`{count: 0}`。然后通过通过 MobX 提供的`autorun`函数定义状态响应函数，例子中是一条打印当前`count`值得的语句，当定义的状态中的任何值发生变化时，该响应函数会立即执行，并且是由 MobX 自动完成的。最后是修改状态，和操作对象属性一样，例子中是对`count`属性进行自增操作。
>
> ## MobX 状态派生模型
>从上一节中我们知道 MobX 的状态响应函数是在状态变化时执行某些操作。实际应用中这些操作可分为两类：带副作用（如打印日志、渲染 UI、请求网络等）和不带副作用（如计算数组的长度）。MobX 将这些操作统称为**派生(Derivations)**，即可从应用状态中派生出来的任何内容。
> 
> 其中不带副作用的操作是一个纯函数，一般是根据当前状态计算返回一个新的值。为了区分带副作用和不带副作用的两类情况，MobX 将 Derivations 概念细分成两个概念 **Reactions** 和 **Computed values** 来区分二者。此外，MobX 还提供了一个可选的概念 **Action** 来表示对 State 的修改操作，用于约束和预测应用状态的修改行为。这些概念汇总在一起如下图：
>
> ![](https://camo.githubusercontent.com/d7c20321b6d79fbccfb769674c00c7c2cc79b97bf17b650e6e9d71dd7cb98f3d/68747470733a2f2f63646e2e6a7364656c6976722e6e65742f67682f7768696e632f7069632d6265642f323032302d31312d32302f313630353838373830323236372d6d6f62785f706963322e706e67)
>
> 下面是一个简单的示例，完整展示了上图中涉及的所有概念。
>
> index.html 文件
>
> ```
><html>
>   <body>
>     <div id="container"></div>
>   </body>
> </html>
> ```
> 
> index.js 文件
>
> ```js
>import { observable, autorun, computed, action } from "mobx"
> 
> const containerEl = document.querySelector("#container");
> 
> // State
> const store = observable({
>   count: 0
> });
> 
> // Actions
> window.increaseCount = action(() => store.count++);
> 
> // Computed values
> const doubleCount = computed(() => 2 * store.count);
> 
> // Reactions
> autorun(() => {
>   containerEl.innerHTML = `
>     <span>${store.count} * 2 = ${doubleCount.get()}</span>
>     <button onclick='window.increaseCount()'>+1</button>
>   `;
> });
> 
> // 0 * 2 = 0
> // (点击按钮)
> // 1 * 2 = 2
> // (点击按钮)
> // 2 * 2 = 4
> ```
> 
> 这个示例展示了一个简单的乘法，点击按钮后数据会自增，同时计算的结果也随之更新。相比上一个例子，它有几个值得注意的区别：
>
> * 通过`computed`定义计算值，并返回计算值对象`doubleCount`，通过其`get/set`方法访问内部计算值。`computed`接受一个返回计算值的函数，在函数内部可以使用`observable`定义的状态数据（如`count`），每当`count`的值发生变化时，`doubleCount`内部的计算值会自动更新。
>* 通过`action`定义状态修改操作，`action`接受一个函数，并返回一个签名相同的函数。在函数内部可直接修改`obsevable`定义的状态（如`count`)触发`autorun`和`computed`重新运行。
> * `autorun`中的响应操作替换成了渲染 UI，每当状态发生变化时重新渲染 UI。
> 
> ## MobX 核心 API 解析
>MobX 的 API 可分为四类：定义状态（`observable`）、响应状态（`autorun`, `computed`）、修改状态（`action`）、辅助函数。下面挑出最核心的 API 进行重点介绍，但不会涉及 API 的详细用法（请参考[MobX Api Reference](https://mobx.js.org/refguide/api.html)）。
> 
> > MobX 目前同时支持 v4 和 v5 两个版本，这两个版本的 API 是相同的（功能相同），他们的区别在于内部实现数据响应的方式不同，如果使用 v4 版本时需要注意文档中标注的注意情况。
>
> ### `observable`
>`observable`用于定义可观察状态，其类型定义如下（已简化）：
> 
> ```ts
><T extends Object>(value: T): T & IObservableObject;
> <T = any>(value: T[]): IObservableArray<T>;
> <K = any, V = any>(value: Map<K, V>): ObservableMap<K, V>;
> <T = any>(value: Set<T>): ObservableSet<T>;
> ```
> 
> 它接受`Object/Array/Map/Set`类型的数据作为参数，并返回对应数据的代理对象，代理对象和原数据类型具有相同的接口，你可以像使用原始数据一样使用代理对象。例如：
>
> ```js
>import { observable } from "mobx";
> 
> const object = observable({a: 1})
> console.log(object.a)
> 
> const array = observable([1, 2])
> console.log(array[0])
> 
> const map = observable(new Map({a: 1}))
> console.log(map.get('a'))
> 
> const set = observable(new Set([1, 2]))
> console.log(set.has(1))
> ```
> 
> `observable`观察的数据可以嵌套，嵌套的数据也会被观察。例如：
>
> ```js
>import { observable, autorun } from "mobx";
> 
> const store = observable({
>   a: {
>     b: [1, 2]
>   }
> })
> 
> autorun(() => console.log(store.a.b[0]))
> // 1
> 
> store.a.b[0] += 1
> // 2
> ```
> 
> `observable`支持动态添加可观察状态。例如：
>
> ```js
>import { observable, autorun } from "mobx";
> 
> const store = observable({});
> 
> autorun(() => {
>   console.log("a =", store.a);
> });
> // a = undefined
> 
> store.a = 1;
> // a = 1
> ```
> 
> > 动态添加可观察状态，仅适用于 MobX v5+ 版本，MobX v4 及以下版本需要借助辅助函数，详见[Direct Observable manipulation](https://mobx.js.org/refguide/object-api.html)。
>
> ### `autorun`
>`autorun`用于定义响应函数，其类型定义如下（已简化）：
> 
> ```ts
>autorun(reaction: () => any): IReactionDisposer;
> ```
> 
> `autorun`接受一个响应函数 reaction，并在定义时立即执行一次 reaction 函数， reaction 函数内部可以执行带有副作用的操作。 以后，每当依赖状态发生变化时，`autorun`自动重新运行 reaction 函数。`autorun`第一次运行 reaction 函数是为了搜集依赖状态——运行 reaction 过程中实际使用的状态（通过`obj.name`或`obj['name']`解引用方式使用的状态）。
>
> 例如下面例子，`autorun`中使用了状态`a`，因此当状态`a`的值发生变化时，会执行响应函数。而状态`b`虽然被列为可观察状态，但由于在`autorun`中没有被实际使用，因此当状态`b`的值发生变化时，不会执行 响应函数 。这是 MobX 的“聪明”之处，它能根据状态的实际使用情况，细粒度地控制更新范围。由于减少了不必要的执行开销，从而提升了程序性能。
>
> ```js
>import { observable, autorun } from "mobx";
> 
> const store = observable({
>   a: 1,
>   b: 2
> });
> 
> autorun(() => {
>   console.log("a =", store.a);
> });
> // a = 1
> 
> store.a += 1;
> // a = 2
> store.b += 1;
> // (无输出)
> ```
> 
> 让我们回顾一下 MobX 的使用，通过`observable`定义状态，在`autorun`中使用状态，当`autorun`中使用到的状态发生变化时，该`autorun`重新执行。绝大部分情况下它都工作的很好，如果你遇到修改了状态而`autorun`没有如你预期的那样运行，这时候你需要深入了解 MobX 是如何响应状态变化的，推荐阅读[What does MobX react to?](https://mobx.js.org/best/react.html)。
>
> ### `computed`
>`computed`用于定义计算值，其类型定义如下（已简化）：
> 
> ```ts
><T>(func: () => T) => { get(): T, set(value: T): void}
> ```
> 
> `computed`与`autorun`相似，他们都会在依赖的状态发生变化时会重新运行，不同之处是`computed`接收的是**纯函数**并且返回一个计算值，这个计算值在状态变化时会自动更新，计算值可以在`autorun`中使用。
>
> 例如下面例子中，`ca`是一个计算值，它依赖状态`a`的值，当状态`a`的值发生变化时，`ca`会重新计算值。计算值`ca`是一个“装箱”对象，需要通过`get/set`访问内部值，只有这样才能保持计算值的引用不变而内部值又是可变的。
>
> ```js
>import { observable, autorun, computed } from "mobx";
> 
> const store = observable({
>   a: 1
> });
> 
> const ca = computed(() => {
>   return 10 * store.a;
> });
> 
> autorun(() => {
>   console.log(`${store.a} * 10 = ${ca.get()}`);
> });
> // 1 * 10 = 10
> 
> 
> store.a += 1;
> // 2 * 10 = 20
> store.a += 1;
> // 3 * 10 = 30
> ```
> 
> 由于`computed`被视作是纯函数，MobX 提供了许多开箱即用的优化措施，例如对计算值的缓存和惰性计算。
>
> **`computed`值会被缓存**
>
> 每当读取`computed`值时，如果其依赖的状态或其他`computed`值未发生变化，则使用上次的缓存结果，以减少计算开销，对于复杂的`computed`值，缓存可以大大提高性能。
>
> 例如下面例子中，`computed`值`ca`和`cb`分别依赖状态`a`和`b`，第一次执行`autorun`时，`ca`和`cb`都会重新计算，然后修改状态`a`的值，第二次执行`autorun`时，只有`ca`会重新计算，而`cb`则使用上次的缓存结果。
>
> ```js
>import { observable, autorun, computed } from "mobx";
> 
> const store = observable({
>   a: 1,
>   b: 1
> });
> 
> const ca = computed(() => {
>   console.log("recomputed ca");
>   return 10 * store.a;
> });
> 
> const cb = computed(() => {
>   console.log("recomputed cb");
>   return 10 * store.b;
> });
> 
> autorun(() => {
>   console.log(
>     `a = ${store.a}, ca = ${ca.get()}, b = ${store.b}, cb = ${cb.get()}`
>   );
> });
> // recomputed ca
> // recomputed cb
> // a = 1, ca = 10, b = 1, cb = 10
> 
> store.a += 1;
> // recomputed ca
> // a = 2, ca = 20, b = 1, cb = 10
> ```
> 
> > 为了观察`computed`的计算过程，插入了打印日志的语句，这会带有副作用，实际中不要这样做
>
> **`computed`值会惰性计算**
>
> 只有`computed`值被使用时才重新计算值。反言之，即使`computed`值依赖的状态发生了变化，但是它暂时没有被使用，那么它不会重新计算。
>
> 例如下面例子中，`computed`值`ca`依赖状态`a`。当状态`a`的值小于 3 时，`autorun`运行时只打印状态`a`的值，由于`computed`值`ca`未被使用，所以`ca`不会从新计算。当状态`a`的值增长到 3 以后，`autorun`运行时同时打印状态`a`和`computed`值`ca`，由于`computed`值`ca`被使用了，所以`ca`会重新计算。
>
> ```js
>import { observable, autorun, computed } from "mobx";
> 
> const store = observable({
>   a: 1
> });
> 
> const ca = computed(() => {
>   console.log("recomputed ca");
>   return 10 * store.a;
> });
> 
> autorun(() => {
>   if (store.a >= 3) {
>     console.log(`a = ${store.a}, ca = ${ca.get()}`);
>   } else {
>     console.log(`a = ${store.a}`);
>   }
> });
> // a = 1
> 
> store.a += 1;
> // a = 2
> 
> store.a += 1;
> // recomputed ca
> // a = 3, ca = 30
> ```
> 
> ### `action`
>`action`用于定义状态修改操作，其类型定义如下（已简化）：
> 
> ```ts
><T extends Function>(fn: T) => T
> ```
> 
> 虽然没有`action`也可以直接修改状态，但是通过`action`显式地修改状态，使得状态的变化可预测（状态的变化能定位到是哪个`action`引起的）。此外，`action`函数是事务型的，通过`action`修改状态时，响应函数不会立即执行，而是等到`action`结束后才执行，这有助于提升性能。
>
> 例如下面例子中，展示了修改状态的两种方式，一种是直接修改状态`a`，另一种是通过调用预先定义的`action`修改状态。值得注意的是，在一次`action`中连续两次修改状态`a`的值，只会触发一次`autorun`的执行。
>
> ```js
>import { observable, autorun, action } from "mobx";
> 
> const store = observable({
>   a: 1
> });
> 
> autorun(() => {
>   console.log(`a = ${store.a}`);
> });
> // a = 1
> 
> store.a += 1;
> // a = 2
> store.a += 1;
> // a = 3
> 
> const increaseA = action(() => {
>   store.a += 1;
>   store.a += 1;
> });
> increaseA();
> // a = 5
> ```
> 
> 对 MobX 进行一些配置后，可以使`action`成为修改状态的唯一方式，这可以避免不受约束的修改状态行为发生，有利于提升项目的可维护性。
>
> 例如下面例子中，配置 [ enforceActions](https://mobx.js.org/refguide/api.html#configure) 为`"always"`后，就只能通过`action`修改状态了，如果尝试直接修改状态将会触发异常。
>
> ```js
>import { observable, autorun, action,  configure } from "mobx";
> 
> // 强制只能通过 action 修改状态
> configure({
>   enforceActions: "always"
> });
> 
> const store = observable({
>   a: 1
> });
> 
> autorun(() => {
>   console.log(`a = ${store.a}`);
> });
> // a = 1
> 
> // store.a += 1;
> // 直接修改状态，将会抛出如下异常
> // Error: [mobx] Since strict-mode is enabled, changing observed 
> // observable values outside actions is not allowed. 
> // Please wrap the code in an `action` if this change is intended.
> 
> const increaseA = action(() => {
>   store.a += 1;
> });
> increaseA();
> // a = 2
> ```
> 
> ## MobX 与 React 集成
>MobX 是框架无关的，你可以单独使用它，也可以与任何流行的 UI 框架进行一起使用，MobX 官方提供了 React/Vue/Angular 等流行框架的[绑定实现](https://github.com/mobxjs)。MobX 最常见的是与 React 一起使用，他们的绑定实现是 [mobx-react](https://github.com/mobxjs/mobx-react)（或 [mobx-react-lite](https://github.com/mobxjs/mobx-react-lite)）。
> 
> mobx-react 提供了一个`observer`方法， 它是一个高阶组件，它接收 React 组件并返回一个新的 React 组件，返回的新组件能响应（通过`observable`定义的）状态的变化，即组件能在可观察状态变化时自动更新。`observer`方法是对 MobX 提供的`autorun`方法和 React 组件更新机制的封装，以便于在 React 中使用，你依然可以在 React 中使用`autorun`来更新组件。下面是`observer`方法的类型声明，它支持组件类和函数组件。
>
> ```ts
>function observer<T extends React.ComponentClass | React.FunctionComponent>(target: T): T
> ```
> 
> 下面是组件类使用 MobX 的示例，通过`observable`定义可观察状态，并通过`observer`包裹组件，之后组件事件处理方法中修改状态后，组件会自动更新，无需手动调用 React 的`setState()`来更新组件。
>
> ```js
>import React from "react";
> import { observable } from "mobx";
> import { observer } from "mobx-react";
> 
> class Counter extends React.Component {
>   constructor(props) {
>     super(props);
>     this.store = observable({
>       count: 0
>     });
>   }
> 
>   render() {
>     return (
>       <button onClick={() => this.store.count++}>
>         {this.store.count}
>       </button>
>     )
>   }
> }
> 
> export default observer(Counter);
> ```
> 
> 下面是函数组件使用 MobX 的示例，与上面类组件类似，区别是使用 mobx-react 提供的`useLocalStore`定义客观察状态，`useLocalStore`内部也是使用`observable`定义可观察状态。
>
> ```js
>import React, { useMemo } from "react";
> import { observable } from "mobx";
> import { observer,  useLocalStore } from "mobx-react";
> 
> const Counter = () => {
>   const store = useLocalStore(() => ({
>     count: 0
>   }));
>   // 等价于下面
>   // const store = useMemo(() => observable({ count: 0 }), []);
>   return (
>     <button onClick={() => store.count++}>
>       {store.count}
>     </button>
>   )
> };
> 
> export default observer(Counter);
> ```
> 
> ## 小结
>MobX 的设计哲学是“可从应用状态中派生的任何内容都应当自动的被派生”，后半句有两个关键字：**自动**和**派生**。心中秉持这一设计哲学，再来看 MobX 的派生状态模型就比较清晰了，Computed values 和 Reactions 都可以视作是从 State 中派生出的，State 变化时触发 Computed values 的重新计算和 Reations 的重新运行。为了让派生能自动的进行，MobX 通过`Object.definePropery`或`Proxy`方式拦截对象的读写操作，从而允许用户以自然的方式来修改状态，MobX 负责更新派生的内容。
> 
> MobX 提供了几个核心 API 来帮助定义状态（`observable`）、响应状态（`autorun`, `computed`）和修改状态（`action`），通过这些 API 可以让程序立即具备响应式能力。这些 API 接口并不复杂，但要熟练使用，需要深入理解 MobX 响应机制，文中通过一些简单的示例来辅助理解这些 API 的行为。
>
> MobX 可以单独使用，也可以与任何流行的 UI 框架一起使用，Github 上可以找到 MobX 与流行框架的绑定实现。不过 MobX 最常见的是与 React 一起使用，mobx-react 是流行的 MobX 和 React 的绑定实现库，本文介绍了它在组件类和函数组件上的一些基本用法。
>
> ## 参考
>* [MobX Documentations](https://mobx.js.org/)
> * [mobx-react](https://github.com/mobxjs/mobx-react)