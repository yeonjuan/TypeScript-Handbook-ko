# 소개 (Introduction)

타입스크립트의 핵심 원칙 중 하나는 타입 검사가 값(value)들이 가지고 있는 *형태(shape)*에 초점을 맞추고 있는 것입니다.
이는 "덕 타이핑(duck typing)" 혹은 "구조적 서브타이핑 (structural subtyping)"이라고도 합니다.
타입스크립트에서, 인터페이스는 이런 타입들의 이름을 짓는 역할을 하고 코드 안의 계약을 정의하는 것뿐만 아니라 프로젝트 외부에서 사용하는 계약을 정의합니다.

# 첫 번째 인터페이스 (Our First Interface)

인터페이스가 어떻게 동작하는지 확인하기 위한 가장 간단한 예제:

```ts
function printLabel(labeledObj: { label: string }) {
    console.log(labeledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

타입 체커는 `printLabel`에 대한 호출을 확인합니다.
`printLabel` 함수는 `문자열` 타입의 `label`을 갖는 객체를 하나의 매개변수로 가집니다.
이 객체가 실제로는 이보다 더 많은 프로퍼티를 갖고 있지만, 컴파일러는 *최소* 필요한 프로퍼티가 있고, 이것의 타입이 잘 맞는지만 검사합니다.
타입스크립트가 관대하지 않은 몇 가지 케이스들이 있는데, 이는 나중에 다룰 것입니다.

이번엔 문자열 타입의 프로퍼티 `label`을 갖고 있음을 기술하는 인터페이스를 사용하여 같은 예제를 다시 사용해보겠습니다.

```ts
interface LabeledValue {
    label: string;
}

