# 测试React应用

在Facebook，我们使用Jest测试React应用程序。

## 设置

### 使用创建React应用程序#进行设置

如果您是React的新手，我们建议您使用Create React App。它可以使用，并与Jest一起发货!您只需要添加用于渲染快照的反应测试渲染器。

运行

```
yarn add --dev react-test-renderer
```

没有创建React App#的安装程序
如果您有一个现有的应用程序，您需要安装一些软件包，以使所有程序都能很好地协同工作。我们正在使用babel-jest包和反应babel预设来转换测试环境中的代码。另请参见 [using babel](https://www.jestjs.cn/docs/getting-started#using-babel).

运行

```
yarn add --dev jest babel-jest @babel/preset-env @babel/preset-react react-test-renderer
```

您的 package.json 应该看起来像这样（其中 \<current-version\> 是软件包的实际最新版本号）。请添加脚本和jest配置条目：

```
// package.json
  "dependencies": {
    "react": "<current-version>",
    "react-dom": "<current-version>"
  },
  "devDependencies": {
    "@babel/preset-env": "<current-version>",
    "@babel/preset-react": "<current-version>",
    "babel-jest": "<current-version>",
    "jest": "<current-version>",
    "react-test-renderer": "<current-version>"
  },
  "scripts": {
    "test": "jest"
  }
```

```
// babel.config.js
module.exports = {
  presets: ['@babel/preset-env', '@babel/preset-react'],
};
```

你可以愉快的开始了！

## 快照测试编号

让我们为呈现超链接的Link组件创建快照测试：

```
// Link.react.js
import React from 'react';

const STATUS = {
  HOVERED: 'hovered',
  NORMAL: 'normal',
};

export default class Link extends React.Component {
  constructor(props) {
    super(props);

    this._onMouseEnter = this._onMouseEnter.bind(this);
    this._onMouseLeave = this._onMouseLeave.bind(this);

    this.state = {
      class: STATUS.NORMAL,
    };
  }

  _onMouseEnter() {
    this.setState({class: STATUS.HOVERED});
  }

  _onMouseLeave() {
    this.setState({class: STATUS.NORMAL});
  }

  render() {
    return (
      <a
        className={this.state.class}
        href={this.props.page || '#'}
        onMouseEnter={this._onMouseEnter}
        onMouseLeave={this._onMouseLeave}
      >
        {this.props.children}
      </a>
    );
  }
}
```

现在，让我们使用React的测试渲染器和Jest的快照功能与组件交互，捕获渲染的输出并创建快照文件：

```
// Link.react.test.js
import React from 'react';
import renderer from 'react-test-renderer';
import Link from '../Link.react';

test('Link changes the class when hovered', () => {
  const component = renderer.create(
    <Link page="http://www.facebook.com">Facebook</Link>,
  );
  let tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // manually trigger the callback
  tree.props.onMouseEnter();
  // re-rendering
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();

  // manually trigger the callback
  tree.props.onMouseLeave();
  // re-rendering
  tree = component.toJSON();
  expect(tree).toMatchSnapshot();
});
```

当您运行 yarn test 或 jest 时，这将生成一个输出文件，如下所示：

```
// __tests__/__snapshots__/Link.react.test.js.snap
exports[`Link changes the class when hovered 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;

exports[`Link changes the class when hovered 2`] = `
<a
  className="hovered"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;

exports[`Link changes the class when hovered 3`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}>
  Facebook
</a>
`;
```

下次运行测试时，渲染的输出将与以前创建的快照进行比较。快照应与代码更改一起提交。当快照测试失败时，您需要检查它是预期的还是意外的更改。如果需要更改，您可以使用 Jest -u 调用 Jest 以覆盖现有快照。

此示例的代码可在 [examples/snapshot](https://github.com/facebook/jest/tree/master/examples/snapshot) 中获得。

### 使用Mocks, Enzyme 和 React 16进行快照测试

使用 Enzyme  和 React 16+ 时，快照测试有一个警告。如果您使用以下样式模拟模块：

```
jest.mock('../SomeDirectory/SomeComponent', () => 'SomeComponent');
```

然后，您将在控制台中看到警告：

```
Warning: <SomeComponent /> is using uppercase HTML. Always use lowercase HTML tags in React.

# Or:
Warning: The tag <SomeComponent> is unrecognized in this browser. If you meant to render a React component, start its name with an uppercase letter.
```

React 16触发这些警告，因为它检查元素类型的方式，而模拟的模块失败了这些检查。您的选择是：

1. 渲染为文本。这样，您就不会在快照中看到传递给模拟组件的道具，但这很简单：

   ```
   jest.mock('./SomeComponent', () => () => 'SomeComponent');
   ```

   

2. 渲染为自定义元素。DOM“自定义元素”不检查任何内容，也不应该发出警告。它们是小写的，名字上有破折号。

   ```
   jest.mock('./Widget', () => () => <mock-widget />);
   ```

   

3. 使用 react-test-renderer 。测试渲染器不关心元素类型，并且会很乐意接受 e.g. `SomeComponent`。您可以使用测试渲染器检查快照，并使用酶单独检查组件行为。

4. 一起禁用警告（应在开玩笑设置文件中完成）：

   ```
   jest.mock('fbjs/lib/warning', () => require('fbjs/lib/emptyFunction'));
   ```

   

这通常不应该是您的选择，因为有用的警告可能会丢失。但是，在某些情况下，例如，当测试反应原生的组件时，我们正在将反应原生标记呈现到DOM中，许多警告是无关紧要的。另一个选项是切换console.warn并抑制特定警告。

## DOM测试

如果您想断言和操作渲染的组件，您可以使用[react-testing-library](https://github.com/kentcdodds/react-testing-library), [Enzyme](http://airbnb.io/enzyme/), or React's [TestUtils](https://reactjs.org/docs/test-utils.html)。以下两个示例使用react-testing-library 和 Enzyme。

### react-testing-library

您必须运行 yarn add --dev @testing-library/react 使用 react-testing-library。

让我们实现一个复选框，在两个标签之间交换：

```
// CheckboxWithLabel.js

import React from 'react';

export default class CheckboxWithLabel extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isChecked: false};

    // bind manually because React class components don't auto-bind
    // https://reactjs.org/blog/2015/01/27/react-v0.13.0-beta-1.html#autobinding
    this.onChange = this.onChange.bind(this);
  }

  onChange() {
    this.setState({isChecked: !this.state.isChecked});
  }

  render() {
    return (
      <label>
        <input
          type="checkbox"
          checked={this.state.isChecked}
          onChange={this.onChange}
        />
        {this.state.isChecked ? this.props.labelOn : this.props.labelOff}
      </label>
    );
  }
}
```

```
// __tests__/CheckboxWithLabel-test.js
import React from 'react';
import {cleanup, fireEvent, render} from '@testing-library/react';
import CheckboxWithLabel from '../CheckboxWithLabel';

