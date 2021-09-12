# 测试奖杯和测试分类

如何最清晰的解释测试奖杯？

请允许我沉浸在一点个人回忆中。如果你对测试奖杯不熟悉，请看这个图片：

![](./assets/trophy.jpg)

[我最初在一条推文中介绍了这一点，并附上了我用 Google Drive 制作的快速绘图。](https://twitter.com/kentcdodds/status/960723172591992832?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E960723172591992832%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fthe-testing-trophy-and-testing-classifications)

我是在发表了一篇题为[“写测试，不需要太多。主要写集成”](https://kentcdodds.com/blog/write-tests)的博文后想到这个“测试奖杯”的"。

这是我对 [Guillermo Rauch](https://twitter.com/rauchg) 大约一年前的推文的[看法](https://twitter.com/rauchg/status/807626710350839808?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E807626710350839808%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fthe-testing-trophy-and-testing-classifications)。
我不能替 Guillermo 发言，但我非常同意他所说的话，因为我作为一名 UI 工程师的经验，以及我个人是如何在这种情况下理解 "integration"(“集成”) 一词的。

在我职业生涯中，我写的几乎所有代码都是直接在浏览器中运行的，或者是为帮助我在浏览器中运行代码的工具。所以对我来说，"单元"、"集成"和 "端到端"这些术语自然会通过这种经验的视角来看待。事实上，我在奖杯中加入了"静态"，因为在 JavaScript 的世界中，并不像主流语言中介绍[“测试金字塔”](https://martinfowler.com/bliki/TestPyramid.html)一样，静态是已有的东西。（译者注：主流语言 C++, Java 等都有不同程度的对类型的支持）

我之所以先说明这个背景，是为了帮助你理解测试奖杯。我从来没有考虑过它是否适用于微服务甚至是后端服务。我认为我的代码库是孤立的，并试图将我可以在自己的代码库内编写的测试类型进行分类。 我一直认为端到端测试是在没有任何 mocking(更实际的情况中，尽可能少的 mocking)的情况下，试图验证功能是否正常的。

因此，我只能把我自己的代码的测试分为 "单元 "或 "集成"。我认为 "单元 "是指包含逻辑的单个函数、类或对象。这里，我决定（简单地）对它们进行分类。

- 单元测试是指测试那些没有依赖（合作者）的单元，或者测试那些依赖被 mock 的单元。

- 集成测试是那些测试多个单元相互集成的测试。

最终，我开发了 [Testing Library](https://testing-library.com/)，来加强那些对我来说最有效的测试实践。

https://twitter.com/kentcdodds/status/980799621080391680?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E980799621080391680%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fthe-testing-trophy-and-testing-classifications

根据我自己的定义，Testing Library 可以用来测试单个 React 组件（单元测试）。也可以测试整个页面, [通过 MSW](https://kentcdodds.com/blog/stop-mocking-fetch)（集成测试）mock HTTP 请求的。也可以用很少的 mocks 来测试整个应用程序（端到端测试）。甚至在必要时测试[单个 React hooks](https://kentcdodds.com/blog/how-to-test-custom-react-hooks)（较低级别的单元测试）。Testing Library 现在是最流行的、是事实上的标准......呃......用来测试 React 应用的测试库，而且 对于测试 DOM 来说，同样的事情也正在发生。2020 年 5 月，它在[ThoughtWorks 技术雷达上获得了 "Adopt "的殊荣。](https://www.thoughtworks.com/radar/languages-and-frameworks/react-testing-library)

我预计有人会对这篇博文回复。"为什么你一开始就要编造自己的定义？只要使用现有的那些就可以了。" 所以我在你问之前先回答："你希望我在两打不同的定义中选择哪个作为自己的定义？" 😂😭 。Martin Fowler 在他关于[testing shapes](https://martinfowler.com/articles/2021-test-shapes.html)的文章中， 近似地引用了一位 "测试专家 "在 20 世纪 90 年代被问及他们如何定义 "单元测试"时的答案：

> "在我培训课程的首个上午，我学习了单元测试的 24 种不同定义"。

这是一个可悲的状态，而且不幸的是，自 90 年代以来一直如此。事实就是这样。我必须选择对我有意义的东西，作为一个教育工作者，我必须选择对我所教的人最有意义的东西。从采取我的建议的人的反应来看，这个决定是不错的。

在讨论你是否能证明测试是有效的时候，[Tim Bray](https://twitter.com/timbray)（在他的文章[testing in the Twenties)](https://www.tbray.org/ongoing/When/202x/2021/05/15/Testing-in-2021)中），正确地说道：

> 我们不要自欺欺人地认为我们的软件测试原则组成了科学的知识。

我想说的是，这适用于关于测试的一切，而不仅仅是它是否有效（它可以是有效的）。任何试图对所有这些术语进行单一定义的努力都是徒劳的。我记得我在 Assert(JS)的演讲（我在那里发表了我的演讲 [Write Tests.Not too many.Mostly Integration.](https://www.youtube.com/watch?v=Fha2bVoC8SE&list=PLV5CVI1eNcJgNqzNwcs4UKrlJdhfDjshf)），我观察到每个演讲在测试的建议方面都有很大的不同。但我现在想想，我认为很多差异可以归因于我们对测试术语的定义，而不是我们如何努力去达到有信心。

[Justin Searls](https://twitter.com/searls)（他也在那年的 Assert(JS)会议上[发言](https://www.youtube.com/watch?v=Af4M8GMoxi4)）在推特上说得很好：

> [人们喜欢争论写哪种类型的测试的比例，但这是在分散注意力。几乎没有一个团队会写出具有表现力的测试，这些测试建立了清晰的边界，运行迅速而可靠，而且只有在有用的情况下才会失败。请关注这些东西。](https://twitter.com/searls/status/1393385209089990659?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E1393385209089990659%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fthe-testing-trophy-and-testing-classifications)

分类是很重要的，这样我们才能就这个问题进行对话。不幸的是，在进行富有成效的对话之前，你们几乎需要就如何定义这些术语达成共识。但最终这真的不重要。正如 Justin 所说，这是一种分散注意力的做法。特别是当这么多的代码库在危险边缘，而又没有一个自动化的方法来保证他们的修改可以安全部署的时候。

## 结论

总之，希望这有助于澄清一些事情。总结一下。当试图将测试奖杯应用于你的情况时，请在单个代码库的代码中考虑它。它绝对适用于后端，但我只考虑了单体，而不是微服务，甚至是 serverless 功能（我同意 [Tim 的观点](https://www.tbray.org/ongoing/When/202x/2021/05/15/Testing-in-2021)，如果可以的话，我们大多数人都应该编写单体）。

测试奖杯（当被理解时）让我（和无数其他的人）明确了测试工作的重点。它被正确解释时，帮助我牢记这一关键原则：

> [你的测试越是类似于你的软件的使用方式，它们就越能给你带来信心。](https://twitter.com/kentcdodds/status/977018512689455106?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E977018512689455106%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fthe-testing-trophy-and-testing-classifications)

这是测试 Testing Library ƒ 的指导原则，也是我对我面临的每一个测试问题的思考方式。

**记住**，这都是为了获得良好的投资回报，其中"回报"是"信心"，"投资"是"时间"。如果我们有无限的时间，那么尝试对事物进行分类就没有必要了，我们就可以永远写测试了！但我们没有，所以我希望能帮助你在决定把你的精力放在哪里。

P.S. 如果你想了解我对测试的更多想法，我的博客上有[很多关于这个主题的文章](https://kentcdodds.com/blog?q=test)。下面是我推荐你接下来阅读的几篇具体文章。

- Confidently Shipping Code: Why I care about testing.
- Static vs Unit vs Integration vs E2E Testing for Frontend Apps: What these mean, why they matter, and why they don't. ⭐️ This one has code examples you might find instructive if you'd like more concrete examples of how I think about these different classifications of tests.
- Testing Implementation Details: Testing implementation details is a recipe for disaster. Why is that? And what does it even mean?
- Avoid the Test User: How your UI code has only two users, but the wrong tests can add a third.
- Should I write a test or fix a bug: How to prioritize tests relative to everything else.
- How to know what to test: Practical advice to help you determine what to test.
