# 介绍一下 react-testing-library

一个更简单的 Enzyme 的替代品, 鼓励良好的测试实践。

两周前，我写了一个新的库! 我已经想了很久了。[但两周前我开始相当认真地对待它](https://twitter.com/kentcdodds/status/974278185540964352?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E974278185540964352%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fintroducing-the-react-testing-library)。

继续阅读，了解我所说的 "破坏性做法 "是什么意思。

Testing library 的 emoji 符号是山羊。没有特别的原因...

> 简单而完整的 React DOM 测试工具，鼓励良好的测试实践。

## 问题所在

你想为你的 React 组件编写可维护的测试。作为这个目标的一部分，你希望你的测试避免包括你的组件的实现细节，而是专注于使你的测试给你组件正常工作的信心。作为这个目标的一部分，你希望你的测试库从长远来看是可维护的，所以你的组件的重构（对实现的改变，但不是功能）不会破坏你的测试，也不会拖累你和你的团队。

## 这个解决方案

react-testing-library 是一个非常轻量级的解决方案，用于测试 React 组件。它在 react-dom 和 react-dom/test-utils 的基础上提供轻量级的实用功能，以鼓励更好的测试实践。它的主要指导原则是：

[你的测试越是类似于你的软件的使用方式，它们就越能给你带来信心。](https://twitter.com/kentcdodds/status/977018512689455106?ref_src=twsrc%5Etfw%7Ctwcamp%5Etweetembed%7Ctwterm%5E977018512689455106%7Ctwgr%5E%7Ctwcon%5Es1_c10&ref_url=https%3A%2F%2Fkentcdodds.com%2Fblog%2Fintroducing-the-react-testing-library)

因此，你的测试将与实际的 DOM 节点一起工作，而不是处理渲染的 React 组件的实例。这个库所提供的工具有助于以与用户相同的方式查询 DOM。通过标签文本找到表单元素（就像用户那样），通过文本找到链接和按钮（就像用户那样）。它还提供了一种推荐的方法，即通过"data-testid"来寻找元素，作为元素的 "逃生舱"，在这种情况下，文本内容和标签没有意义或没有作用。

这个库鼓励你的应用程序更易于访问，并允许你让你的测试更接近于以用户的方式使用你的组件，这使得你的测试给你更多的信心，当真正的用户使用它时，你的应用程序会正常工作。

这个库是 Enzyme 的替代品。虽然你可以使用 enzyme 本身来遵循这些准则，但由于 enzyme 提供的所有额外的 utilities（方便测试实现细节的实用程序），执行起来比较困难。请在 [FAQ](https://testing-library.com/docs/react-testing-library/faq) 中阅读更多关于这方面的内容。

这个库不是：

1. 一个测试运行器或框架
2. 特定于一个测试框架（尽管我们推荐 Jest 作为我们的首选，但该库可用于任何框架，甚至可用于 codesandbox！）。

## 例子：

### 基础例子：

```js
// hidden-message.js
import * as React from "react";
// NOTE: React Testing Library works with React Hooks _and_ classes just as well
// and your tests will be the same however you write your components.
function HiddenMessage({ children }) {
  const [showMessage, setShowMessage] = React.useState(false);
  return (
    <div>
      <label htmlFor="toggle">Show Message</label>
      <input
        id="toggle"
        type="checkbox"
        onChange={(e) => setShowMessage(e.target.checked)}
        checked={showMessage}
      />
      {showMessage ? children : null}
    </div>
  );
}

export default HiddenMessage;
// __tests__/hidden-message.js
// These imports are something you'd normally configure Jest to import for you automatically.
// Learn more in the setup docs: https://testing-library.com/docs/react-testing-library/setup#skipping-auto-cleanup
import "@testing-library/jest-dom/extend-expect";
// NOTE: jest-dom adds handy assertions to Jest and is recommended, but not required
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import * as React from "react";
import HiddenMessage from "../hidden-message";
test("shows the children when the checkbox is checked", () => {
  const testMessage = "Test Message";
  render(<HiddenMessage>{testMessage}</HiddenMessage>);
  // query* functions will return the element or null if it cannot be found
  // get* functions will return the element or throw an error if it cannot be found
  expect(screen.queryByText(testMessage)).toBeNull();
  // the queries can accept a regex to make your selectors more resilient to content tweaks and changes.
  userEvent.click(screen.getByLabelText(/show/i));
  // .toBeInTheDocument() is an assertion that comes from jest-dom
  // otherwise you could use .toBeDefined()
  expect(screen.getByText(testMessage)).toBeInTheDocument();
});
```

实际例子：

```js
// login.js
import * as React from "react";
function Login() {
  const [state, setState] = React.useReducer((s, a) => ({ ...s, ...a }), {
    resolved: false,
    loading: false,
    error: null,
  });

  function handleSubmit(event) {
    event.preventDefault();
    const { usernameInput, passwordInput } = event.target.elements;
    setState({ loading: true, resolved: false, error: null });
    window
      .fetch("/api/login", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          username: usernameInput.value,
          password: passwordInput.value,
        }),
      })
      .then((r) => r.json())
      .then(
        (user) => {
          setState({ loading: false, resolved: true, error: null });
          window.localStorage.setItem("token", user.token);
        },
        (error) => {
          setState({ loading: false, resolved: false, error: error.message });
        }
      );
  }

  return (
    <div>
      <form onSubmit={handleSubmit}>
        <div>
          <label htmlFor="usernameInput">Username</label>
          <input id="usernameInput" />
        </div>
        <div>
          <label htmlFor="passwordInput">Password</label>
          <input id="passwordInput" type="password" />
        </div>
        <button type="submit">Submit{state.loading ? "..." : null}</button>
      </form>
      {state.error ? <div role="alert">{state.error.message}</div> : null}
      {state.resolved ? (
        <div role="alert">Congrats! You're signed in!</div>
      ) : null}
    </div>
  );
}

export default Login;
// __tests__/login.js
// again, these first two imports are something you'd normally handle in
// your testing framework configuration rather than importing them in every file.
import "@testing-library/jest-dom/extend-expect";
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import * as React from "react";
import Login from "../login";

test("allows the user to login successfully", async () => {
  // mock out window.fetch for the test
  const fakeUserResponse = { token: "fake_user_token" };
  jest.spyOn(window, "fetch").mockImplementationOnce(() => {
    return Promise.resolve({
      json: () => Promise.resolve(fakeUserResponse),
    });
  });

  render(<Login />);
  // fill out the form
  userEvent.type(screen.getByLabelText(/username/i), "chuck");
  userEvent.type(screen.getByLabelText(/password/i), "norris");
  userEvent.click(screen.getByText(/submit/i));
  // just like a manual tester, we'll instruct our test to wait for the alert
  // to show up before continuing with our assertions.
  const alert = await screen.findByRole("alert");
  // .toHaveTextContent() comes from jest-dom's assertions
  // otherwise you could use expect(alert.textContent).toMatch(/congrats/i)
  // but jest-dom will give you better error messages which is why it's recommended
  expect(alert).toHaveTextContent(/congrats/i);
  expect(window.localStorage.getItem("token")).toEqual(fakeUserResponse.token);
});
```

从这个例子中最重要的启示是:

> 测试的编写方式与用户使用你的应用程序的方式相类似。

让我们进一步探讨这个问题...

假设我们有一个 GreetingFetcher 组件，为用户获取一个问候语。它可能会渲染一些像这样的 HTML。

```html
<div>
  <label for="name-input">Name</label>
  <input id="name-input" />
  <button>Load Greeting</button>
  <div data-testid="greeting-text" />
</div>
```

所以功能点是：设置名字，点击 "load Greeting" 按钮，服务器请求加载带有该名称的问候语文本。

在你的测试中，你需要找到<input \/>，这样你就可以把它的`value`设置为某种东西。传统的观点认为你可以使用 CSS 选择器中的 `id` 属性：`#name-input`。但这是用户找到该输入的方法吗？肯定不是！他们看了看屏幕，然后找到这个输入。他们看着屏幕，找到标签为 "姓名 "的输入，然后填入。所以这就是我们的测试中 `getByLabelText` 的作用。它根据标签来获取表单控件。

通常在使用 Enzyme 的测试中，为了找到"Load Greeting"按钮，你可能会使用一个 CSS 选择器，甚至通过组件 `displayName` 或组件构造器来寻找。但是当用户想加载问候语时，他们并不关心这些实现细节，相反，他们会找到并点击写着 "Load Greeting" 的按钮。而这正是我们的测试用 `getByText` 帮助函数所做的事情！这就是我们的测试。

此外，等待的过程与用户所做的完全相似。他们等待问候语文本出现，无论需要多长时间。在我们的测试中，我们正在模拟它，所以它基本上是即时发生的，但我们的测试实际上并不关心它需要多长时间。我们不需要在测试中使用 setTimeout 或任何东西。我们只是说。"嘿，等到问候语节点出现。" (注意，在这种情况下，它使用了一个 `data-testid` 属性，这是在通过其他机制找到一个元素没有意义的情况下的一个逃生舱口。[`data-testid` 肯定比其他方法好](https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change)。

## 高层次的概述 API

最初，该库只提供了 `queryByTestId` 这个工具，正如我的博文 "[让你的 UI 测试对变化有弹性](https://kentcdodds.com/blog/making-your-ui-tests-resilient-to-change)"中所建议的那样。但是由于 Bergé Greg 对那篇博文的反馈，以及 Jamie White 精彩（而且很短！）的[演讲](https://youtu.be/qfnkDyHVJzs?t=5h39m19s)的启发，我又增加了几个，现在我对这个解决方案更加满意。

你可以在官方文档中阅读更多关于这个库和它的 API。下面是这个库给你的高层次概述。

- `Simulate`：来自 [react-dom/test-utilsSimulate](https://reactjs.org/docs/test-utils.html#simulate) object 的 Simulate 工具的再导出。
- `wait`：允许你在测试中等待一段非决定性的时间。通常情况下，你应该 mock 出 API 请求或动画，但即使你处理的是立即 resolved 的 Promises，你也需要你的测试等待事件循环的下一次 tick，而`wait` 在这方面非常出色。(这要归功于 Łukasz Gozda Gandecki，他引入了这个功能来替代（现已废弃的）flushPromises API）。
- `render`: 这是该库的核心部分。它相当简单。它用 `document.createElement` 创建一个 `div`，然后使用 ReactDOM.render 来渲染该 `div`。

`render` 函数返回以下对象和工具。

- `container`: 你的组件被渲染到的那个 div
- `unmount`: 对`ReactDOM.unmountComponentAtNod` 的一个简单封装，用来卸载你的组件（例如，为了方便测试 `componentWillUnmount`）。
- `getByLabelText`: 获取一个与标签相关的表单控件
- `getByPlaceholderText`: 占位符不是标签的适当替代品，但如果这对你的使用情况更有意义，它是可用的。
- `getByText`: 通过其文本内容获取任何元素。
- `getByAltText`: 通过它的 alt 属性值获取一个元素（比如<img>）。
- `getByTestId`: 通过它的 data-testid 属性来获取一个元素。

如果找不到元素，每个`get*`工具都会抛出一个有用的错误信息。还有一个相关的 `query*`API，它将返回 `null`，而不是抛出一个错误，这对于断言一个元素不在 DOM 中很有用。

另外，对于这些 `get* `工具，要找到一个匹配的元素，你可以传递这些参数:

- 一个不区分大小写的子串：`lo world` 匹配 `Hello World`
- 一个正则。/^Hello World$/ 匹配 Hello World
- 一个接受文本和元素的函数。`(text, el) => el.tagName === 'SPAN' && text.startsWith('Hello')` 将匹配一个内容以 `Hello` 开头的 `span`。

## 自定义 Jest 匹配器

感谢 Anto Aravinth Belgin Rayen，我们也有了一些方便的自定义 Jest 匹配器。

- `toBeInTheDOM`: 断定一个元素是否存在于 DOM 中。
- `toHaveTextContent`: 检查给定元素是否有文本内容。

> 注意：现在这些已经被提取到 jest-dom 中了，它由 Ernesto García 维护。

## 总结

这个库的一个很大的特点是它没有测试实现细节的工具。它专注于提供鼓励良好的测试和软件实践的实用程序。我希望通过使用 react-testing-library，你的 React 测试代码会更容易被理解和维护。
