---
title: Webpack
path: /blog
tags:
  - Javascript
  - Webpack
date: 2019-08-03 11:54:00
thumbnail: ../images/posts/webpack.png
---

<img class="hero" src="https://user-images.githubusercontent.com/39963468/62406370-f7bce200-b5e5-11e9-9352-f110fa8f02b9.png">

<span class="contents">**Contents**</span>

- History of Web Performance and Javascript
  - Why webpack?
  - What is Webpack?
  - Problems with Script Loading
  - IIFE
  - History of Modules
  - History of Modules
- Configuration
- Webpack Bundle Walkthrough
- Loaders
  - Babel Loader
  - CSS Loader
  - Handling Images Using Webpack
- Build a SPA bundling
- Performance With Webpack
  - Code Splitting Example
  - Analyze bundle
- Webpack Dev Server

## History of Web Performance and Javascript

### Why wepback?

웹팩을 왜 사용하는지 알아보려면 웹 생태계 흐름을 조금 살펴보면 이해하기가 좋습니다. 지금과 같이 Single Page Application이 유행하기 전 웹앱에서 서버는 템플릿을 이용해 HTML을 내려주었습니다. Server side templating은 좀 이전의 웹애플리케이션에서 많이 사용되던 방법이에요. 브라우저에서 서버로 요청이 가면 서버에서 작업을 다 하고 완성된 html, css를 내려줘서 화면을 보여줬어요. 그리고 유저가 메뉴나 버튼을 눌러 페이지 이동을 하면, 같은 작업이 반복되는거죠. 서버에 요청이 다시 가고 새로운 html 응답을 보냅니다.

<img style="width: 60%;" src="https://user-images.githubusercontent.com/39963468/94360798-14ba4b80-00eb-11eb-9be3-8f224dced31e.jpg" />

이후 SPA에서 서버는 기본적인 뼈대만 있는 Html을 제공하고, 복잡한 웹 앱의 UI 구성은 자바스크립트 코드를 이용하게 되었어요. 하지만, SPA이 애니메이션이나 여러 비동기 작업 또한 처리하기 때문에 기존의 Server Side Templating에 비해 매우 큰 뭉치의 자바스크립트 코드를 전송하게 되었습니다.

<figure>
<img style="margin-bottom: 0;" src="https://user-images.githubusercontent.com/39963468/95595651-5eaa1680-0a87-11eb-8a4b-56b177ba0cfc.png" />
 <center>
    <figcaption>자바스크립트 사이즈 추이</figcaption> 
 </center>
</figure>

SPA로 넘어오며 코드의 양이 많아지고 파일의 사이즈도 커졌어요. 한 파일의 자바스크립트로만 작업을 하는 것이 어렵기 때문에 모듈화가 필요해 졌고, 웹팩과 같은 번들링 툴의 필요성이 생겼습니다.

### What is webpack?

웹 앱의 규모가 커지고 복잡해 지면서 모듈이 도입됩니다. 모듈을 이용해 코드를 편리하게 나누어 작업할 수 있게 되었어요. 하지만, 여기서도 다른 고려 사항이 생겼습니다. 묘듈화 된 각 자바스크립트 파일이 또다른 자바스크립트 파일에 의존성을 가지기 때문에 코드 실행 순서 등 묘듈간 의존성을 고려해야 합니다. 그리고, 다수의 자바스크립트 파일을 HTTP를 통해 전달하는 것은 성능에도 매우 좋지 않겠죠.

<img src="https://user-images.githubusercontent.com/39963468/94360885-9316ed80-00eb-11eb-971e-cbe2a7b7b3c5.jpg" />

SPA 개발에서 위 문제 해결을 위해 웹팩이 등장했습니다. 웹팩에 여러 좋은 기능이 많지만 코어는 모듈화된 자바크스립트 파일을 올바른 실행 순서를 보장하며 하나의 파일로 만들어 주는 것입니다.

HTTP2를 언급할 수도 있겠지만, 현재의 프론트엔드 앱에서 사용하는 모듈의 수가 너무나 많고 별개의 파일 자바스크립트로 브라우저에 응답을 보낸다고 전체 파알의 사이즈가 작아지지는 않기 때문에 크게 좋은 방법은 아닐거라 생각합니다.

