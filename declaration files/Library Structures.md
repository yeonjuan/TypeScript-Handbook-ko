# 개요 (Overview)

일반적으로, 선언 파일을 *구조화*하는 방법은 라이브러리를 사용하는 방법에 따라 다릅니다.
JavaScript에서 사용할 라이브러리를 제공하는 방법은 여러 가지가 있고, 이에 맞추어 선언 파일을 작성해야 합니다.
이 가이드는 일반적인 라이브러리 패턴을 식별하는 방법과, 그 패턴에 상응하는 선언 파일을 작성하는 방법에 대해 다룹니다.

주요 라이브러리 각각의 구조화 패턴 유형은 [템플릿](./Templates.md) 섹션에 있습니다.
이 템플릿으로 시작하면 더 빠르게 진행할 수 있습니다.

# 라이브러리 종류 식별하기 (Identifying Kinds of Libraries)

첫 번째로, TypeScript 선언 파일이 나타낼 수 있는 라이브러리 종류를 다뤄보겠습니다.
각 종류의 라이브러리를 *사용하는* 방법과, *작성하는* 방법, 그리고 실제 라이브러리들의 예제를 볼 것입니다.

라이브러리의 구조를 식별하는 것은 선언 파일을 작성하는 첫 단계입니다.
*사용법*과 *코드*를 기반으로 구조를 식별하는 방법에 대한 힌트를 제공합니다.
라이브러리의 문서와 구성에 따라서, 어떤 건 다른 것보다 훨씬 쉬울 수 있습니다.
본인에게 더 편한 것을 사용할 것을 추천합니다.

## 전역 라이브러리 (Global Libraries)

