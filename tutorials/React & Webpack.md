이 가이드는 TypeScript를 [React](https://reactjs.org/) 및 [webpack](https://webpack.js.org/)에 연결하는 방법을 알려줍니다.

새로운 프로젝트를 시작하는 경우, 먼저 [React Quick Start guide](https://create-react-app.dev/docs/adding-typescript)를 살펴보세요.

그렇지 않으면 이미 [npm](https://www.npmjs.com/)과 함께 [Node.js](https://nodejs.org/)를 사용하고 있다고 가정합니다.

# 프로젝트 배치 (Lay out the project)

새 디렉토리부터 시작하겠습니다.
지금은 이름을 `proj`라고 지정하지만, 원하는대로 변경할 수 있습니다.

```shell
mkdir proj
cd proj
```

시작하기 위해, 다음과 같은 방식으로 프로젝트를 구성하겠습니다:

```text
proj/
├─ dist/
└─ src/
   └─ components/
```

TypeScript 파일은 `src` 폴더에서 시작하여, TypeScript 컴파일러를 통해 실행한 다음, webpack을 거쳐 `dist`의 `main.js` 파일로 끝납니다.
우리가 작성하는 모든 컴포넌트는 `src/components` 폴더 안에 있습니다.

이것을 기본 뼈대로 구성합니다:

```shell
mkdir src
cd src
mkdir components
cd ..
```

Webpack으로 마지막엔 `dist`폴더를 생성할 것입니다.

# 프로젝트 초기화 (Initialize the project)

이 폴더에 npm 패키지를 설정합니다.

```shell
npm init -y
```

기본값으로 `package.json` 파일이 생성됩니다.

# 의존성 설치 (Install our dependencies)

먼저 Webpack이 설치되어 있는지 확인합니다.

```shell
npm install --save-dev webpack webpack-cli
```

Webpack은 코드와 선택적으로 모든 의존성을 하나의 `.js`파일로 묶는 도구입니다.

이제 선언 파일과 함께 React 및 React-DOM을 `package.json` 파일에 의존성으로 추가하겠습니다:

```shell
npm install --save react react-dom
npm install --save-dev @types/react @types/react-dom
```

`@types/` 접두사는 React와 React-DOM의 선언 파일을 가져오고 싶다는 것을 의미합니다.
일반적으로 `"react"`와 같은 경로를 가져오면, `react` 패키지 자체를 살펴볼 것입니다;
그러나 모든 패키지에 선언 파일이 포함되어 있지 않기 때문에, TypeScript는 `@types/react` 패키지도 찾습니다.
나중에는 이것에 대해 생각할 필요가 없다는 것을 알 수 있습니다.

다음으로, 개발 시 필요한 의존성에 [ts-loader](https://www.npmjs.com/package/ts-loader)와[source-map-loader](https://www.npmjs.com/package/source-map-loader)를 추가합니다.

```shell
npm install --save-dev typescript ts-loader source-map-loader
```

이 두 가지 의존성 모두 TypeScript와 Webpack을 함께 사용할 수 있습니다.
ts-loader는 Webpack이 `tsconfig.json`이라는 TypeScript 표준 구성 파일을 사용하여 TypeScript 코드를 컴파일하도록 도와줍니다.
source-map-loader는 TypeScript의 소스 맵 출력을 사용하여 *고유한* 소스 맵을 생성할 때 Webpack에 알립니다.
이렇게 하면 기존의 TypeScript 소스 코드를 디버깅하는 것 처럼 최종 출력 파일을 디버깅 할 수 있습니다.

ts-loader는 TypeScript의 유일한 로더는 아닙니다.

개발 의존성으로 TypeScript를 설치했습니다.
`npm link typescript`를 사용하여 TypeScript를 전역 복사본에 연결할 수도 있지만, 덜 일반적인 시나리오입니다.

# TypeScript 구성 파일 추가 (Add a TypeScript configuration file)

작성하려는 코드와 필요한 선언 파일 모두 TypeScript 파일로 가져오기를 원할 것입니다.

이렇게 하려면, 입력 파일 목록과 모든 컴파일 설정을 포함하는 `tsconfig.json` 파일을 만들어야 합니다.
프로젝트 루트에 `tsconfig.json`이라는 새 파일을 생성하고, 다음 내용을 채우세요.

```json
{
    "compilerOptions": {
        "outDir": "./dist/",
        "sourceMap": true,
        "noImplicitAny": true,
        "module": "commonjs",
        "target": "es6",
        "jsx": "react"
    }
}
```

`tsconfig.json` 파일에 대한 자세한 내용은 [여기](./tsconfig.json.md)를 참조하세요.

# Write some code

Let's write our first TypeScript file using React.
First, create a file named `Hello.tsx` in `src/components` and write the following:

```ts
import * as React from "react";

export interface HelloProps { compiler: string; framework: string; }

export const Hello = (props: HelloProps) => <h1>Hello from {props.compiler} and {props.framework}!</h1>;
```

Note that while this example uses [function components](https://reactjs.org/docs/components-and-props.html#functional-and-class-components), we could also make our example a little *classier* as well.

```ts
import * as React from "react";

export interface HelloProps { compiler: string; framework: string; }

// 'HelloProps' describes the shape of props.
// State is never set so we use the '{}' type.
export class Hello extends React.Component<HelloProps, {}> {
    render() {
        return <h1>Hello from {this.props.compiler} and {this.props.framework}!</h1>;
    }
}
```

Next, let's create an `index.tsx` in `src` with the following source:

```ts
import * as React from "react";
import * as ReactDOM from "react-dom";

import { Hello } from "./components/Hello";

ReactDOM.render(
    <Hello compiler="TypeScript" framework="React" />,
    document.getElementById("example")
);
```

We just imported our `Hello` component into `index.tsx`.
Notice that unlike with `"react"` or `"react-dom"`, we used a *relative path* to `Hello.tsx` - this is important.
If we hadn't, TypeScript would've instead tried looking in our `node_modules` folder.

We'll also need a page to display our `Hello` component.
Create a file at the root of `proj` named `index.html` with the following contents:

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8" />
        <title>Hello React!</title>
    </head>
    <body>
        <div id="example"></div>

        <!-- Dependencies -->
        <script src="./node_modules/react/umd/react.development.js"></script>
        <script src="./node_modules/react-dom/umd/react-dom.development.js"></script>

        <!-- Main -->
        <script src="./dist/main.js"></script>
    </body>
</html>
```

Notice that we're including files from within `node_modules`.
React and React-DOM's npm packages include standalone `.js` files that you can include in a web page, and we're referencing them directly to get things moving faster.
Feel free to copy these files to another directory, or alternatively, host them on a content delivery network (CDN).
Facebook makes CDN-hosted versions of React available, and you can [read more about that here](http://facebook.github.io/react/downloads.html#development-vs.-production-builds).

# Create a webpack configuration file

Create a `webpack.config.js` file at the root of the project directory.

```js
module.exports = {
    mode: "production",

    // Enable sourcemaps for debugging webpack's output.
    devtool: "source-map",

    resolve: {
        // Add '.ts' and '.tsx' as resolvable extensions.
        extensions: [".ts", ".tsx"]
    },

    module: {
        rules: [
            {
                test: /\.ts(x?)$/,
                exclude: /node_modules/,
                use: [
                    {
                        loader: "ts-loader"
                    }
                ]
            },
            // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
            {
                enforce: "pre",
                test: /\.js$/,
                loader: "source-map-loader"
            }
        ]
    },

    // When importing a module whose path matches one of the following, just
    // assume a corresponding global variable exists and use that instead.
    // This is important because it allows us to avoid bundling all of our
    // dependencies, which allows browsers to cache those libraries between builds.
    externals: {
        "react": "React",
        "react-dom": "ReactDOM"
    }
};
```

You might be wondering about that `externals` field.
We want to avoid bundling all of React into the same file, since this increases compilation time and browsers will typically be able to cache a library if it doesn't change.

Ideally, we'd just import the React module from within the browser, but most browsers still don't quite support modules yet.
Instead libraries have traditionally made themselves available using a single global variable like `jQuery` or `_`.
This is called the "namespace pattern", and webpack allows us to continue leveraging libraries written that way.
With our entry for `"react": "React"`, webpack will work its magic to make any import of `"react"` load from the `React` variable.

You can learn more about configuring webpack [here](https://webpack.js.org/concepts).

# Putting it all together

Just run:

```shell
npx webpack
```

Now open up `index.html` in your favorite browser and everything should be ready to use!
You should see a page that says "Hello from TypeScript and React!"
