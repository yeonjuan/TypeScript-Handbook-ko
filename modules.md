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

// Use function validate
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

`작업중...`
