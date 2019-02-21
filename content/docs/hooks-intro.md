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

在React Conf 2018，Sophie Alpert 和 Dan Abramov 介绍了Hooks, 接下来 Ryan Florence 示范了如何使用它们重构一个应用. 你可以在这里看到这个视频：

<br>

<iframe width="650" height="366" src="//www.youtube.com/embed/dpw9EHDh2bM" frameborder="0" allowfullscreen></iframe>

## 没有破坏性改动 {#no-breaking-changes}

在我们继续之前，请记住以下几点关于Hooks：

* **完全可选。** 你无需重写任何已有代码就可以在一些组件中尝试Hooks。但是如果你不想，你不必现在就去学习或使用Hooks。
* **100%向后兼容。** Hooks不包含任何破坏性改动。
* **现在可用。** Hooks在v16.8.0中已实装。

**没有计划从React中移除classes。** 你可以在本页的[底部部分](#gradual-adoption-strategy)读到更多关于Hooks的渐进策略。

**Hooks不会影响你对React的理解。** 恰恰相反，Hooks提供更直接的API去使用你了解的React概念: props, state, context, refs, 和 lifecycle。我们将稍后展示，Hooks也提供了一种更强大的方式来组合他们。

**如果你只想开始学习Hooks，你可以[直接跳到下一页!](/docs/hooks-overview.html)** 你也可以继续阅读这一页来了解为什么我们要添加Hooks，以及我们将如何在不重写应用的情况下使用他们。

## 动机 {#motivation}

Hooks solve a wide variety of seemingly unconnected problems in React that we've encountered over five years of writing and maintaining tens of thousands of components. Whether you're learning React, use it daily, or even prefer a different library with a similar component model, you might recognize some of these problems.

### It's hard to reuse stateful logic between components {#its-hard-to-reuse-stateful-logic-between-components}

React doesn't offer a way to "attach" reusable behavior to a component (for example, connecting it to a store). If you've worked with React for a while, you may be familiar with patterns like [render props](/docs/render-props.html) and [higher-order components](/docs/higher-order-components.html) that try to solve this. But these patterns require you to restructure your components when you use them, which can be cumbersome and make code harder to follow. If you look at a typical React application in React DevTools, you will likely find a "wrapper hell" of components surrounded by layers of providers, consumers, higher-order components, render props, and other abstractions. While we could [filter them out in DevTools](https://github.com/facebook/react-devtools/pull/503), this points to a deeper underlying problem: React needs a better primitive for sharing stateful logic.

With Hooks, you can extract stateful logic from a component so it can be tested independently and reused. **Hooks allow you to reuse stateful logic without changing your component hierarchy.** This makes it easy to share Hooks among many components or with the community.

We'll discuss this more in [Building Your Own Hooks](/docs/hooks-custom.html).

### Complex components become hard to understand {#complex-components-become-hard-to-understand}

We've often had to maintain components that started out simple but grew into an unmanageable mess of stateful logic and side effects. Each lifecycle method often contains a mix of unrelated logic. For example, components might perform some data fetching in `componentDidMount` and `componentDidUpdate`. However, the same `componentDidMount` method might also contain some unrelated logic that sets up event listeners, with cleanup performed in `componentWillUnmount`. Mutually related code that changes together gets split apart, but completely unrelated code ends up combined in a single method. This makes it too easy to introduce bugs and inconsistencies.

In many cases it's not possible to break these components into smaller ones because the stateful logic is all over the place. It's also difficult to test them. This is one of the reasons many people prefer to combine React with a separate state management library. However, that often introduces too much abstraction, requires you to jump between different files, and makes reusing components more difficult.

To solve this, **Hooks let you split one component into smaller functions based on what pieces are related (such as setting up a subscription or fetching data)**, rather than forcing a split based on lifecycle methods. You may also opt into managing the component's local state with a reducer to make it more predictable.

We'll discuss this more in [Using the Effect Hook](/docs/hooks-effect.html#tip-use-multiple-effects-to-separate-concerns).

### Classes confuse both people and machines {#classes-confuse-both-people-and-machines}

In addition to making code reuse and code organization more difficult, we've found that classes can be a large barrier to learning React. You have to understand how `this` works in JavaScript, which is very different from how it works in most languages. You have to remember to bind the event handlers. Without unstable [syntax proposals](https://babeljs.io/docs/en/babel-plugin-transform-class-properties/), the code is very verbose. People can understand props, state, and top-down data flow perfectly well but still struggle with classes. The distinction between function and class components in React and when to use each one leads to disagreements even between experienced React developers.

Additionally, React has been out for about five years, and we want to make sure it stays relevant in the next five years. As [Svelte](https://svelte.technology/), [Angular](https://angular.io/), [Glimmer](https://glimmerjs.com/), and others show, [ahead-of-time compilation](https://en.wikipedia.org/wiki/Ahead-of-time_compilation) of components has a lot of future potential. Especially if it's not limited to templates. Recently, we've been experimenting with [component folding](https://github.com/facebook/react/issues/7323) using [Prepack](https://prepack.io/), and we've seen promising early results. However, we found that class components can encourage unintentional patterns that make these optimizations fall back to a slower path. Classes present issues for today's tools, too. For example, classes don't minify very well, and they make hot reloading flaky and unreliable. We want to present an API that makes it more likely for code to stay on the optimizable path.

To solve these problems, **Hooks let you use more of React's features without classes.** Conceptually, React components have always been closer to functions. Hooks embrace functions, but without sacrificing the practical spirit of React. Hooks provide access to imperative escape hatches and don't require you to learn complex functional or reactive programming techniques.

>示例
>
>[Hooks一瞥](/docs/hooks-overview.html) is a good place to start learning Hooks.

## 渐进策略 {#gradual-adoption-strategy}

>**太长不看: 没有计划从React中移除classes。**

We know that React developers are focused on shipping products and don't have time to look into every new API that's being released. Hooks are very new, and it might be better to wait for more examples and tutorials before considering learning or adopting them.

We also understand that the bar for adding a new primitive to React is extremely high. For curious readers, we have prepared a [detailed RFC](https://github.com/reactjs/rfcs/pull/68) that dives into motivation with more details, and provides extra perspective on the specific design decisions and related prior art.

**Crucially, Hooks work side-by-side with existing code so you can adopt them gradually.** We are sharing this experimental API to get early feedback from those in the community who are interested in shaping the future of React — and we will iterate on Hooks in the open.

Finally, there is no rush to migrate to Hooks. We recommend avoiding any "big rewrites", especially for existing, complex class components. It takes a bit of a mindshift to start "thinking in Hooks". In our experience, it's best to practice using Hooks in new and non-critical components first, and ensure that everybody on your team feels comfortable with them. After you give Hooks a try, please feel free to [send us feedback](https://github.com/facebook/react/issues/new), positive or negative.

We intend for Hooks to cover all existing use cases for classes, but **we will keep supporting class components for the foreseeable future.** At Facebook, we have tens of thousands of components written as classes, and we have absolutely no plans to rewrite them. Instead, we are starting to use Hooks in the new code side by side with classes.

## 常见问题 {#frequently-asked-questions}

我们准备了一个[Hooks 常见问题页面](/docs/hooks-faq.html)来解答最常见的关于Hooks的问题。

## 下一步 {#next-steps}

在这一页的最后，你应该对Hooks能解决什么问题有了粗略的理解，但可能还有许多细节不清楚。不要担心！**让我们去[下一页](/docs/hooks-overview.html)通过例子学习Hooks。**
