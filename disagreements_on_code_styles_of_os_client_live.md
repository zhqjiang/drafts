# 对目前项目 os-client-live 代码风格的异议

## 阅读本文之前

建议先阅读这几篇翻译的文章：

1. [展示型组件和容器型组件](./Presentational_and_Container_Components.md) - 作者是 Dan Abramov, Redux 的作者，现在是 React.js 核心团成员

原文：[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

2. [对 React Hooks, Redux 和关注点分离的思考](./Thoughts_on_React_Hooks_Redux_and_Separation_of_Concerns.md) - 作者是 Mark Erikson, Redux 现在的维护者

原文：[Blogged Answers: Thoughts on React Hooks, Redux, and Separation of Concerns](https://blog.isquaredsoftware.com/2019/07/blogged-answers-thoughts-on-hooks/)

3. [测试“奖杯”和测试分级](./The_Testing_trophy_and_Testing_Classifications.md) - 作者是 Kent C. Dodds，Testing Library 的作者

原文：[The Testing Trophy and Testing Classifications](https://kentcdodds.com/blog/the-testing-trophy-and-testing-classifications)

4. [前端应用的“静态-单元-集成-端到端测试”](./static_unit_integration_e2e_tests.md) - 作者和上篇文章的作者相同

原文：[Static vs Unit vs Integration vs E2E Testing for Frontend Apps](https://kentcdodds.com/blog/static-vs-unit-vs-integration-vs-e2e-tests)

## 一、 容器型/展示型组件在 hooks 时代是否仍有必要?

#### 来自开源社区有很大影响力开发者的判断，前两篇文章的作者都提到，“容器型/展示型”在 hooks 时代没有必要，Dan Abramov 2015 年写了开头的第一篇文章推荐“容器型/展示型”模式，但是在 2019 年说他认为自定义 Hooks 同样可以把复杂的有状态逻辑分离，所以不再推荐“容器型/展示型”。他曾在推特回复[React 团队从未建议将 "展示型 "和 "容器型 "组件分开。(这甚至不是 React 的术语。)但如果你愿意，你仍然可以遵守它。事实上，自定义 Hooks 使得从组件中提取逻辑更容易](https://twitter.com/dan_abramov/status/1056158199877943297)。Redux 库现在的维护者 Mark Erikson 也多次在推特提到不推荐“”容器型/展示型”。曾说[人们总是对 "容器型/展示型"的概念进行过度解释。它当然是有用的，但人们认为必须作为规则来严格遵守它](https://twitter.com/acemarke/status/1320922322207936513)。

#### 即使这两个作者这样说，也先得思考一下是否“容器型/展示型”和 hooks 函数式组件真的无法和谐共存。当然不是，但如果教条般遵守“容器型/展示型”模式，那么就会有多余的组件层次和冗余代码。

我们首先定义一下什么是教条般遵守的“容器型/展示型”模式

- 1. 展示型组件不能有任何状态（甚至是 UI 状态）或者内联函数

- 2. 在 jsx 中不能包含简单的逻辑表达式

- 3. 展示型组件的子组件只能是展示型组件

如果全部遵守这三种教条的准则，那么重构代码就会尝试把状态，状态处理逻辑提升到很高层次的容器组件中，结果就是这个容器型组件中有很多子孙组件的状态处理函数甚至是渲染逻辑。props 和函数需要传递多层。

如果能摒弃掉第三条，也即能接受“展示组件可以有容器型组件作为子组件，那么当把已有的一层一层组件拆为“容器型/展示型”时，要么选择不断提升状态和处理函数（即和上一段中一样），要么选择加深组件树的层次。比如把层级 App -> B -> C 变成 App -> B_container -> B_presentational -> C_container -> C_Presentational

如果再摒弃掉前两条，那么从结果上来说，这种“展示型/容器型”的分法是和正常组件编写没有太多区别的。

从项目中的代码来看，一个容器组件出现多个子孙组件的状态和处理函数这种情况是最多的。

## 二、代码可维护性和冗余

为了“展示型/容器型”的分法将子孙组件的状态和处理函数不断提升到容器组件中，那么 props 多层传值会影响代码可维护性。比如说如果想重构一个子组件，那么要同时修改这个自组件自身（展示组件） 以及往上一直查询到包含它状态和处理函数的容器组件，从一些状态和处理函数中找到它独有的进行修改。可维护性并不理想。

过于追求“展示型/容器型”，会导致代码的冗余，

我们来看一下菜单题的 basic 模版的代码

```ts
// index.tsx
const handleValueChange = useValueChange(
  node,
  handler.handleMenuClick,
  setBindValue,
  setShowValue
);
const renderLine = useRenderLine(
  theme.primary,
  theme.optFontWeight,
  theme.contrast,
  needIcon,
  getColor,
  handleValueChange
);
// useRnderLine.tsx
const useRenderLine = (
  primary: string,
  optFontWeight: FONT_WEIGHT,
  contrast: string,
  needIcon: (option: CFOption) => boolean,
  getColor: (option: CFOption) => string,
  handleValueChange: (val: string) => void
) =>
  useCallback(
    (option: CFOption) => (
      <SelectLine
        primary={primary}
        contrast={contrast}
        value={option.text}
        key={option.text}
        showIcon={needIcon(option)}
        style={{ color: getColor(option), fontWeight: optFontWeight }}
        onClick={() => handleValueChange(option.text)}
      />
    ),
    [primary, contrast, needIcon, getColor, optFontWeight, handleValueChange]
  );

export default useRenderLine;
```

1. index.tsx 为什么会需要直接引入 userRenderLine？

追求“展示型/容器型” -> 那么需要在 index.tsx 中计算 needIcon,getColor,handleValueChange -> 不想把 needIcon,getColor,handleValueChange 再传递到子组件中 -> 那么就把渲染<SelectLine \/>的逻辑提到上层 -> 实现 userRenderLine。

2. 实现 useRendereLine 以及很多项目中的其他自定义 hooks 为什么需要用 useCallback 包装？

一方面是因为从 Core 中传来的数据结构不够理想很多是复杂 object，另一方面也是把<SelectLine \/>提到上层导致， index.tsx 中组件范围内会有多个计算值，每个计算值改变一次，那么整个组件重新渲染一次，所有内联值和内联函数也会计算一次，为了性能考虑，使用 useMemo, useCallback 暂存这些值或函数。如果<SelectLine \/>的渲染逻辑在组件树更底层，我不敢打包票不需要 useCallback 等手段，起码必要性要减小很多。

我们来尝试维护一下 SelectLine 的代码，首先需要在`MenuDrawer/index.tsx`中找到 renderLine 的调用，然后需要到 `userRenderLine.tsx` 中找到 SelectLine，然后到`select-line/index.tsx`找到具体 dom 结构。我们想看一下 needIcon, getColor, handleValueChang 是如何由 Core 中数据计算出来的，那么需要分别到 useGetColor, useNeedIcon, useValueChange 中分别查找，好在这些 hooks 的代码都很简单。但等一下，useGetColor 需要用到的 bindValue 需要到 useInitState 中查找是怎么计算出来的。

在开头的第二篇文章中提到过

> Hooks 的使用确实导致了与依赖关系更强的耦合

如果再加上“容器型/展示型”的教条，一个容器型组件中多个自定义 hooks 树形调用，那么更加难以解耦和分离关注点。

## 三、 项目中的代码重用性

os-client-live 项目本质上是个没有很多复杂逻辑的 UI 项目，真正核心的逻辑在 os-client-core，虽然 os-client-core 复杂的数据结构在使用时不是百分百理想，但是也为 os-cient-live 方便编写 UI 提供了极大方便。 以后 live 可能需要的变化，大部分都是增加新的 UI。真的实际做起来，复制粘贴代码恐怕会更多一些，我认为应该要追求代码精简和代码风格统一，而不是引入“容器型/展示型”增加复杂度。

## 四、 是否应该为了单元测试的覆盖率而用自定义 hooks 强制抽出 “useState, useEffect 以及内联值，内联函数”? 测试的目的是什么？目前单元测试追求 100% 是否真的那么有价值？

按照测试金字塔，单元测试的性价比确实是最高的。那为什么我们不只写单元测试？因为单元测试只能给我们某个短小片段能够正确工作的信心，而不是代码组合在一起也能正确工作的信心。一般来说，useState, useEffect，内联值，内联函数的逻辑都是和单个组件能否正确工作高度相关的，即使测试一个组件的这些代码需要用到某种意义上的“集成”测试，组件的“集成”比抽出自定义 hooks 的测试更难写一点。但是带来的回报（即信心）比强制抽出这些代码单独测试来的高。

有时，还是推荐抽出自定义 hooks，特别是有调用 api，改变外部值等等这种副作用的代码时。

如果状态逻辑或计算逻辑过于复杂，也可以按照“容器型/展示型”分割组件，比如把

```js
const A = () => {};
```

分割成

```js
const A_container = () => {
  // 计算复杂的逻辑
  const props = {};
  return <A_presentational {...props} />;
};

const A_presentational = () => {};
```

分辩哪些逻辑需要被抽出为自定义 hooks，或者留在组件内部。比单纯抽出所有逻辑到自定义 hooks 要花费更多的精力，以及更多的时间编写测试，思考组件正确工作的方式。但得到的回报是更多的。

单元测试和集成测试（或者叫组件测试）不应该分离开，如开头第三篇第四篇文章所说，只要集合不同测试策略达到目的就行了。按照目前项目配置，Enzyme 和 Testing library 配合同时编写单元测试和集成测试，也是能达到 100% 覆盖率的。

## 五、原子化并不等于“容器型/展示型组件”的二分法

“容器型/展示型组件”导致的状态和函数通过多层 props 传值，或者增加了组件的层次，多个自定义 hooks 树形调用，都会导致更强的耦合。这与原子化是相违背的。React 应用的原子化，在理想中我认为应该是 hooks 组件有足够内聚的逻辑和足够明确的边界。

## 六、是否有必要使用 ahooks 中 useUpdateEffect，甚至于，是否有必要使用 ahooks?

目前仍在使用的 ahooks 中有 "useUpdateEffect,useBoolean,useDebounceFn,useThrottleFn", "useBoolean, useDebounceFn, useThrottleFn" 完全可以从 ahooks 拷贝源码到项目中或者自己用 lodash 和内置 hooks 包装。

关于 useUpdateEffect，这个相比 useEffect 可以减少一次运算，其他方面没有区别。也可以去掉。

上面的观点都是为了精简掉 ahooks，虽然也减少不了太多打包体积。

## 七、过度使用 useMemo 和 useCallback?

useMemo 和 useCallback 作为优化性能的手段，大量使用其实意味着整个应用的架构存在问题。传给组件的值不是合理的。

扩展阅读:
[新版 react 中，usecallback 和 usememo 是不是值得大量使用?](https://www.zhihu.com/question/390974405)
[useCallback/useMemo 的使用误区](https://juejin.cn/post/6847902217261809671)

## 八、为什么我推荐 testing libaray 多过 Enzyme?

1. 首先是对 react hooks 的支持上，testing library 超过 Enzyme。

2. 看看 testing libaray 的作者自己是[怎么说的](./introducing-the-react-testing-library.md)
   [原文](https://kentcdodds.com/blog/introducing-the-react-testing-library)˝