### Problems with Script Loading

자바스크립트를 브라우저에서 로드하는 방법은 두 가지가 있습니다. 스크립트 태그의 속성을 이용해 자바스크립트 파일을 패칭하던가, HTML 파일 내 스크립트 태그에 직접 작성하는 방법이 있습니다.

```html
<script src="/some/javascript/file/path"></script>

// or

<script>
  console.log('Are you sure you wannna write codes here?');
</script>
```

하지만, 위의 방식으로 앱을 작성하면 scope, size, readability에서 문제가 발생합니다. 참고로, 브라우저는 동시에 데이터를 페칭하는 수에 제한이 있어서 많은 수의 스크립트 태그를 생성하면 성능에 문제가 발생할 수 있습니다.

### IIFE

스코프가 겹치는 문제 해결하기 위해 IIFE를 사용해, 함수 단위로 스코프를 형성할 수 있습니다.(캡슐화)

Make, Grunt, Gulp, Broccoli, Brunch, Stealjs 등은 캡술화된 자바스크립트 파일을 연결하는 목적으로 사용됐습니다. 하지만 모든 것이 그렇듯이, 이 방식에도 단점이 있습니다. 단점 중 하나는 Dead code를 거르지 않는다는 것입니다.

### History of Modules

#### commonJS

```
Syntax example
 - require
 - module.exports
```

Node.js 덕분에 자바스크립트 모듈이 생겨났어요. 만약 여러분이 자바스크립트로 앱을 만들어 보았다면 Node.js를 사용했을 것입니다. webpack 또한 node.js를 기반으로 작동합니다. node.js는 commonJS의 모듈을 착용해, 자바스크립트 파일을 모듈화 할 수 있게 해 줍니다.

#### ESM MODULES

```
Syntax example
 - import
 - export
```

NPM은 node 환경에서 개발에 필요한 모듈을 매우 쉽게 설치할 수 있게 해 주는 도구이며, 자바스크립트를 이용한 개발을 쉽고 빠르게 만들어 줍니다. 하지만 여기서 문제점은 commonJS 모듈은 브라우저에서 작동을 하지 않는다는 것입니다.

ECMAScript 모듈을 브라우저에서 사용할 수 있습니다. 이를 통해 캡슐화, 확장성 등 여러 문제를 해결할 수 있습니다. 하지만 브라우저에서 모듈을 사용하는 것은 아직은 매우 느립니다. 런타임에서 모듈이 유효한지 확인 후 import하고, 해당 모듈이 참조하는 다른 모듈에 같은 작업을 거치기 때문이라고 합니다. 현 상황에서는 UX에 좋지 않을 거에요.

#### Webpack

