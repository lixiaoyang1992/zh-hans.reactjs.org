---
id: hooks-intro
title: Hooks 简介
permalink: docs/hooks-intro.html
next: hooks-overview.html
---

*Hooks*是React 16.8中的增加的特性。它使你在class之外使用state和其他React特性。

```js{4,5}
import React, { useState } from 'react';

function Example() {
  // 声明一个新的叫做“count”的state变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

这个叫做`useState`的新函数是我们学习的第一个“Hooks”，这个例子只是一个开场戏。如果你现在还一头雾水，不必担心。

**你可以在[下一页](/docs/hooks-overview.html)开始学习Hooks 。** 在这一页，我们将继续解释我们为什么在React中加入Hooks，以及它将如何帮助你写出更好的应用。

>注意
>
>React 16.8.0是第一个支持Hooks的版本。升级时请升级所有的package，包括React DOM。React Native将在下一个稳定版本中支持Hooks。

## 视频介绍 {#video-introduction}

在React Conf 2018，Sophie Alpert 和 Dan Abramov 介绍了Hooks, 接下来 Ryan Florence 演示了如何使用它们重构一个应用. 你可以在这里看到这个视频：

<br>

<iframe width="650" height="366" src="//www.youtube.com/embed/dpw9EHDh2bM" frameborder="0" allowfullscreen></iframe>

## 没有破坏性改动 {#no-breaking-changes}

在我们继续之前，关于Hooks请记住以下几点：

* **完全可选。** 你无需重写任何已有代码就可以在一些组件中尝试Hooks。但是如果你不想，你不必现在就去学习或使用Hooks。
* **100%向后兼容。** Hooks不包含任何破坏性改动。
* **现在可用。** Hooks在v16.8.0中已实装。

**没有计划从React中移除classes。** 你可以在本页的[底部部分](#gradual-adoption-strategy)读到更多关于Hooks的渐进策略。

**Hooks不会影响你对React的理解。** 恰恰相反，Hooks提供更直接的API去使用你已知的React概念: props, state, context, refs, 和 lifecycle。我们将稍后展示，Hooks也提供了一种更强大的方式来组合他们。

**如果你只想开始学习Hooks，你可以[直接跳到下一页!](/docs/hooks-overview.html)** 你也可以继续阅读这一页来了解为什么我们要添加Hooks，以及我们将如何在不重写应用的情况下使用他们。

## 动机 {#motivation}

Hooks解决了我们五年来编写和维护成千上万的组件时遇到的各种各样看起来不相关的问题。无论你正在学习React，或每天使用，或者更愿意尝试另一个和React有相似组件模型的框架，你也许对这些问题似曾相识。

### 在组件之间复用包含状态的逻辑很难 {#its-hard-to-reuse-stateful-logic-between-components}

React没有提供给一个组件“附上”可复用的行为的途径（例如，把组件连接到store）。如果你用过React一段时间，你也许会熟悉解决这个问题的设计模式，比如[render props](/docs/render-props.html)和[higher-order components](/docs/higher-order-components.html) 。但是这些模式需要你重新组织你的组件结构，这可能会很笨重，是你的代码难以理解。如果你在React DevTools中观察一个典型的React应用，你会发现一个由providers, consumers, 高阶组件, render props等其他抽象层组成的嵌套地狱。我们可以[在DevTools过滤掉它们](https://github.com/facebook/react-devtools/pull/503)，这指出了一个更深层次的问题：React需要更好的原函数以复用包含状态的逻辑。

你可以使用Hooks从组件中提取包含状态的逻辑，使得这些逻辑可以单独测试并复用。**Hooks使你在无需修改组件结构的情况下复用包含状态的逻辑。**这使得在众多组件间或社区内分享Hooks很便捷。

我们将在[自定义 Hooks](/docs/hooks-custom.html)展开更多讨论。

### 复杂的组件变得难以理解 {#complex-components-become-hard-to-understand}

我们经常维护一些组件，他们一开始很简单但是逐渐变得充满了状态逻辑和副作用。每一个生命周期常常包含一些不相关的逻辑。例如，组件常常在`componentDidMount`和`componentDidUpdate`中获取数据。但是，同一个`componentDidMount`方法可能也包含一些和之前无关的逻辑如设置事件监听，之后在`componentWillUnmount`中清除。强关联需要对照修改的代码被分离，而完全不相关的代码在一个方法中组合在了一起。很容易导致bug和不一致。

在很多情况下，由于包含状态的逻辑到处都是，所以不可能将这些组件拆分成更小的。也很难测试他们。这是很多人将React和单独状态管理库结合使用的原因。但是，那常常引入了太多的抽象，使你在不同的文件之间来回切换，使得复用变得更加困难。

为了解决这个问题， **Hooks使你将一个组件中相互关联的部分切分成更小的函数(比如设置一个订阅或获取数据)**，而不是强制按照生命周期划分。你还可以使用一个reducer来管理组件的本地状态，以使其更加可预测。

我们将在[使用 Effect Hook](/docs/hooks-effect.html#tip-use-multiple-effects-to-separate-concerns)展开更多讨论。

### 人类和机器都对Classes感到困惑 {#classes-confuse-both-people-and-machines}

除了使代码复用和管理更困难外，我们还发现classes是学习React的一大屏障。你必须去理解JavaScript和其他语言差异巨大的`this`的工作方式。你不得不记得绑定事件处理器。没有稳定的[语法提案](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/)，这些代码非常啰嗦。人们可以很好地理解props, state和自顶向下的数据流，但又难以掌握classes。即便在有经验的React开发者中，函数式和类式组件的差异和在什么情况下使用也会导致分歧。

另外，React已经发布五年了，我们希望它能在下一个五年保持相关。就像[Svelte](https://svelte.technology/), [Angular](https://angular.io/), [Glimmer](https://glimmerjs.com/)和其它的库展示的那样，组件的[预先编译](https://en.wikipedia.org/wiki/Ahead-of-time_compilation)有巨大的潜力。 Especially if it's not limited to templates. Recently, we've been experimenting with [component folding](https://github.com/facebook/react/issues/7323) using [Prepack](https://prepack.io/), and we've seen promising early results. However, we found that class components can encourage unintentional patterns that make these optimizations fall back to a slower path. Classes present issues for today's tools, too. For example, classes don't minify very well, and they make hot reloading flaky and unreliable. We want to present an API that makes it more likely for code to stay on the optimizable path.

To solve these problems, **Hooks let you use more of React's features without classes.** Conceptually, React components have always been closer to functions. Hooks embrace functions, but without sacrificing the practical spirit of React. Hooks provide access to imperative escape hatches and don't require you to learn complex functional or reactive programming techniques.

>示例
>
>[Hooks一瞥](/docs/hooks-overview.html) is a good place to start learning Hooks.

## 渐进策略 {#gradual-adoption-strategy}

>**太长不看: 没有计划从React中移除classes。**

We know that React developers are focused on shipping products and don't have time to look into every new API that's being released. Hooks are very new, and it might be better to wait for more examples and tutorials before considering learning or adopting them.

We also understand that the bar for adding a new primitive to React is extremely high. For curious readers, we have prepared a [detailed RFC](https://github.com/reactjs/rfcs/pull/68) that dives into motivation with more details, and provides extra perspective on the specific design decisions and related prior art.

**最重要的是，Hooks和现有代码可以同时工作，你可以逐渐的采用他们。** We are sharing this experimental API to get early feedback from those in the community who are interested in shaping the future of React — and we will iterate on Hooks in the open.

Finally, there is no rush to migrate to Hooks. We recommend avoiding any "big rewrites", especially for existing, complex class components. It takes a bit of a mindshift to start "thinking in Hooks". In our experience, it's best to practice using Hooks in new and non-critical components first, and ensure that everybody on your team feels comfortable with them. After you give Hooks a try, please feel free to [send us feedback](https://github.com/facebook/react/issues/new), positive or negative.

We intend for Hooks to cover all existing use cases for classes, but **we will keep supporting class components for the foreseeable future.** At Facebook, we have tens of thousands of components written as classes, and we have absolutely no plans to rewrite them. Instead, we are starting to use Hooks in the new code side by side with classes.

## 常见问题 {#frequently-asked-questions}

我们准备了一个[Hooks 常见问题页面](/docs/hooks-faq.html)来解答最常见的关于Hooks的问题。

## 下一步 {#next-steps}

在这一页的最后，你应该对Hooks能解决什么问题有了粗略的理解，但可能还有许多细节不清楚。不要担心！**让我们去[下一页](/docs/hooks-overview.html)通过例子学习Hooks。**
