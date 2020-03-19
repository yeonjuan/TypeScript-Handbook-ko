# 정의 파일 이론: 심층 분석 (Definition File Theory: A Deep Dive)

원하는 API 형태를 제공하는 모듈을 만드는 것은 까다로울 수 있습니다.
예를 들어, `new`의 사용에 따라 호출할 때 다른 타입을 생성하는 모듈을 원할 수 있고,
  계층에 노출 된 다양한 명명된 타입을 가지고 있으며,
  모듈 객체에 대한 여러 프로퍼티도 가질 수 있습니다.

이 가이드에서는, 익숙한 API를 노출하는 복잡한 정의 파일에 대해 작성하는 도구를 제공합니다.
또한 옵션이 다양하기 때문에 여기서는 모듈 (또는 UMD) 라이브러리에 중점을 둡니다.

## 주요 컨셉 (Key Concepts)

TypeScript 작동 방식에 대해 여러 주요 개념을 이해하여
  정의의 형태를 만드는 방법을 완전히 이해할 수 있습니다.

### 타입 (Types)

이 가이드를 읽고 있다면, 아마도 TypeScript의 타입에 대해 이미 알고 있을 것입니다.
보다 명확하게하기 위해, 다음과 같이 *타입*을 도입했습니다:

* 타입 별칭 선언 (`type sn = number | string;`)
* 인터페이스 선언 (`interface I { x: number[]; }`)
* 클래스 선언 (`class C { }`)
* 열거형 선언 (`enum E { A, B, C }`)
* 타입을 가리키는 `import` 선언

이러한 각 선언 방식은 새로운 타입 이름을 만듭니다.

### 값 (Values)

타입과 마찬가지로 값이 무엇인지 이미 알고 있을 것입니다.
값은 표현식에서 참조할 수 있는 런타임 이름입니다.
`let x = 5;`에서 `x`라고 불리는 값을 생성합니다.

다시 말하지만,  다음과 같이 값을 만듭니다.

* `let`, `const`, 그리고 `var` 선언
* 값을 포함하는 `네임스페이스` 또는 `모듈` 선언
* `열거형` 선언
* `클래스` 선언
* 값을 참조하는 `import` 선언
* `함수` 선언

### 네임스페이스 (Namespaces)

타입은 *네임스페이스* 안에 존재할 수 있습니다.
예를 들어, `let x: A.B.C` 이란 선언이 있다면,
  타입 `C`는 `A.B` 네임스페이스에서 온 것 입니다.

이 구별은 미묘하지만 중요합니다 -- 여기서 `A.B`는 타입이거나 값일 필요는 없습니다.

## 간단한 조합: 하나의 이름, 여러 의미 (Simple Combinations: One name, multiple meanings)

`A`라는 이름이 있으면, `A`에 대해 타입, 값 또는 네임스페이스라는 세가지 다른 의미를 찾을 수 있습니다.
이름을 해석하는 방법은 사용하는 컨텍스트에 따라 다릅니다.
예를 들어 `let m: A.A = A;` 선언에서,
  `A`는 먼저 네임스페이스로 사용 된 다음, 타입의 이름으로, 그 다음 값으로 사용됩니다.
즉 완전히 다른 선언을 의미할 수 있습니다!

약간은 혼란스러워 보이지만, 과하게 사용하지 않는 한 실제로 매우 편리합니다.
결합 동작의 유용한 측면을 살펴 보겠습니다.

### 내부 조합 (Built-in Combinations)

영리한 사람이라면, *타입*과 *값* 목록에서 `클래스`가 둘 다 나온 것을 눈치챘을 것입니다.
`class C { }` 선언은 두 가지를 만듭니다:
  클래스 인스턴스의 형태를 나타내는 *타입* `C`와 
  클래스 생성자를 나타내는 *값* `C` 입니다.
열거형 선언도 비슷하게 동작합니다.

### 사용자 조합 (User Combinations)

모듈 파일 `foo.d.ts`을 작성했습니다:

```ts
export var SomeVar: { a: SomeType };
export interface SomeType {
  count: number;
}
```

그 다음 사용했습니다:

```ts
import * as foo from './foo';
let x: foo.SomeType = foo.SomeVar.a;
console.log(x.count);
```

잘 작동하지만, `SomeType`과 `SomeVar`는 이름이 같도록
  밀접하게 관련되어 있다고 상상할 수 있습니다.
결합을 사용하여 같은 이름 `Bar`를 두 가지 다른 객체 (값과 타입)를 표시 할 수 있습니다:

```ts
export var Bar: { a: Bar };
export interface Bar {
  count: number;
}
```

이 경우 사용하는 코드를 해체할 수 있는 아주 좋은 기회입니다:

```ts
import { Bar } from './foo';
let x: Bar = Bar.a;
console.log(x.count);
```

여기서도 `Bar`를 타입과 값으로 사용했습니다.
`Bar` 값을 `Bar` 타입으로 선언할 필요가 없었으며 -- 독립적입니다.

## Advanced Combinations

Some kinds of declarations can be combined across multiple declarations.
For example, `class C { }` and `interface C { }` can co-exist and both contribute properties to the `C` types.

This is legal as long as it does not create a conflict.
A general rule of thumb is that values always conflict with other values of the same name unless they are declared as `namespace`s,
  types will conflict if they are declared with a type alias declaration (`type s = string`),
  and namespaces never conflict.

Let's see how this can be used.

### Adding using an `interface`

We can add additional members to an `interface` with another `interface` declaration:

```ts
interface Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

This also works with classes:

```ts
class Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

Note that we cannot add to type aliases (`type s = string;`) using an interface.

### Adding using a `namespace`

A `namespace` declaration can be used to add new types, values, and namespaces in any way which does not create a conflict.

For example, we can add a static member to a class:

```ts
class C {
}
// ... elsewhere ...
namespace C {
  export let x: number;
}
let y = C.x; // OK
```

Note that in this example, we added a value to the *static* side of `C` (its constructor function).
This is because we added a *value*, and the container for all values is another value
  (types are contained by namespaces, and namespaces are contained by other namespaces).

We could also add a namespaced type to a class:

```ts
class C {
}
// ... elsewhere ...
namespace C {
  export interface D { }
}
let y: C.D; // OK
```

In this example, there wasn't a namespace `C` until we wrote the `namespace` declaration for it.
The meaning `C` as a namespace doesn't conflict with the value or type meanings of `C` created by the class.

Finally, we could perform many different merges using `namespace` declarations.
This isn't a particularly realistic example, but shows all sorts of interesting behavior:

```ts
namespace X {
  export interface Y { }
  export class Z { }
}

// ... elsewhere ...
namespace X {
  export var Y: number;
  export namespace Z {
    export class C { }
  }
}
type X = string;
```

In this example, the first block creates the following name meanings:

* A value `X` (because the `namespace` declaration contains a value, `Z`)
* A namespace `X` (because the `namespace` declaration contains a type, `Y`)
* A type `Y` in the `X` namespace
* A type `Z` in the `X` namespace (the instance shape of the class)
* A value `Z` that is a property of the `X` value (the constructor function of the class)

The second  block creates the following name meanings:

* A value `Y` (of type `number`) that is a property of the `X` value
* A namespace `Z`
* A value `Z` that is a property of the `X` value
* A type `C` in the `X.Z` namespace
* A value `C` that is a property of the `X.Z` value
* A type `X`

## Using with `export =` or `import`

An important rule is that `export` and `import` declarations export or import *all meanings* of their targets.

<!-- TODO: Write more on that. -->