현재 웹 애플리케이션을 제작을 위해 성능상 가장 좋은 방법 중 하나는 webpack을 이용하는 것입니다. webpack은 풍부한 생태계를 가지고 있고, aync bundling을 빌드 과정에서 하며 이 빌드 과정에서 추가 최적화를 할 수 있습니다. Dan Abramov가 2012년 webpack 관련 질문을 stackoverflow에 올렸으며, webpack의 초기 author인 Tobias가 답변한 [링크](https://stackoverflow.com/questions/24581873/what-exactly-is-hot-module-replacement-in-webpack)가 있는데 참고하면 웹팩 공부에 좋을거 같아요. webpack과 redux가 널리 사용되기 전의 내용입니다.

## Configuration

webpack을 실행하기 위해 최소한 두 가지 설정은 필요합니다. 첫 번째는 bundling process가 시작할 위치이며 일반적으로 애플리케이션의 root 파일이 됩니다. entry로 지정한 파일이 앱 실행시 먼저 실행되며, 해당 파일을 기준으로 웹팩은 의존성을 가지는 다른 모듈을 번들링 프로세스에 포함시킵니다.

두 번째로 필수적인 설정 속성은 `output`이에요. [output](https://webpack.js.org/configuration/output/)은 번들링한 파일을 어디에, 어떠한 이름으로 저장할지 지정할 수 있는 속성입니다.

entry와 output만 설정한 configuration을 아래와 같이 작성해 볼 수 있어요.

```javascript
const path = require('path');

const config = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'bundle.js'
  }
};

module.exports = config;
```

실행을 위해서 package.json에 스크립트를 추가해 줍니다. 스크립트를 추가해 실행해야 프로젝트 폴더 내 node_modules 폴더의 webpack으로 실행이 됩니다. 그렇지 않으면 글로벌로 설치된 webpack으로 실행이 되어 상이한 버전으로 인한 오류가 생길 수 있어요.

```json
{
  "build": "webpack"
}
```

Entry로 설정한 index.js에 간단히 코드를 작성하고 빌드를 해 봅시다. 그러면 빌드한 정보가 나오는데 실제 index.js보다 번들링 된 파일의 크기가 큰 것을 확인할 수 있습니다.

<img src="https://user-images.githubusercontent.com/39963468/89664713-8a3b4400-d912-11ea-84d7-57d78edceeb3.png" />

왜 그런지 번들링 된 파일을 살펴보도록 할게요.

<blockquote>
참고참고 
Output의 경우 절대 경로로 지정해 주어야 해요. 또 경로 구분문 문제 등도 있기 때문에 일반적으로 path 모듈과 node 환경에서 제공한는 __dirname을 사용합니다. __dirname은 프로젝트 폴더의 절대 경로를 알려줍니다.

OS 별로 경로 구분자가 다르기 때문에 path 모듈을 일반적으로 사용함.
크게 windows 타입과 POSIX 타입이 있는데, 제가 사용하는 맥은 유닉스 기반 운영체제로 POSIX 타입에 속합니다.

- windows 경로 예시: C:₩Users₩eric
- POSIX 타입 경로 예시: /home/eric
  </blockquote>

## Webpack Bundle Walkthrough

아래 예시의 webpack의 entry 자바스크립트 파일 예시를 만들어 보았습니다. root 요소에 헤더를 추가하는 소소한 예시입니다.

```javascript
// index.js
import renderTo from './lib/renderTo';
import Header from './lib/Header';

renderTo(document.querySelector('#root'), Header);
```

웹팩이 내부적으로 앱에 사용하는 여러 모듈을 연결시켜 주는데, 아래의 코드는 번들링 된 파일의 내부를 간단히 나타내 보았어요. 실제는 각 모듈 함수에 이름은 붙지 않고 IIFE로 구성 돼 있습니다.

```javascript
(function(modules) {
  function __webpack_require__(moduleId) {}
  return __webpack_require__(0);
})([
  function indexJs() {
    // index.js module logic
  },
  function renderTo() {
    // renderTo.js module logic
  },
  function Header() {
    // Header.js module logic
  },
  function greeting() {
    // greeting.js module logic
  }
]);
```

실제 번들링 된 파일은 다른 모듈 시스템 대응이나 모듈 캐싱 등의 여러 부가적인 기능이 추가되어 복잡하지만 기본적인 웹팩의 기능은 이것이에요.

위 index.js를 webpack으로 번들링 했을 때의 [output](https://gist.github.com/ChanwheKim/1bd8fb02d6ad4ac31a31ef732e21936e)을 조금 살펴보겠습니다.

webpack의 mode를 "none"으로 설정하면, 아래와 같이 압축되지 않은 번들링 파일을 확인할 수 있습니다. 가장 바깥쪽을 살펴보면 위에서 형상화한 코드처럼 IIFE가 있고 인자로 배열이 들어가 있습니다. 그리고 배열에는 아직 호출되지 않은 IIFE가 들어있는데 각 IIFE는 index.js 파일에서 import한 모듈입니다.

```javascript
/******/ (function(modules) {
  //... 생략 .../
})(
  /******/ [
    /* 인덱스 0 */
    /***/ function(module, __webpack_exports__, __webpack_require__) {
      //... 생략 .../
    },
    /* 인덱스 1 */
    /***/ function(module, __webpack_exports__, __webpack_require__) {
      //... 생략 .../
    },
    /* 인덱스 2 */
    /***/ function(module, __webpack_exports__, __webpack_require__) {
      //... 생략 .../
    },
    /* 인덱스 3 */
    /***/ function(module, __webpack_exports__, __webpack_require__) {
      //... 생략 .../
    }
  ]
);
```

차레로 위에서부터 코드를 대략적으로만 보면,

```javascript
(function(modules) {
  // 이곳에서 모듈을 캐시합니다.
  var installedModules = {};

  // The require function
  function __webpack_require__(moduleId) {
    // 만약 함수 실행 시 캐시된 모듈이 있다면, 그 모듈의 .exports 속성을 반환합니다.
    if (installedModules[moduleId]) {
      return installedModules[moduleId].exports;
    }

    // 캐시된 것이 없으면, 모듈을 생성하고 변수 module에 담습니다.
    var module = (installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    });

    // 모듈 함수를 실행합니다. 함수 내부에서 다른 모듈이 필요할 수도 있기 때문에 __webpack_require__ 함수를 인자로 전달합니다.
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

    // 모듈이 load 상태를 true로 할당합니다.
    module.l = true;

    // 모듈을 반환합니다.
    return module.exports;
  }

  // ...생략... //
});
```

**webpack_require** 함수가 하는 일은 인자로 전달 받는 배열(IIFE)의 아이디를 받아서 모듈의 exports 속성을 반환합니다.

commonJS는 export default라는 문법이 없고 module.exports를 사용합니다. 아래의 코드는 이러한 차이를 바벨 사용 시 일치시키기 위한 함수입니다.

```javascript
// define __esModule on exports
__webpack_require__.r = function(exports) {
  if (typeof Symbol !== 'undefined' && Symbol.toStringTag) {
    Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
  }
  Object.defineProperty(exports, '__esModule', { value: true });
};
```

그리고 마지막으로 **webpack_require**함수를 실행하는데, 인자로 entry point의 id를 전달해 entry module을 실행합니다.

```javascript
(function(modules) {
  // ... 생략 ... //
  return __webpack_require__((__webpack_require__.s = 0));
})([
  // ... 생략 ... //
]);
```

위에서 살펴본 **webpack_require** 함수가 실행되면 아래 코드가 그 내부에서 호출되어 배열로 전달한 IIFE 모듈 중 전달 받은 id에 해당하는 모듈이 실행됩니다.

```javascript
// __webpack_require__ 함수 내부
modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
```

여기까지 간단한 앱을 webpack이 번들링 한 내용을 살펴보았습니다. 재미있죠? :)

## Loaders

웹팩의 목적은 여러 자바스크립트 모듈을 하나의 파일로 연결 시켜주는 것이라고 언급했어요. 그렇다고 해도, 웹팩에는 유용하며 실제 개발 환경에서 꼭 사용하는 여러 기능들이 있습니다.

우선 웹팩에는 Loader가 있습니다. Loader는 특정 파일이 bundle.js에 포함되기 전 preprocessing을 해 줍니다. 설정은 아래와 같이 해 줄 수 있어요. 문서에 좀 더 소상히 적혀있습니다.

```javascript
const config = {
  module: {
    rules: [
      { test: /\.ts$/, use: “ts-loader” },
      { test: /\.js$/, use: “babel-loader” },
    ],
  },
}
```

보편적으로 많이 사용하는 Loader로 babel-loader가 있습니다.

### Babel Loader

Babel Loader는 Babel이 웹팩과 호환 될 수 있도록 레이어를 생성하는 역할을 해요. 실제 코드는 @babel/core가 변환해 줍니다. 하지만 `@babel/core`는 코드를 어떠한 기준으로 변환해야 하는지 모르기 때문에 `@babel/preset-env`이 그 역할을 해 줍니다. `@babel/preset-env`에 컴파일 룰을 별도로 설정해 주어야 해요. 해서, 기본적으로 이 세 가지는 install이 필요합니다.

```
yarn add babel-loader @babel/core @babel/preset-env --dev
```

로더에 어떠한 파일도 적용할 수 있지만, 기본적으로 해당 로더의 목적에 맞는 파일에 적용되도록 설정을 해 줍니다. 웹팩 번들링 시 아래와 같이 js 파일에 babel-loader를 설정할 수 있습니다.

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.(js)$/,
        loader: 'babel-loader',
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                targets: {
                  browsers: ['last 2 versions']
                }
              }
            ]
          ]
        }
      }
    ]
  }
};
```

### CSS Loader

웹팩을 이용해 CSS를 처리하면, 여러 CSS 파일을 만들어 작업할 수 있고 필요한 자바스크립트 파일에서 import 후 사용할 수 있습니다.

웹팩을 이용해 CSS를 처리하기 위해서 `css-loader`, `style-loader` 두 개의 로더가 필요합니다. `css-loader`는 웹팩이 css 파일을 parsing 할 수 있도록 해 줍니다. `style-loader`는 css-loader가 전달한 데이터를 HTML의 style 태그에 추가해 주는 역할을 합니다.

webpack 설정 파일의 module.rules에 아래와 같이 설정할 수 있습니다.

```javascript
  {
    test: /\.css$/,
    use: ["style-loader", "css-loader"]
  }
