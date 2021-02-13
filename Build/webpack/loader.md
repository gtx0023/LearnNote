## loader路径引起的错误

近日遇到一个webpack loader的问题

主要报错

```javascript
ERROR in ./src/index.js 10:4
Module parse failed: Unexpected token (10:4)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
|
| ReactDOM.render(
>     <Root />,
|     document.getElementById('root'),
| );

```

但是loader部分是复制的，一直摸不着头脑为什么会出错。为什么别人是对的，我确实错的

后来看到一个帖子如下

## Question

## [React / Webpack - “Module parse failed: Unexpected token - You may need an appropriate loader to handle this file type.”](https://stackoverflow.com/questions/48570731/react-webpack-module-parse-failed-unexpected-token-you-may-need-an-appro)

I am trying to build a simple React app using Webpack, however during run time from the Webpack dev server I get the following error:

```
> ERROR in ./client/components/home.js Module parse failed: Unexpected
> token (8:3) You may need an appropriate loader to handle this file
> type. |   render(){ |         return( |           <h1>Hello World!</h1> |         ) |     } 
> @ ./client/app.js 13:12-43  @ multi
> (webpack)-dev-server/client?http://localhost:8080 ./client/app.js
```

Here is my Webpack.config.js file contents:

```
var path = require('path');
var debug = process.env.NODE_ENV !== 'production';
var webpack = require('webpack');


module.exports = {
    context: __dirname,
    devtool: debug ? 'inline-sourcemap' : null,
    entry: './client/app.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'app.js'
    },
    module: {
        loaders: [{
            test: /\.js?$/,
            exclude: /(node_modules|bower_components)/,
            include: [path.resolve(__dirname, 'client/app.js')],
            loader: 'babel-loader',
            query: {
                presets: ['react', 'es2015', 'stage-3'],
                plugins: ['transform-react-jsx']
            }
        }, {
            test: /\.scss$/,
            include: [path.resolve(__dirname, 'sass/style.scss')],
            loaders: ['style-loader' ,'css-loader' ,'sass-loader']
        }]
    },
    plugins: debug ? [] : [
        new webpack.optimize.DedupePlugin(),
        new webpack.optimize.OccurenceOrderPlugin(),
        new webpack.optimize.UglifyJsPlugin({
            mangle: false,
            sourcemap: false
        }),
    ],
};
```

app.js file contents import React from 'react' import ReactDOM from 'react-dom' import { Switch, BrowserRouter, Route } from 'react-router-dom' import Home from './components/home.js'

```
ReactDOM.render((
     <BrowserRouter>
          <Switch>
            <Route exact path="/" component={Home}/>
        </Switch>
     </BrowserRouter>
     ),
     document.getElementById('app')
)
```

And below is my folder structure as follows:

 [![enter image description here](https://i.stack.imgur.com/jaYYE.png)](https://i.stack.imgur.com/jaYYE.png)



## Answer

I think it might have something to do with your `includes` property for your loader:

```
// Here you are telling babel to ONLY transpile client/app.js. One file.
include: [path.resolve(__dirname, 'client/app.js')]
```

Should be:

```
// Instead, it needs to transpile ALL .js files under /client
include: [path.resolve(__dirname, 'client')], 
```



在回复中我注意到，回答者把问题聚焦在loader的路径上。所以我检查了自己的路径。发现因为文件夹被我调整了位置，但是路径没变，所以这个就是问题的根本原因
