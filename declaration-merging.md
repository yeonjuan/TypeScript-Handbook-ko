# 선언 병합 (Declaration Merging)

## 소개 (Introduction)

TypeScript의 독특한 개념들 중 일부는 타입(type) 레벨에서 JavaScript 객체의 형태를 설명합니다. 그중 TypeScript만의 특별한 예시로 '선언 병합(declaration merging)' 개념이 있습니다. 이 개념을 이해한다면 기존의 JavaScript 작업을 진행할 때 이점이 많을 것입니다. 또한 이 개념은 고급 추상화 개념으로의 문을 열어줄 것입니다.

본론으로 돌아가서, "선언 병합"은 컴파일러가 같은 이름으로 선언된 두 개의 개별적인 선언을 하나의 정의로 합쳐준다는 것을 의미합니다. 병합된 정의는 원래 두 선언 각각의 기능을 모두 갖게 됩니다. 병합할 선언이 몇 개든 상관없습니다; 이 개념은 단지 두 개의 선언만 합치도록 제한되어 있지 않습니다.

## 기본 사용법 (Basic Concepts)

TypeScript에선, 선언은 네임스페이스(namespace), 타입, 값(value) 중 최소 한 종류의 개체를 생성합니다. 네임스페이스-생성 선언은, 점 표기법을 사용하여 접근할 이름을 가진 네임스페이스를 생성합니다. 타입-생성 선언은, 선언된 형태로 표시되며 주어진 이름에 바인딩 된 타입을 생성합니다. 마지막으로, 값-생성 선언은 출력된 JavaScript에서 확인할 수 있는 값을 생성합니다.

|        선언 타입       | 네임스페이스 | 타입 | 값 |
|----------------------|:--------:|:---:|:---:|
| 네임스페이스(Namespace) |     X    |     |  X  |
| 클래스 (Class)         |         |  X  |  X  |
| 열거형 (Enum)          |         |  X  |  X  |
| 인터페이스 (Interface)  |         |  X  |     |
| 타입 별칭 (Type Alias) |          |  X  |     |
| 함수(Function)        |          |     |  X  |
| 변수(Variable)        |          |     |  X  |

각 선언이 어떤 것을 생성하는지 이해한다면, 선언 병합을 할 때 어떤 것이 병합되는지 이해하는 데에 도움이 될 것입니다.

## 인터페이스의 병합 (Merging Interfaces)

아마도 선언 병합 중에 가장 일반적이고, 단순한 유형은 인터페이스 병합일 것입니다. 가장 기본적인 수준에서, 이 병합은 각 선언의 멤버(member)를 같은 이름의 인터페이스에 단순히 결합(join)합니다.

```ts
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = {height: 5, width: 6, scale: 10};
```

각 인터페이스의 비함수(Non-function) 멤버는 고유해야 합니다. 만약 해당 멤버들이 고유하지 않다면, 모두 같은 타입이어야 합니다. 컴파일러는 각 인터페이스에 같은 이름이지만 다른 타입을 가진 비함수 멤버가 있을 경우, 오류를 일으킬 것입니다.

For function members, each function member of the same name is treated as describing an overload of the same function.
Of note, too, is that in the case of interface `A` merging with later interface `A`, the second interface will have a higher precedence than the first.

That is, in the example:

```ts
interface Cloner {
    clone(animal: Animal): Animal;
}

interface Cloner {
    clone(animal: Sheep): Sheep;
}

interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
```

The three interfaces will merge to create a single declaration as so:

```ts
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

Notice that the elements of each group maintains the same order, but the groups themselves are merged with later overload sets ordered first.

One exception to this rule is specialized signatures.
If a signature has a parameter whose type is a *single* string literal type (e.g. not a union of string literals), then it will be bubbled toward the top of its merged overload list.

For instance, the following interfaces will merge together:

```ts
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
    createElement(tagName: string): HTMLElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

The resulting merged declaration of `Document` will be the following:

```ts
interface Document {
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```

# Merging Namespaces

Similarly to interfaces, namespaces of the same name will also merge their members.
Since namespaces create both a namespace and a value, we need to understand how both merge.

To merge the namespaces, type definitions from exported interfaces declared in each namespace are themselves merged, forming a single namespace with merged interface definitions inside.

To merge the namespace value, at each declaration site, if a namespace already exists with the given name, it is further extended by taking the existing namespace and adding the exported members of the second namespace to the first.

The declaration merge of `Animals` in this example:

```ts
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

is equivalent to:

```ts
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

This model of namespace merging is a helpful starting place, but we also need to understand what happens with non-exported members.
Non-exported members are only visible in the original (un-merged) namespace. This means that after merging, merged members that came from other declarations cannot see non-exported members.

We can see this more clearly in this example:

```ts
namespace Animal {
    let haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // Error, because haveMuscles is not accessible here
    }
}
```

Because `haveMuscles` is not exported, only the `animalsHaveMuscles` function that shares the same un-merged namespace can see the symbol.
The `doAnimalsHaveMuscles` function, even though it's part of the merged `Animal` namespace can not see this un-exported member.

# Merging Namespaces with Classes, Functions, and Enums

Namespaces are flexible enough to also merge with other types of declarations.
To do so, the namespace declaration must follow the declaration it will merge with. The resulting declaration has properties of both declaration types.
TypeScript uses this capability to model some of the patterns in JavaScript as well as other programming languages.

## Merging Namespaces with Classes

This gives the user a way of describing inner classes.

```ts
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

The visibility rules for merged members is the same as described in the 'Merging Namespaces' section, so we must export the `AlbumLabel` class for the merged class to see it.
The end result is a class managed inside of another class.
You can also use namespaces to add more static members to an existing class.

In addition to the pattern of inner classes, you may also be familiar with the JavaScript practice of creating a function and then extending the function further by adding properties onto the function.
TypeScript uses declaration merging to build up definitions like this in a type-safe way.

```ts
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith"));
```

Similarly, namespaces can be used to extend enums with static members:

```ts
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

# Disallowed Merges

Not all merges are allowed in TypeScript.
Currently, classes can not merge with other classes or with variables.
For information on mimicking class merging, see the [Mixins in TypeScript](./Mixins.md) section.

# Module Augmentation

Although JavaScript modules do not support merging, you can patch existing objects by importing and then updating them.
Let's look at a toy Observable example:


```ts
// observable.ts
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.ts
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}
```

This works fine in TypeScript too, but the compiler doesn't know about `Observable.prototype.map`.
You can use module augmentation to tell the compiler about it:

```ts
// observable.ts
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.ts
import { Observable } from "./observable";
declare module "./observable" {
    interface Observable<T> {
        map<U>(f: (x: T) => U): Observable<U>;
    }
}
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}


// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map(x => x.toFixed());
```

The module name is resolved the same way as module specifiers in `import`/`export`.
See [Modules](./Modules.md) for more information.
Then the declarations in an augmentation are merged as if they were declared in the same file as the original.

However, there are two limitations to keep in mind: 

1. You can't declare new top-level declarations in the augmentation -- just patches to existing declarations.
2. Default exports also cannot be augmented, only named exports (since you need to augment an export by its exported name, and `default` is a reserved word - see [#14080](https://github.com/Microsoft/TypeScript/issues/14080) for details)

## Global augmentation

You can also add declarations to the global scope from inside a module:

```ts
// observable.ts
export class Observable<T> {
    // ... still no implementation ...
}

declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
}
```

Global augmentations have the same behavior and limits as module augmentations.