```

로더는 오른쪽부터 실행 되기 때문에, 위 순서에 맞춰서 두 로더를 추가해 주어야 해요.

웹팩 설정에서 index.html과 어떠한 연관도 짓지 않았는데, 번들링 파일을 통해서 html header 태그에 우리가 모듈에서 import 한 css를 추가해 주었습니다. 실제 번들링 된 파일을 살펴보면 아래와 같지요.

```javascript
var ___CSS_LOADER_EXPORT___ = _node_modules_css_loader_dist_runtime_api_js__WEBPACK_IMPORTED_MODULE_0___default()(
  false
);
// Module
___CSS_LOADER_EXPORT___.push([module.i, 'img {\n  border: 10px solid black;\n}\n', '']);
```

위와 같이 코드를 통해 스타일을 헤더에 추가해 줄 수도 있지만, [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)를 사용하면 별도의 스타일 파일로 생성할 수도 있습니다.

### Handling Images Using Webpack

애플리케이션에 이미지를 추가할 때 웹팩을 통해서도 빌드 과정에서 이미지를 처리할 수 있는데, 이를 통해 성능 개선을 할 수 있습니다. 이를 위해 url-loader가 필요합니다. [url-loader](https://webpack.js.org/loaders/url-loader/)를 사용하면 제한 사이즈를 설정해, 해당 사이즈 밑으로는 이미지를 번들 파일에 포함할 수 있고, 그렇지 않으면 별도의 이미지 파일로 생성할 수 있습니다.

```javascript
  {
    test: /\.(jpe?g|png|svg)$/,
    loader: [
      { loader: 'url-loader', options: { limit: 10240 } },
      'image-webpack-loader',
    ],
  },
