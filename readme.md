[![Npm Package](https://img.shields.io/npm/v/web-webpack-plugin.svg?style=flat-square)](https://www.npmjs.com/package/web-webpack-plugin)
[![Npm Downloads](http://img.shields.io/npm/dm/web-webpack-plugin.svg?style=flat-square)](https://www.npmjs.com/package/web-webpack-plugin)
[![Dependency Status](https://david-dm.org/gwuhaolin/web-webpack-plugin.svg?style=flat-square)](https://npmjs.org/package/web-webpack-plugin)

### [中文文档  Chinese](https://github.com/gwuhaolin/web-webpack-plugin/blob/master/readme_zh.md)

# Install
```bash
npm i web-webpack-plugin --save-dev
```
```js
const { WebPlugin, AutoWebPlugin } = require('web-webpack-plugin');
```


# Feature


## output html file [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/out-html)
**webpack config**
```js
module.exports = {
    entry: {
        A: './a',
        B: './b',
    },
    plugins: [
        new WebPlugin({
            // file name for output file, required.
            // pay attention not to duplication of name,as is will cover other file
            filename: 'index.html',
            // this html's requires entry,must be an array.dependent resource will inject into html use the order entry in array.
            requires: ['A', 'B'],
        }),
    ]
};
```

will output an file named `index.html`,this file will auto load js file generated by webpack form entry  `A` and `B`,the out html as below:

**output html**
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
</head>
<body>
<script src="A.js"></script>
<script src="B.js"></script>
</body>
</html>
```

**output directory**
```
├── A.js
├── B.js
└── index.html
```



## use html template [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/use-template)
**webpack config**
```js
module.exports = {
    entry: {
        A: './a',
        B: './b',
    },
    plugins: [
        new WebPlugin({
            filename: 'index.html',
            // html template file path（full path relative to webpack.config.js）
            template: './template.html',
            requires: ['A', 'B'],
        }),
    ]
};
```

**html template**
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <!--load a chunk file config and output in webpack-->
    <script src="B"></script>
    <!--load a local reset style file direct without local var webpack-->
    <link rel="stylesheet" href="./reset.min.css?_inline">
    <!--load a local google analyze file direct without local var webpack-->
    <script src="./google-analyze.js"></script>
</head>
<body>
<!--SCRIPT-->
<footer>web-webpack-plugin</footer>
</body>
</html>
```
- use `<script src="B"></script>` in html template to load required entry, the `B` in `src="B"` means entry name config in `webpack.config.js`
- comment `<!--SCRIPT-->` means a inject position ,except for resource load by `<script src></script>` left required resource config in `WebPlugin's requires option`. if there has no `<!--SCRIPT-->` in html template left required script will be inject ad end of `body` tag.
    
**output html**
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <!--load a chunk file config and output in webpack-->
    <script src="B.js"></script>
    <!--load a local reset style file direct without local var webpack-->
    <style>body {
        background-color: rebeccapurple;
    }</style>
    <!--load a local google analyze file direct without local var webpack-->
    <script src="google-analyze.js"></script>
</head>
<body>
<script src="A.js"></script>
<footer>web-webpack-plugin</footer>

</body>
</html>
```    



## config resource attribute [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/config-resource)
every resource required by html,it can config some attribute as below:
- `_dist` only load in production environment
- `_dev` only load in dev environment
- `_inline` inline resource content info html,inline script and css
- `_ie` resource only required IE browser,to achieve by `[if IE]>resource<![endif]` comment

there has two way to config resource attribute:

### config in html template

**webpack config**
```js
module.exports = {
    entry: {
        'ie-polyfill': './ie-polyfill',
        inline: './inline',
        dev: './dev',
        googleAnalytics: './google-analytics',
    },
    plugins: [
        new WebPlugin({
            filename: 'index.html',
            template: './template.html'
        }),
    ]
};
```

**html template**
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <script src="inline?_inline"></script>
    <script src="ie-polyfill?_ie"></script>
</head>
<body>
<script src="dev?_dev"></script>
<!--load a local google analyze file direct without local var webpack-->
<script async src="./google-analytics.js?_dist"></script>
</body>
</html>
```
[output html file](https://github.com/gwuhaolin/web-webpack-plugin/blob/master/demo/config-resource/dist-template/index.html)

### config in `webpack.config.js`

**webpack config**
```js
module.exports = {
    plugins: [
        new WebPlugin({
            filename: 'index.html',
            requires: {
                'ie-polyfill': {
                    _ie: true
                },
                'inline': {
                    _inline: true,
                    _dist: true
                },
                'dev': {
                    _dev: true
                },
                //load a local google analyze file direct without local var webpack
                './google-analytics.js': {
                    _dist: true
                }
            }
        }),
    ]
};
```
[output html file](https://github.com/gwuhaolin/web-webpack-plugin/blob/master/demo/config-resource/dist-js/index.html)




## auto detect html entry [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/auto-plugin)
`AutoWebPlugin` can find all page entry in an directory,then auto config an `WebPlugin` for every page to output an html file,you can use it as below:

**webpack config**
```js
module.exports = {
    plugins: [
        new AutoWebPlugin(
            // the directory hold all pages
            './src/', 
            {
            // the template file path used by all pages
            template: './src/template.html',
            // javascript main file for current page,if it is null will use index.js in current page directory as main file
            entity: null,
            // CommonsChunkPlugin options for all pages entry find by AutoWebPlugin.
            // if this is null will not do commonsChunk action
            commonsChunk: {
                name: 'common',// name prop is require,output filename
                minChunks: 2,// come from CommonsChunkPlugin
            },
            // pre append to all page's entry
            preEntrys:['./path/to/file1.js'],
            // post append to all page's entry
            postEntrys:['./path/to/file2.js'],
            // whether output a pagemap.json file which contain all pages has been resolved with AutoWebPlugin in this way:
            // {"page name": "page url",}
            outputPagemap: true,
        }),
    ]
};
```

**src directory**
```
── src
│   ├── home
│   │   └── index.js
│   ├── ie_polyfill.js
│   ├── login
│   │   └── index.js
│   ├── polyfill.js
│   ├── signup
│   │   └── index.js
│   └── template.html
```

**output directory**
```
├── dist
│   ├── common.js
│   ├── home.html
│   ├── home.js
│   ├── ie_polyfill.js
│   ├── login.html
│   ├── login.js
│   ├── polyfill.js
│   ├── signup.html
│   └── signup.js
```
`AutoWebPlugin` find all page `home login signup` directory in `./src/`,for this three page `home login signup` will use `index.js` as main file and output three html file `home.html login.html signup.html`

### ignorePages attribute
`ignorePages` page name list will not ignore by AutoWebPlugin(Not output html file for this page name),type is array of string.

### template attribute
`template` if template is a string , i will regard it as file path for html template（full path relative to webpack.config.js）
In the complex case,You can set the template to a function, as follows using the current page directory index.html file as the current page template file

**webpack config**
```js
const path = require('path');
module.exports = {
    plugins: [
        new AutoWebPlugin('./src/', {
            // Template files used by all pages
            template: (pageName) => {
                return path.resolve('./src',pageName,'index.html');
            },
        }),
    ]
};
```
### entity attribute
The entity property is similar to template, and also supports callback functions for complex situations. But if the entity is empty to use the current page directory `index.jsx?` As the entrance
 
 
 
## config publicPath [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/public-path)
 


## load css [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/extract-css)
The resource for each entity may contain css code.
If you want to extract the css code to load alone rather than sneaking into the js where you need to load
[extract-text-webpack-plugin](https://github.com/webpack/extract-text-webpack-plugin) 
Separated css code, the rest of the things to me, I will automatically deal with the same as the above js css

**webpack config**
```js
// webpack.config.js
module.exports = {
    module: {
        loaders: [
            {
                test: /\.css$/,
                loader: ExtractTextPlugin.extract({
                    fallbackLoader: 'style-loader',
                    loader: 'css-loader'
                })
            }
        ]
    },
    entry: {
        1: './1',
        2: './2',
        3: './3',
        4: './4',
    },
    plugins: [
        new ExtractTextPlugin('[name].css'),
        new WebPlugin({
            filename: 'index.html',
            template: './template.html',
            requires: ['1', '2', '3', '4']
        }),
    ]
};
```

**html template**
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="1">
    <link rel="stylesheet" href="2?_inline">
    <link rel="stylesheet" href="3?_ie">
    <script src="1"></script>
    <!--STYLE-->
</head>
<body>
<script src="2"></script>
<!--SCRIPT-->
<footer>footer</footer>
</body>
</html>
```

**output html**
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <link rel="stylesheet" href="1.css">
    <style>
    /**2.css**/
    body {
        background-color: rebeccapurple;
    }</style>
    <!--[if IE]>
    <link rel="stylesheet" href="3.css">
    <![endif]-->
    <script src="1.js"></script>
    <link rel="stylesheet" href="4.css">
</head>
<body>
<script src="2.js"></script>
<script src="3.js"></script>
<script src="4.js"></script>
<footer>footer</footer>
</body>
</html>
```

**output directory**
```
├── 1.css
├── 1.js
├── 2.css
├── 2.js
├── 3.css
├── 3.js
├── 4.css
├── 4.js
└── index.html
``` 
 

 
# Distinguish the environment
This plugin takes into account both **development** environment and **production** environment. And only if `process.env.NODE_ENV = production` current environment is **production** environment, others are considered to be development environment.
`webpack -p` will use DefinePlugin define `NODE_ENV=production`。


# Version of the supported node.js
This plugin uses a lot of es6 syntax, support the latest node.js LTS version
