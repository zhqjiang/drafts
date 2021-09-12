# 展示型和容器型组件

[原文](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

Dan Abramov

2015 年 3 月 23 日 - 阅读时间大约 5 分钟

> 2019 年的更新：我在很久以前写了这篇文章，后来我的观点发生了变化。尤其是，我不再建议像这样拆分你的组件了。如果你发现在你的代码库中这种方式很自然，这种模式可以很方便。但是我很多次发现这种模式被没有任何必要的强制实行，并且带有教条主义的狂热。我发现它有用的主要原因是它让我把复杂的有状态逻辑与组件的其他方面分开。自定义 Hooks 让我做同样的事情，不需要主观的划分。出于历史原因，这篇文章被保留下来，但不要太认真看待它。

在编写 React 应用程序时，我发现有一种简单的模式非常有用。如果你已经写了一段时间的 React，你可能已经发现了它。[这篇文章](https://medium.com/@learnreact/container-components-c0e67432e005)很好地解释了它，但我想再补充几点。

如果你把组件分为两类，你会发现你的组件更容易被重用和理解。我称它们为容器型组件和展示型组件，但我也听说过胖子/瘦子、智慧/愚笨、有状态组件/纯组件、荧幕/组件等等。这些术语意思不完全相同，但核心思想是相似的。

我所认为的**展示型**组件。

- 关注事物的外观。

- 可能同时包含展示组件和容器组件，并且通常有一些 DOM 元素和它们自己的样式。

- 通常允许包含 this.props.children 。

- 对应用程序的其他部分没有依赖性，例如 Flux Actions 或 Store。(译者注：Flux 是一种应用架构, facebook 有官方实现，但是更常用的是它的替代品 Redux 和 Mobx)

- 不指定数据是如何加载或变更的。

- 完全通过 props 接收数据和回调函数。

- 很少有自己的状态（如果有，也是 UI 的状态而不是数据）。

- 除非它们需要状态、生命周期钩子或性能优化，否则将被写成函数型组件（Functional Components）。

- 例子: Page, Sidebar, Story, UserInfo, List.

我所认为的**容器型**组件。

- 关注事物的运作方式。

- 可能同时包含展示性组件和容器组件，但除了一些包裹性的 div 之外，通常没有任何自己的 DOM 元素，也没有任何样式。

- 为展示型或其他容器组件提供数据和行为。

- 调用 Flux actions，并将其作为回调函数提供给展示型组件。
- 通常是有状态的，因为它们往往作为数据源。

- 通常使用高阶组件生成，如 React Redux 的 connect()，Relay 的 createContainer()，或 Flux Utils 的 Container.create()，而不是手动编写。

- 例子：UserPage, FollowersSidebar, StoryContainer, FollowedUserList.

## 这种模式的益处

- 更好地分离关注点。通过这种方式编写组件，你可以更好地理解你的应用程序和你的用户界面。

- 更好的重用性。你可以在完全不同的状态源中使用相同的展示型组件，并将这些组件变成可以进一步重用的独立容器组件。

- 展示型式组件本质上是你的应用程序的 "调色板"。你可以把它们放在一个页面上，让设计者调整它们的所有变化，而不需要接触应用程序的逻辑。你可以在该页面上运行截图回归测试。

- 这强迫你提取 "布局组件"，如 Sidebar、Page、ContextMenu，并使用 this.props.children 而不是在几个容器组件中重复相同的标记和布局。

**请记住，组件并不一定需要返回 DOM**。它们只需要在 UI 关注点之间提供组合边界。

利用好这个优势。

## 何时引入容器型组件？

我建议你在构建你的应用程序时，首先只使用展示性组件。最终你会意识到，你在中间组件中传递了太多的 props。当你注意到有些组件并不使用它们收到的 props，而只是将它们转发下去，并且当子组件们需要更多的数据时，你不得不重写那些中间组件时，这就是引入一些容器型组件的好时机。这样，你就可以把数据和行为 props 送到子组件上，而不会给树中间的无关组件带来负担。

这是一个不断重构的过程，所以不要试图在第一次就把它做好。当你尝试使用这种模式时，你会发展出一种直觉，知道什么时候该提取一些容器，就像你知道什么时候该提取一个函数一样。我的[Redux Egghead 系列](https://egghead.io/series/getting-started-with-redux)可能也会对你有所帮助!

## 其他二分法

重要的是你要明白，展示型组件和容器型组件之间的区别不是技术上的区别。相反，他们的区别在于目的上的区别。

相比之下，这里有几个相关（但不同！）的技术上的区别。

- 有状态和无状态。有些组件使用 React setState()方法，有些不使用。虽然容器组件倾向于有状态，而展示组件倾向于无状态，但这并不是一个硬性规定。展示型组件可以是有状态的，而容器型组件也可以是无状态的。

- Classes 和 Functions。从 React 0.14 开始，组件可以被声明为 Classes 和 Functions。函数组件能被更简单的定义，但它们缺乏目前只有 Class 组件才有的某些功能。其中一些限制可能会在未来消失，但它们今天仍然存在。因为函数组件更容易理解，我建议你使用它们，除非你需要状态、生命周期钩子或性能优化，这些功能目前只有类组件才有。（译者注：React Hooks 引入后，函数组件的功能已经覆盖绝大部分 class 组件的功能）

- 纯粹和不纯粹。人们说，如果一个组件能保证在相同的 props 和 states 下返回相同的结果，那么它就是纯的。纯组件既可以定义为类，也可以定义为函数，既可以有状态，也可以无状态。纯组件的另一个重要方面是，它们不依赖于 props 或 state 的深度改变，所以它们的渲染性能可以通过 shouldComponentUpdate()钩子中的浅层比较来优化。目前只有类可以定义 shouldComponentUpdate()，但这在未来可能会改变。

展示型组件和容器型都可以应用到这些二分法中。根据我的经验，展示型组件往往是无状态的纯函数组件，而容器型组件往往是有状态的类（class）组件。然而这不是一个规则，而是一种观察，我见过完全相反的情况，在特定情况下是有意义的。

不要把展示型和容器组件的分离当作教条。有时它并不重要，或者很难划清界限。如果你对某个特定的组件应该是展示型的还是容器性的感到不确定，现在决定可能还为时过早。不要紧张！

## 例子

这个 Michael Chan 写的[片段](https://gist.github.com/chantastic/fc9e3853464dffdb1e3c)很好的诠释了这种模式。

## 延伸阅读

- [Getting Started with Redux](https://app.egghead.io/playlists/fundamentals-of-redux-course-from-dan-abramov-bd5cc867)
- [Mixins are Dead, Long Live Composition](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)
- [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005)
- [Atomic Web Design](https://bradfrost.com/blog/post/atomic-web-design/)
- [Building the Facebook News Feed with Relay](http://facebook.github.io/react/blog/2015/03/19/building-the-facebook-news-feed-with-relay.html)(注：这个链接已经打不开)

## 脚注

> - 在本文的早期版本中，我称它们为 "智能 "和 "哑巴 "组件，但这对展示型组件来说过于苛刻，而且最重要的是，没有真正解释它们在目的上的区别。我更喜欢新的术语，我希望你也能这样做!
> - 在这篇文章的早期版本中，我声称展示型组件只应该包含其他展示型组件。我不再这样认为了。一个组件是一个展示型组件还是一个容器型组件是它的实现细节。你应该能够在不修改任何调用的情况下用一个容器型组件来替换一个展示型组件。因此，展示型组件和容器型组件都可以很好地包含其他展示型或容器型组件。