```

이후 limit보다 작은 사이즈의 이지미를 애플리케이션에 포함 후 빌드를 하면 번들된 파일에서 base64로 인코딩 된 이미지를 확인할 수 있습니다.

```javascript
var _866_200x300 = 'data:image/jpeg;base64,/9j....생략';
```

로더에 대해서는 이 정도 개념만 알아도, 문서 보며 적용할 수 있을거에요. 각 모듈 타입 별 어떠한 전처리 작업이 필요할 때 사용한다는 것만 알고 있으면 될 거 같습니다.

## Build a SPA bundling

이제 클라인언트 사이드에서 실행될 리액트 앱의 번들링 파일을 생성해 보도록 할게요. 우선 리액트 앱 번들을 위해 필요한 최소한의 모듈을 npm 또는 yarn을 이용해 설치해 주세요. 저는 아래와 같이 설치해 주었습니다.

```json
{
  "dependencies": {
    "react": "^16.13.1",
    "react-dom": "^16.13.1"
  },
  "devDependencies": {
    "@babel/core": "^7.11.6",
    "@babel/preset-env": "^7.11.5",
    "@babel/preset-react": "^7.10.4",
    "babel-loader": "^8.1.0",
    "css-loader": "^0.26.4",
    "style-loader": "^0.13.2",
    "webpack": "^4.44.1",
    "webpack-cli": "^3.3.12"
  }
}
```

그리고 index.html 파일도 생성해 주세요. 폴더 구조는 아래와 같아요.

```
Super Simple React App Folder
|__ node_modules
|__ src
|    |__ App.jsx
|    |__ Greeting.jsx
|    |__ Header.css
|    |__ Header.jsx
|    |__ index.jsx
|__ index.html
|__ package.sjon
|__ webpack.config.js
```

그리고 앞서 살펴본 babel-loader를 먼저 추가할 거에요. 리액트 앱을 번들링 하는 작업이기 때문에 jsx를 일반 자바스크립트 코드로 변환을 위해 `@babel/preset-react`를 추가해 주었습니다.

```javascript
module.exports = {
  entry: './src/index.jsx',

  module: {
    rules: [
      {
        test: /\.(js|jsx)$/,
        loader: 'babel-loader',
        exclude: /node_modules/,
        options: {
          presets: [
            [
              '@babel/preset-env',
              {
                targets: {
                  browsers: ['last 2 versions']
                }
              }
            ],
            '@babel/preset-react'
          ]
        }
      }
    ]
  },

  output: {
    path: path.join(__dirname, 'dist'),
    filename: 'bundle.js'
  }
};
```

추가로 style-loader와 css-loader도 추가해 줍시다. 여기까지 작성 후 터미널에서 webpack을 입력하면 dist 폴더에 bundle 된 자바스크립트 파일을 확인해 볼 수 있어요.

## Webpack Dev Server

개발환경에서 코드를 수정할 때마다 확인을 위해 매번 webpack을 실행시키는 작업은 생산성을 떨어뜨릴거에요. 웹팩의 흐름은 자바스크립트와 기타 파일을 번들링 후 브라우저에게 줍니다. 브라우저는 전달 받은 스크립트 태그의 소스를 우리의 프로젝트 폴더로 요청을 합니다. 이러한 작업을 수월하게 해 주는 툴이 이 webpack-dev-server입니다.

Webpack-dev-server는 브라우저와 output의 중간에서 매개체 역할을 합니다. 메모리 상에 번들 데이터를 가지고 있고, output된 내용을 지켜보다 일부가 바뀌면 해당 부분만 빌드를 다시 해 Express를 이용해 브라우저에 전달합니다. 추가로, 인증이나 데이터베이스 접근, web socket 등의 작업이 필요하면 어떠한 흐름으로 처리하게 되는지 알아봐도 좋을거 같네요.

<img src="https://user-images.githubusercontent.com/39963468/95603382-27d8fe00-0a91-11eb-8fb9-873fa82710fa.jpg" />

## Performance with Webpack

웹 애플리케이션의 첫 랜딩 페이지가 유저에게 유의미한 화면을 보여주고, 유저와 상호 작용할 수 있는 상태가 얼마나 빨리 되는 지는 클라이언트 개발에서 중요한 요소입니다. 플랫폼 이슈를 제외하고 여기에 크게 영향을 미치는 요소를 몇 개 나열해 보면 아래와 같아요.

```
- 초기 진입 시 자바스크립트 다운로드 크기
- 초기 진입 시 CSS 다운로드 크기
- 초기 진입 시 네트워트 요청 수
```

이 중 자바스크립트의 영향이 가장 큽니다. 아래의 자료는 인터넷에서 찾은 랜딩 페이지 성능과 관련된 이상적인 수치라고 합니다.

```
                         이상적인 목표