*전역* 라이브러리는 전역 스코프 (즉, `import` 형식을 사용하지 않음)에서 접근 가능한 라이브러리입니다.
많은 라이브러리는 사용을 위해 간단히 하나 이상의 전역 변수를 노출합니다.
예를 들어, [jQuery](https://jquery.com/)를 사용한다면, `$` 변수를 참조해서 사용할 수 있습니다:

```ts
$(() => { console.log('hello!'); } );
```

HTML 스크립트 태그로 라이브러리를 사용하는 방법은 라이브러리 문서에서 지침을 볼 수 있습니다:

```html
<script src="http://a.great.cdn.for/someLib.js"></script>
```

오늘날, 대부분의 전역에서 접근 가능한 유명 라이브러리들은 실제로 UMD 라이브러리로 작성되어 있습니다 (아래를 참조).
UMD 라이브러리 문서는 전역 라이브러리 문서와 구별하기 어렵습니다.
전역 선언 파일을 작성하기 전에, 실제로는 UMD가 아닌지 확인하십시오.

### 코드에서 전역 라이브러리 식별하기 (Identifying a Global Library from Code)

전역 라이브러리 코드는 대게 엄청 간단합니다.
전역 "Hello, world" 라이브러리는 다음과 같습니다:

```js
function createGreeting(s) {
    return "Hello, " + s;
}
```

혹은 다음과 같습니다:

```js
window.createGreeting = function(s) {
    return "Hello, " + s;
}
```

전역 라이브러리의 코드를 보면, 보통 다음을 볼 수 있습니다:

* 최상위 레벨 `var`문 이나 `function`선언
* 하나 이상의 `window.someName` 할당
* DOM 인터페이스 `document` 혹은 `window`가 존재한다고 가정

다음은 볼 수 *없습니다*:

* `require` 이나 `define` 같은 모듈 로더 검사 혹은 사용
* `var fs = require("fs");` 형태의 CommonJS/Node.js-스타일 import
* `define(...)` 호출
* 라이브러리를 `require` 혹은 import하는 방법에 대해 설명하는 문서

### 전역 라이브러리 예제 (Examples of Global Libraries)

전역 라이브러리를 UMD 라이브러리로 바꾸는게 쉽기 때문에, 전역 스타일로 작성한 인기 라이브러리는 거의 없습니다.
하지만, 크기가 작고 DOM이 필요한 (혹은 의존성이 *없는*) 라이브러리는 여전히 전역입니다.

### 전역 라이브러리 템플릿 (Global Library Template)

템플릿 파일 [`global.d.ts`](./templates/global.d.ts.md)은 예제 라이브러리 `myLib`를 정의합니다.
["이름 충돌 방지" 각주](#preventing-name-conflicts)를 반드시 읽어보세요.

## 모듈형 라이브러리 (Modular Libraries)

어떤 라이브러리는 모듈 로더 환경에서만 동작합니다.
예를 들어, `express`는 Node.js에서만 동작하고 반드시 CommonJS의 `require` 함수로 로드되어야 합니다.

ECMAScript 2015 (ES2015, ECMAScript 6, ES6로도 잘 알려진), CommonJS와 RequireJS는 *모듈*을 *importing*하는 비슷한 개념을 가지고 있습니다.
JavaScript의 CommonJS (Node.js)를 예를 들면, 다음과 같이 작성합니다

```js
var fs = require("fs");
```

TypeScript나 ES6에서는, `import` 키워드가 같은 목적을 제공합니다:

```ts
import fs = require("fs");
```

일반적으로 모듈형 라이브러리의 문서에서 다음 코드들 중 하나를 볼 수 있습니다:

```js
var someLib = require('someLib');
```

혹은

```js
define(..., ['someLib'], function(someLib) {

});
```

전역 모듈과 마찬가지로 UMD 모듈의 문서에서도 이 예제들을 볼 수 있으므로, 코드나 문서를 반드시 확인하세요.

### 코드에서 모듈 라이브러리 식별하기 (Identifying a Module Library from Code)

모듈형 라이브러리는 일반적으로 다음 중 몇 가지를 반드시 가지고 있습니다:

* `require` 혹은 `define`에 대한 무조건적인 호출
* `import * as a from 'b';` 혹은 `export c;` 같은 선언문
* `exports` 혹은 `module.exports`에 대한 할당

다음과 같은 것은 거의 갖지 않습니다:

* `window` 혹은 `global` 프로퍼티에 대한 할당

### 모듈형 라이브러리의 예제 (Examples of Modular Libraries)

많은 유명한 Node.js 라이브러리들은 [`express`](http://expressjs.com/), [`gulp`](http://gulpjs.com/), [`request`](https://github.com/request/request)와 같은 모듈군 안에 있습니다.

## *UMD*

*UMD* 모듈은 모듈로 (import를 통해) 사용할 수 있고 혹은 전역으로도 (모듈 로더 없는 환경에서 실행될 때) 사용할 수 있습니다.
[Moment.js](http://momentjs.com/) 같은 많은 유명한 라이브러리들은 이 방법으로 작성되었습니다.
예를 들어, Node.js나 RequireJS를 사용하면, 다음과 같이 작성합니다:

```ts
import moment = require("moment");
console.log(moment.format());
```

반면에 바닐라 브라우저 환경에서는 다음과 같이 쓸 수 있습니다:

```js
console.log(moment.format());
```

### UMD 라이브러리 식별하기 (Identifying a UMD library)

[UMD modules](https://github.com/umdjs/umd)은 모듈 로더 환경의 유무를 검사합니다.
이는 다음과 같이 보이는 찾기 쉬운 패턴입니다.

```js
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["libName"], factory);
    } else if (typeof module === "object" && module.exports) {
        module.exports = factory(require("libName"));
    } else {
        root.returnExports = factory(root.libName);
    }
}(this, function (b) {
```

만약 라이브러리의 코드, 특히 파일 상단에서 `typeof define`, `typeof window` 혹은 `typeof module`에 대한 테스트를 보았다면, 거의 대부분 UMD 라이브러리입니다.

UMD 라이브러리 문서에서는 `require`를 보여주는 "Node.js에서 사용하기" 예제를 종종 설명하고
  "브라우저에서 사용하기" 예제에서는 `<script>` 태그를 사용해서 스크립트를 로드하는 방법을 보여줍니다.

### UMD 라이브러리 예제 (Examples of UMD libraries)

유명한 라이브러리 대부분은 UMD 패키지로 사용할 수 있습니다.
예로는 [jQuery](https://jquery.com/), [Moment.js](http://momentjs.com/), [loadash](https://loadash.com/) 등 더 많이 있습니다.

### 템플릿 (Template)

모듈은 세 가지 템플릿을 사용할 수 있습니다,
  [`module.d.ts`](./templates/module.d.ts.md), [`module-class.d.ts`](./templates/module-class.d.ts.md) 그리고 [`module-function.d.ts`](./templates/module-function.d.ts.md).

만약 모듈을 함수처럼 *호출*할 수 있으면 [`module-function.d.ts`](./templates/module-function.d.ts.md)을 사용하세요:

```js
var x = require("foo");
// 참고: 함수로 'x'를 호출
var y = x(42);
```

[각주 "ES6가 모듈 호출 시그니처에 미치는 영향"](#the-impact-of-es6-on-module-plugins)를 반드시 읽어보세요

만약 모듈이 `new`를 사용하여 *생성*할 수 있다면 [`module-class.d.ts`](./templates/module-class.d.ts.md)를 사용하세요:

```js
var x = require("bar");
// 참고: 'new' 연산자를 import된 변수에 사용
var y = new x("hello");
```

이런 모듈에도 같은 [각주](#the-impact-of-es6-on-module-plugins)가 적용됩니다.

만약 모듈이 위 사항에 해당되지 않다면, [`module.d.ts`](./templates/module.d.ts.md) 파일을 사용하세요.

## *모듈 플러그인* 혹은 *UMD 플러그인* (*Module Plugin* or *UMD Plugin*)

*모듈 플러그인*은 다른 모듈 (UMD나 모듈)의 형태를 변경합니다.
예를 들어, Moment.js에서, `moment-range`는 `moment` 객체에 새로운 `range` 메서드를 추가합니다.

선언 파일 작성을 위해, 모듈이 일반 모듈로 변경되든 UMD 모듈로 변경되든 같은 코드를 작성합니다.

### 템플릿 (Template)

[`module-plugin.d.ts`](./templates/module-plugin.d.ts.md) 템플릿을 사용하세요.

## *Global Plugin*

A *global plugin* is global code that changes the shape of some global.
As with *global-modifying modules*, these raise the possibility of runtime conflict.

For example, some libraries add new functions to `Array.prototype` or `String.prototype`.

### Identifying global plugins

Global plugins are generally easy to identify from their documentation.

You'll see examples that look like this:

```js
var x = "hello, world";
// Creates new methods on built-in types
console.log(x.startsWithHello());

var y = [1, 2, 3];
// Creates new methods on built-in types
console.log(y.reverseAndSort());
```

### Template

Use the [`global-plugin.d.ts`](./templates/global-plugin.d.ts.md) template.

## *Global-modifying Modules*

A *global-modifying module* alters existing values in the global scope when they are imported.
For example, there might exist a library which adds new members to `String.prototype` when imported.
This pattern is somewhat dangerous due to the possibility of runtime conflicts,
  but we can still write a declaration file for it.

### Identifying global-modifying modules

Global-modifying modules are generally easy to identify from their documentation.
In general, they're similar to global plugins, but need a `require` call to activate their effects.

You might see documentation like this:

```js
// 'require' call that doesn't use its return value
var unused = require("magic-string-time");
/* or */
require("magic-string-time");

var x = "hello, world";
// Creates new methods on built-in types
console.log(x.startsWithHello());

var y = [1, 2, 3];
// Creates new methods on built-in types
console.log(y.reverseAndSort());
```

### Template

Use the [`global-modifying-module.d.ts`](./templates/global-modifying-module.d.ts.md) template.

# Consuming Dependencies

There are several kinds of dependencies your library might have.
This section shows how to import them into the declaration file.

## Dependencies on Global Libraries

If your library depends on a global library, use a `/// <reference types="..." />` directive:

```ts
/// <reference types="someLib" />

function getThing(): someLib.thing;
```

## Dependencies on Modules

If your library depends on a module, use an `import` statement:

```ts
import * as moment from "moment";

function getThing(): moment;
```

## Dependencies on UMD libraries

### From a Global Library

If your global library depends on a UMD module, use a `/// <reference types` directive:

```ts
/// <reference types="moment" />

function getThing(): moment;
```

### From a Module or UMD Library

If your module or UMD library depends on a UMD library, use an `import` statement:

```ts
import * as someLib from 'someLib';
```

Do *not* use a `/// <reference` directive to declare a dependency to a UMD library!

# Footnotes

## Preventing Name Conflicts

Note that it's possible to define many types in the global scope when writing a global declaration file.
We strongly discourage this as it leads to possible unresolvable name conflicts when many declaration files are in a project.

A simple rule to follow is to only declare types *namespaced* by whatever global variable the library defines.
For example, if the library defines the global value 'cats', you should write

```ts
declare namespace cats {
    interface KittySettings { }
}
```

But *not*

```ts
// at top-level
interface CatsKittySettings { }
```

This guidance also ensures that the library can be transitioned to UMD without breaking declaration file users.

## The Impact of ES6 on Module Plugins

Some plugins add or modify top-level exports on existing modules.
While this is legal in CommonJS and other loaders, ES6 modules are considered immutable and this pattern will not be possible.
Because TypeScript is loader-agnostic, there is no compile-time enforcement of this policy, but developers intending to transition to an ES6 module loader should be aware of this.

## The Impact of ES6 on Module Call Signatures

Many popular libraries, such as Express, expose themselves as a callable function when imported.
For example, the typical Express usage looks like this:

```ts
import exp = require("express");
var app = exp();
```

In ES6 module loaders, the top-level object (here imported as `exp`) can only have properties;
  the top-level module object is *never* callable.
The most common solution here is to define a `default` export for a callable/constructable object;
  some module loader shims will automatically detect this situation and replace the top-level object with the `default` export.

## Library file layout

The layout of your declaration files should mirror the layout of the library.

A library can consist of multiple modules, such as

```Text
myLib
  +---- index.js
  +---- foo.js
  +---- bar
         +---- index.js
         +---- baz.js
```

These could be imported as

```js
var a = require("myLib");
var b = require("myLib/foo");
var c = require("myLib/bar");
var d = require("myLib/bar/baz");
```

Your declaration files should thus be

```Text
@types/myLib
  +---- index.d.ts
  +---- foo.d.ts
  +---- bar
         +---- index.d.ts
         +---- baz.d.ts
```
