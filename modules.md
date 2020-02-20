# 소개 (Introduction)

ECMAScript 2015부터 JavaScript에는 모듈 개념이 있습니다. TypeScript는 이 개념을 공유합니다.

모듈은 전역 범위가 아닌 그 자체 범위 내에서 실행됩니다; 즉 모듈 내에서 선언된 변수, 함수, 클래스 등은 [`export` 양식]() 중 하나를 사용하여 명시적으로 내보내지 않는 한 모듈 외부에서 보이지 않습니다. 반대로 다른 모듈에서 내보낸 변수, 함수, 클래스, 인터페이스 등을 사용하기 위해서는 [`import` 양식]() 중 하나를 사용하여 가져와야 합니다.

모듈은 선언형입니다; 모듈 간의 관계는 파일 수준에서 imports와 exports 관점에서 지정됩니다.

모듈은 모듈 로더를 사용하여 다른 모듈을 가져옵니다. 모듈 로더는 런타임 시에 모듈을 실행하기 전에 모듈의 모든 의존성을 찾고 실행해야 합니다. JavaScript에서 사용하는 유명한 모듈 로더로는 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 모듈 용 Node.js의 로더와 웹 애플리케이션의 [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 모듈 용 [RequireJS](https://requirejs.org/) 로더가 있습니다.

ECMAScript 2015와 마찬가지로 TypeScript는 최상위 수준의 `import` 혹은 `export`가 포함된 모든 파일을 모듈로 간주합니다. 반대로 최상위 수준의  `import` 혹은 `export` 선언이 없는 파일은 전역 범위에서 사용할 수 있는 스크립트로 처리됩니다(모듈도 마찬가지).

# 내보내기 (Export)

## 선언 내보내기 (Exporting a declaration)

`export` 키워드를 추가하여 모든 선언 (변수, 함수, 클래스, 타입 별칭, 인터페이스)를 내보낼 수 있습니다.

##### StringValidator.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

##### ZipCodeValidator.ts

```ts
import { StringValidator } from "./StringValidator";

export const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

## 내보내기 문 (Export statements)

내보내기 문은 사용자를 위해 내보낼 이름을 바꿔야 할 때 편리합니다. 위의 예제는 다음과 같이 작성할 수 있습니다:

```ts
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

## 다시-내보내기 (Re-exports)

종종 모듈은 다른 모듈을 확장하고 일부 기능을 부분적으로 노출합니다. 다시-내보내기(re-export)는 지역적으로 가져오거나 지역 변수를 도입하지 않습니다.

##### ParseIntBasedZipCodeValidator.ts

```ts
export class ParseIntBasedZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && parseInt(s).toString() === s;
    }
}

// 기존 validator의 이름을 변경 후 내보냄
export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";
```

선택적으로, 하나의 모듈은 하나 혹은 여러 개의 모듈을 감쌀 수 있고, 그 내보내기들을 `export * from "module"` 구문을 사용하여 결합할 수 있습니다.

##### AllValidators.ts

```ts
export * from "./StringValidator"; // 'StringValidator' 인터페이스를 내보냄
export * from "./ZipCodeValidator";  // 'ZipCodeValidator' 와 const 'numberRegexp' 클래스를 내보냄
export * from "./ParseIntBasedZipCodeValidator"; // 'ParseIntBasedZipCodeValidator' 클래스를 내보냄
                                                 // 'ZipCodeValidator.ts' 모듈 에 있는
                                                 // 'ZipCodeValidator' 클래스를
                                                 // 'RegExpBasedZipCodeValidator' 라는 별칭으로 다시 내보냄
```

# 가져오기 (Import)

가져오기는 모듈에서 내보내기 만큼 쉽습니다. 내보낸 선언은 아래의 `import` 양식 중 하나를 사용하여 가져옵니다.

## 모듈에서 단일 내보내기를 가져오기 (Import a single export from a module)

```ts
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();
```

이름을 수정해서 가져올 수 있습니다.

## 전체 모듈을 단일 변수로 가져와 모듈 내보내기 접근에 사용하기 (Import the entire module into a single variable, and use it to access the module exports)

```ts
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```

## 부수효과만을 위해 모듈 가져오기 (Import a module for side-effects only)

권장되지는 않지만, 일부 모듈은 다른 모듈에서 사용할 수 있도록 일부 전역 상태로 설정합니다. 이러한 모듈은 어떤 내보내기도 없거나, 사용자가 내보내기에 관심이 없습니다. 이러한 모듈을 가져오기 위해, 다음처럼 사용하세요:

```ts
import "./my-module.js"
```

## 타입 가져오기 (Importing Types)

TypeScript 3.8 이전에서는 `import`를 사용하여 타입을 가져올 수 있습니다. TypeScript 3.8에서는 `import` 문을 사용하거나 `import type`을 사용하여 타입을 가져올 수 있습니다.

```ts
// 동일한 import를 재사용하기
import {APIResponseType} from "./api";

// 명시적으로 import type을 사용하기
import type {APIResponseType} from "./api";
```

`import type`은 항상 JavaScript에서 제거되며, 바벨 같은 도구는 컴파일러 플래그인 `isolatedModules`를 통해 코드에 대해 더 나은 가정을 할 수 있습니다. [3.8 릴리즈 정보](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports)에서 더 많은 정보를 읽을 수 있습니다.

# 기본 내보내기 (Default exports)

각 모듈은 선택적으로 `default` 내보내기(default export)를 내보낼 수 있습니다. 기본 내보내기는 `default` 키워드로 표시됩니다; 모듈당 하나의 `default` 내보내기만 가능합니다. `default` 내보내기는 다른 가져오기 양식을 사용하여 가져옵니다.

`default` 내보내기는 정말 편리합니다. 예를 들어 jQuery와 같은 라이브러리는 `jQuery` 혹은 `$`와 같은 기본 내보내기를 가질 수 있으며, `$`나 `jQuery`와 같은 이름으로 가져올 수 있습니다.

##### [JQuery.d.ts](https://github.com/DefinitelyTyped/DefinitelyTyped/blob/master/types/jquery/JQuery.d.ts)

```ts
declare let $: JQuery;
export default $;
```

##### App.ts

```ts
import $ from "jquery";

$("button.continue").html( "Next Step..." );
```

클래스 및 함수 선언은 기본 내보내기로 직접 작성될 수 있습니다. 기본 내보내기 클래스 및 함수 선언 이름은 선택사항 입니다.

##### ZipCodeValidator.ts

```ts
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}
```

##### Test.ts

```ts
import validator from "./ZipCodeValidator";

let myValidator = new validator();
```

혹은

##### StaticZipCodeValidator.ts

```ts
const numberRegexp = /^[0-9]+$/;

export default function (s: string) {
    return s.length === 5 && numberRegexp.test(s);
}
```

##### Test.ts

```ts
import validate from "./StaticZipCodeValidator";

let strings = ["Hello", "98052", "101"];

// validate 함수 사용하기
strings.forEach(s => {
  console.log(`"${s}" ${validate(s) ? "matches" : "does not match"}`);
});
```

`default` 내보내기는 값도 가능합니다.

##### OneTwoThree.ts

```ts
export default "123";
```

##### Log.ts

```ts
import num from "./OneTwoThree";

console.log(num); // "123"
```

## x로 모두 내보내기 (Export all as x)

TypeScript 3.8에서는 다음 이름이 다른 모듈로 다시-내보내기 될 때 단축어처럼 `export * as ns`를 사용할 수 있습니다.

```ts
export * as utilities from "./utilities";
```

모듈에서 모든 의존성을 가져와 내보낸 필드로 만들면, 다음과 같이 가져올 수 있습니다:

```ts
import { utilities } from "./index";
```

# `export =`와 `import = require()` (`export =` and `import = require()`)

CommonJS와 AMD 둘 다 일반적으로 모듈의 모든 내보내기를 포함하는 `exports` 객체의 개념을 가지고 있습니다.

또한 `exports` 객체를 사용자 정의 단일 객체로 대체하는 기능도 지원합니다. 기본 내보내기는 이 동작에서 대체 역할을 합니다; 하지만 둘은 호환되지는 않습니다. TypeScript는 기존의 CommonJS와 AMD 워크플로우를 모델링 하기 위해 `export =`를 지원합니다.

`export =` 구문은 모듈에서 내보내지는 단일 객체를 지정합니다. 클래스, 인터페이스, 네임스페이스, 함수 혹은 열거형이 될 수 있습니다.

`export = `를 사용하여 모듈을 내보낼 때, TypeScript에 특정한 `import module = require("module")`를 사용하여 모듈을 가져와야 합니다.

##### ZipCodeValidator.ts

```ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

##### Test.ts

```ts
import zip = require("./ZipCodeValidator");

// 시험용 샘플
let strings = ["Hello", "98052", "101"];

// 사용할 Validators
let validator = new zip();

// 각 문자열이 각 validator를 통과했는지 보여줍니다
strings.forEach(s => {
  console.log(`"${ s }" - ${ validator.isAcceptable(s) ? "matches" : "does not match" }`);
});
```

# 모듈을 위한 코드 생성 (Code Generation for Modules)

컴파일 중에는 지정된 모듈 대상에 따라 컴파일러는 Node.js ([CommonJS](http://wiki.commonjs.org/wiki/CommonJS)), require.js ([AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)), [UMD](https://github.com/umdjs/umd), [SystemJS](https://github.com/systemjs/systemjs), 또는 [ECMAScript 2015 native modules](http://www.ecma-international.org/ecma-262/6.0/#sec-modules) (ES6) 모듈-로딩 시스템에 적합한 코드를 생성합니다. 생성된 코드의 `define`, `require` 그리고 `register` 호출 기능에 대한 자세한 정보는 각 모듈 로더의 문서를 확인하세요.

이 간단한 예제는 가져오기 및 내보내기 중에 사용된 이름이 모듈 로딩 코드로 변환되는 방법을 보여줍니다.

##### SimpleModule.ts

```ts
import m = require("mod");
export let t = m.something + 1;
```

##### AMD / RequireJS SimpleModule.js

```js
define(["require", "exports", "./mod"], function (require, exports, mod_1) {
    exports.t = mod_1.something + 1;
});
```

##### CommonJS / Node SimpleModule.js

```js
var mod_1 = require("./mod");
exports.t = mod_1.something + 1;
```

##### UMD SimpleModule.js

```js
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        var v = factory(require, exports); if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./mod"], factory);
    }
})(function (require, exports) {
    var mod_1 = require("./mod");
    exports.t = mod_1.something + 1;
});
```

##### System SimpleModule.js

```js
System.register(["./mod"], function(exports_1) {
    var mod_1;
    var t;
    return {
        setters:[
            function (mod_1_1) {
                mod_1 = mod_1_1;
            }],
        execute: function() {
            exports_1("t", t = mod_1.something + 1);
        }
    }
});
```

##### Native ECMAScript 2015 modules SimpleModule.js

```js
import { something } from "./mod";
export var t = something + 1;
```

# 간단한 예제 (Simple Example)

아래에서는 각 모듈에서 단일 이름으로 내보내기 위해 이전 예제에서 사용한 Validator 구현을 통합합니다.

컴파일 하려면, 명령 줄에서 모듈 대상을 지정해야 합니다. Node.js의 경우, `--module commonjs`를 사용하세요; require.js의 경우 `--module amd`를 사용하세요. 예를 들면:

```Shell
tsc --module commonjs Test.ts
```

컴파일이 되면, 각 모듈은 별도의 `.js`파일이 됩니다. 참조 태그와 마찬가지로, 컴파일러는 `import`문을 따라 의존적인 파일들을 컴파일 합니다.

##### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

##### LettersOnlyValidator.ts

```ts
import { StringValidator } from "./Validation";
const lettersRegexp = /^[A-Za-z]+$/;
export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

##### ZipCodeValidator.ts

```ts
import { StringValidator } from "./Validation";
const numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

##### Test.ts

```ts
import { StringValidator } from "./Validation";
import { ZipCodeValidator } from "./ZipCodeValidator";
import { LettersOnlyValidator } from "./LettersOnlyValidator";
// 시험용 샘플
let strings = ["Hello", "98052", "101"];
// 사용할 validator
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();
// 각 문자열이 validator를 통과하는지 보여줌
strings.forEach(s => {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" :
        "does not match" } ${ name }`);
    }
});
```

`작업중...`
