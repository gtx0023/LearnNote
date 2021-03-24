# 与webpack一起使用

Jest可以用于使用webpack来管理资产、样式和编译的项目。webpack确实比其他工具提供了一些独特的挑战，因为它直接与应用程序集成，允许管理样式表、图像和字体等资产，以及编译到JavaScript语言和工具的广阔生态系统。

## 一个webpack的例子

让我们从一个常见的webpack配置文件开始，并将其转换为Jest设置。

```
// webpack.config.js
module.exports = {
  module: {
    loaders: [
      {exclude: ['node_modules'], loader: 'babel', test: /\.jsx?$/},
      {loader: 'style-loader!css-loader', test: /\.css$/},
      {loader: 'url-loader', test: /\.gif$/},
      {loader: 'file-loader', test: /\.(ttf|eot|svg)$/},
    ],
  },
  resolve: {
    alias: {
      config$: './configs/app-config.js',
      react: './vendor/react-master',
    },
    extensions: ['', 'js', 'jsx'],
    modules: [
      'node_modules',
      'bower_components',
      'shared',
      '/shared/vendor/modules',
    ],
  },
};
```

如果您有由Babel转换的JavaScript文件，您可以通过安装 babel-jest [插件](https://www.jestjs.cn/docs/getting-started#using-babel)来启用对Babel的支持。非Babel JavaScript转换可以使用Jest的 [转换](https://www.jestjs.cn/docs/webpack) 配置选项处理。 

## 处理静态资产

接下来，让我们配置Jest以优雅地处理资产文件，如样式表和图像。通常，这些文件在测试中并不特别有用，所以我们可以安全地模拟它们。但是，如果您使用的是CSS模块，那么最好为className查找模拟代理。

```
// package.json
{
  "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js"
    }
  }
}
```

还有模拟文件本身：

```
// __mocks__/styleMock.js

module.exports = {};
```

```
// __mocks__/fileMock.js

module.exports = 'test-file-stub';
```

## 模拟CSS模块

您可以使用ES6代理模拟CSS模块：

```
yarn add --dev identity-obj-proxy
```

然后，样式对象上的所有className查找都将按原样返回（例如，styles.foobar==='foobar'）。这对于React[快照测试](https://www.jestjs.cn/docs/snapshot-testing)来说非常方便。

```
// package.json (for CSS Modules)
{
  "jest": {
    "moduleNameMapper": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/__mocks__/fileMock.js",
      "\\.(css|less)$": "identity-obj-proxy"
    }
  }
}
```

> 请注意，默认情况下，节点6中启用了代理。如果您还不在节点6上，请确保使用 node --harmony_proxies node_modules/.bin/jest

如果moduleNameMapper无法满足您的要求，您可以使用Jest的转换配置选项来指定如何转换资产。例如，返回文件基名的转换器（例如 require('logo.jpg'）；返回'logo')可以写成：

```
// fileTransformer.js
const path = require('path');

module.exports = {
  process(src, filename, config, options) {
    return 'module.exports = ' + JSON.stringify(path.basename(filename)) + ';';
  },
};
```

```
// package.json (for custom transformers and CSS Modules)
{
  "jest": {
    "moduleNameMapper": {
      "\\.(css|less)$": "identity-obj-proxy"
    },
    "transform": {
      "\\.(jpg|jpeg|png|gif|eot|otf|webp|svg|ttf|woff|woff2|mp4|webm|wav|mp3|m4a|aac|oga)$": "<rootDir>/fileTransformer.js"
    }
  }
}
```

我们已经告诉Jest忽略匹配样式表或图像扩展名的文件，而是需要我们的模拟文件。您可以调整正则表达式以匹配webpack配置处理的文件类型。 

注意：如果您将babel-jest与其他代码预处理器一起使用，则必须显式定义babel-jest作为JavaScript代码的转换器，以将.js文件映射到babel-jest模块。

```
"transform": {
  "\\.js$": "babel-jest",
  "\\.css$": "custom-transformer",
  ...
}
```

## 配置Jest以查找我们的文件

现在杰斯特知道如何处理我们的文件，我们需要告诉它如何找到它们。对于webpack的 模块目录 和 扩展选项，Jest的 模块目录 和 模块文件扩展 选项中有直接的模拟。

```
// package.json
{
  "jest": {
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],

    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
    }
  }
}
```

注意：<rootDir>是一个特殊的令牌，它将被Jest替换为项目的根。大多数情况下，除非在配置中指定自定义rootDir选项，否则这将是package.json所在的文件夹。 

类似地，webpack的 resolve.root 选项的功能类似于设置 NODE_PATH env 变量，您可以设置该变量，或使用modulePaths 选项。

```
// package.json
{
  "jest": {
    "modulePaths": ["/shared/vendor/modules"],
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],
    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js"
    }
  }
}
```

最后，我们必须处理 webpack别名。为此，我们可以再次使用 moduleNameMapper 选项。

```
// package.json
{
  "jest": {
    "modulePaths": ["/shared/vendor/modules"],
    "moduleFileExtensions": ["js", "jsx"],
    "moduleDirectories": ["node_modules", "bower_components", "shared"],

    "moduleNameMapper": {
      "\\.(css|less)$": "<rootDir>/__mocks__/styleMock.js",
      "\\.(gif|ttf|eot|svg)$": "<rootDir>/__mocks__/fileMock.js",

      "^react(.*)$": "<rootDir>/vendor/react-master$1",
      "^config$": "<rootDir>/configs/app-config.js"
    }
  }
}
```

就是这样!webpack是一个复杂而灵活的工具，因此您可能必须进行一些调整以处理特定应用程序的需求。幸运的是，对于大多数项目来说，Jest应该足够灵活，可以处理您的webpack配置。 

注意：对于更复杂的webpack配置，您可能还需要调查项目，如：[babel-plugin-webpack-loaders](https://github.com/istarkov/babel-plugin-webpack-loaders).

# 与webpack 2一起使用

webpack 2为ES模块提供了本机支持。但是，Jest运行在Node中，因此需要将ES模块传输到CommonJS模块中。因此，如果您使用的是webpack 2，您很可能希望配置Babel，以便仅在测试环境中将ES模块传输到CommonJS模块。

```
// .babelrc
{
  "presets": [["env", {"modules": false}]],

  "env": {
    "test": {
      "plugins": ["transform-es2015-modules-commonjs"]
    }
  }
}
```

> 注意：Jest缓存文件以加快测试执行速度。如果您更新了.babelrc，而 Jest 仍然无法工作，请尝试使用 --no-cache 运行Jest。 

如果使用动态导入 (`import('some-file.js').then(module => ...)`), 则需要启用 dynamic-import-node 插件。

```
// .babelrc
{
  "presets": [["env", {"modules": false}]],

  "plugins": ["syntax-dynamic-import"],

  "env": {
    "test": {
      "plugins": ["dynamic-import-node"]
    }
  }
}
```

有关如何将Jest与Webpack与React、Redux和Node一起使用的示例，您可以在此处查看一个。
