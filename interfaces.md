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