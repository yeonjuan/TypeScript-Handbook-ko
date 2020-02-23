* [타입-전용 Imports 와 Exports](#type-only-imports-exports)
* [ECMAScript 비공개 필드](#ecmascript-private-fields)
* [`export * as ns` 구문](#export-star-as-namespace-syntax)
* [최상위-레벨 `await`](#top-level-await)
* [JSDoc 프로퍼티 지정자](#jsdoc-modifiers)
* [Better Directory Watching on Linux and `watchOptions`](#better-directory-watching)
* ["Fast and Loose" Incremental Checking](#assume-direct-dependencies)

## <span id="type-only-imports-exports" /> 타입-전용 Imports 와 Exports (Type-Only Imports and Exports)

이 기능은 대부분의 사용자가 생각할 필요가 없을 수도 있지만; `--isolatedModules`, TypeScript의 `transpileModule` API, 또는 Babel에서 문제가 발생하면 이 기능이 관련 있을 수 있습니다.

TypeScript 3.8은 타입-전용 imports, exports를 위한 새로운 구문이 추가되었습니다.

```ts
import type { SomeThing } from "./some-module.js";

export type { SomeThing };
```

`import type`은 타입 표기와 선언에 사용될 선언만 import 합니다.
이는 *항상* 완전히 제거되므로, 런타임에 남아있는 것은 없습니다.
마찬가지로, `export type`은 타입 문맥에 사용할 export만 제공하며, 이 또한 TypeScript의 출력물에서 제거됩니다.

클래스는 런타임에 값을 가지고 있고 디자인-타임에 타입이 있으며 사용은 상황에-따라 다르다는 것을 유의해야 합니다.
클래스를 import 하기 위해 `import type`을 사용하면, 확장 같은 것은 할 수 없습니다.

```ts
import type { Component } from "react";

interface ButtonProps {
    // ...
}

class Button extends Component<ButtonProps> {
    //               ~~~~~~~~~
    // error! 'Component' only refers to a type, but is being used as a value here.

    // ...
}
```

이전에 Flow를 사용해본 적이 있다면, 이 구문은 상당히 유사합니다.
한 가지 차이점은 코드가 모호해 보이지 않도록 몇 가지 제한을 두었다는 것입니다.

```ts
// 'Foo'만 타입인가? 혹은 모든 import 선언이 타입인가?
// 이는 명확하지 않기 때문에 오류로 처리합니다.

import type Foo, { Bar, Baz } from "some-module";
//     ~~~~~~~~~~~~~~~~~~~~~~
// error! A type-only import can specify a default import or named bindings, but not both.
```

`import type`과 함께, TypeScript 3.8은 런타임 시 사용되지 않는 import에서 발생하는 작업을 제어하기 위해 새로운 컴파일러 플래그를 추가합니다.: `importsNotUsedAsValues`.
이 플래그는 3 가지 다른 값을 가집니다:

* `remove`: 이는 imports를 제거하는 현재 동작이며, 계속 기본값으로 작동할 것이며, 기존 동작을 바꾸는 변화가 아닙니다.
* `preserve`: 이는 사용되지 않는 값들을 모두 *보존*합니다. 이로 인해 imports/side-effects가 보존될 수 있습니다.
* `error`: 이는 모든 (`preserve` option 처럼) 모든 imports를 보존하지만, import 값이 타입으로만 사용될 경우 오류를 발생시킵니다. 이는 실수로 값을 import하지 않지만 사이드 이팩트 import를 명시적으로 만들고 싶을 때 유용합니다.

이 기능에 대한 더 자세한 정보는, `import type`선언이 사용될수 있는 범위를 확대하는 [pull request](https://github.com/microsoft/TypeScript/pull/35200), 와 [관련된 변경 사항](https://github.com/microsoft/TypeScript/pull/36092/)에서 찾을 수 있습니다.

## <span id="ecmascript-private-fields" /> ECMAScript 비공개 필드 (ECMAScript Private Fields)

TypeScript 3.8 은 ECMAScript의 [stage-3 클래스 필드 제안](https://github.com/tc39/proposal-class-fields/)의 비공개 필드를 지원합니다.

```ts
class Person {
    #name: string

    constructor(name: string) {
        this.#name = name;
    }

    greet() {
        console.log(`Hello, my name is ${this.#name}!`);
    }
}

let jeremy = new Person("Jeremy Bearimy");

jeremy.#name
//     ~~~~~
// 프로퍼티 '#name'은 'Person' 클래스 외부에서 접근할 수 없습니다.
// 이는 비공개 식별자를 가지기 때문입니다.
```

일반적인 프로퍼티들(`private` 지정자로 선언한 것도)과 달리, 비공개 필드는 몇 가지 명심해야 할 규칙이 있습니다.
그중 몇몇은:

* 비공개 필드는 `#` 문자로 시작합니다. 때때로 이를 *비공개 이름(private names)* 이라고 부릅니다.
* 모든 비공개 필드 이름은 이를 포함한 클래스 범위에서 유일합니다.
* `public` 또는 `private` 같은 TypeScript 접근 지시자는 비공개 필드로 사용될 수 없습니다.  
* JS 사용자로부터도 비공개 필드는 이를 포함한 클래스 밖에서 접근하거나 탐지할 수 없습니다! 때때로 이를 *강한 비공개(hard privacy)* 라고 부릅니다.

"강한" 비공개와 별도로, 비공개 필드의 또 다른 장점은 유일하다는 것입니다.
예를 들어, 일반적인 프로퍼티 선언은 하위클래스에서 덮어쓰기 쉽습니다.

```ts
class C {
    foo = 10;

    cHelper() {
        return this.foo;
    }
}

class D extends C {
    foo = 20;

    dHelper() {
        return this.foo;
    }
}

let instance = new D();
// 'this.foo' 는 각 인스턴스마다 같은 프로퍼티를 참조합니다.
console.log(instance.cHelper()); // '20' 출력
console.log(instance.dHelper()); // '20' 출력
```

비공개 필드에서는, 포함하고 있는 클래스에서 각각의 필드 이름이 유일하기 때문에 이에 대해 걱정하지 않아도 됩니다.

```ts
class C {
    #foo = 10;

    cHelper() {
        return this.#foo;
    }
}

class D extends C {
    #foo = 20;

    dHelper() {
        return this.#foo;
    }
}

let instance = new D();
// 'this.#foo' 는 각 클래스안의 다른 필드를 참조합니다.
console.log(instance.cHelper()); // '10' 출력
console.log(instance.dHelper()); // '20' 출력
```

알아 두면 좋은 또 다른 점은 다른 타입으로 비공개 필드에 접근하면 `TypeError` 를 발생한다는 것입니다.

```ts
class Square {
    #sideLength: number;

    constructor(sideLength: number) {
        this.#sideLength = sideLength;
    }

    equals(other: any) {
        return this.#sideLength === other.#sideLength;
    }
}

const a = new Square(100);
const b = { sideLength: 100 };

// Boom!
// TypeError: attempted to get private field on non-instance
// 이는 `b` 가 `Square`의 인스턴스가 아니기 때문에 실패 합니다.
console.log(a.equals(b));
```

마자막으로, 모든 일반 `.js` 파일 사용자들의 경우, 비공개 필드는 *항상* 할당되기 전에 선언되어야 합니다.

```js
class C {
    // '#foo' 선언이 없습니다.
    // :(

    constructor(foo: number) {
        // SyntaxError!
        // '#foo'는 쓰여지기 전에 선언되어야 합니다.
        this.#foo = foo;
    }
}
```

JavaScript는 항상 사용자들에게 선언되지 않은 프로퍼티에 접근을 허용했지만, TypeScript는 항상 클래스 프로퍼티 선언을 요구했습니다.
비공개 필드는, `.js` 또는 `.ts` 파일에서 동작하는지 상관없이 항상 선언이 필요합니다.

```js
class C {
    /** @type {number} */
    #foo;

    constructor(foo: number) {
        // 동작합니다.
        this.#foo = foo;
    }
}
```

구현에 대한 더 많은 정보는, [the original pull request](https://github.com/Microsoft/TypeScript/pull/30829)를 참고하세요

### Which should I use?

이미 TypeScript 유저로서 어떤 종류의 비공개를 사용해야 하는지에 대한 많은 질문을 받았습니다: 주로, "`private` 키워드를 사용해야 하나요 아니면 ECMAScript의 해시/우물 (`#`) 비공개 필드를 사용해야 하나요?"
상황마다 다릅니다!

프로퍼티에서, TypeScript의 `private` 지정자는 완전히 지워집니다 - 이는 런타임에서는 완전히 일반 프로퍼티처럼 동작하며 이것이 `private` 지정자로 선언되었다고 알릴 방법이 없습니다.
`private` 키워드를 사용할 때, 비공개는 오직 컴파일-타임/디자인-타임에만 시행되며, JavaScript 사용자에게는 전적으로 의도-기반입니다.

```ts
class C {
    private foo = 10;
}

// 이는 컴파일 타임에 오류이지만
// TypeScript 가 .js 파일로 출력했을 때는
// 잘 동작하며 '10'을 출력합니다.
console.log(new C().foo);    // '10' 출력
//                  ~~~
// error! Property 'foo' is private and only accessible within class 'C'.

// TypeScript 오류를 피하기 위한 "해결 방법" 으로
// 캄파일 타임에 이것을 허용합니다.
console.log(new C()["foo"]); // prints '10'
```

이 같은 종류의 "약한 비공개(soft privacy)"는 사용자가 API에 접근할 수 없는 상태에서 일시적으로 작업을 하는 데 도움이 되며, 어떤 런타임에서도 동작합니다.

반면에, ECMAScript의 `#` 비공개는 완벽하게 클래스 밖에서 접근 불가능합니다.

```ts
class C {
    #foo = 10;
}

console.log(new C().#foo); // SyntaxError
//                  ~~~~
// TypeScript 는 오류를 보고 하며 *또한*
// 런타임에도 동작하지 않습니다.

console.log(new C()["#foo"]); // undefined 출력
//          ~~~~~~~~~~~~~~~
// TypeScript 는 'noImplicitAny' 하에서 오류를 보고하며
// `undefined`를 출력합니다.
```

이런 강한 비공개(hard privacy)는 아무도 내부를 사용할 수 없도록 엄격하게 보장하는데 유용합니다.
만약 라이브러리 작성자일 경우, 비공개 필드를 제거하거나 이름을 바꾸는 것이 급격한 변화를 초래서는 안됩니다.

언급했듯이, 다른 장점은 ECMAScript의 `#` 비공개가 *진짜* 비공개이기 때문에 서브클래싱을 쉽게 할 수 있다는 것입니다.
ECMAScript `#` 비공개 필드를 사용하면, 어떤 서브 클래스도 필드 네이밍 충돌에 대해 걱정할 필요가 없습니다.
TypeScript의 `private`프로퍼티 선언에서는, 사용자는 여전히 상위 클래스에 선언된 프로퍼티를 짓밟지  않도록 주의해야 합니다.

한 가지 더 생각해봐야 할 것은 코드가 실행되기를 의도하는 곳입니다.
현재 TypeScript는 이 기능을 ECMAScript 2015 (ES6) 이상 버전을 대상으로 하지 않으면 지원할 수 없습니다.
이는 하위 레벨 구현이 비공개를 강제하기 위해 `WeakMap`을 사용하는데, `WeakMap`은 메모리 누수를 잃으키지 않도록 폴리필될 수 없기 때문입니다.
반면, TypeScript의 `private`-선언 프로퍼티는 모든 대상에서 동작합니다- ECMAScript3에서도!

마지막 고려 사항은 속도 일수 있습니다: `private` 프로퍼티는 다른 어떤 프로퍼티와 다르지 않기 때문에, 어떤 런타임을 대상으로 하단 다른 프로퍼티와 마찬가지로 접근 속도가 빠를 수 있습니다.
반면에, `#` 비공개 필드는 `WeakMap`을 이용해 다운 레벨 되기 때문에 사용 속도가 느려질 수 있습니다.
어떤 런타임은 `#` 비공개 필드 구현을 최적화 하고, 더 빠른 `WeakMap`을 구현하고 싶을 수 있지만, 모든 런타임에서 그렇지 않을 수 있습니다.

## <span id="export-star-as-namespace-syntax" /> `export * as ns` 구문 (`export * as ns` Syntax)

다른 모듈의 모든 멤버를 하나의 멤버로 내보내는 단일 진입점을 갖는 것은 일반적입니다.

```ts
import * as utilities from "./utilities.js";
export { utilities };
```

이는 매우 흔해서 ECMAScript2020은 최근에 이 패턴을 지원하기 위해서 새로운 구문을 추가했습니다.

```ts
export * as utilities from "./utilities.js";
```

이것은 JavaScript에 대한 훌륭한 삶의 질의 향상이며, TypeScript 3.8은 이 구문을 지원합니다.
모듈 대상이 `es2020` 이전인 경우, TypeScript는 첫 번째 줄의 코드 스니펫을 따라서 무언가를 출력할 것입니다.

## <span id="top-level-await" /> 최상위-레벨 `await` (Top-Level `await`)

TypeScript 3.8은 "최상위-레벨 `await`"이라는 편리한 ECMAScript 기능을 지원합니다.

JavaScript 사용자는 `await`을 사용하기 위해 `async` 함수를 도입하는 경우가 많으며, 이를 정의한 후 즉시 함수를 호출합니다.

```js
async function main() {
    const response = await fetch("...");
    const greeting = await response.text();
    console.log(greeting);
}

main()
    .catch(e => console.error(e))
```

이전의 JavaScript(유사한 기능을 가진 다른 언어들과 함께)에서 `await`은 `async` 함수 내에서 만 허용되었기 때문입니다.
하지만 최상위-레벨 `await`로, 우리는 모듈의 최상위 레벨에서 `await`을 사용할 수 있습니다.

```ts
const response = await fetch("...");
const greeting = await response.text();
console.log(greeting);

// 모듈인지 확인
export {};
```

유의할 점이 있습니다: 최상위-레벨 `await`은 *module*의 최상위 레벨에서만 동작하며, 파일은 TypeScript가`import`나 `export`를 찾을 때에만 모듈로 간주됩니다. 일부 기본적인 경우에 `export {}`와 같은 보일러 플레이트를 작성하여 이를 확인할 필요가 있습니다.

이러한 경우가 예상되는 모든 환경에서 최상위 레벨 `await`은 동작하지 않을 수 있습니다. 현재, `target` 컴파일러 옵션이 `es2017` 이상이고, `module`이 `esnext` 또는 `system`인 경우에만 최상위 레벨 `await`을 사용할 수 있습니다. 몇몇 환경과 번들러내에서의 지원은 제한적으로 작동하거나 실험적 지원을 활성화해야 할 수도 있습니다.

구현에 관한 더 자세한 정보는 [original pull request을 확인하세요](https://github.com/microsoft/TypeScript/pull/35813).

## <span id="es2020-for-target-and-module" /> `es2020`용 `target`과 `module`   (`es2020` for `target` and `module`)

TypeScript 3.8은 `es2020`을 `module`과 `target` 옵션으로 지원합니다. 이를 통해 선택적 체이닝 (optional chaining), nullish 병합 (nullish coalescing), `export * as ns` 그리고 동적인 `import(...)` 구문과 같은 ECMAScript 2020 기능이 유지됩니다. 또한 `bigint` 리터럴이 `esnext` 아래에 안정적인 `target`을 갖는 것을 의미합니다.

## <span id="jsdoc-modifiers" /> JSDoc 프로퍼티 지정자 (JSDoc Property Modifiers)

TypeScript 3.8는 `allowJs` 플래그를 사용하여 JavaScript 파일을 지원하고 `checkJs` 옵션이나 `// @ts-check` 주석을 `.js` 파일 맨 위에 추가하여 JavaScript 파일의 *타입-검사*를 지원합니다.

JavaScript 파일에는 타입-검사를 위한 전용 구문이 없기 때문에 TypeScript는 JSDoc을 활용합니다. TypeScript 3.8은 프로퍼티에 대한 몇 가지 새로운 JSDoc 태그를 인식합니다.

먼저 접근 지정자입니다: `@public`, `@private` 그리고 `@protected`입니다. 이 태그들은 TypeScript 내에서 각각 `public`, `private`, `protected`와 동일하게 동작합니다.

```js
// @ts-check

class Foo {
    constructor() {
        /** @private */
        this.stuff = 100;
    }

    printStuff() {
        console.log(this.stuff);
    }
}

new Foo().stuff;
//        ~~~~~
// 오류! 'stuff' 프로퍼티는 private 이기 때문에 오직 'Foo' 클래스 내에서만 접근이 가능합니다.
```

* `@public`은 항상 암시적이며 생략될 수 있지만, 어디서든 해당 프로퍼티에 접근 가능을 의미합니다.
* `@private`은 오직 프로퍼티를 포함하는 클래스 내에서 해당 프로퍼티 사용 가능을 의미합니다.
* `@protected`는 프로퍼티를 포함하는 클래스와 파생된 모든 하위 클래스내에서 해당 프로퍼티를 사용할 수 있지만, 포함하는 클래스의 인스턴스에서는 해당 프로퍼티를 사용할 수 없습니다.

다음으로 `@readonly` 지정자를 추가하여 프로퍼티가 초기화 과정 내에서만 값이 쓰이는 것을 보장합니다.

```js
// @ts-check

class Foo {
    constructor() {
        /** @readonly */
        this.stuff = 100;
    }

    writeToStuff() {
        this.stuff = 200;
        //   ~~~~~
        // 'stuff'는 읽기-전용(read-only) 프로퍼티이기 때문에 할당할 수 없습니다.
    }
}

new Foo().stuff++;
//        ~~~~~
// 'stuff'는 읽기-전용(read-only) 프로퍼티이기 때문에 할당할 수 없습니다.
```

## <span id="better-directory-watching" /> Better Directory Watching on Linux and `watchOptions`

TypeScript 3.8 ships a new strategy for watching directories, which is crucial for efficiently picking up changes to `node_modules`.

For some context, on operating systems like Linux, TypeScript installs directory watchers (as opposed to file watchers) on `node_modules` and many of its subdirectories to detect changes in dependencies.
This is because the number of available file watchers is often eclipsed by the of files in `node_modules`, whereas there are way fewer directories to track.

Older versions of TypeScript would *immediately* install directory watchers on folders, and at startup that would be fine; however, during an npm install, a lot of activity will take place within `node_modules` and that can overwhelm TypeScript, often slowing editor sessions to a crawl.
To prevent this, TypeScript 3.8 waits slightly before installing directory watchers to give these highly volatile directories some time to stabilize.

Because every project might work better under different strategies, and this new approach might not work well for your workflows, TypeScript 3.8 introduces a new `watchOptions` field in `tsconfig.json` and `jsconfig.json` which allows users to tell the compiler/language service which watching strategies should be used to keep track of files and directories.

```json5
{
    // Some typical compiler options
    "compilerOptions": {
        "target": "es2020",
        "moduleResolution": "node",
        // ...
    },

    // NEW: Options for file/directory watching
    "watchOptions": {
        // Use native file system events for files and directories
        "watchFile": "useFsEvents",
        "watchDirectory": "useFsEvents",

        // Poll files for updates more frequently
        // when they're updated a lot.
        "fallbackPolling": "dynamicPriority"
    }
}
```

`watchOptions` contains 4 new options that can be configured:

* `watchFile`: the strategy for how individual files are watched. This can be set to
    * `fixedPollingInterval`: Check every file for changes several times a second at a fixed interval.
    * `priorityPollingInterval`: Check every file for changes several times a second, but use heuristics to check certain types of files less frequently than others.
    * `dynamicPriorityPolling`: Use a dynamic queue where less-frequently modified files will be checked less often.
    * `useFsEvents` (the default): Attempt to use the operating system/file system's native events for file changes.
    * `useFsEventsOnParentDirectory`: Attempt to use the operating system/file system's native events to listen for changes on a file's containing directories. This can use fewer file watchers, but might be less accurate.
* `watchDirectory`: the strategy for how entire directory trees are watched under systems that lack recursive file-watching functionality. This can be set to:
    * `fixedPollingInterval`: Check every directory for changes several times a second at a fixed interval.
    * `dynamicPriorityPolling`: Use a dynamic queue where less-frequently modified directories will be checked less often.
    * `useFsEvents` (the default): Attempt to use the operating system/file system's native events for directory changes.
* `fallbackPolling`: when using file system events, this option specifies the polling strategy that gets used when the system runs out of native file watchers and/or doesn't support native file watchers. This can be set to
    * `fixedPollingInterval`: *(See above.)*
    * `priorityPollingInterval`: *(See above.)*
    * `dynamicPriorityPolling`: *(See above.)*
* `synchronousWatchDirectory`: Disable deferred watching on directories. Deferred watching is useful when lots of file changes might occur at once (e.g. a change in `node_modules` from running `npm install`), but you might want to disable it with this flag for some less-common setups.

For more information on these changes, [head over to GitHub to see the pull request](https://github.com/microsoft/TypeScript/pull/35615) to read more.

## <span id="assume-direct-dependencies" /> "Fast and Loose" Incremental Checking

TypeScript 3.8 introduces a new compiler option called `assumeChangesOnlyAffectDirectDependencies`.
When this option is enabled, TypeScript will avoid rechecking/rebuilding all truly possibly-affected files, and only recheck/rebuild files that have changed as well as files that directly import them.

For example, consider a file `fileD.ts` that imports `fileC.ts` that imports `fileB.ts` that imports `fileA.ts` as follows:

```text
fileA.ts <- fileB.ts <- fileC.ts <- fileD.ts
```

In `--watch` mode, a change in `fileA.ts` would typically mean that TypeScript would need to at least re-check `fileB.ts`, `fileC.ts`, and `fileD.ts`.
Under `assumeChangesOnlyAffectDirectDependencies`, a change in `fileA.ts` means that only `fileA.ts` and `fileB.ts` need to be re-checked.

In a codebase like Visual Studio Code, this reduced rebuild times for changes in certain files from about 14 seconds to about 1 second.
While we don't necessarily recommend this option for all codebases, you might be interested if you have an extremely large codebase and are willing to defer full project errors until later (e.g. a dedicated build via a `tsconfig.fullbuild.json` or in CI).

For more details, you can [see the original pull request](https://github.com/microsoft/TypeScript/pull/35711).
