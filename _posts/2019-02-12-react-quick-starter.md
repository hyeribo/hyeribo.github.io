---
title: "[ReactJS] create-react-app 사용하지 않고 리액트 시작하기"
date: 2019-02-12 20:08:00 +0900
categories: react
---

리액트 프로젝트를 시작할때, 제일 빠르고 간단한 방법은 create-react-app을 사용하는것이다.  
(참고: [https://facebook.github.io/create-react-app/docs/getting-started](https://facebook.github.io/create-react-app/docs/getting-started))

근데 나는 이거 쓰기 싫다. 의존성 모듈이 너무 많고 웹팩 설정도 직접 해주는데, 코드가 너무 길어서 커스텀 할 엄두가 안난다. 그러니까 처음부터 만들어보겠다.

기본적으로 사용할 기능은 대략 아래와 같다.

- 리액트 v16.8.1~
- 웹팩 v4
- 바벨 v7
- react-hot-loader v4.5.4~

<br/>

## 1. 프로젝트 생성 & npm init
```
$ mkdir react-quick-starter
$ cd react-quick-starter
$ npm init -y
```
- -y : default 설정으로 즉시 init

<br/>

## 2. 리액트 모듈 설치
```
$ npm install --save react react-dom react-hot-loader
```
- --save : package.json의 dependencies에 해당 모듈을 추가한다.
- react : 리액트 라이브러리
- react-dom : 리액트 컴포넌트가 DOM에 접근할 수 있게 해주는 라이브러리. 항상 react 모듈과 동일한 버전을 유지하자.
- react-hot-loader : hot loader 설치. 프로덕션 모드일때는 자동으로 인식하여 실행되지 않기때문에 --save 옵션을 사용해도 된다.

<br/>

## 3. 웹팩 패키지 설치
```
$ npm install --save-dev webpack webpack-cli webpack-dev-server
```
- --save-dev : package.json의 devDependencies에 해당 모듈을 추가한다. 프로덕션 배포시 install 되지 않는다.
- webpack : 여러 코드를 브라우저가 이해할 수 있도록 변환해주고, 번들링해준다. (참고: [웹팩4 설정하기](/development/what-is-webpack/)) 
- webpack-cli : webpack 명령어를 사용할수 있게 해준다. v4 부터 필수임.
- webpack-dev-server : 개발단계에서 hot-reloading이 가능한 웹서버를 실행시켜준다.

``` 
$ npm install --save-dev css-loader style-loader url-loader file-loader html-webpack-plugin
```
- css-loader, style-loader : css 코드를 다룬다.
- url-loader : 크기가 작은 이미지나 글꼴 파일 등을 base64로 변환하여 번들 파일에 포함시킨다.
- file-loader : 파일을 다룬다.
- html-webpack-plugin : 번들링할때 html파일을 생성해준다. 컴파일 할 때마다 파일 이름에 해시가 포함되는경우 유용하다.

<br/>

## 4. 바벨 모듈 설치
바벨 v6 이하에서는 바벨 패키지들을 babel-[패키지명] 으로 install했지만, 바벨 v7 부터는 바벨 패키지들이 @babel라는 namespace 안에 속하게 되어 @babel/[패키지명] 으로 install하게 되었다.

```
$ npm install --save @babel/polyfill
```
- babel-polyfill : [regenerator runtime](https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js)와 [core-js](https://github.com/zloirock/core-js)를 내부적으로 래핑한 모듈. 참고: [(번역)Babel에 대한 모든 것](https://jaeyeophan.github.io/2017/05/16/Everything-about-babel/#babel-polyfill)


```
$ npm install --save-dev babel-loader @babel/core @babel/preset-env @babel/preset-react
```
- @babel/preset-env : 브라우저에 필요한 자바스크립트 버전을 알아서 파악해서 polyfill을 넣어준다. (stage-x 플러그인을 지원하지 않으니 참고하자. [링크](https://babeljs.io/blog/2018/07/27/removing-babels-stage-presets))

<br/>

## 5. 디렉토리 생성
```
/src
/src/assets
/src/components
/dist
/dist/favicon.ico (파비콘 파일을 추가한다. 샘플 파일: https://reactjs.org/favicon.ico)
```

<br/>

## 6. 파일 생성

**webpack.config.js** : 웹팩 설정 파일

```
const webpack = require("webpack");
const path = require("path");
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = env => ({
  mode: env.mode,
  entry: {
    bundle: ['@babel/polyfill', path.resolve(__dirname, "src", "app.js")],
    vendor: [
      '@babel/polyfill',
      'react',
      'react-dom',
    ]
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    publicPath: '/',
    filename: '[name].[hash].js'
  },
  devServer: {
    inline: true,
    contentBase: path.resolve(__dirname, "dist"),
    historyApiFallback: true,
    open: true,
    // hot:true
    // compress: true,
    // port: 9000
  },
  resolve: {
    extensions: [".js", ".jsx"],
    alias: {
      "@root": path.resolve('src'),
      "@assets": path.resolve('src/assets'),
      "@components": path.resolve('src/components')
    }
  },
  plugins: [
    new webpack.DefinePlugin({
      'process.env.NODE_ENV': JSON.stringify(env.mode)
    }),
    new HtmlWebpackPlugin({
      template: 'src/index.html',
      favicon: 'dist/favicon.ico'
    })
  ],
  devtool: env.mode === 'development' ? "source-map" : '',
  optimization: {
    splitChunks: {
      cacheGroups: {
        vendor: {
          chunks: 'initial',
          test: 'vendor',
          name: 'vendor',
          enforce: true
        }
      }
    }
  },
  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ['babel-loader']
      },
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.(png|jpg)$/i,
        use: [
          {
            loader: "url-loader",
            options: {
              name: 'img/[hash].[ext]',
              limit: 10000
            }
          }
        ]
      }
    ]
  }
});
```

<br/>

**.babelrc** - 바벨 설정파일

```
{
  "presets": [
    "@babel/env",
    "@babel/react"
  ],
}
```

<br/>

**src/index.html**
 
```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="initial-scale=1, width=device-width"/>
  <title>React Quick Starter</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>

```

<br/>

**src/app.js**

```
import React from 'react';

const Home = () => (
  <div>
    Hello React!
  </div>
)

export default Home;
```
- 리액트 v16.8.1 부터는 **React Hooks**가 정식 릴리즈되었으므로 class형 컴포넌트를 사용하지 말아보자!


<br/>
**src/components/Home.js**

```
import ReactDOM from 'react-dom';
import React from 'react';
import { hot } from 'react-hot-loader/root';

import Home from '@components/Home';

const render = Home => {
  const App = hot(Home);
  ReactDOM.render(<App/>, document.getElementById("root"));
}
render(Home);
```
- react-hot-loader : 변경사항을 Live-reload 한다.
- react-hot-loader/root : react-hot-loader v4.5.4 버전부터는 `AppContainer`가 아닌 `react-hot-loader/root`의 `hot`을 사용한다. ([다른 방법으로 react-hot-loader 사용하기](https://www.npmjs.com/package/react-hot-loader#getting-started))


<br/>


## 7. package.json에 script 추가
```
"scripts": {
  "start": "webpack-dev-server --env.mode=development",
  "build": "webpack --env.mode=production",
  "test": "echo \"Error: no test specified\" && exit 1"
}

```

<br/>

이것으로 리액트 프로젝트를 시작하기위한 기본 환경구성을 끝냈다.

참고 소스코드: [https://github.com/hr5959/react-quick-starter](https://github.com/hr5959/react-quick-starter)
