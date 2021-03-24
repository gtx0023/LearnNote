# 快照测试 

当您想确保UI不会意外更改时，快照测试是一个非常有用的工具。 

典型的快照测试用例渲染UI组件，获取快照，然后将其与存储在测试旁边的参考快照文件进行比较。如果两个快照不匹配，测试将失败：更改是意外的，或者引用快照需要更新到UI组件的新版本。

## 使用Jest

进行快照测试 在测试React组件时，也可以采取类似的方法。您可以使用测试渲染器快速为React树生成可序列化的值，而不是渲染图形UI，这将需要构建整个应用程序。考虑链接组件的以下示例测试：

```
import React from 'react';
import renderer from 'react-test-renderer';
import Link from '../Link.react';

it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.facebook.com">Facebook</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

第一次运行此测试时，Jest会创建一个快照文件，该文件看起来如下：

```
exports[`renders correctly 1`] = `
<a
  className="normal"
  href="http://www.facebook.com"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Facebook
</a>
`;
```

快照工件应与代码更改一起提交，并作为代码审查过程的一部分进行审查。Jest使用漂亮的格式使快照在代码审查期间可读。在随后的测试运行中，Jest将将渲染的输出与上一个快照进行比较。如果它们匹配，测试就会通过。如果它们不匹配，则测试运行程序在代码中发现了一个应该修复的错误（在本例中为<Link>组件），或者实现已更改，需要更新快照。

> 注意：快照的作用域直接指向您呈现的数据——在我们的示例中，带有页面道具传递给它的<Link />组件。这意味着，即使任何其他文件在<Link />组件中缺少道具（比如说，App.js），它仍然会通过测试，因为测试不知道<Link />组件的用法，而且它的作用域只限于Link。React.js。此外，在其他快照测试中使用不同的道具渲染相同的组件不会影响第一个测试，因为测试彼此不知道。

有关快照测试如何工作以及我们为什么构建它的更多信息，请访问发布博客帖子。我们建议您阅读此博客文章，以很好地了解何时应该使用快照测试。我们还建议观看这个书呆子视频，关于快照测试与Jest。

## 更新快照

在引入错误后，快照测试何时失败是很容易发现的。当发生这种情况时，请继续修复问题，并确保快照测试再次通过。现在，让我们来谈谈快照测试因故意实现更改而失败的情况。 

如果我们故意更改示例中Link组件指向的地址，则可能会出现这种情况。

```
// Updated test case with a Link to a different address
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="http://www.instagram.com">Instagram</Link>)
    .toJSON();
  expect(tree).toMatchSnapshot();
});
```

在这种情况下，Jest将打印此输出：

![img](https://www.jestjs.cn/assets/images/failedSnapshotTest-28fddb1a7aa06fe502f8a74c0b0049ef.png)

由于我们刚刚更新了组件以指向不同的地址，因此预期此组件的快照会发生更改是合理的。我们的快照测试用例失败，因为更新组件的快照不再与此测试用例的快照工件匹配。 

要解决此问题，我们需要更新快照工件。您可以使用一个标志运行Jest，该标志将告诉它重新生成快照：

```
jest --updateSnapshot
```

通过运行上述命令，继续并接受更改。如果您愿意，您还可以使用等效的单字符-u标志重新生成快照。这将为所有失败的快照测试重新生成快照工件。如果我们由于无意中的错误而有任何其他失败的快照测试，我们需要在重新生成快照之前修复该错误，以避免记录错误行为的快照。 

如果您想限制重新生成的快照测试用例，则可以传递额外的--testNamePattern标志，以仅为匹配模式的测试重新录制快照。

 您可以通过克隆快照示例、修改Link组件和运行Jest来尝试此功能。

## 交互式快照模式

失败的快照也可以在监视模式下以交互方式更新：

![img](https://www.jestjs.cn/assets/images/interactiveSnapshot-dafbb7e3f643c06fad03dd1468431cf9.png)

进入交互式快照模式后，Jest将逐次引导您完成失败的快照测试，并让您有机会查看失败的输出。 

从此处，您可以选择更新该快照或跳到下一个快照：

![img](https://www.jestjs.cn/assets/images/interactiveSnapshotUpdate-a17d8d77f94702048b4d0e0e4c580719.gif)

完成后，Jest将在返回观看模式之前给您一个总结：

![img](https://www.jestjs.cn/assets/images/interactiveSnapshotDone-6ad54afc34b56a3791860110c9dac06d.png)

内联快照

内联快照的行为与外部快照（.snap文件）相同，只是快照值会自动写回源代码。这意味着您可以获得自动生成快照的好处，而不必切换到外部文件以确保写入了正确的值。 

> 内联快照由Prettier提供支持。要使用内联快照，您必须在项目中安装更漂亮的快照。写入测试文件时，您的Prettier配置将受到尊重。 
>
> 如果您在Jest找不到它的位置安装了漂亮的，您可以使用“漂亮的Path”配置属性告诉Jest如何找到它。 

示例： 

首先，您编写一个测试，调用.toMatchInlineSnapshot()，没有参数：

```
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot();
});
```

下次运行Jest时，将评估树，并将快照作为参数写入toMatchInlineSnapshot：

```
it('renders correctly', () => {
  const tree = renderer
    .create(<Link page="https://prettier.io">Prettier</Link>)
    .toJSON();
  expect(tree).toMatchInlineSnapshot(`
<a
  className="normal"
  href="https://prettier.io"
  onMouseEnter={[Function]}
  onMouseLeave={[Function]}
>
  Prettier
</a>
`);
});
```

这就是它的全部!您甚至可以使用--updateSnapshot或在--watch模式下使用u键更新快照。

## 属性匹配器

通常，要快照的对象中会生成一些字段（如ID和日期）。如果您尝试对这些对象进行快照，它们将强制快照在每次运行时失败：

```
it('will fail every time', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot();
});