- 초기 자바스크립트 파일 200kb 이하 (uncompressed)
- 초기 css 파일 100kb 이하 (uncompressed)
- 90% code coverage
```

모바일 디바이스나 네트워크 환경이 좋지 않은 국가를 대상으로 앱을 만든다면, 충족은 못 시켜도 위의 기준을 염두에 두고 작업을 해봐도 좋을거에요.

Code Spliting은 웹팩의 가장 큰 장점 중 하나입니다. Code Splitting이란 비동기로 로드할 수 있는 별개의 자바스크립트 chunk를 생성하는 것을 말해요. 그리고 <strong>이 작업은 빌드 시 진행됩니다.</strong> Dynamic이란 용어 때문에 혼동될 수 있는데, 현재 기준으로(2020.10) 런타임 환경 조건에 따른 Code Splitting은 할 수 없습니다. Dynamic이라고 해도 빌드 시 조건에 의해서만 Code Splitting을 할 수 있어요.

Code Splitting은 클라이언트에 Initial Page에서 사용 안 하는 코드는 전달하지 않기 위해 사용합니다. 이유는 빠르게 유저에게 UI를 보여주기 위해서입니다.

예로, 애플리케이션의 로그인 페이지에 유저가 접근했다고 가정해 봅시다. 이 로그인 페이지에는 로그인 폼 한 개만 UI에 존재하고 이에 대한 로직만 필요할 거에요. Code Splitting을 하면 지금 필요한 최소한의 코드만 로드를 하고, 유저가 로그인 후 애플리케이션 작동에 필요한 나머지 로직을 로드할 수 있습니다.

<figure>
<img style="margin-bottom: 0px;" src="https://user-images.githubusercontent.com/39963468/95746850-70cbc500-0cd2-11eb-8099-0413d2460579.png" />
 <center>
    <figcaption>로그인 페이지 관련 로직만 내려줘도 좋음</figcaption> 
 </center>
</figure>

그래서 아래와 같은 케이스에서 Code Splitting을 고려해 볼 수 있어요.

- 클라이언트 앱의 사이즈가 큰 경우
- Page 내에서 조건에 의해 렌더링 하는 컴포넌트
- 각 Page 별 라우트 등

### Code Splitting Example

Code Splitting을 하기 위해서는 현재 stage 4에 있는 dynamic import를 사용해야 해요.

```javascript
// webpack's entry file
import getButton from './lib/getButton';
import Header from './lib/Header';

