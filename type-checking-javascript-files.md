# JavaScript 파일의 타입 검사 (Type Checking JavaScript Files)

TypeScript 2.3 이상의 버전에서는 `--checkJs`를 사용하는 `.js` 파일에서 타입 검사 및 오류 보고를 지원합니다.

`// @ts-nocheck` 주석을 달아 일부 파일에서 타입 검사를 건너뛸 수 있으며; 반대로 `// @ts-check` 주석을 달아 `--checkJs`를 사용하지 않고 일부 `.js` 파일에 대해서만 타입 검사를 하도록 선택할 수 있습니다.
또한 특정 부분의 앞 줄에 `// @ts-ignore`를 달아 에러를 무시할 수도 있습니다. `tsconfig.json`이 있는 경우, JavaScript 검사는 `noImplicitAny`, `strictNullChecks` 등의 엄격한 플래그를 우선시 한다는 점을 알아두세요.
하지만, JavaScript 검사의 상대적인 느슨함 덕분에 엄격한 플래그와 결합하여 사용하는 것은 놀라운 결과를 보여줄 것입니다.

`.ts` 파일과 `.js` 파일은 타입을 검사하는 방법에 몇 가지 주목할만한 차이점이 있습니다:

## 타입 정보에 쓰일 수 있는 JSDoc 타입 (JSDoc types are used for type information)

`.js` 파일에서는, 흔히 `.ts` 파일처럼 타입을 추론해볼 수 있습니다. 타입을 추론할 수 없는 경우, `.ts`의 타입 표시와 같은 방법으로 JSDoc을 사용해 이를 지정할 수 있습니다.
TypeScript와 마찬가지로, `--noImplicitAny`는 컴파일러가 타입을 유추할 수 없는 부분에서 오류를 보고할 것입니다. (확장 가능한(open-ended) 객체 리터럴을 제외하고; 자세한 내용은 아래를 참고하세요.)

선언에 JSDoc 표시를 사용하면 해당 선언의 타입을 설정할 수 있습니다. 예를 들면:

```js
/** @type {number} */
var x;

x = 0;      // 성공
x = false;  // 오류: 불리언(boolean)에는 숫자를 할당할 수 없음
```