// Note: running cleanup afterEach is done automatically for you in @testing-library/react@9.0.0 or higher
// unmount and cleanup DOM after the test is finished.
afterEach(cleanup);

it('CheckboxWithLabel changes the text after click', () => {
  const {queryByLabelText, getByLabelText} = render(
    <CheckboxWithLabel labelOn="On" labelOff="Off" />,
  );

  expect(queryByLabelText(/off/i)).toBeTruthy();

  fireEvent.click(getByLabelText(/off/i));

  expect(queryByLabelText(/on/i)).toBeTruthy();
});
```

此示例的代码可在以下网址获得：[examples/react-testing-library](https://github.com/facebook/jest/tree/master/examples/react-testing-library).

### Enzyme

你必须运行 yarn add --dev enzyme 才能使用 Enzyme 。如果您使用的是15.5.0以下的React版本，您还需要安装React-addons-test-utils。

让我们使用 Enzyme  替代 react-testing-library 从上面重写测试。在本示例中，我们使用了 Enzyme's [shallow renderer](http://airbnb.io/enzyme/docs/api/shallow.html)。

```
// __tests__/CheckboxWithLabel-test.js

import React from 'react';
import {shallow} from 'enzyme';
import CheckboxWithLabel from '../CheckboxWithLabel';

test('CheckboxWithLabel changes the text after click', () => {
  // Render a checkbox with label in the document
  const checkbox = shallow(<CheckboxWithLabel labelOn="On" labelOff="Off" />);

  expect(checkbox.text()).toEqual('Off');

  checkbox.find('input').simulate('change');

  expect(checkbox.text()).toEqual('On');
});
```

此示例的代码可在以下网址获得：[examples/enzyme](https://github.com/facebook/jest/tree/master/examples/enzyme)

定制转换器
如果您需要更高级的功能，您也可以构建自己的转换器。这里有一个使用 @babel/core 的示例，而不是使用 babel-jest：

```
// custom-transformer.js
'use strict';

const {transform} = require('@babel/core');
const jestPreset = require('babel-preset-jest');

module.exports = {
  process(src, filename) {
    const result = transform(src, {
      filename,
      presets: [jestPreset],
    });

    return result || src;
  },
};
```

不要忘记安装 @babel/core 和 babel-preset-jest 软件包，以便此示例工作。

要使此功能与Jest一起工作，您需要使用以下命令更新Jest配置："transform": {"\\\\.js$": "path/to/custom-transformer.js"}

如果您想构建一个具有babel支持的变压器，您也可以使用 babel-jest 来编写一个变压器，并传递您的自定义配置选项：

```
const babelJest = require('babel-jest');

module.exports = babelJest.createTransformer({
  presets: ['my-custom-preset'],
});
```

有关更多详细信息，请[参阅专用文档](https://www.jestjs.cn/docs/code-transformation#writing-custom-transformers)。
