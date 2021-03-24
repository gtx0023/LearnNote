# 手动模拟

手动模拟用于使用模拟数据构建功能。例如，您可能希望创建一个手动模拟，以允许您使用虚假数据，而不是访问网站或数据库等远程资源。这可确保您的测试将是快速的，而不是片状的。 

## 模拟用户模块

手动模拟是通过在紧靠模块的__mocks__/子目录中写入模块来定义的。例如，要在模型目录中模拟一个名为user的模块，请创建一个名为user.js的文件，并将其放入模型/_模拟__目录中。请注意，__模拟__文件夹区分大小写，因此在某些系统上命名目录__MOCKS__会中断。 

> 当我们在测试中要求该模块时，需要显式调用jest.mock（'./moduleName'）。

## 示例

```
.
├── config
├── __mocks__
│   └── fs.js
├── models
│   ├── __mocks__
│   │   └── user.js
│   └── user.js
├── node_modules
└── views
```

当给定模块存在手动模拟时，Jest的模块系统将在显式调用jest.mock（'moduleName'）时使用该模块。但是，当automock设置为true时，即使未调用jest.mock（'moduleName'），也将使用手动模拟实现而不是自动创建的模拟。要选择退出此行为，您需要在应使用实际模块实现的测试中显式调用jest.unmock（'moduleName'）。 

> 注意：为了正确模拟，Jest需要jest.mock（'moduleName'）与要求/导入语句处于同一作用域。 

这里是一个人为的示例，其中我们有一个模块，它提供了给定目录中所有文件的摘要。在这种情况下，我们使用核心（内置）fs模块。

```
// FileSummarizer.js
'use strict';

const fs = require('fs');

function summarizeFilesInDirectorySync(directory) {
  return fs.readdirSync(directory).map(fileName => ({
    directory,
    fileName,
  }));
}

exports.summarizeFilesInDirectorySync = summarizeFilesInDirectorySync;
```

由于我们希望我们的测试避免实际撞击磁盘（这非常缓慢和脆弱），我们通过扩展自动模拟为fs模块创建手动模拟。我们的手动模拟将实现fs API的自定义版本，我们可以在这些版本上构建用于测试：

```
// __mocks__/fs.js
'use strict';

const path = require('path');

const fs = jest.createMockFromModule('fs');

// This is a custom function that our tests can use during setup to specify
// what the files on the "mock" filesystem should look like when any of the
// `fs` APIs are used.
let mockFiles = Object.create(null);
function __setMockFiles(newMockFiles) {
  mockFiles = Object.create(null);
  for (const file in newMockFiles) {
    const dir = path.dirname(file);

    if (!mockFiles[dir]) {
      mockFiles[dir] = [];
    }
    mockFiles[dir].push(path.basename(file));
  }
}

// A custom version of `readdirSync` that reads from the special mocked out
// file list set via __setMockFiles
function readdirSync(directoryPath) {
  return mockFiles[directoryPath] || [];
}

fs.__setMockFiles = __setMockFiles;
fs.readdirSync = readdirSync;

module.exports = fs;
```

现在我们写测试。请注意，我们需要明确告知我们要模拟fs模块，因为它是一个核心节点模块：

```
// __tests__/FileSummarizer-test.js
'use strict';

jest.mock('fs');

describe('listFilesInDirectorySync', () => {
  const MOCK_FILE_INFO = {
    '/path/to/file1.js': 'console.log("file1 contents");',
    '/path/to/file2.txt': 'file2 contents',
  };

  beforeEach(() => {
    // Set up some mocked out file info before each test
    require('fs').__setMockFiles(MOCK_FILE_INFO);
  });

  test('includes all files in the directory in the summary', () => {
    const FileSummarizer = require('../FileSummarizer');
    const fileSummary = FileSummarizer.summarizeFilesInDirectorySync(
      '/path/to',
    );

    expect(fileSummary.length).toBe(2);
  });
});
```

此处显示的示例模拟使用jest.createMockFromModule生成自动模拟，并覆盖其默认行为。这是推荐的方法，但完全是可选的。如果您根本不想使用自动模拟，您可以从模拟文件导出自己的函数。完全手动模拟的一个缺点是它们是手动的——这意味着您必须在模块模拟更改时随时手动更新它们。正因为如此，当自动模拟满足您的需求时，最好使用或扩展它。 

为了确保手动模拟与其实际实现保持同步，在导出之前，要求在手动模拟中使用jest.requireActual(moduleName)使用真实模块，并使用模拟函数修改它可能会很有用。 

此示例的代码可在示例/手动模拟中获得。

## 与ES模块导入一起使用

如果您使用的是ES模块导入，那么您通常倾向于将导入语句放在测试文件的顶部。但通常，您需要指示Jest在模块使用模拟之前使用它。因此，Jest将自动将jest.mock调用提升到模块的顶部（在任何导入之前）。要了解有关此信息的更多信息并查看其实际行动，请参见此回购。 

## JSDOM中没有实现的模拟方法

如果某些代码使用了JSDOM（Jest使用的DOM实现）尚未实现的方法，则测试它并不容易。例如，window.matchMedia()就是这样。Jest返回TypeError: window.matchMedia不是函数，无法正确执行测试。 

在这种情况下，在测试文件中模拟匹配媒体应解决以下问题：

```
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: jest.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: jest.fn(), // deprecated
    removeListener: jest.fn(), // deprecated
    addEventListener: jest.fn(),
    removeEventListener: jest.fn(),
    dispatchEvent: jest.fn(),
  })),
});
```

如果在测试中调用的函数（或方法）中使用了window.matchMedia()，则此操作有效。如果直接在测试文件中执行window.matchMedia(),Jest会报告相同的错误。在这种情况下，解决方案是将手动模拟移动到一个单独的文件中，并在测试文件之前将此模拟包含在测试中：

```
import './matchMedia.mock'; // Must be imported before the tested file
import {myMethod} from './file-to-test';

describe('myMethod()', () => {
  // Test the method here...
});
```

