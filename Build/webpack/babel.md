## .babelrc 配置导致的 jsx相关问题

在配置React的时候出现如下报错：

```javascript
Module build failed (from ./node_modules/happypack/loader.js):
SyntaxError: G:\react\webr\src\index.js: Support for the experimental syntax 'jsx' isn't currently enabled
```

一开始以为是 happypack 的配置问题

通篇百度之后发现 并没有关于 jsx 部分的配置

当关联搜索 React+Support for the experimental syntax 'jsx' isn't currently enabled

出现 一个 这样的帖子

[experimental syntax 'jsx' isn't currently enabled (react storybook)](https://stackoverflow.com/questions/64103567/experimental-syntax-jsx-isnt-currently-enabled-react-storybook)

Reciveving the following error when running storybook:

```
WARNING in ./src/stories/button.stories.tsx
Module build failed (from ./node_modules/babel-loader/lib/index.js):
SyntaxError: /<path_to_project>/src/stories/button.stories.tsx: Support for the experimental syntax 'jsx' isn't currently enabled (45:35):

  43 | const onClick = () => console.log('clicked');
  44 | 
> 45 | stories.add(NAME, () => { return (<Button onClick={onClick}>Hello Button</Button> );})
     |                                   ^
  46 | 

Add @babel/preset-react (https://git.io/JfeDR) to the 'presets' section of your Babel config to enable transformation.
If you want to leave it as-is, add @babel/plugin-syntax-jsx (https://git.io/vb4yA) to the 'plugins' section to enable parsing.
    at Parser._raise (/<path_to_project>/node_modules/@babel/parser/lib/index.js:766:17)
```

However, `.babelrc` does contain the preset:

```
{
    "presets": [
        [
            "@babel/preset-env"
        ],
        [
            "@babel/preset-react"
        ],
        [
            "@babel/typescript"
        ]
    ],
}
```

As a side note - I have installed `storybook-addon-jsx` and registered it via `addons.js`.

所以怀疑是和 .babelrc 配置有关。和参考代码比较之后发现 果然 .babelrc 没有添加

添加之后 问题 解决