function printLabel(labeledObj: LabeledValue) {
    console.log(labeledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`LabeledValue` 인터페이스는 이전 예제의 요구사항을 똑같이 기술하는 이름으로 사용할 수 있습니다.
이 인터페이스는 똑같이 `문자열` 타입의 `label` 프로퍼티 하나를 가진다는 것을 의미하고 있습니다.
다른 언어에서처럼 `printLabel`에 전달한 객체가 이 인터페이스를 구현해야 한다고 명시적으로 얘기할 필요는 없습니다.
여기서 중요한 것은 형태뿐입니다. 함수에 전달된 객체가 나열된 요구 조건을 충족한다면 허용됩니다.

타입 체커는 프로퍼티들이 어떤 순서로 올 것인지에 대해서는 요구하지 않습니다. 단지 인터페이스가 요구하는 프로퍼티들이 존재하는지와 그것들이 요구하는 타입을 가졌는지만을 확인합니다.

# 선택적 프로퍼티 (Optional Properties)

인터페이스의 모든 프로퍼티가 필요한 것은 아닙니다.
어떤 조건에서만 존재하거나 아예 없을 수도 있습니다.
이런 선택적 프로퍼티들은 객체 안의 몇 개의 프로퍼티만 채워 함수에 전달하는 "option bags"와 같은 패턴을 만들 때 유용합니다.

이 패턴의 예제를 한번 보겠습니다:

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
    let newSquare = {color: "white", area: 100};
    if (config.color) {
        newSquare.color = config.color;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

선택적 프로퍼티를 가지는 인터페이스는 다른 인터페이스와 비슷하게 작성되고, 선택적 프로퍼티는 선언에서 프로퍼티 이름 끝에 `?`를 붙여 표시합니다.

선택적 프로퍼티의 이점은 인터페이스에 속하지 않는 프로퍼티 사용을 방지하면서, 사용 가능한 속성을 기술하는 것입니다.
예를 들어, `createSquare`안의 `color` 프로퍼티 이름을 잘못 타이핑하면, 에러 메시지로 알려줍니다:

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    let newSquare = {color: "white", area: 100};
    if (config.clor) {
        // Error: Property 'clor' does not exist on type 'SquareConfig'
        newSquare.color = config.clor;
    }
    if (config.width) {
        newSquare.area = config.width * config.width;
    }
    return newSquare;
}

let mySquare = createSquare({color: "black"});
```

# 읽기전용 프로퍼티 (Readonly properties)

어떤 프로퍼티들은 객체가 생성될 때만 수정 가능해야합니다.
프로퍼티 이름 앞에 `readonly`를 넣어서 이를 지정할 수 있습니다:

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}
```

객체 리터럴을 할당하여 `Point`를 생성합니다.
할당 후에는 `x`, `y`를 수정할 수 없습니다.

```ts
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript에서는 모든 변경 메서드(Mutating Methods)가 제거된 `Array<T>`와 동일한 `ReadonlyArray<T>` 타입을 제공합니다. 그래서 생성 후에 배열을 변경하지 않음을 보장할 수 있습니다.

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

예제 마지막 줄에서 일반 배열을 `ReadonlyArray`로 재할당하는 것이 금지됨을 확인할 수 있습니다.
타입 단언(type assertion)으로 오버라이드하는 것은 가능합니다.

```ts
a = ro as number[];
```

## `readonly` vs `const`

`readonly`와 `const` 중에 어떤 것을 사용할 지 기억하기 가장 쉬운 방법은 변수에 쓸 것인지 프로퍼티에 쓸 것인지 질문해 보는 것입니다.
변수는 `const`를 사용하고 프로퍼티는 `readonly`를 사용합니다

# 초과 프로퍼티 검사 (Excess Property Checks)

인터페이스의 첫 번째 예제에서 TypeScript가 `{ label: string; }`을 기대해도 `{ size: number; label: string; }`를 허용해주었습니다. 우리는 또 선택적 프로퍼티를 배우고, 소위 "option bags"을 기술할 때, 유용하다는 것을 배웠습니다.

하지만, 순진하게 그 둘을 결합하면 에러가 발생할 수 있습니다.
예를 들어, `createSquare`를 사용한 마지막 예제를 보겠습니다:

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

`color`대신에 *`colour`* 로 인수를 잘못 전달했을 경우, 일반 JavaScript에선 이런 경우 조용히 에러가 발생합니다.

`width` 프로퍼티는 적합하고, `color` 프로퍼티는 없고, 추가 `colour` 프로퍼티는 중요하지 않기 때문에, 이 프로그램이 올바르게 입력되었다고 생각 할 수 있습니다.

하지만, TypeScript는 이 코드에 버그가 있을 수 있다고 생각할  것입니다.
객체 리터럴은 다른 변수에 할당할 때나 인수로 전달할 때, 특별한 처리를 받고, *초과 프로퍼티 검사 (excess property checking)*를 받습니다.
만약 객체 리터럴이 "대상 타입 (target type)"이 갖고 있지 않은 프로퍼티를 갖고 있으면, 에러가 발생합니다.

```ts
// error: Object literal may only specify known properties, but 'colour' does not exist in type 'SquareConfig'. Did you mean to write 'color'?
let mySquare = createSquare({ colour: "red", width: 100 });
```

이 검사를 피하는 방법은 사실 정말 간단합니다.
가장 간단한 방법은 타입 단언을 사용하는 것입니다:

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

하지만, 객체가 특별한 경우에서 사용될 때, 추가 프로퍼티가 있음을 확신한다면, 문자열 인덱스 서명(string index signatuer)을 추가하는 것이 더 나은 방법입니다.
만약 `SquareConfig` `color`와 `width` 프로퍼티를 위와 같은 타입으로 갖고 있고, *또한* 다른 프로퍼티를 가질 수 있다면, 다음과 같이 정의할 수 있습니다.

```ts
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

나중에 인데스 서명에 대해 좀 더 다룰 것입니다. 하지만 여기서는 `SquareConfig`가 여러 프로퍼티가 가질 수 있고, 그 프로퍼티들이 `color`나 `width`가 아니라면, 그들의 타입은 중요하지 않습니다.

이 검사를 피하는 마지막 방법은 놀랍게도 객체를 다른 변수에 할당하는 것입니다.
`squareOptions`가 추가 프로퍼티 검사를 받지 않기 때문에, 컴파일러는 에러를 주지 않습니다.

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

`squareOptions`와 `SquareConfig` 사이에 공통 프로퍼티가 있는 경우에만 위와 같은 방법을 사용할 수 있습니다.
이 예제에서는, `width`가 그 경우입니다. 하지만 만약에 변수가 공통 객체 프로퍼티가 없으면 에러가 납니다. 예를 들어:

```ts
let squareOptions = { colour: "red" };
let mySquare = createSquare(squareOptions);
```

위처럼 간단한 코드의 경우, 이 검사를 "피하는" 방법을 시도하지 않는 것이 좋습니다.
메서드가 있거고 상태를 가지는 등 더 복잡한 객체 리터럴에서 이 방법을 생각해볼 수 있습니다. 하지만 초과 프로퍼티 에러의 대부분은 실제 버그입니다.
그 말은, 만약 옵션 백 같은 곳에서 초과 프로퍼티 검사 문제가 발생하면, 타입 정의를 수정해야 할 필요가 있습니다.
예를 들어, 만약 `createSquare`에 `color`나 `colour` 모두 전달해도 괜찮다면, `squareConfig`가 이를 반영하도록 정의를 수정해야 합니다.