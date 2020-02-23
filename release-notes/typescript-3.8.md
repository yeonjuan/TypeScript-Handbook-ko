* [타입-전용 Imports 와 Exports](#type-only-imports-exports)
* [ECMAScript 비공개 필드](#ecmascript-private-fields)
* [`export * as ns` Syntax](#export-star-as-namespace-syntax)
* [Top-Level `await`](#top-level-await)
* [JSDoc Property Modifiers](#jsdoc-modifiers)
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
* `preserve`: 이는 사용되지 않는 값들을 모두 *보존*합니다. 이로 인해 imports/사이트-이펙트가 보존될 수 있습니다.
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
* JS 사용자로부터도 비공개 필드는 이를 포함한 클래스 밖에서 접근하거나 탐지할 수 없습니다! 때때로 이를 *강한 비공개(hard privacy)*라고 부릅니다.

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

## <span id="export-star-as-namespace-syntax" /> `export * as ns` Syntax

It's often common to have a single entry-point that exposes all the members of another module as a single member.

```ts
import * as utilities from "./utilities.js";
export { utilities };
```

This is so common that ECMAScript 2020 recently added a new syntax to support this pattern!

```ts
export * as utilities from "./utilities.js";
```

This is a nice quality-of-life improvement to JavaScript, and TypeScript 3.8 implements this syntax.
When your module target is earlier than `es2020`, TypeScript will output something along the lines of the first code snippet.

## <span id="top-level-await" /> Top-Level `await`

TypeScript 3.8 provides support for a handy upcoming ECMAScript feature called "top-level `await`".

JavaScript users often introduce an `async` function in order to use `await`, and then immediately called the function after defining it.

```js
async function main() {
    const response = await fetch("...");
    const greeting = await response.text();
    console.log(greeting);
}

main()
    .catch(e => console.error(e))
```

This is because previously in JavaScript (along with most other languages with a similar feature), `await` was only allowed within the body of an `async` function.
However, with top-level `await`, we can use `await` at the top level of a module.

```ts
const response = await fetch("...");
const greeting = await response.text();
console.log(greeting);

// Make sure we're a module
export {};
```

Note there's a subtlety: top-level `await` only works at the top level of a *module*, and files are only considered modules when TypeScript finds an `import` or an `export`.
In some basic cases, you might need to write out `export {}` as some boilerplate to make sure of this.

Top level `await` may not work in all environments where you might expect at this point.
Currently, you can only use top level `await` when the `target` compiler option is `es2017` or above, and `module` is `esnext` or `system`.
Support within several environments and bundlers may be limited or may require enabling experimental support.

For more information on our implementation, you can [check out the original pull request](https://github.com/microsoft/TypeScript/pull/35813).

## <span id="es2020-for-target-and-module" /> `es2020` for `target` and `module`

TypeScript 3.8 supports `es2020` as an option for `module` and `target`.
This will preserve newer ECMAScript 2020 features like optional chaining, nullish coalescing, `export * as ns`, and dynamic `import(...)` syntax.
It also means `bigint` literals now have a stable `target` below `esnext`.

## <span id="jsdoc-modifiers" /> JSDoc Property Modifiers

TypeScript 3.8 supports JavaScript files by turning on the `allowJs` flag, and also supports *type-checking* those JavaScript files via the `checkJs` option or by adding a `// @ts-check` comment to the top of your `.js` files.

Because JavaScript files don't have dedicated syntax for type-checking, TypeScript leverages JSDoc.
TypeScript 3.8 understands a few new JSDoc tags for properties.

First are the accessibility modifiers: `@public`, `@private`, and `@protected`.
These tags work exactly like `public`, `private`, and `protected` respectively work in TypeScript.

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
// error! Property 'stuff' is private and only accessible within class 'Foo'.
```

* `@public` is always implied and can be left off, but means that a property can be reached from anywhere.
* `@private` means that a property can only be used within the containing class.
* `@protected` means that a property can only be used within the containing class, and all derived subclasses, but not on dissimilar instances of the containing class.

Next, we've also added the `@readonly` modifier to ensure that a property is only ever written to during initialization.

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
        // Cannot assign to 'stuff' because it is a read-only property.
    }
}

new Foo().stuff++;
//        ~~~~~
// Cannot assign to 'stuff' because it is a read-only property.
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
