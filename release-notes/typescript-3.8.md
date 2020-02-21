* [Type-Only Imports and Exports](#type-only-imports-exports)
* [ECMAScript Private Fields](#ecmascript-private-fields)
* [`export * as ns` 구문](#export-star-as-namespace-syntax)
* [최상위-레벨 `await`](#top-level-await)
* [JSDoc 프로퍼티 지정자](#jsdoc-modifiers)
* [Better Directory Watching on Linux and `watchOptions`](#better-directory-watching)
* ["Fast and Loose" Incremental Checking](#assume-direct-dependencies)

## <span id="type-only-imports-exports" /> Type-Only Imports and Export

This feature is something most users may never have to think about; however, if you've hit issues under `--isolatedModules`, TypeScript's `transpileModule` API, or Babel, this feature might be relevant.

TypeScript 3.8 adds a new syntax for type-only imports and exports.

```ts
import type { SomeThing } from "./some-module.js";

export type { SomeThing };
```

`import type` only imports declarations to be used for type annotations and declarations.
It *always* gets fully erased, so there's no remnant of it at runtime.
Similarly, `export type` only provides an export that can be used for type contexts, and is also erased from TypeScript's output.

It's important to note that classes have a value at runtime and a type at design-time, and the use is context-sensitive.
When using `import type` to import a class, you can't do things like extend from it.

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

If you've used Flow before, the syntax is fairly similar.
One difference is that we've added a few restrictions to avoid code that might appear ambiguous.

```ts
// Is only 'Foo' a type? Or every declaration in the import?
// We just give an error because it's not clear.

import type Foo, { Bar, Baz } from "some-module";
//     ~~~~~~~~~~~~~~~~~~~~~~
// error! A type-only import can specify a default import or named bindings, but not both.
```

In conjunction with `import type`, TypeScript 3.8 also adds a new compiler flag to control what happens with imports that won't be utilized at runtime: `importsNotUsedAsValues`.
This flag takes 3 different values:

* `remove`: this is today's behavior of dropping these imports. It's going to continue to be the default, and is a non-breaking change.
* `preserve`: this *preserves* all imports whose values are never used. This can cause imports/side-effects to be preserved.
* `error`: this preserves all imports (the same as the `preserve` option), but will error when a value import is only used as a type. This might be useful if you want to ensure no values are being accidentally imported, but still make side-effect imports explicit.

For more information about the feature, you can [take a look at the pull request](https://github.com/microsoft/TypeScript/pull/35200), and [relevant changes](https://github.com/microsoft/TypeScript/pull/36092/) around broadening where imports from an `import type` declaration can be used.

## <span id="ecmascript-private-fields" /> ECMAScript Private Fields

TypeScript 3.8 brings support for ECMAScript's private fields, part of the [stage-3 class fields proposal](https://github.com/tc39/proposal-class-fields/).

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
// Property '#name' is not accessible outside class 'Person'
// because it has a private identifier.
```

Unlike regular properties (even ones declared with the `private` modifier), private fields have a few rules to keep in mind.
Some of them are:

* Private fields start with a `#` character. Sometimes we call these *private names*.
* Every private field name is uniquely scoped to its containing class.
* TypeScript accessibility modifiers like `public` or `private` can't be used on private fields.
* Private fields can't be accessed or even detected outside of the containing class - even by JS users! Sometimes we call this *hard privacy*.

Apart from "hard" privacy, another benefit of private fields is that uniqueness we just mentioned.
For example, regular property declarations are prone to being overwritten in subclasses.

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
// 'this.foo' refers to the same property on each instance.
console.log(instance.cHelper()); // prints '20'
console.log(instance.dHelper()); // prints '20'
```

With private fields, you'll never have to worry about this, since each field name is unique to the containing class.

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
// 'this.#foo' refers to a different field within each class.
console.log(instance.cHelper()); // prints '10'
console.log(instance.dHelper()); // prints '20'
```

Another thing worth noting is that accessing a private field on any other type will result in a `TypeError`!

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
// This fails because 'b' is not an instance of 'Square'.
console.log(a.equals(b));
```

Finally, for any plain `.js` file users, private fields *always* have to be declared before they're assigned to.

```js
class C {
    // No declaration for '#foo'
    // :(

    constructor(foo: number) {
        // SyntaxError!
        // '#foo' needs to be declared before writing to it.
        this.#foo = foo;
    }
}
```

JavaScript has always allowed users to access undeclared properties, whereas TypeScript has always required declarations for class properties.
With private fields, declarations are always needed regardless of whether we're working in `.js` or `.ts` files.

```js
class C {
    /** @type {number} */
    #foo;

    constructor(foo: number) {
        // This works.
        this.#foo = foo;
    }
}
```

For more information about the implementation, you can [check out the original pull request](https://github.com/Microsoft/TypeScript/pull/30829)

### Which should I use?

We've already received many questions on which type of privates you should use as a TypeScript user: most commonly, "should I use the `private` keyword, or ECMAScript's hash/pound (`#`) private fields?"
It depends!

When it comes to properties, TypeScript's `private` modifiers are fully erased - that means that at runtime, it acts entirely like a normal property and there's no way to tell that it was declared with a `private modifier.
When using the `private` keyword, privacy is only enforced at compile-time/design-time, and for JavaScript consumers it's entirely intent-based.

```ts
class C {
    private foo = 10;
}

// This is an error at compile time,
// but when TypeScript outputs .js files,
// it'll run fine and print '10'.
console.log(new C().foo);    // prints '10'
//                  ~~~
// error! Property 'foo' is private and only accessible within class 'C'.

// TypeScript allows this at compile-time
// as a "work-around" to avoid the error.
console.log(new C()["foo"]); // prints '10'
```

The upside is that this sort of "soft privacy" can help your consumers temporarily work around not having access to some API, and also works in any runtime.

On the other hand, ECMAScript's `#` privates are completely inaccessible outside of the class.

```ts
class C {
    #foo = 10;
}

console.log(new C().#foo); // SyntaxError
//                  ~~~~
// TypeScript reports an error *and*
// this won't work at runtime!

console.log(new C()["#foo"]); // prints undefined
//          ~~~~~~~~~~~~~~~
// TypeScript reports an error under 'noImplicitAny',
// and this prints 'undefined'.
```

This hard privacy is really useful for strictly ensuring that nobody can take use of any of your internals.
If you're a library author, removing or renaming a private field should never cause a breaking change.

As we mentioned, another benefit is that subclassing can be easier with ECMAScript's `#` privates because they *really* are private.
When using ECMAScript `#` private fields, no subclass ever has to worry about collisions in field naming.
When it comes to TypeScript's `private` property declarations, users still have to be careful not to trample over properties declared in superclasses.

One more thing to think about is where you intend for your code to run.
TypeScript currently can't support this feature unless targeting ECMAScript 2015 (ES6) targets or higher.
This is because our downleveled implementation uses `WeakMap`s to enforce privacy, and `WeakMap`s can't be polyfilled in a way that doesn't cause memory leaks.
In contrast, TypeScript's `private`-declared properties work with all targets - even ECMAScript 3!

A final consideration might be speed: `private` properties are no different from any other property, so accessing them is as fast as any other property access no matter which runtime you target.
In contrast, because `#` private fields are downleveled using `WeakMap`s, they may be slower to use.
While some runtimes might optimize their actual implementations of `#` private fields, and even have speedy `WeakMap` implementations, that might not be the case in all runtimes.

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

이전의 JavaScript(유사한 기능을 가진 다른 언어들과 함께)에서 `await`은 `async` 함수 내에서 만 허용되었기 때문입니다. 하지만 최상위-레벨 `await`에서, 우리는 모듈의 최상위-레벨에서 `await`을 사용할 수 있습니다.

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
