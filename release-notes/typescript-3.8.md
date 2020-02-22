* [Type-Only Imports and Exports](#type-only-imports-exports)
* [ECMAScript Private Fields](#ecmascript-private-fields)
* [`export * as ns` Syntax](#export-star-as-namespace-syntax)
* [Top-Level `await`](#top-level-await)
* [JSDoc Property Modifiers](#jsdoc-modifiers)
* [리눅스에서 더 나은 디렉터리 감시와 `watchOptions`](#better-directory-watching)
* ["빠르고 느슨한" 증분 검사](#assume-direct-dependencies)

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

## <span id="better-directory-watching" /> 리눅스에서 더 나은 디렉터리 감시와 `watchOptions`

TypeScript 3.8에서는 `node_modules`의 변경사항을 효율적으로 수집하는데 중요한 새로운 디렉터리 감시 전략을 제공합니다.

리눅스와 같은 운영체제에서 TypeScript는 `node_modules`에 디렉터리 왓쳐(파일 왓쳐와는 반대로)를 설치하고, 종속성의 변화를 감지하기 위해 많은 하위 디렉터리를 설치합니다.
왜냐하면 사용 가능한 파일 왓쳐의 수는 종종 'node_modules'의 파일 수에 의해 가려지기 때문이고, 추적할 디렉터리 수가 적기 때문입니다.

TypeScript의 이전 버전은 폴더에 디렉터리 왓쳐를 *즉시* 설치하고, 초기에는 괜찮을 겁니다; 하지만, npm install 할 때, `node_modules`안에서 많은 일들이 발생할 것이고, TypeScript를 압도하여, 종종 에디터 세션을 아주 느리게 만듭니다.
이를 방지하기 위해, TypeScript 3.8은 디렉터리 왓쳐를 설치하기 전에 조금 기다려서 변동성이 높은 디렉터리에게 안정될 수 있는 시간을 줍니다.

왜냐하면 모든 프로젝트는 다른 전략에서 작동을 더 잘할 수 있고, 이 새로운 방법은 당신의 작업 흐름에서는 잘 맞지 않을 수 있습니다. TypeScript 3.8은 파일과 디렉터리를 감시하는데 어떤 감시 전략을 사용할지 컴파일러/언어 서비스에 알려줄 수 있도록 `tsconfig.json`과 `jsconfig.json`에 `watchOptions`란 새로운 필드를 제공합니다.

```json5
{
    // 일반적인 컴파일러 옵션들
    "compilerOptions": {
        "target": "es2020",
        "moduleResolution": "node",
        // ...
    },

    // NEW: 파일/디렉터리 감시를 위한 옵션
    "watchOptions": {
        // 파일과 디렉터리에 네이티브 파일 시스템 이벤트 사용
        "watchFile": "useFsEvents",
        "watchDirectory": "useFsEvents",

        // 업데이트가 빈번할 때
        // 업데이트하기 위해 더 자주 파일을 폴링
        "fallbackPolling": "dynamicPriority"
    }
}
```

`watchOptions`는 구성할 수 있는 4가지 새로운 옵션이 포함되어 있습니다.

* `watchFile`: 각 파일의 감시 방법 전략. 다음과 같이 설정할 수 있습니다:
    * `fixedPollingInterval`: 고정된 간격으로 모든 파일의 변경을 1초에 여러 번 검사합니다.
    * `priorityPollingInterval`: 모든 파일의 변경을 1초에 여러 번 검사합니다, 하지만 휴리스틱을 사용하여 특정 타입의 파일은 다른 타입의 파일보다 덜 자주 검사합니다.
    * `dynamicPriorityPolling`: 동적 큐를 사용하여 덜-자주 수정된 파일은 적게 검사합니다.
    * `useFsEvents` (디폴트): 파일 변화에 운영체제/파일 시스템의 네이티브 이벤트를 사용합니다.
    * `useFsEventsOnParentDirectory`: 파일을 포함하고 있는 디렉터리 변경을 감지할 때, 운영체제/파일 시스템의 네이티브 이벤트를 사용합니다. 파일 검사자를 적게 사용할 수 있지만, 덜 정확할 수 있습니다.
* `watchDirectory`: 재귀적인 파일-감시 기능이 없는 시스템 안에서 전체 디렉터리 트리가 감시되는 전략. 다음과 같이 설정할 수 있습니다:
    * `fixedPollingInterval`: 고정된 간격으로 모든 디렉터리의 변경을 1초에 여러 번 검사합니다.
    * `dynamicPriorityPolling`: 동적 큐를 사용하여 덜-자주 수정된 디렉터리는 적게 검사합니다.
    * `useFsEvents` (디폴트): 디렉터리 변경에 운영체제/파일 시스템의 네이티브 이벤트를 사용합니다.
* `fallbackPolling`: 파일 시스템 이벤트를 사용할 때, 이 옵션은 시스템이 네이티브 파일 왓쳐가 부족하거나/혹은 지원하지 않을 때, 사용되는 폴링 전략을 지정합니다. 다음과 같이 설정할 수 있습니다.
    * `fixedPollingInterval`: *(위를 참조하세요.)*
    * `priorityPollingInterval`: *(위를 참조하세요.)*
    * `dynamicPriorityPolling`: *(위를 참조하세요.)*
* `synchronousWatchDirectory`: 디렉터리의 연기된 감시를 비활성화합니다. 연기된 감시는 많은 파일이 한 번에 변경될 때 유용합니다 (예를 들어, `npm install`을 실행하여 `node_modules`의 변경), 하지만 덜-일반적인 설정을 위해 비활성화할 수도 있습니다.

이 변경의 더 자세한 내용은 [head over to GitHub to see the pull request](https://github.com/microsoft/TypeScript/pull/35615)를 읽어보세요.

## <span id="assume-direct-dependencies" /> "빠르고 느슨한" 증분 검사

TypeScript 3.8은 새로운 컴파일러 옵션 `assumeChangesOnlyAffectDirectDepencies`을 제공합니다.
이 옵션이 활성화되면, TypeScript는 정말로 영향을 받은 파일들은 재검사/재빌드하지않고, 변경된 파일뿐만 아니라 직접 import 한 파일만 재검사/재빌드 합니다.

예를 들어, 다음과 같이 `fileA.ts`를 import 한 `fileB.ts`를 import 한 `fileC.ts`를 import 한 `fileD.ts`를 살펴봅시다:

```text
fileA.ts <- fileB.ts <- fileC.ts <- fileD.ts
```

`--watch` 모드에서는, `fileA.ts`의 변경이 `fileB.ts`, `fileC.ts` 그리고 `fileD.ts`를 TypeScript가 재-검사해야 한다는 의미입니다.
`assumeChangesOnlyAffectDirectDependencies`에서는 `fileA.ts`의 변경은 `fileA.ts`와 `fileB.ts`만 재-검사하면 됩니다.

Visual Studio Code와 같은 코드 베이스에서는, 특정 파일의 변경에 대해 약 14초에서 약 1초로 재빌드 시간을 줄여주었습니다.
이 옵션을 모든 코드 베이스에서 추천하는 것은 아니지만, 큰 코드 베이스를 가지고 있고, 나중까지 전체 프로젝트 오류를 기꺼이 연기하겠다면 (예를 들어, `tsconfig.fullbuild.json`이나 CI를 통한 전용 빌드) 흥미로울 것입니다.

더 자세한 내용은 [see the original pull request](https://github.com/microsoft/TypeScript/pull/35711)에서 보실 수 있습니다.
