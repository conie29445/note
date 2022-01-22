---
title: 20220122_frontend-webpack
date: 2022-01-22 22:44:39
tags: 前端專案工具
---

【筆記】Webpack
===
[教學文](https://medium.com/@Mike_Cheng1208/%E4%BB%80%E9%BA%BC%E6%98%AFwebpack-%E4%BD%A0%E9%9C%80%E8%A6%81webpack%E5%97%8E-2d8f9658241d)
[中文手冊](https://webpack.docschina.org/configuration/output/)


|      | gulp                        | webpack                                 |
| ---- | --------------------------- | --------------------------------------- |
| 短詞說明 | 自動化工具                       | 模組化打包工具                                 |
| 工作內容 | 前端、開發雜事自動化，用於繁瑣的任務管理，簡化前端流程 | 當前端引入大量模組 (CSS、JS)時，可透過 Webpack 來作模組化開發 |
| 適合範圍 | 廣                           | 較小                                      |
| 輸出   | 依據需求調整                      | 全部都打成一包                                 |


# 專案結構
一般我們的專案會有兩個很重要的資料夾src與dist，這兩個資料夾是什麼？
* src : 專門放我們Preprocess的檔案，包括es6、pug、sass、vue、jsx等檔案，這個資料夾不會丟上去server部署。
* dist : 經過webpack編譯打包後，產生出瀏覽器看得懂的html、css、js，要部署也是這個資料夾去部署。
# webpack
- 想引入npm第三方套件, 瀏覽器對es6支援度低
- 把每個資源都看成模組
- 是一個以自己編寫設定檔然後透過指令去驅動的一個自動化工具
- 我們必須透過javascript的引入語法讓webpack知道需要它幫忙編譯什麼東西，例如pug或是sass等
- 進入點的js專門注入（import）那些Preprocess，讓那些Preprocess可以通過這個進入點的js讓webpack去編譯它，進而打包到dist資料夾

## 安裝webpack
* npm init
            產生package.json
            npm init -y 
            不想回答那些問題時
* npm install webpack webpack-cli --save-dev
  npm install後產生node_modules

[Pug and Webpack](https://medium.com/@NorthBei/%E5%A6%82%E6%9E%9C%E4%BD%A0%E6%98%AF%E5%B8%B8%E5%88%87%E7%89%88%E7%9A%84%E5%89%8D%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%B8%AB-%E4%BD%A0%E4%B8%80%E5%AE%9A%E8%A6%81%E7%9F%A5%E9%81%93pug-8b2cbc0a784c)
## 執行webpack
- 新版本webpack**必須透過npm script**才可以執行
# 實作 Webpack 打包
webpack 的主要概念如下(也是我們在 config 裡要設定的參數)：
- 我們使用的 ./ 是一種相對路徑, 代表**現在所在的資料夾**
- entry: 待打包的 source 從哪裡來 ，可放object
- context: 指定所有entry的資料夾
```javascript
    context: path.resolve(__dirname, 'src'),
    entry: {
        index: './index.js',
        about: './about.js'
    }
```
- output: 打包完要匯出到哪
  - bundle, assets和其它你所打包或使用webpack載入的任何內容
- __dirname: nodejs中的特殊變數，當前執行目錄之所在目錄的完整目錄位置
- [name] 會依照entry的名稱來更改output的名稱
```javascript
//例子
entry: {
  cc_index: './index.js'
},
output: {
  filename: '[name].js'
}
// result: 產生 cc_index.js
```
```javascript
// [name]　會依照entry的name來更改output
output: {
    path: path.resolve(__dirname,"dist"),
    filename: '[name].js'
}
```
- module
rules: 每個loader的規則
```javascript
// test　正規表示式
module: {
    rules: [{
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
    }]
}
```
loader: 將 webpack 看不懂的檔案轉成 webpack 看的懂 js, loader由後往前執行
- plugins: 使用什麼套件來處理這些檔案, 處理loader無法處理的事s
```javascript
plugins: [
    extractCSS
]
```
```javascript
// webpack.config.js
var path = require('path')
module.exports = {
  entry: ['./src/index'],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js'
  }
```
### Path
#### path.resolve()
- 將「相對路徑」或「路徑片段」解析成絕對路徑
- __dirname: nodejs中的特殊變數，當前執行目錄之所在目錄的完整目錄位置
### Resolve 路徑可縮寫
resolve: 調整模組與**entry的路徑**
- modules: 在webpack加上resolve module在**引入模塊**時，可以省略路徑
- extensions: 可以省略副檔名，因檔名重覆會出現問題，建議只省略js
> output不可使用縮寫
> 僅限開發
```javascript
// webpack config js
resolve: {
    modules: [
        path.resolve('src'),
        path.resolve('src/js'),
        path.resolve('src/scss'),
        // 沒加這行 webpack-dev-server會讀不到自己
        path.resolve('node_modules')
    ],
    extensions: ['.js']
}
```
# webpack 套件 & loader
- loader執行順序： **由後往前執行**
#### cross-env (windows開發者需加入)
```
npm i -D cross-env
```
NODE_ENV為node.js其中一個環境變數
```
NODE_ENV
```
```javascript
process.env.NODE_ENV
```
#### extract-text-webpack-plugin: 獨立拆分出檔案 (webpack-4 以前)
```javascript
// 安裝　非release版
npm install --save-dev extract-text-webpack-plugin@next

//用法 webpack.config.js
const ExtractTextPlugin = require('extract-text-webpack-plugin');
// Create multiple instances
const extractCSS = new ExtractTextPlugin('css/[name].css');

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: extractCSS.extract([ 'css-loader', 'postcss-loader' ])
      },
      {
        test: /\.less$/i,
        use: extractLESS.extract([ 'css-loader', 'less-loader' ])
      },
    ]
  },
  plugins: [
    extractCSS,
    extractLESS
  ]
};
```
#### mini-css-extract-plugin
// publicPath : 解決css中背景圖的路徑問題
```javascript
//webpack config
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const devMode = process.env.NODE_ENV !== 'production'

{
    test: /\.(sa|sc)ss$/,
    use: [
        {
            loader: MiniCssExtractPlugin.loader,
            options: {
                publicPath: "../"
            }
        },
        'css-loader',
        'postcss-loader',
        'sass-loader',
    ]
}

//　實體化
// filename output
plugins: [
    new MiniCssExtractPlugin({
        filename: 'css/[name].css'
    })
]
```

#### postCSS: 使用js轉換css的工具，搭配autoprefixer加入前綴: -webkit-
npm i postcss-loader autoprefixer
[說明](https://cythilya.github.io/2018/08/10/postcss/)
```javascript
// postcss.config.js
module.exports = {
    plugins: [
        require('autoprefixer')({
            browsers: [
                '> 1%',
                'last 5 versions',
                'Firefox > 45',
                'iOS >= 8',
                'Safari >= 8',
                'ie >= 10'
            ]
        })
    ]
}
```
#### sass-loader
#### postcss-loader
  - 使用js轉換css的工具
  - 搭配autoprefixer加入瀏覽器的prefix: -webkit-, -moz-
```javascript
// npm install -D sass-loader node-sass postcss-loader autoprefixer
    test: /\.(sass|scss)$/,
    use: [
        'style-loader',
        'css-loader',
        'postcss-loader',
        'sass-loader'
    ]
```  
把css檔案獨立出來的作法
``` javascript
  test: /\.(sass|scss)$/,
  use: extractCSS.extract([
      'css-loader',
      'postcss-loader',
      'sass-loader'
  ])
``` 
#### file-loader: 搬移檔案, 不用經過loader
```javascript
    test: /\.html$/,
    use: [{
        loader: 'file-loader',
        options: {
            name: '[path][name].[ext]'
            //　路徑，檔名，副檔名
        }
    }]
```

#### webpack-dev-server:
把src資料夾內容，做預處理編譯放在記憶體裡
```javascript
// webpack.config.js設定
    devServer: {
        compress: true,
        port: 3000,
        stats: {
            assets: true,
            cached: false,
            chunkModules: false,
            chunkOrigins: false,
            chunks: false,
            colors: true,
            hash: false,
            modules: false,
            reasons: false,
            source: false,
            version: false,
            warnings: false
        }
    }
// package.json設定 script, --open: 開瀏覽器
"server": "cross-env NODE_ENV=development webpack-dev-server --open"
// 區網中使用
"server-c": "cross-env NODE_ENV=development webpack-dev-server --host 區域ip --open"
``` 
#### babel: 轉換高版本的js，for舊瀏覽器的轉譯器
安裝時參考官方文件([Babel · The compiler for next generation JavaScript](https://babeljs.io/))
npm install -D babel-loader @babel/core @babel/preset-env webpack
@babel/core　程式需要調用babel的api進行編譯
@babel/preset-env 使用最新版本的編譯器，不用理會哪些語法需轉換
```javascript
module: {
  rules: [
    {
      test: /\.m?js$/,
      exclude: /(node_modules|bower_components)/,
      use: {
        loader: 'babel-loader',
        options: {
          presets: ['@babel/preset-env']
        }
      }
    }
  ]
}
```
#### @babel/polyfill: 有些語法ie不支援，靠@babel/polyfill來解決舊ie問題 (winXP)
```
npm install @babel/polyfill
```
> 沒有要support 舊型的ie就不建議裝
```
//在js import
import '@babel-polyfill';
```
#### url loader
檔案小於8192bytes時，轉成base64 URIs，通常不搭配extractCss，路徑會有問題
建議用官方的limit: 8192 bytes
```javascript
{
    test: /\.(jpe?g|png|gif)$/,
    use: [{
        loader: 'url-loader',
        options: {
            limit: 8192,
            name: '[path][name].[ext]?[hash:8]'
        }
    }]
}
```
> 在 sass 裡面要使用載入圖片的話可以使用 ~ 符號來代替路徑
> 不使用的話可以用原本的 ../ 的方式來指向路徑

#### image-webpack-loader: 壓縮圖片
先壓縮圖片再轉成base64
```javascript
{
    loader: 'image-webpack-loader',
    options: {
        publicPath: '../',
        mozjpeg: {
            progressive: true,
            quality: 65
        },
        optipng: {
            enabled: false,
        },
        pngquant: {
            quality: '65-90',
            speed: 4
        },
        gifsicle: {
            interlaced: false,
        }
    }
}
```
#### copy-webpack-plugin：　原封不動複製至目的資料夾，可能需要搭配『**file-loader**』
If you want webpack-dev-server to write files to the output directory during development, you can force it with the **write-file-webpack-plugin**
libs（自定義）：　放不用經過loader轉換的檔案，如.mp3、字型檔
> 若js或sass有引入libs檔案時，需透過『**file-loader**』判別副檔名
```javascript
// webpack config
const CopyWebpackPlugin = require('copy-webpack-plugin')

plugins: [
    new CopyWebpackPlugin([
        { from: 'libs', to: 'libs' }
    ])
]
```
```javascript
// sass, js..等有引入assets檔案時，需透過『file-loader』判別副檔案
{
    test: /\.(woff|woff2|ttf|eot)$/,
    loader: 'file-loader',
    options: {
        name: '[path][name].[ext]?[hash:8]'
    }
}
```
#### ProvidePlugin: 透過ProvidePlugin讓每支js裡不用import第三方套件就可以在全域取得
ProvidePlugin是原本就在webpack裡面的一個功能，所以我們先抓取他的模組
**非必要盡量不要使用 ProvidePlugin 這個做法!!!**
npm install jquery (production mode也會用到，所以不加-dev)
```javascript
var webpack = require('webpack');

// plugins
new webpack.ProvidePlugin({
    $: 'jquery',
    jQuery: 'jquery',
    'window.jQuery': 'jquery',
    // '自定義名字': 'jquery'
})
```
#### Html-Webpack-Plugin-template: 管理所有的html＆要載入的js
[options](https://github.com/jantimon/html-webpack-plugin#options)
npm install --save-dev html-webpack-plugin
If you have any CSS assets in webpack's output (for example, CSS extracted with the ExtractTextPlugin) then these will be included with **\<link\>** tags in the HTML head. 若css獨立出來，則透過htmlwebpackplugin，會自動引入
Chunks就是對每個不同的html注入不同的 js
- 使用pug一定必須搭配此plugins
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin');

// plugins
new HtmlWebpackPlugin({
  title: 'Webpack前端自動化開發',
  filename: 'index.html',
  template: 'html/template.html',
  viewport: 'width=device-width, initial-scale=1.0',
  description: 'Webpack前端自動化開發，讓你熟悉現代前端工程師開發的方法',
  Keywords: 'Webpack前端自動化開發、前端、工程師、線上教學、教學範例',
  chunks: ['index' ],
}),

```


# require & import
- require: es6 **前**模組化開發方式
- import: es6 **後**模組化開發方式

| require/module.exports        | import/export           |
| :------------- |:-------------|
| CommonJS 的規範     | ES6 的規範    |
|Node.js 有支援，瀏覽器沒有   | node.js、瀏覽器部分支援   |

#### 模塊檔名：含相對路徑
| 輸出        | 引入           |
| :------------- |:-------------|
| module.exports = 模塊       | var 名稱 = require("模塊檔名")      |
| export default 模塊;        | import 名稱 from "模塊檔名"        |
# dependencies devdependencies
* dependencies : 使用在已經發布的環境下，換句話說，是指發布後仍然需要依賴使用的 plug-in。舉個例子來說，如果我需要使用 jQuery 與 AngularJs 來開發，就算開發完之後發佈到伺服器，我仍然需要依賴 jQuery 與 AngularJs 的套件，這些套件會在發布後繼續使用。
用法：當我執行 npm install –production 或是註明 NODE_ENV 變量值为為 production 時，只會下載 dependencies 中的套件。
* devDependencies : 使用在開發中的環境下，意思是指——只單純會在開發時應用到的 plug-in。同樣舉個例子，如果我在開發時需要使用 Js ES6 並使用 babel 轉換成 ES5，或是我希望可以使用 gulp-stylus 的套件來使用，但在發佈之後，我們並不會在用到 gulp-stylus 這個套件。換句話說，他只需要存在於開發環境中，而不需要繼續放到發布環境裏。
用法：鍵入 npm install 時，會同時抓下來 dependencies & devDependencies 兩個節點之中的套件。

# Vendor.js 與 Entry.js
透過Vendor.js獨立出來可以有效率的進行打包
```javascript
  optimization: {
      splitChunks: {
          cacheGroups: {
              vendor: {
                  test: /node_modules/,
                  name: 'vendor',
                  chunks: 'initial',
                  enforce: true
              }
          }
      }
  },
```
```javascript
  new HtmlWebpackPlugin({
    title: 'index',
    filename: 'index.html',
    template: 'template/template.html',
    chunks: ['vendor', 'index' ],
  }),
```
# 打包與排除
- include 哪些目錄中的文件需要進行 loader 轉換
- exclude 哪些目錄中的文件不需要進行 loader 轉換

# 問題集
- [打包圖片與壓縮的介紹](https://codingwife.com/2018/03/23/webpack/webpack03/)

# Pug 使用
npm install --save-dev html-loader pug-html-loader
**pug的轉換一定要搭配html-webpack-plugin來使用**
chunks: 注入 js / 獨立css檔案
```javascript
// plugins
  new HtmlWebpackPlugin({
      title: 'pug轉換',
      filename: 'views/about.html',
      template: 'pug/about.pug',
      chunks: ['about']
  })
```

```javascript
// html壓縮
  new HtmlWebpackPlugin({
      title: 'pug轉換',
      filename: 'views/about.html',
      template: 'pug/about.pug',
      chunks: ['about'],
      minify: {
          collapseWhitespace: true,
          removeComments: true,
          removeRedundantAttributes: true,
          removeScriptTypeAttributes: true,
          removeStyleLinkTypeAttributes: true,
          useShortDoctype: true
      }
  })
```
# Bootstrap
- Importing JavaScript
```
import 'bootstrap';
```
- Importing Styles
```
@import "~bootstrap/scss/bootstrap";
```
# Vue 開發
```
// --save-dev
npm install vue-loader -D
npm install vue-template-compiler -D
npm install vue-html-loader -D
npm install vue-style-loader -D

// --save
npm install vue -S
```
- 引入VueLoaderPlugin
```javascript
  var VueLoaderPlugin = require('vue-loader/lib/plugin');
```
- 加入 vue-loader 跟 vue-style-loader，移除 include 設定
```javascript
// module rule
  {
      test: /\.(vue)$/,
      use: 'vue-loader'
  }
// 修改sass loader設定
  {
    test: /\.(sa|sc)ss$/,
    use: [
        'vue-style-loader',
        {
            loader: MiniCssExtractPlugin.loader,
            options: {
                publicPath: "../"
            }
        },
        'css-loader',
        'postcss-loader',
        'sass-loader',
    ]
  },
 // plugins
  new VueLoaderPlugin()
```
- new VueLoaderPlugin

## Vue 資料夾結構
  src/
    - components/
    - js/
    - App.vue
    - index.js (進入點)
## Vue 套件
- Vetur (VS Code套件)

## 單純編譯指令
```
webpack <entry> <output>
```

