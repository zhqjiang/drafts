# Blogged Answers: 对 React Hooks, Redux 和注意点分离的思考

发布于 2019 年 7 月 10 日

这篇文章属于 [Blogged Answers](https://blog.isquaredsoftware.com/series/blogged-answers) 系列

关于使用 React Hooks 所涉及的“权衡利弊”的一些看法

## 介绍

最近我看到很多关于 React Hooks 使用的问题，特别是 React-Redux hooks。最常见的问题是这样的：

- 我如何测试依赖外部数据源的组件，比如 Redux store？
- 使用 hooks 和 context 不会减少关注点分离吗？
- 在组件中内联编写一堆 hooks 和回调产生的“噪音”怎么处理？

实际上，我对 Hooks 带来的"权衡利弊"有很多想法。今天早些时候，[我在 Twitter 上写了一个很长的 thread](https://twitter.com/acemarke/status/1149000836200181760)，其中有一些想法，我想在这里扩展一下这个 thread。

在我进一步阐述之前，有一些注意事项。我已经写了一些使用 Hooks 的小例子，但我还没有在真实世界的应用中认真使用它们。特别是我们新的 [React-Redux Hooks API](https://react-redux.js.org/api/hooks) 也是如此--我推动了设计和发布这些 API 的工作，但我自己只简单地在实际中使用过它们。我也不是测试方面的专家。我知道很多理论，只是我自己没有实际写过那么多测试（单元或集成）。所以，我想说的是，这些只是一些观点和观察，我很可能是错的，你应该对事情做出自己的判断。

## Hooks、HOC（High Order Components） 和权衡利弊

自从 Hooks 被宣布后，React 社区就为其疯狂了。虽然有很多合理的怀疑，但社区中的大部分人都对 Hooks 跃跃欲试，把它们吹捧为未来编写 React 代码的 "理所应当的方式"。通常来说，应该用一些更现实的期望来缓和这种突变和激动。

从某种意义上说，Hooks 并没有给你带来什么新东西。class 组件一直都有状态和副作用。Hooks 只是让你在函数组件中做同样的事情。从这个意义上说，什么都没有改变。

同时，Hooks 又极大地改变了一些事情。原本分割在多个生命周期中的逻辑现在被集中在一起。Hooks 需要使用和了解闭包，而不是`this`。同样的结果，但写法完全不同。特别是，使用闭包的结果是组件要大得多，因为你必须写内联函数来捕获作用域范围内的变量。

与任何技术一样，这也是一种权衡利弊。集中在一起通常是好的，但复杂的组件可能会非常长。有些可以通过提取自定义 Hooks 来缓解。(我想你可以用内联定义函数来做类似的事情--有一些像 `makeClickHandler(a,b,c)`这样的工厂函数来把实际的代码移到组件之外，但这开始变得有点愚蠢和过度抽象了。)

正因为如此，Hooks 在 "关注点分离 "和 "简单性 "方面有意做出了与 HOC 不同的权衡利弊。

- HOCs 提倡编写普通组件以 props 接收所有数据，保持它们的通用性、解耦性和潜在的可重用性。HOCs 的缺点是众所周知的：潜在的名称冲突、组件树和 DevTools 中的额外层次、极其复杂的静态类型和边缘情况，如 Ref 访问。

- Hooks 缩小了组件树，促进了作为普通函数的逻辑提取，更容易静态类型化，并且更容易组合。但是，Hooks 的使用确实导致了与依赖关系更强的耦合--不仅仅是对 React-Redux 的使用，而是对 context 的任何使用。

在这个意义上，Hooks 故意远离了 "关注点分离"。一个组件现在明确地期望从某个地方读取它自己的数据并渲染它。你可以为获取数据和渲染编写单独的组件，但是也意味着你已经重新发明了 HOC。

## React-Redux Hooks 的使用模式

Redux 团队一直提倡 "保持组件对 Redux 无意识"。

> 它们应该像其他 React 组件一样，简单地接收数据和函数作为 props。这最终使得测试和重用你自己的组件更加容易。

现在，我们的新 Hooks API 并没有彻底改变什么。我们一直鼓励使用选择器函数（selector 函数）从存储状态中提取值--`useSelector()`只是把它正式化了（你甚至可以用`useSelector()`重用你现有的`mapState`函数，前提是你要么把结果暂存（memoize）下来，要么用`shallowEqual`作为比较函数）。

同时，你一直可以只使用 `connect()(MyComponent)`，在组件中做一些异步逻辑，然后显式使用 `this.props.dispatch(someAction)`。然而，[我们不鼓励这样做，理由是它使异步代码的可重用性降低](https://redux.js.org/faq/actions#how-can-i-represent-side-effects-such-as-ajax-calls-why-do-we-need-things-like-action-creators-thunks-and-middleware-to-do-async-behavior)：

> 一般来说，Redux 建议带有副作用的代码应该是 action 创建过程的一部分。虽然该逻辑可以在 UI 组件内执行，但一般来说，将该逻辑提取到一个可重用的函数中是有意义的，这样可以从多个地方调用相同的逻辑(同一个 action creator function)。

这也是[action creators、thunks 和 mapDispatch 存在](https://blog.isquaredsoftware.com/2016/10/idiomatic-redux-why-use-action-creators/)的部分原因：在调用现场没有明确提到 `dispatch` 的情况下实现 dispatch，并实现通用的展示型组件。

不过，在我们的 React-Redux Hooks 中，你只有 `useDispatch()`。所以，你总是有直接可用的东西。人们可能会放弃 thunks(redux-thunk)，在 `useEffect()`中编写异步逻辑，并直接在组件中进行`dispatch`。通过提取`Custom hooks`，它还可以使 React 和 Redux 以一些新的方式混合在一起，可以参考这个[从 connect()转换到 hooks 的例子](https://blog.logrocket.com/how-to-convert-your-existing-redux-containers-to-hooks/)。

(题外话： "容器型/展示型 "一直被社区过度解读。Dan(译者注：[原 Redux 开发者，现 React 核心团队开发人员](https://twitter.com/dan_abramov))后来[很大程度上否认了他的原帖](./Presentational_and_Container_Components.md)，当我们[修改 Redux 文档](https://github.com/reduxjs/redux/issues/3313)时，我们会重写 ["React 使用 "教程](https://redux.js.org/basics/usage-with-react)页面，不再强调这个概念。)

所以是的，我们的 React-Redux Hooks 肯定会将组件与 Redux 更紧密地结合在一起，因为对于任何使用 context 而不是 props 的组件来说都是这样的。组件会对上下文实例和所提供的数据有一种隐含的依赖性。

## 结论

就像大多数编程一样：不是说这些方法中的任何一个是对的或错的。而是这些方法有不同的权衡，每个团队都必须做出自己的决定，哪些权衡对他们更重要。

你想在多大程度上分离关注点？如果你喜欢"shallow"测试组件(译者注：比如 Enzyme 的 shallow)，Hooks 可能不适合你，因为它们真的需要“集成”类型的测试。你是喜欢更干净的组件树、集中和更容易的静态类型，还是更多的关注点分离和间接性？

从长远来看，看到这个问题在生态系统中的影响将是非常有趣的。Hooks 只出现了几个月，所以社区仍在努力解决如何最好地利用它们的问题。我们花了很多年才从 "mixins "到 "HOCs"，然后发现"render props"是什么东西。我们将在很长一段时间内探索 Hooks 的使用模式。

另外一个想法：我经常观察到，[React 和 Redux 的用户可以大致分为两种观点。"以应用为中心 "与 "以组件为中心 "的设计](https://twitter.com/acemarke/status/1056669495354421249)。我认为围绕 Hooks 的权衡问题与这些思维方式的一些方面有关。React 是你真正的"应用"吗？或者它只是 UI 层，而真正的应用是保存在组件树之外的逻辑和数据？这两种观点都是很有道理的，同样有不同的权衡。

## 更多

也请看我在 ReactBoston 2019 的演讲 ["Hooks, HOCs, and Tradeoffs"](https://blog.isquaredsoftware.com/2019/09/presentation-hooks-hocs-tradeoffs/)，其中进一步讨论了这个话题。