import './styles/styles.css';

const button = getButton('Click');
const root = document.getElementById('root');

root.appendChild(Header);
root.appendChild(button);

button.addEventListener('click', () => {
  import('./lib/puppy').then(puppyModule => {
    root.appendChild(puppyModule.default);
  });
});
```

아래와 같이 페이지 로드 후 유저가 버튼 클릭 시 dynamic import로 split된 chunk를 가져와서 그려주도록 구성했습니다. 버튼 클릭 시 요청을 통해 필요한 모듈을 추가로 불러오는 것은 콘솔을 통해 확인할 수 있습니다. 웹팩은 처음에는 jsonp를 통해 모듈을 가져오지만, 한 번 패치한 모듈은 메모리에 캐시하기 때문에 두 번째 부터는 패치를 하지 않습니다.

<img src="https://user-images.githubusercontent.com/39963468/95463990-6b116f00-09b4-11eb-9181-79d2da284a75.gif" />

dynamic import로 불러오는 모듈은 번들된 파일 내부에서 jsonp 요청으로 로드하는 것을 확인해 볼 수 있습니다.

```javascript
// bundle.js
// start chunk loading
var script = document.createElement('script');
var onScriptComplete;

script.charset = 'utf-8';
script.timeout = 120;

if (__webpack_require__.nc) {
  script.setAttribute('nonce', __webpack_require__.nc);
}

script.src = jsonpScriptSrc(chunkId);

/** 생략 **/

script.onerror = script.onload = onScriptComplete;

document.head.appendChild(script);
```

### Analyze bundle

초기 진입 시 내려주는 자바스크립트의 coverage를 확인하기 위해 크롬 개발자 툴의 coverage를 이용할 수 있어요. 비교를 위해 위 강아지 코드에서 lodash를 import 후 coverage 확인을 해 봅니다. import 후 미사한 상태에서 확인해 보면 아래와 같이 각 파일 별 coverage를 참고할 수 있어요.

<img src="https://user-images.githubusercontent.com/39963468/95557088-44544680-0a4f-11eb-8ed5-291172a43b5e.png" />

해당 모듈을 아래처럼 변경하고 다시 확인해 보면, 초기 진입 시 번들 파일에 포함하지 않는 것을 확인할 수 있어요. 글고, 사용 시 로드하는 것을 확인해 볼 수 있습니다.

```javascript
import getButton from './lib/getButton';
import Header from './lib/Header';

// import lodash from "lodash";
const lodash = () => import('lodash');
```

이상이어요.

## Reference

- [What exactly is Hot Module Replacement in Webpack?](https://stackoverflow.com/questions/24581873/what-exactly-is-hot-module-replacement-in-webpack)
- [The cost of javascript in 2018](https://medium.com/@addyosmani/the-cost-of-javascript-in-2018-7d8950fbb5d4))