// Snapshot
exports[`will fail every time 1`] = `
Object {
  "createdAt": 2018-05-19T23:36:09.816Z,
  "id": 3,
  "name": "LeBron James",
}
`;
```

对于这些情况，Jest允许为任何属性提供非对称匹配器。在写入或测试快照之前，将检查这些匹配器，然后将其保存到快照文件中，而不是接收到的值：

```
it('will check the matchers and pass', () => {
  const user = {
    createdAt: new Date(),
    id: Math.floor(Math.random() * 20),
    name: 'LeBron James',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    id: expect.any(Number),
  });
});

// Snapshot
exports[`will check the matchers and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "id": Any<Number>,
  "name": "LeBron James",
}
`;
```

任何不是匹配器的给定值都将被精确检查并保存到快照中：

```
it('will check the values and pass', () => {
  const user = {
    createdAt: new Date(),
    name: 'Bond... James Bond',
  };

  expect(user).toMatchSnapshot({
    createdAt: expect.any(Date),
    name: 'Bond... James Bond',
  });
});

// Snapshot
exports[`will check the values and pass 1`] = `
Object {
  "createdAt": Any<Date>,
  "name": 'Bond... James Bond',
}
`;
```

## 最佳实践

快照是识别应用程序中意外接口更改的绝佳工具——无论该接口是API响应、UI、日志还是错误消息。与任何测试策略一样，您应该了解一些最佳实践，以及您应该遵循的准则，以便有效地使用它们。

1. 将快照视为代码

   提交快照并将其作为常规代码审查流程的一部分进行审查。这意味着像对待项目中的任何其他类型的测试或代码一样对待快照。 

   通过保持快照的重点、简短和使用强制这些风格约定的工具，确保快照是可读的。 

   如前所述，Jest使用漂亮的格式使快照可读，但您可能会发现引入其他工具很有用，如eslint-插件-jest，其无大快照选项，或快照差异及其组件快照比较功能，以促进提交简短、重点突出的断言。 

   其目标是使查看拉取请求中的快照变得容易，并与在测试套件失败时重新生成快照的习惯作斗争，而不是检查其失败的根本原因。

   

2. 测试应该是确定性的

   你的测试应该是确定性的。在未更改的组件上多次运行相同的测试，每次都应产生相同的结果。您有责任确保生成的快照不包括特定于平台的数据或其他非确定性数据。

   例如，如果您有一个使用Date.now()的时钟组件，则每次运行测试用例时，从该组件生成的快照将不同。在这种情况下，我们可以模拟Date.now()方法，以在每次运行测试时返回一致的值：

   ```
   Date.now = jest.fn(() => 1482363367071);
   ```

   现在，每次运行快照测试用例时，Date.now()都将一致返回1482363367071。这将导致为此组件生成相同的快照，无论测试何时运行。

   

3. 使用描述性快照名称

   始终努力对快照使用描述性测试和/或快照名称。最佳名称描述预期的快照内容。这使得审阅者更容易在审阅期间验证快照，并且任何人都可以在更新之前知道过时的快照是否正确。 

   例如，比较：

   ```
   exports[`<UserName /> should handle some test case`] = `null`;
   
   exports[`<UserName /> should handle some other test case`] = `
   <div>
     Alan Turing
   </div>
   `;
   ```

   To:

   ```
   exports[`<UserName /> should render null`] = `null`;
   
   exports[`<UserName /> should render Alan Turing`] = `
   <div>
     Alan Turing
   </div>
   `;
   ```

   由于后面的内容准确地描述了输出中的预期内容，因此更清楚地看到它何时出错：

   ```
   exports[`<UserName /> should render null`] = `
   <div>
     Alan Turing
   </div>
   `;
   
   exports[`<UserName /> should render Alan Turing`] = `null`;
   ```

   

# 常见问题

- 快照是否自动写入持续集成（CI）系统?

  - 否，从Jest 20起，当Jest在CI系统中运行而不显式传递--updateSnapshot时，Jest中的快照不会自动写入。预计所有快照都是在CI上运行的代码的一部分，由于新快照自动通过，因此它们不应通过CI系统上的测试运行。建议始终提交所有快照，并将它们保持在版本控制中。

    

- 是否应该提交快照文件?

  - 是的，所有快照文件都应与它们所涵盖的模块及其测试一起提交。它们应该被视为测试的一部分，类似于Jest中任何其他断言的值。事实上，快照表示源模块在任何给定时间点的状态。这样，当修改源模块时，Jest就可以知道与上一个版本相比发生了什么变化。它还可以在代码审查期间提供许多额外的上下文，在这些上下文中，审查者可以更好地研究您的更改。

    

- 快照测试是否只适用于React组件?

  - React和React Native组件是快照测试的一个很好的用例。但是，快照可以捕获任何可序列化的值，并且应在目标测试输出是否正确的任何时候使用。Jest存储库包含许多测试Jest本身输出、Jest断言库输出以及来自Jest代码库各个部分的日志消息的示例。请参阅在Jest repo中快照CLI输出的示例。

    

- 快照测试和可视化回归测试有什么区别?

  - 快照测试和视觉回归测试是测试UI的两种不同方式，它们的目的不同。视觉回归测试工具拍摄网页的屏幕截图，并逐个像素比较生成的图像。使用快照，测试值将序列化，存储在文本文件中，并使用差异算法进行比较。有不同的权衡需要考虑，我们列出了在Jest博客中构建快照测试的原因。

    

- 快照测试是否取代单元测试?

  - 快照测试只是Jest附带的20多个断言之一。快照测试的目的不是取代现有的单元测试，而是提供额外的价值，使测试无痛苦。在某些情况下，快照测试可能会消除对一组特定功能（例如React组件）的单元测试的需要，但它们也可以协同工作。

    

- 快照测试在生成文件的速度和大小方面的性能是什么?

  - Jest被重写时考虑到了性能，快照测试也不例外。由于快照存储在文本文件中，因此这种测试方式快速可靠。Jest为调用toMatchSnapshot匹配器的每个测试文件生成一个新文件。快照的大小相当小：作为参考，Jest代码库本身中所有快照文件的大小都小于300 KB。

    

- 如何解决快照文件中的冲突?

  - 快照文件必须始终表示它们所涵盖的模块的当前状态。因此，如果要合并两个分支，并且在快照文件中遇到冲突，则可以手动解决冲突，也可以通过运行Jest并检查结果来更新快照文件。

    

- 是否可以将测试驱动的开发原则与快照测试一起应用?

  - 虽然可以手动写入快照文件，但这通常是不可能的。快照有助于确定测试所涵盖的模块的输出是否发生了更改，而不是一开始就提供设计代码的指导。

    

- 代码覆盖率是否与快照测试一起工作?

  - 是的，以及任何其他测试。