사용 가능한 JSDoc 패턴 목록은 [이곳](#supported-jsdoc)에서 확인할 수 있습니다.

## 클래스 본문의 할당으로부터 추론된 프로퍼티 (Properties are inferred from assignments in class bodies)

ES2015에는 클래스에 프로퍼티를 선언할 수 있는 수단이 없습니다. 프로퍼티는 객체 리터럴과 같이 동적으로 할당됩니다.

`.js` 파일에서, 컴파일러는 클래스 본문 내부에서 할당된 프로퍼티로부터 프로퍼티들을 추론합니다.
생성자가 정의되어 있지 않거나, 생성자에서 정의된 타입이 `undefined`나 `null`이 아닐 경우, 프로퍼티의 타입은 생성자에서 주어진 타입과 동일합니다.
전자에 해당 프로퍼티의 경우, 할당되었던 모든 값들의 타입을 가진 유니언 타입이 됩니다.
생성자에 정의된 프로퍼티는 항상 존재하는 것으로 가정하는 반면, 메서드, getter, setter에서만 정의된 프로퍼티는 선택적인 것으로 간주합니다.

```js
class C {
    constructor() {
        this.constructorOnly = 0
        this.constructorUnknown = undefined
    }
    method() {
        this.constructorOnly = false // 오류, constructorOnly는 Number 타입임
        this.constructorUnknown = "plunkbat" // 성공, constructorUnknown의 타입은 string | undefined
        this.methodOnly = 'ok'  // 성공, 그러나 methodOnly는 undefined 타입 또한 허용됨
    }
    method2() {
        this.methodOnly = true  // 이 또한 성공, methodOnly의 타입은 string | boolean | undefined
    }
}
```

프로퍼티가 클래스 본문에서 설정되지 않았다면, 알 수 없는 것으로 간주합니다.
클래스에 읽기 전용 프로퍼티가 있는 경우, 생성자에서 선언에 JSDoc을 사용하여 타입을 추가하여 표시합니다.
이후엔 초기화 하더라도 값을 지정할 필요가 없습니다.

```js
class C {
    constructor() {
        /** @type {number | undefined} */
        this.prop = undefined;
        /** @type {number | undefined} */
        this.count;
    }
}

let c = new C();
c.prop = 0;          // 성공
c.count = "string";  // 오류: string 은 number|undefined에 할당할 수 없음
```

## 생성자 함수와 클래스는 동일합니다 (Constructor functions are equivalent to classes)

ES2015 이전에는, JavaScript는 클래스 대신 생성자 함수를 사용했습니다.
컴파일러는 이러한 패턴을 지원하며 생성자 함수를 ES2015 클래스와 동일한 것으로 이해합니다.
앞서 설명한 프로퍼티 추론 규칙 또한 정확히 같은 방식으로 작용합니다.

```js
function C() {
    this.constructorOnly = 0
    this.constructorUnknown = undefined
}
C.prototype.method = function() {
    this.constructorOnly = false // 오류
    this.constructorUnknown = "plunkbat" // 성공, 타입은 string | undefined가 됨
}
```

## CommonJS 모듈 지원 (CommonJS modules are supported)

`.js` 파일에서, TypeScript는 CommonJS 모듈 포맷을 이해합니다.
`exports`와 `module.exports` 할당은 내보내기(export) 선언으로 인식됩니다.
마찬가지로, `require` 함수 호출은 모듈 가져오기(import)로 인식됩니다. 예를 들어:

```js
// `import module "fs"`와 같음
const fs = require("fs");

// `export function readFile`과 같음
module.exports.readFile = function(f) {
    return fs.readFileSync(f);
}
```

JavaScript의 모듈 지원은 TypeScript의 모듈 지원보다 구문적으로 훨씬 관용적입니다.
따라서 대부분의 할당과 선언의 조합이 지원됩니다.

## 클래스, 함수, 객체 리터럴은 네임스페이스임 (Classes, functions, and object literals are namespaces)

`.js` 파일에 있는 클래스는 네임스페이스입니다.
예를 들어, 다음과 같이 클래스를 중첩하는 데에 사용할 수 있습니다:

```js
class C {
}
C.D = class {
}
```

그리고 pre-ES2015에선, 정적 메서드를 나타내는 데에 사용할 수도 있습니다.

```js
function Outer() {
  this.y = 2
}
Outer.Inner = function() {
  this.yy = 2
}
```

또한 간단한 네임스페이스를 생성하는 데에 사용할 수도 있습니다:

```js
var ns = {}
ns.C = class {
}
ns.func = function() {
}
```

다른 번형도 허용됩니다:

```js
// 즉시 호출 함수(IIFE)
var ns = (function (n) {
  return n || {};
})();
ns.CONST = 1

// 전역으로 기본 설정
var assign = assign || function() {
  // 여기엔 코드를
}
assign.extra = 1
```

## 객체 리터럴은 확장 가능 (Object literals are open-ended)

`.ts` 파일에서, 변수 선언을 초기화하는 객체 리터럴은 선언에 해당 타입을 부여합니다.
원본 리터럴에 명시되어 있지 않은 새 멤버는 추가될 수 없습니다.
이 규칙은 `.js` 파일에선 완화됩니다; 객체 리터럴은 원본에 정의되지 않은 새로운 프로퍼티를 조회하고 추가하는 것이 허용되는 확장 가능한 타입(인덱스 시그니처(index signature))을 갖습니다.
예를 들어:

```js
var obj = { a: 1 };
obj.b = 2;  // 허용됨
```

객체 리터럴은 마치 닫힌 객체가 아니라 열린 맵(maps)으로 다뤄지도록 `[x:string]: any`와 같은 인덱스 시그니처를 가진 것처럼 동작합니다.

다른 특정 JavaScript 검사 동작과 마찬가지로, 해당 동작은 변수에 JSDoc 타입을 지정하여 변경할 수 있습니다.
예를 들어:

```js
/** @type {{a: number}} */
var obj = { a: 1 };
obj.b = 2;  // 오류, {a: number}타입엔 b 프로퍼티가 없음
```

## null, undefined 및 빈 배열 이니셜라이저는 any 혹은 any[] 타입임 (null, undefined, and empty array initializers are of type any or any[])

null 또는 undefined로 초기화된 변수나 매개변수 또는 프로퍼티는, 엄격한 null 검사가 있더라도 any 타입을 갖게 될 것입니다.
[]로 초기화된 변수나 매개변수 또는 프로퍼티는, 엄격한 null 검사가 있더라도 any[] 타입을 갖게 될 것입니다.
위에서 설명한 여러 이니셜라이저(initializer)를 갖는 프로퍼티만이 유일한 예외입니다.

```js
function Foo(i = null) {
    if (!i) i = 1;
    var j = undefined;
    j = 2;
    this.l = [];
}
var foo = new Foo();
foo.l.push(foo.i);
foo.l.push("end");
```

## 함수 매개변수는 기본적으로 선택 사항임 (Function parameters are optional by default)

Since there is no way to specify optionality on parameters in pre-ES2015 Javascript, all function parameters in `.js` file are considered optional.
Calls with fewer arguments than the declared number of parameters are allowed.

It is important to note that it is an error to call a function with too many arguments.

For instance:

```js
function bar(a, b) {
    console.log(a + " " + b);
}

bar(1);       // OK, second argument considered optional
bar(1, 2);
bar(1, 2, 3); // Error, too many arguments
```

JSDoc annotated functions are excluded from this rule.
Use JSDoc optional parameter syntax to express optionality. e.g.:

```js
/**
 * @param {string} [somebody] - Somebody's name.
 */
function sayHello(somebody) {
    if (!somebody) {
        somebody = 'John Doe';
    }
    console.log('Hello ' + somebody);
}

sayHello();
```

## Var-args parameter declaration inferred from use of `arguments`

A function whose body has a reference to the `arguments` reference is implicitly considered to have a var-arg parameter (i.e. `(...arg: any[]) => any`). Use JSDoc var-arg syntax to specify the type of the arguments.

```js
/** @param {...number} args */
function sum(/* numbers */) {
    var total = 0
    for (var i = 0; i < arguments.length; i++) {
      total += arguments[i]
    }
    return total
}
```

## Unspecified type parameters default to `any`

Since there is no natural syntax for specifying generic type parameters in Javascript, an unspecified type parameter defaults to `any`.

### In extends clause

For instance, `React.Component` is defined to have two type parameters, `Props` and `State`.
In a `.js` file, there is no legal way to specify these in the extends clause. By default the type arguments will be `any`:

```js
import { Component } from "react";

class MyComponent extends Component {
    render() {
        this.props.b; // Allowed, since this.props is of type any
    }
}
```

Use JSDoc `@augments` to specify the types explicitly. for instance:

```js
import { Component } from "react";

/**
 * @augments {Component<{a: number}, State>}
 */
class MyComponent extends Component {
    render() {
        this.props.b; // Error: b does not exist on {a:number}
    }
}
```

### In JSDoc references

An unspecified type argument in JSDoc defaults to any:

```js
/** @type{Array} */
var x = [];

x.push(1);        // OK
x.push("string"); // OK, x is of type Array<any>

/** @type{Array.<number>} */
var y = [];

y.push(1);        // OK
y.push("string"); // Error, string is not assignable to number

```

### In function calls

A call to a generic function uses the arguments to infer the type parameters. Sometimes this process fails to infer any types, mainly because of lack of inference sources; in these cases, the type parameters will default to `any`. For example:

```js
var p = new Promise((resolve, reject) => { reject() });

p; // Promise<any>;
```

# Supported JSDoc

The list below outlines which constructs are currently supported when using JSDoc annotations to provide type information in JavaScript files.

Note any tags which are not explicitly listed below (such as `@async`) are not yet supported.

* `@type`
* `@param` (or `@arg` or `@argument`)
* `@returns` (or `@return`)
* `@typedef`
* `@callback`
* `@template`
* `@class` (or `@constructor`)
* `@this`
* `@extends` (or `@augments`)
* `@enum`

The meaning is usually the same, or a superset, of the meaning of the tag given at usejsdoc.org.
The code below describes the differences and gives some example usage of each tag.

## `@type`

You can use the "@type" tag and reference a type name (either primitive, defined in a TypeScript declaration, or in a JSDoc "@typedef" tag).
You can use any Typescript type, and most JSDoc types.

```js
/**
 * @type {string}
 */
var s;

/** @type {Window} */
var win;

/** @type {PromiseLike<string>} */
var promisedString;

// You can specify an HTML Element with DOM properties
/** @type {HTMLElement} */
var myElement = document.querySelector(selector);
element.dataset.myData = '';

```

`@type` can specify a union type &mdash; for example, something can be either a string or a boolean.

```js
/**
 * @type {(string | boolean)}
 */
var sb;
```

Note that parentheses are optional for union types.

```js
/**
 * @type {string | boolean}
 */
var sb;
```

You can specify array types using a variety of syntaxes:

```js
/** @type {number[]} */
var ns;
/** @type {Array.<number>} */
var nds;
/** @type {Array<number>} */
var nas;
```

You can also specify object literal types.
For example, an object with properties 'a' (string) and 'b' (number) uses the following syntax:

```js
/** @type {{ a: string, b: number }} */
var var9;
```

You can specify map-like and array-like objects using string and number index signatures, using either standard JSDoc syntax or Typescript syntax.

```js
/**
 * A map-like object that maps arbitrary `string` properties to `number`s.
 *
 * @type {Object.<string, number>}
 */
var stringToNumber;

/** @type {Object.<number, object>} */
var arrayLike;
```

The preceding two types are equivalent to the Typescript types `{ [x: string]: number }` and `{ [x: number]: any }`. The compiler understands both syntaxes.

You can specify function types using either Typescript or Closure syntax:

```js
/** @type {function(string, boolean): number} Closure syntax */
var sbn;
/** @type {(s: string, b: boolean) => number} Typescript syntax */
var sbn2;
```

Or you can just use the unspecified `Function` type:

```js
/** @type {Function} */
var fn7;
/** @type {function} */
var fn6;
```

Other types from Closure also work:

```js
/**
 * @type {*} - can be 'any' type
 */
var star;
/**
 * @type {?} - unknown type (same as 'any')
 */
var question;
```

### Casts

Typescript borrows cast syntax from Closure.
This lets you cast types to other types by adding a `@type` tag before any parenthesized expression.

```js
/**
 * @type {number | string}
 */
var numberOrString = Math.random() < 0.5 ? "hello" : 100;
var typeAssertedNumber = /** @type {number} */ (numberOrString)
```

### Import types

You can also import declarations from other files using import types.
This syntax is Typescript-specific and differs from the JSDoc standard:

```js
/**
 * @param p { import("./a").Pet }
 */
function walk(p) {
    console.log(`Walking ${p.name}...`);
}
```

import types can also be used in type alias declarations:

```js
/**
 * @typedef { import("./a").Pet } Pet
 */

/**
 * @type {Pet}
 */
var myPet;
myPet.name;
```

import types can be used to get the type of a value from a module if you don't know the type, or if it has a large type that is annoying to type:

```js
/**
 * @type {typeof import("./a").x }
 */
var x = require("./a").x;
```

## `@param` and `@returns`

`@param` uses the same type syntax as `@type`, but adds a parameter name.
The parameter may also be declared optional by surrounding the name with square brackets:

```js
// Parameters may be declared in a variety of syntactic forms
/**
 * @param {string}  p1 - A string param.
 * @param {string=} p2 - An optional param (Closure syntax)
 * @param {string} [p3] - Another optional param (JSDoc syntax).
 * @param {string} [p4="test"] - An optional param with a default value
 * @return {string} This is the result
 */
function stringsStringStrings(p1, p2, p3, p4){
  // TODO
}
```

Likewise, for the return type of a function:

```js
/**
 * @return {PromiseLike<string>}
 */
function ps(){}

/**
 * @returns {{ a: string, b: number }} - May use '@returns' as well as '@return'
 */
function ab(){}
```

## `@typedef`, `@callback`, and `@param`

`@typedef` may be used to define complex types.
Similar syntax works with `@param`.

```js
/**
 * @typedef {Object} SpecialType - creates a new type named 'SpecialType'
 * @property {string} prop1 - a string property of SpecialType
 * @property {number} prop2 - a number property of SpecialType
 * @property {number=} prop3 - an optional number property of SpecialType
 * @prop {number} [prop4] - an optional number property of SpecialType
 * @prop {number} [prop5=42] - an optional number property of SpecialType with default
 */
/** @type {SpecialType} */
var specialTypeObject;
```

You can use either `object` or `Object` on the first line.

```js
/**
 * @typedef {object} SpecialType1 - creates a new type named 'SpecialType1'
 * @property {string} prop1 - a string property of SpecialType1
 * @property {number} prop2 - a number property of SpecialType1
 * @property {number=} prop3 - an optional number property of SpecialType1
 */
/** @type {SpecialType1} */
var specialTypeObject1;
```

`@param` allows a similar syntax for one-off type specifications.
Note that the nested property names must be prefixed with the name of the parameter:

```js
/**
 * @param {Object} options - The shape is the same as SpecialType above
 * @param {string} options.prop1
 * @param {number} options.prop2
 * @param {number=} options.prop3
 * @param {number} [options.prop4]
 * @param {number} [options.prop5=42]
 */
function special(options) {
  return (options.prop4 || 1001) + options.prop5;
}
```

`@callback` is similar to `@typedef`, but it specifies a function type instead of an object type:

```js
/**
 * @callback Predicate
 * @param {string} data
 * @param {number} [index]
 * @returns {boolean}
 */
/** @type {Predicate} */
const ok = s => !(s.length % 2);
```

Of course, any of these types can be declared using Typescript syntax in a single-line `@typedef`:

```js
/** @typedef {{ prop1: string, prop2: string, prop3?: number }} SpecialType */
/** @typedef {(data: string, index?: number) => boolean} Predicate */
```

## `@template`

You can declare generic types with the `@template` tag:

```js
/**
 * @template T
 * @param {T} x - A generic parameter that flows through to the return type
 * @return {T}
 */
function id(x){ return x }
```

Use comma or multiple tags to declare multiple type parameters:

```js
/**
 * @template T,U,V
 * @template W,X
 */
```

You can also specify a type constraint before the type parameter name.
Only the first type parameter in a list is constrained:

```js
/**
 * @template {string} K - K must be a string or string literal
 * @template {{ serious(): string }} Seriousalizable - must have a serious method
 * @param {K} key
 * @param {Seriousalizable} object
 */
function seriousalize(key, object) {
  // ????
}
```

## `@constructor`

The compiler infers constructor functions based on this-property assignments, but you can make checking stricter and suggestions better if you add a `@constructor` tag:

```js
/**
 * @constructor
 * @param {number} data
 */
function C(data) {
  this.size = 0;
  this.initialize(data); // Should error, initializer expects a string
}
/**
 * @param {string} s
 */
C.prototype.initialize = function (s) {
  this.size = s.length
}

var c = new C(0);
var result = C(1); // C should only be called with new
```

With `@constructor`, `this` is checked inside the constructor function `C`, so you will get suggestions for the `initialize` method and an error if you pass it a number. You will also get an error if you call `C` instead of constructing it.

Unfortunately, this means that constructor functions that are also callable cannot use `@constructor`.

## `@this`

The compiler can usually figure out the type of `this` when it has some context to work with. When it doesn't, you can explicitly specify the type of `this` with `@this`:

```js
/**
 * @this {HTMLElement}
 * @param {*} e
 */
function callbackForLater(e) {
    this.clientHeight = parseInt(e) // should be fine!
}
```

## `@extends`

When Javascript classes extend a generic base class, there is nowhere to specify what the type parameter should be. The `@extends` tag provides a place for that type parameter:

```js
/**
 * @template T
 * @extends {Set<T>}
 */
class SortableSet extends Set {
  // ...
}
```

Note that `@extends` only works with classes. Currently, there is no way for a constructor function extend a class.

## `@enum`

The `@enum` tag allows you to create an object literal whose members are all of a specified type. Unlike most object literals in Javascript, it does not allow other members.

```js
/** @enum {number} */
const JSDocState = {
  BeginningOfLine: 0,
  SawAsterisk: 1,
  SavingComments: 2,
}
```

Note that `@enum` is quite different from, and much simpler than, Typescript's `enum`. However, unlike Typescript's enums, `@enum` can have any type:

```js
/** @enum {function(number): number} */
const Math = {
  add1: n => n + 1,
  id: n => -n,
  sub1: n => n - 1,
}
```

## More examples

```js
var someObj = {
  /**
   * @param {string} param1 - Docs on property assignments work
   */
  x: function(param1){}
};

/**
 * As do docs on variable assignments
 * @return {Window}
 */
let someFunc = function(){};

/**
 * And class methods
 * @param {string} greeting The greeting to use
 */
Foo.prototype.sayHi = (greeting) => console.log("Hi!");

/**
 * And arrow functions expressions
 * @param {number} x - A multiplier
 */
let myArrow = x => x * x;

/**
 * Which means it works for stateless function components in JSX too
 * @param {{a: string, b: number}} test - Some param
 */
var fc = (test) => <div>{test.a.charAt(0)}</div>;

/**
 * A parameter can be a class constructor, using Closure syntax.
 *
 * @param {{new(...args: any[]): object}} C - The class to register
 */
function registerClass(C) {}

/**
 * @param {...string} p1 - A 'rest' arg (array) of strings. (treated as 'any')
 */
function fn10(p1){}

/**
 * @param {...string} p1 - A 'rest' arg (array) of strings. (treated as 'any')
 */
function fn9(p1) {
  return p1.join();
}
```

## Patterns that are known NOT to be supported

Referring to objects in the value space as types doesn't work unless the object also creates a type, like a constructor function.

```js
function aNormalFunction() {

}
/**
 * @type {aNormalFunction}
 */
var wrong;
/**
 * Use 'typeof' instead:
 * @type {typeof aNormalFunction}
 */
var right;
```

Postfix equals on a property type in an object literal type doesn't specify an optional property:

```js
/**
 * @type {{ a: string, b: number= }}
 */
var wrong;
/**
 * Use postfix question on the property name instead:
 * @type {{ a: string, b?: number }}
 */
var right;
```

Nullable types only have meaning if `strictNullChecks` is on:

```js
/**
 * @type {?number}
 * With strictNullChecks: true -- number | null
 * With strictNullChecks: off  -- number
 */
var nullable;
```

Non-nullable types have no meaning and are treated just as their original type:

```js
/**
 * @type {!number}
 * Just has type number
 */
var normal;
```

Unlike JSDoc's type system, Typescript only allows you to mark types as containing null or not.
There is no explicit non-nullability -- if strictNullChecks is on, then `number` is not nullable.
If it is off, then `number` is nullable.