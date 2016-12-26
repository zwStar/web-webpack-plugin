# [中文文档](https://github.com/gwuhaolin/web-webpack-plugin/blob/master/readme_zh.md)

# install
```bash
npm i web-webpack-plugin --save-dev
```
```js
const WebWebpackPlugin = require('web-webpack-plugin');
const { WebPlugin, AutoWebPlugin } = WebWebpackPlugin;
```


# output html file [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/out-html)
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
            // this html's require entry,must be an array.dependent resource will inject into html use the order entry in array.
            require: ['A', 'B'],
        }),
    ]
};
```

will output an file named `index.html`,this file will auto load js file generated by webpack form entry  `A` and `B`,the out html as below:
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
output directory structure:
```
├── A.js
├── B.js
└── index.html
```


# use html template [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/use-template)
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
            require: ['A', 'B'],
        }),
    ]
};
```
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <script src="B"></script>
</head>
<body>
<!--SCRIPT-->
<footer>web-webpack-plugin</footer>
</body>
</html>
```
- use `<script src="B"></script>` in html template to load required entry, the `B` in `src="B"` means entry name config in `webpack.config.js`
- comment `<!--SCRIPT-->` means a inject position ,except for resource load by `<script src></script>` left required resource config in `WebPlugin's require option`. if there has no `<!--SCRIPT-->` in html template left required script will be inject ad end of `body` tag.
    
output html:
```html
<!DOCTYPE html>
<html lang="zh-cn">
<head>
    <meta charset="UTF-8">
    <script src="B.js"></script>
</head>
<body>
<script src="A.js"></script>
<footer>web-webpack-plugin</footer>
</body>
</html>
```    


# config resource attribute [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/config-resource)
every resource required by html,it can config some attribute as below:
- `_dist` only load in production environment
- `_dev` only load in dev environment
- `_inline` inline resource content info html,inline script and css
- `_ie` resource only required IE browser,to achieve by `[if IE]>resource<![endif]` comment

there has two way to config resource attribute:

### config in html template
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
template.html
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
<script async src="googleAnalytics?_dist"></script>
</body>
</html>
```
[output html file](https://github.com/gwuhaolin/web-webpack-plugin/blob/master/demo/config-resource/dist-template/index.html)

### config in `webpack.config.js`
```js
module.exports = {
    plugins: [
        new WebPlugin({
            filename: 'index.html',
            require: {
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
                'googleAnalytics': {
                    _dist: true
                }
            }
        }),
    ]
};
```
[output html file](https://github.com/gwuhaolin/web-webpack-plugin/blob/master/demo/config-resource/dist-js/index.html)


# auto detect html entry [demo](https://github.com/gwuhaolin/web-webpack-plugin/tree/master/demo/auto-plugin)
`AutoWebPlugin` can find all page entry in an directory,then auto config an `WebPlugin` for every page to output an html file,you can use it as below:
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
            // extract common chunk for all pages and then put it into a file named common,if it is null then not do extract action
            // achieve by CommonsChunkPlugin
            commonsChunk: 'common',
        }),
    ]
};
```
src directory:
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
output directory:
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
`AutoWebPlugin` find all page `home login signup` directory in `./src/`,for this three page `home login signup` will use `index.js` as main file and output three html file home.html login.html signup.html`

### template attribute
`template` if template is a string , i will regard it as file path for html template（full path relative to webpack.config.js）
In the complex case,You can set the template to a function, as follows using the current page directory index.html file as the current page template file
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
The entity property is similar to template, and also supports callback functions for complex situations. But if the entity is empty to use the current page directory index.jsx? As the entrance
 
 
# Distinguish the environment
This plug-in takes into account both *development* environment and *production* environment. And only if the `DefinePlugin` plugin defines` NODE_ENV = production` is the current environment is *production* environment, others are considered to be development environment.
```js
new webpack.DefinePlugin({
    'process.env': {
        'NODE_ENV': JSON.stringify('production')
    }
})
``` 
webpack -p will define `DefinePlugin NODE_ENV=production`。