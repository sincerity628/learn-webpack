<h1 id="0">learn-webpack</h1>

reference:
- [Webpack中文文档](https://www.webpackjs.com/concepts/)

目录：

- [1. 初始化 npm](#1)
- [2. 安装 webpack & webpack-cli](#2)
- [3. 管理模块间的关系](#3)
- [4. 配置 webpack](#4)
- [5. 加载 css loader](#5)
- [6. 加载 scss loader](#6)

---
### 文件结构：
```
|- .git
|- assets
|- README.md
|- .gitignore
|- src
  |- app
    |- utils
      - inputs-are-valid.js
      - parse-inputs.js
    - alert.service.js
    - app.js
    - component.service.js
|- index.html
```

<h3 id="1">1. 初始化 npm</h3>

```
npm init -y
```

此时出现package.json文件:

```
|- .git
|- assets
|- README.md
|- .gitignore
|- src
  |- app
    |- utils
      - inputs-are-valid.js
      - parse-inputs.js
    - alert.service.js
    - app.js
    - component.service.js
|- package.json
|- index.html
```

- 将其中的script清空，同时加上private配置

package.json:
```js
"private": true,
"scripts": {

}
```

##### [⬆️回到顶部⬆️](#0)

---

<h3 id="2">2. 安装 webpack & webpack-cli</h3>

```
npm install --save-dev webpack webpack-cli
```

安装成功后，package.json文件出现：
```js
"devDependencies": {
  "webpack": "^4.42.1",
  "webpack-cli": "^3.3.11"
}
```

- 在package.json文件中自定义一个script

package.json:
```js
"scripts": {
  "start": "webpack"
}
```
意味着，在终端执行```npm start```命令时会执行```webpack```命令。

- 终端在```/learn-webpack```目录下执行```npm start```命令

![npm-start-error1](./assets/screen-shots/npm-start-error.png)
![npm-start-error1](./assets/screen-shots/npm-start-error(1).png)

会发现有如上的报错，原因是 webpack 配置中的入口（entry）默认值为```/src/index.js```，而当前项目中的src文件夹中没有对应的 JS 执行入口文件 ```index.js```，所以会报错。[参考](https://www.webpackjs.com/concepts/#%E5%85%A5%E5%8F%A3-entry-)

- 在/src目录下新建立一个文件index.js
index.js:
```js
alert('hey.');
```

```
|- src
  |- app
    |- utils
      - inputs-are-valid.js
      - parse-inputs.js
    - alert.service.js
    - app.js
    - component.service.js
  - index.js
```

- 再次执行```npm start```命令

![npm-start-success](./assets/screen-shots/npm-start-success.png)

先忽略warning，可以看到，命令执行成功，并且产生了一个```/dist```文件夹，里面有一个```main.js```文件，是 webpack 打包好的执行文件。

```
|- .git
|- assets
|- dist
  - main.js
|- node_modules
|- README.md
|- .gitignore
|- src
|- package.json
|- index.html
```

main.js:

![main.js](./assets/screen-shots/main.png)

文件开始是 webpack 产生的默认配置代码，在文件代码结尾处可以看到入口文件```index.js```中的内容。

- 在```index.html```尾部引入此打包好的文件，打开项目页面，可以看到```index.js```中的内容成功执行。

index.html:
```html
<body>
  ...
  <script src="./dist/main.js"></script>
</body>
```

![alert](./assets/screen-shots/alert.png)

这就意味着，我们可以在入口文件调用启动项目所需的函数，然后在文件中管理好各个接口之间的调用关系（利用```export```和```import```），最后只需要生成一个总的文件，并且只调用一次就可以达到目的，而不需要关注各个文件的调用顺序（webpack会解决这个问题）。

##### [⬆️回到顶部⬆️](#0)

---

<h3 id="3">3. 管理模块间的关系</h3>

分析项目需求，每个文件之间的依赖关系如下：

![relation](./assets/screen-shots/relationship.png)

- 根据这个关系，在各个文件中进行 import 和 export，然后在```index.js```文件中调用```run()```函数。

index.js:
```js
import { AlertService } from './app/alert.service';
import { ComponentService } from './app/component.service';
import { run } from './app/app';

const alertService = new AlertService();
const componentService = new ComponentService();

run(alertService, componentService);
```

- 重新打包```index.js```文件

![build](./assets/screen-shots/build.png)

可以看到，webpack 成功将各个模块联系到了一起，集合在```main.js```文件中。

- 在index.html文件中，只需调用打包好的main.js文件即可，不需要像之前按照顺序将每个项目依赖文件全部引入了。

index.html（原）：
```html
<body>
  ...
  <script src="./src/app/alert.service.js"></script>
  <script src="./src/app/component.service.js"></script>
  <script src="./src/app/utils/inputs-are-valid.js"></script>
  <script src="./src/app/utils/parse-inputs.js"></script>
  <script src="./src/app/app.js"></script>
  <script src="./dist/main.js"></script>
<body>
```

index.html（现）：
```html
<body>
  ...
  <script src="./dist/main.js"></script>
<body>
```

##### [⬆️回到顶部⬆️](#0)

---

<h3 id="4">4. 配置 webpack</h3>

以上都是 webpack 的默认常规操作，因为我们还没有对它进行实际的配置。

- 新建一个```webpack.config.js```文件（名字随意）

```
|- .git
|- assets
|- dist
  - main.js
|- node_modules
|- README.md
|- .gitignore
|- src
|- package.json
|- index.html
|- webpack.config.js
```

- 根据文档，将之前的常规配置写下来并保存：
```js
const path = require('path');

module.exports = {
  mode: 'development', // 生成的main.js文件不被聚合，可以更好的理解代码
  devtool: 'none', // 更好理解main.js中的内容
  entry: './src/index.js', // 入口
  output = { // 输出打包文件
    filename: 'main.js',
    path: path.resolve(__dirname, 'dist')
  }
}
```

- 要使用我们写好的配置，在```package.json```文件中修改

package.json:
```js
{
  "scripts": {
    "start": "webpack --config webpack.config.js"
    // webpack后面加上 --config 配置文件名称
  },
}
```

- 再次终端运行```npm start```，查看生成的```/dist/main.js```文件

main.js:

![main](./assets/screen-shots/main1.png)

##### [⬆️回到顶部⬆️](#0)

---

<h3 id="5">5. 加载 css loader</h3>

- 在```/src```文件夹中创建一个```main.css```文件

main.css:
```css
body {
  background-color: lightblue;
}
```

```
|- src
  |- app
  - index.js
  - main.css
```

要想使用这个样式文件，若不直接在```index.html```中调用的话，我们可以在入口文件中，将这个样式文件加载进来，然后使用 webpack 中的 loader 会自动在项目中引入对应的样式。

- 在入口文件```index.js```中引入```main.css```
```js
import './main.css';
```

现在查看项目，是不会看到背景颜色的变化的，还需要进一步的配置。

- 参考[文档](https://www.webpackjs.com/loaders/)，找到其中的**样式**部分

![doc](./assets/screen-shots/doc.png)

- 根据文档提示，安装style-loader和css-loader
```
npm install --save-dev css-loader style-loader
```

安装成功：

package.json:
```js
{
  "devDependencies": {
    "css-loader": "^3.4.2",
    "style-loader": "^1.1.3",
    "webpack": "^4.42.1",
    "webpack-cli": "^3.3.11"
  }
}
```

- 在```webpack.config.js```配置文件中添加对应配置
```js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.css$/, // 所有以.css结尾的文件
        use: ['style-loader', 'css-loader']
        // css-loader 将css代码转化为js代码，存放在输出文件中
        // style-loader 将输出文件中的被转化的样式添加到项目DOM中，以style标签的形式
        // 顺序不要颠倒，先使用的css-loader反而在后！！！
      },
    ]
  }
}
```
[参考：style-loader](https://www.webpackjs.com/loaders/style-loader/)

- 终端执行```npm start```

- 查看项目，发现样式已经被添加

![background](./assets/screen-shots/background.png)

style-loader 生成的```style```标签：

![style-tag](./assets/screen-shots/style.png)

##### [⬆️回到顶部⬆️](#0)

---

<h3 id="6">6. 加载 scss loader</h3>

目前项目中使用了 Bootstarp 样式库，现在要更改其中的一些基本颜色，比如项目中出现的按钮（primary）颜色，需要对其中的样式代码进行覆盖。

- 在```index.html```文件中删除对 Bootstrap 的引用

index.html 中删除:
```html
<link
  rel="stylesheet"
  href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css"
  integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
  crossorigin="anonymous"
/>
```

可以看到项目中引用的样式消失了：

![delete](./assets/screen-shots/delete.png)

- 将 Bootstrap 下载到本地
```
npm i --save-dev bootstrap
```
package.json:
```js
{
  "devDependencies": {
    "bootstrap": "^4.4.1",
    ...
  }
}
```
Bootstrap 是利用 sass 预处理进行打包的，所以此时在项目中也使用 sass。

- 将之前的```main.css```改为```main.scss```，并引入相应的 Bootstrap文件，改变需要改的颜色。

```
|- src
  |- app
  - index.js
  - main.scss
```

main.scss:
```scss
$primary: teal;
$danger: lightsalmon;

@import "~bootstrap/scss/bootstrap";

body {
  background-color: lightblue;
}
```
[参考](https://getbootstrap.com/docs/4.0/getting-started/webpack/)

根据文档：[sass-loader](https://www.webpackjs.com/loaders/sass-loader/)，我们还需要额外的 loaders 来将```.scss```文件中的代码转换为```.css```文件。

- 安装```sass-loader node-sass```
```
npm i --save-dev sass-loader node-sass
```

package.json:
```js
"devDependencies": {
  ...
  "node-sass": "^4.13.1",
  "sass-loader": "^8.0.2"
}
```

- 修改对应的配置文档

webpack.config.js:
```js
module.exports = {
  ...
  module: {
    rules: [
      {
        test: /\.scss$/,
        use: [
          'style-loader', // 将 js 字符串生成为 style 节点
          'css-loader', // 将 css 转化为 commonjs 模块
          'sass-loader' // 将 sass 编译成 css
        ]
      },
    ]
  }
}
```

- 更改入口文件```index.js```中的引用

index.js：
```js
import './main.scss';
```

- 终端执行```npm start```

- 查看项目，发现样式已经被添加，对应的颜色也有变化
  - error-warning 对应的 danger 颜色（原：淡红）改为了 lightsalmon；
  - input outline 对应 primary 颜色（原：蓝）改为了 teal；
  - button 对应的 primary 颜色被改为了teal。

![background1](./assets/screen-shots/background1.png)

style-loader 生成的```style```标签（连同Bootstrap的样式）：
![style1-tag](./assets/screen-shots/style1.png)

##### [⬆️回到顶部⬆️](#0)
