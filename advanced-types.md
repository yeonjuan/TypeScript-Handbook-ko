# 목차 (Table of contents)

[교차 타입 (Intersection Types)](#교차-타입-Intersection-Types)

[유니언 타입 (Union Types)](#유니언-타입-union-types)

[타입 가드와 차별 타입 (Type Guards and Differentiating Types)](#타입-가드와-차별-타입-type-guards-and-differentiating-types)
* [사용자-정의 타입 가드 (User-Defined Type Guards)](#사용자-정의-타입-가드-user-defined-type-guards)
  * [타입 서술어 사용하기 (Using type predicates)](#타입-서술어-사용하기-using-type-predicates)
  * [`in` 연산자 사용하기 (Using the `in` operator)](#in-연산자-사용하기-using-the-in-operator)

[널러블 타입 (Nullable types)](#널러블-타입-nullable-types)
* [선택적 매개변수와 프로퍼티 (Optional parameters and properties)](#선택적-매개변수와-프로퍼티-optional-parameters-and-properties)
* [타입 가드와 타입 단언 (Type guards and type assertions)](#타입-가드와-타입-단언-type-guards-and-type-assertions)

[타입 별칭 (Type Aliases)](#타입-별칭-type-aliases)
* [인터페이스 vs. 타입 별칭 (Interfaces vs. Type Aliases)](#인터페이스-vs-타입-별칭-interfaces-vs-type-aliases)

[문자열 리터럴 타입 (String Literal Types)](#문자열-리터럴-타입-string-literal-types)

[숫자 리터럴 타입 (Numeric Literal Types)](#숫자-리터럴-타입-numeric-literal-types)

[열거형 멤버 타입 (Enum Member Types)](#열거형-멤버-타입-enum-member-types)

# 교차 타입 (Intersection Types)

교차 타입은 여러 타입을 하나로 결합합니다.
기존 타입을 합쳐 필요한 모든 기능을 가진 하나의 타입을 얻을 수 있습니다.
예를 들어, `Person & Serializable & Loggable`은 `Person` *과* `Serializable` *그리고* `Loggable`입니다.
즉, 이 타입의 객체는 세 가지 타입의 모든 멤버를 갖게 됩니다.

기존 객체-지향 틀과는 맞지 않는 믹스인(mixin)이나 다른 컨셉들에서 교차 타입이 사용되는 것을 볼 수 있습니다.
(JavaScript에는 이런 것들이 많습니다!)
믹스인을 어떻게 만드는지 간단한 예제를 보겠습니다:

```ts
function extend<First, Second>(first: First, second: Second): First & Second {
    const result: Partial<First & Second> = {};
    for (const prop in first) {
        if (first.hasOwnProperty(prop)) {
            (result as First)[prop] = first[prop];
        }
    }
    for (const prop in second) {
        if (second.hasOwnProperty(prop)) {
            (result as Second)[prop] = second[prop];
        }
    }
    return result as First & Second;
}

class Person {
    constructor(public name: string) { }
}

interface Loggable {
    log(name: string): void;
}

class ConsoleLogger implements Loggable {
    log(name) {
        console.log(`Hello, I'm ${name}.`);
    }
}

const jim = extend(new Person('Jim'), ConsoleLogger.prototype);
jim.log(jim.name);
```

# 유니언 타입 (Union Types)

유니언 타입은 교차 타입과 밀접하게 관련되어 있지만, 매우 다르게 사용됩니다.
가끔, `숫자`나 `문자열`을 매개변수로 기대하는 라이브러리를 사용할 때가 있습니다.
예를 들어, 다음 함수를 사용할 때입니다:

```ts
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // "    Hello world"를 반환합니다.
```

`padLeft`의 문제는 매개변수 `padding`이 `any` 타입으로 되어있다는 것입니다.
즉, `숫자`나 `문자열` 둘 다 아닌 인수로 호출할 수 있다는 것이고, TypeScript는 이를 괜찮다고 받아들일 것입니다.

```ts
let indentedString = padLeft("Hello world", true); // 컴파일 타임에 통과되고, 런타임에 오류.
```

전통적인 객체지향 코드에서, 타입의 계층을 생성하여 두 타입을 추상화할 수 있습니다.
이는 더 명시적일 수는 있지만, 좀 과하다고 할 수도 있습니다.
`padLeft`의 기존 버전에서 좋은 점은 그냥 원시값을 전달할 수 있다는 것입니다.
즉 사용법이 간단하고 간결합니다.
이 새로운 방법은 다른 곳에서 이미 존재하는 함수를 사용하려 할 때, 도움이 되지 않습니다.

`any` 대신에, *유니언 타입*을 매개변수 `padding`에 사용할 수 있습니다:

```ts
/**
 * 문자열을 받고 왼쪽에 "padding"을 추가합니다.
 * 만약 'padding'이 문자열이라면, 'padding'은 왼쪽에 더해질 것입니다.
 * 만약 'padding'이 숫자라면, 그 숫자만큼의 공백이 왼쪽에 더해질 것입니다.
 */
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true); // 컴파일 중에 오류
```

유니언 타입은 값이 여러 타입 중 하나임을 설명합니다.
세로 막대 (`|`)로 각 타입을 분리하여 사용합니다. 그래서 `number | string | boolean`은 값의 타입이 `number`, `string` 혹은 `boolean`이 될 수 있음을 나타냅니다.

유니언 타입인 값을 가지고 있으면, 유니언에 있는 모든 타입에 공통인 멤버에만 접근할 수 있습니다.

```ts
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // 성공
pet.swim();    // 오류
```

유니언 타입은 여기서 약간 까다로울 수 있으나, 익숙해지는데 약간의 직관만 있으면 됩니다.
만약 값이 `A | B` 타입을 가지고 있으면, `A` *와* `B` 둘 다 가지고 있는 멤버가 있다는 것만 *확실히* 알고 있습니다.
이 예제에서, `Bird`는 `fly`로 부르는 멤버를 가지고 있습니다.
`Bird | Fish`로 타입이 지정된 변수가 `fly` 메서드를 가지고 있는지 확신할 수 없습니다
만약 변수가 실제로 런타임에 `Fish`이면, `pet.fly()`를 호출하는 것은 오류입니다.

# 타입 가드와 차별 타입 (Type Guards and Differentiating Types)

유니언 타입은 값이 가질 수 있는 타입이 겹쳐질 수 있는 상황을 모델링하는데 유용합니다.
`Fish`가 있는지 구체적으로 알고 싶을 때, 무슨일이 벌어질까요?
JavaScript에서 가능한 두 값을 구분하는 흔한 관용구는 멤버의 존재를 검사하는 것입니다.
앞에서 말했듯이, 유니언 타입의 모든 구성 성분을 가지고 있다고 보장되는 멤버에만 접근할 수 있습니다.

```ts
let pet = getSmallPet();

// 이런 각각의 프로퍼티들에 접근하는 것은 오류를 발생시킵니다
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

같은 코드를 동작하게 하려면, 타입 단언을 사용해야 합니다:

```ts
let pet = getSmallPet();

if ((pet as Fish).swim) {
    (pet as Fish).swim();
} else if ((pet as Bird).fly) {
    (pet as Bird).fly();
}
```

## 사용자-정의 타입 가드 (User-Defined Type Guards)

타입 단언을 여러 번 사용해야만 했던 것을 주목해야 합니다.
만약 검사를 실시했을 때, 각 브랜치 안의 `pet`의 타입을 알 수 있었다면 훨씬 좋았을 것입니다.

마침 TypeScript에는 *타입 가드*라는 것이 있습니다.
타입 가드는 어떤 스코프 안에서의 타입을 보장하는 런타임 검사를 시행한다는 표현입니다.

### 타입 서술어 사용하기 (Using type predicates)

타입 가드를 정의하기 위해, 간단하게 반환 타입이 *타입 서술어*인 함수를 정의하면 됩니다:

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
```

`pet is Fish`는 이 예제에서의 타입 서술어입니다.
서술어는 `parameterName is Type` 형태이고, `parameterName`는 반드시 현재 함수 시그니처에서 매개변수의 이름이어야 합니다.

`isFish`가 어떤 변수와 함께 호출될 때마다, TypeScript는 기존 타입과 호환된다면 그 변수를 특정 타입으로 *제한*할 것입니다.

```ts
// 이제 'swim'과 'fly'에 대한 모든 호출은 허용됩니다

if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```

TypeScript가 `pet`이 `if`문 안에서 `Fish`라는 것을 알고 있을뿐만 아니라; `else`문 안에서 `Fish`가 *없다*는 것을 알고 있으므로, `Bird`를 반드시 가지고 있어야합니다.

### `in` 연산자 사용하기 (Using the `in` operator)

`in` 연산자는 타입을 좁히는 표현으로 작용합니다.

`n in x` 표현에서, `n`은 문자열 리터럴 혹은 문자열 리터럴 타입이고 `x`는 유니언 타입입니다. "true" 분기에서는 선택적 혹은 필수 프로퍼티 `n`을 가지는 타입으로 좁히고, "false" 분기에서는 선택적 혹은 누락된 프로퍼티 `n`을 가지는 타입으로 좁혀집니다.

```ts
function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly();
}
```

# 널러블 타입 (Nullable types)

TypeScript는 두 가지 특별한 타입 `null`과 `undefined`가 있는데, 각각 값이 null과 undefined를 가집니다.

[기본 타입](./basic-types.md)에서 짧게 언급한 바 있습니다.
기본적으로, 타입 체커는 `null`과 `undefined`를 아무것에나 할당할 수 있다고 간주합니다.
실제로 `null`과 `undefined`는 모든 타입의 유효한 값입니다.
즉, 막고 싶어도 모든 타입에 할당되는 것을 *막을 수* 없습니다.
`null`의 개발자, Tony Hoare는 이를 두고["백만 불짜리 실수 (billion dollar mistake)"](https://en.wikipedia.org/wiki/Null_pointer#History)라고 부릅니다.

`--strictNullChecks` 플래그는 이를 해결합니다: 변수를 선언할 때, 자동으로 `null`이나 `undefined`를 포함하지 않습니다.
유니언 타입을 사용하여 명시적으로 포함할 수 있습니다.

```ts
let s = "foo";
s = null; // error, 'null' is not assignable to 'string'
let sn: string | null = "bar";
sn = null; // 성공

sn = undefined; // error, 'undefined' is not assignable to 'string | null'
```

TypeScript는 JavaScript에서의 의미와 맞추기 위해 `null`과 `undefined`를 다르게 처리합니다.
`string | null`은 `string | undefined`와 `string | undefined | null`과는 다른 타입입니다.

TypeScript 3.7 이후부터는 널러블 타입을 간단하게 다룰 수 있게 [optional chaining](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-7.html#optional-chaining)를 사용할 수 있습니다.

## 선택적 매개변수와 프로퍼티 (Optional parameters and properties)

`--strictNullChecks`를 적용하면, 자동으로 선택적 매개변수 `| undefined`가 추가됩니다.

```ts
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // error, 'null' is not assignable to 'number | undefined'
```

선택적 프로퍼티도 마찬가지입니다.

```ts
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // error, 'undefined' is not assignable to 'number'
c.b = 13;
c.b = undefined; // 성공
c.b = null; // error, 'null' is not assignable to 'number | undefined'
```

## 타입 가드와 타입 단언 (Type guards and type assertions)

널러블 타입이 유니언으로 구현되기 때문에, `null`을 제거하기 위해 타입 가드를 사용할 필요가 있습니다
다행히, JavaScript에서 작성했던 코드와 동일합니다.

```ts
function f(sn: string | null): string {
    if (sn == null) {
        return "default";
    }
    else {
        return sn;
    }
}
```

여기서 `null`의 제거는 명백해 보이지만, 간단한 연산자를 사용할 수도 있습니다.

```ts
function f(sn: string | null): string {
    return sn || "default";
}
```

컴파일러가 `null`이나 `undefined`를 제거할 수 없는 상황에서, 타입 단언 연산자를 사용하여 수동으로 제거할 수 있습니다.
구문은 `!`를 후위 표기하는 방법입니다: `identifier!`는 `null`과 `undefined`를 `identifier`의 타입에서 제거합니다.

```ts
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // error, 'name' is possibly null
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}
```

예제는 중첩 함수를 사용합니다. 왜냐하면 컴파일러가 중첩 함수안에서는 null을 제거할 수 없기 때문입니다 (즉시-호출된 함수 표현은 예외).
특히 외부 함수에서 반환할 경우, 중첩 함수에 대한 모든 호출을 추적할 수 없기 때문입니다.
함수가 어디에서 호출되었는지 알 수 없으면, body가 실행될 때 `name`의 타입을 알 수 없습니다.

# 타입 별칭 (Type Aliases)

타입 별칭은 타입의 새로운 이름을 만듭니다.
타입 별칭은 때때로 인터페이스와 유사합니다만, 원시 값, 유니언, 튜플 그리고 손으로 작성해야 하는 다른 타입의 이름을 지을 수 있습니다.

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === "string") {
        return n;
    }
    else {
        return n();
    }
}
```

별칭은 실제로 새로운 타입을 만드는 것은 아닙니다 - 그 타입을 나타내는 새로운 *이름* 을 만드는 것입니다.
원시 값의 별칭을 짓는 것은 문서화의 형태로 사용할 수 있지만, 아주 유용하지 않습니다.

인터페이스처럼, 타입 별칭은 제네릭이 될 수 있습니다 - 타입 매개변수를 추가하고 별칭 선언의 오른쪽에 사용하면 됩니다:

```ts
type Container<T> = { value: T };
```

프로퍼티 안에서 자기 자신을 참조하는 타입 별칭을 가질 수 있습니다:

```ts
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

교차 타입과 같이 사용하면, 아주 놀라운 타입을 만들 수 있습니다.

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

하지만, 타입 별칭을 선언의 오른쪽 이외에 사용하는 것은 불가능합니다.

```ts
type Yikes = Array<Yikes>; // 오류
```

## 인터페이스 vs. 타입 별칭 (Interfaces vs. Type Aliases)

위에서 언급했듯이, 타입 별칭은 인터페이스와 같은 역할을 할 수 있습니다, 하지만, 약간의 미묘한 차이가 있습니다

하나의 차이는 인터페이스는 어디에서나 사용할 수 있는 새로운 이름을 만들 수 있습니다.
타입 별칭은 새로운 이름을 만들지 못합니다 &mdash; 예를 들어, 오류 메시지는 별칭 이름을 사용하지 않습니다.
아래의 코드에서, 에디터 안에서 `interfaced`에 마우스를 올리면  `Interface`를 반환한다고 보여줍니다, 하지만 `aliased`는 객체 리터럴 타입을 반환한다고 보여줍니다.

```ts
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```

TypeScript의 이전 버전에서, 타입 별칭은 extend 하거나 implement 할 수 없었습니다 (다른 타입을 extend/implement 할 수도 없습니다).
2.7 버전부터, 타입 별칭은 교차 타입을 생성함으로써 extend 할 수 있습니다. 예를 들어, `type Cat = Animal & { purrs: true }`.

왜냐하면 [소프트웨어의 이상적인 프로퍼티는 확장에 열려있는 것이다](https://en.wikipedia.org/wiki/Open/closed_principle), 가능하면 항상 타입 별칭보다 인터페이스를 사용해야 하기 때문입니다.

반면에, 만약 인터페이스로 어떤 형태를 표현할 수 없고 유니언이나 튜플 타입을 사용해야 한다면, 일반적으로 타입 별칭을 사용합니다.

# 문자열 리터럴 타입 (String Literal Types)

문자열 리터럴 타입은 문자열이 가져야 하는 정확한 값을 지정할 수 있게 해줍니다.
예제에서 문자열 리터럴 타입은 유니언 타입, 타입 가드, 그리고 타입 별칭과 잘 결합됩니다.
이 기능을 열거형-같은 행동을 문자열과 함께 사용할 수 있습니다.

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // 오류! null이나 undefined를 전달하면 안됩니다
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // 오류: "uneasy"는 여기서 허용하지 않습니다
```

허용되는 3개의 문자열 아무거나 전달할 수 있지만, 그 외 다른 문자열들은 오류를 발생시킵니다.

```text
Argument of type '"uneasy"' is not assignable to parameter of type '"ease-in" | "ease-out" | "ease-in-out"'
```

문자열 리터럴 타입은 오버로드를 구별하기 위해 같은 방법으로 사용할 수 있습니다.

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... 더 많은 오버로드 ...
function createElement(tagName: string): Element {
    // ... 이곳에 코드를 ...
}
```

# 숫자 리터럴 타입 (Numeric Literal Types)

TypeScript는 또한 숫자 리터럴 타입을 갖고 있습니다.

```ts
function rollDice(): 1 | 2 | 3 | 4 | 5 | 6 {
    // ...
}
```

좀처럼 명시적으로 작성하지 않지만, 이슈를 좁히고 버그를 잡는데 유용할 수 있습니다.

```ts
function foo(x: number) {
    if (x !== 1 || x !== 2) {
        //         ~~~~~~~
        // '!==' 연산자는 '1'과 '2' 타입에 적용할 수 없습니다.
    }
}
```

다른 말로, `x`는 `2`와 비교될 때, 반드시 `1`이어야 합니다, 위의 검사에서 유효하지 않은 비교를 만드는 것을 의미합니다.

# 열거형 멤버 타입 (Enum Member Types)

[열거형 섹션](./enums.md#유니언-열거형과-열거형-멤버-타입-union-enums-and-enum-member-types)에서 언급했듯이, 열거형 멤버는 모든 멤버가 리터럴로-초기화될 때 타입을 가집니다.

"싱글톤 타입"에 대해 이야기하는 대부분의 시간에, 비록 많은 유저가 "싱글톤 타입"과 "리터럴 타입"을 서로 교환 가능하게 사용할 것이지만, 열거형 멤버 타입과 숫자/문자열 리터럴 타입 모두 참조합니다.
