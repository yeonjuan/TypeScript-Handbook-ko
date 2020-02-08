# 소개 (Introduction)

기존의 JavaScript는 재사용할 수 있는 컴포넌트를 만들기 위해 함수와 프로토타입 기반의 상속을 사용했지만, 함수를 상속받는 클래스에서 객체가 만들어질 때 객체 지향 접근 방식에 익숙한 프로그래머의 입장에서는 다소 어색함을 느낄 수 있습니다. ECMAScript 6로 알려진 ECMAScript 2015를 시작으로 JavaScript 프로그래머들은 이런 객체 지향적 클래스 기반의 접근 방식을 사용해서 애플리케이션을 만들 수 있습니다. TypeScript에서는 다음 버전의 JavaScript를 기다릴 필요 없이 개발자들이 이러한 기법들을 사용하여 기존의 JavaScript로 컴파일하여 주요 브라우저와 플랫폼에서 동작하게 합니다.

# 클래스 (Classes)

간단한 클래스 기반 예제를 살펴보겠습니다.

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world");
```

C# 이나 Java를 사용해봤다면, 이 예제 코드는 익숙할 것입니다. `Greeter` 라는 새로운 클래스를 선언했습니다. 이 클래스는 3개의 멤버를 가지고 있습니다: 프로퍼티 `greeting`, 생성자 그리고 	`greet`라는 메소드 입니다.

클래스의 멤버를 참조할 때 클래스에서 `this`를 앞에 덧붙입니다. `this`는 멤버에 접근하는 것을 의미합니다.


마지막 줄에서, `new`를 사용하여 클래스 `Greeter`의 인스턴스를 생성합니다. 이 코드는 이전에 정의한 생성자를 호출하여 `Greeter` 형태의 새로운 객체를 만들고, 생성자를 이용하여 초기화합니다.

# 상속 (Inheritance)

TypeScript에서는, 일반적인 객체 지향 패턴을 사용할 수 있습니다. 이미 존재하는 클래스를 상속하여 확장된 클래스를 생성하는 방식은 클래스 기반의 프로그래밍에서 가장 기본적인 패턴 중 하나입니다.

예제를 살펴보겠습니다.

```ts
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal {
    bark() {
        console.log('Woof! Woof!');
    }
}

const dog = new Dog();
dog.bark();
dog.move(10);
dog.bark();
```

가장 기본적으로 상속하는 기능을 보여주는 예제입니다: 클래스는 기초 클래스로부터 프로퍼티와 메서드를 상속받습니다. 여기서, `Dog`은 `extends` 키워드를 사용하여 `Animal`이라는 *기초* 클래스로부터 파생된 *파생* 클래스입니다. 파생된 클래스는 *하위클래스(subclasses)*, 기초 클래스는 *상위클래스(superclasses)* 라고 불리기도 합니다.

`Dog`는 `Animal`의 기능을 확장하기 때문에, `bark()`와 `move()`를 모두 가진 `Dog` 인스턴스를 생성할 수 있습니다.

조금 더 복잡한 예제를 살펴보겠습니다.

```ts
class Animal {
    name: string;
    constructor(theName: string) { this.name = theName; }
    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters);
    }
}

class Horse extends Animal {
    constructor(name: string) { super(name); }
    move(distanceInMeters = 45) {
        console.log("Galloping...");
        super.move(distanceInMeters);
    }
}

let sam = new Snake("Sammy the Python");
let tom: Animal = new Horse("Tommy the Palomino");

sam.move();
tom.move(34);
```

이 예제는 앞에서 언급하지 않은 몇 가지 기능에 대해 다룹니다. 이번에도 `extends` 키워드를 사용하여 `Animal`의 하위클래스인 `Horse`와 `Snake`를 생성합니다.

이전 예제와 한 가지 다른 부분은 파생된 클래스의 생성자 함수는 기초 클래스의 생성자를 실행할 `super()`를 *호출해야 한다는 점*입니다. 더욱이 생성자 내에서 `this`에 있는 프로퍼티에 접근하기 전에 `super()`를 먼저 호출해야 합니다. 이 부분은 TypeScript에서 강하게 요구할 중요한 규칙입니다.

또한 이 예제는 기초 클래스의 메서드를 하위클래스에 특화된 메서드로 오버라이드하는 방법을 보여줍니다. 여기서 `Snake`와 `Horse`는 `Animal`의 `move`를 오버라이드해서 각각 클래스의 특성에 맞게 기능을 가진 `move`를 생성합니다. `tom`은 `Animal`로 선언되었지만 `Horse`의 값을 가지므로 `tom.move(34)`는 `Horse`의 오버라이딩 메서드를 호출합니다.

```ts
Slithering...
Sammy the Python moved 5m.
Galloping...
Tommy the Palomino moved 34m.
```

# Public, private 그리고 protected 지정자 (Public, private, and protected modifiers)

## 기본적인 공개 (Public by default)

우리의 예제에서는, 프로그램 내에서 선언된 멤버들에게 자유롭게  접근할 수 있습니다. 다른 언어의 클래스가 익숙하다면, 위의 예제에서 `public`을 사용하지 않아도 된다는 점을 알 수 있습니다. 예를 들어, C#에서는 명시적으로 각 멤버에 `public`을 붙여야 합니다. TypeScript에서는 기본적으로 각 멤버는 `public`입니다.

명시적으로 멤버를 `public`으로 표시할 수도 있습니다. 이전 섹션의 `Animal` 클래스를 다음과 같은 방식으로 작성할 수 있습니다.

```ts
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

## ECMAScript 비공개 필드 (ECMAScript Private Fields)

TypeScript 3.8에서, TypeScript는 비공개 필드에 관한 JavaScript의 새로운 문법을 지원합니다.

```ts
class Animal {
    #name: string;
    constructor(theName: string) { this.#name = theName; }
}

new Animal("Cat").#name; // 프로퍼티 '#name'은 비공개 식별자이기 때문에 'Animal' 클래스 외부에선 접근할 수 없습니다.
```

이 문법은 JavaScript 런타임에 내장되어 있으며, 각각의 비공개 필드의 격리를 더 잘 보장할 수 있습니다. 현재 TypeScript 3.8 [릴리즈 노트](https://devblogs.microsoft.com/typescript/announcing-typescript-3-8-beta/#type-only-imports-exports)에 이러한 비공개 필드에 대해 자세히 나와있습니다. 

## TypeScript의 `private` 이해하기 (Understanding TypeScript’s `private`)

TypeScript에는 멤버를 포함하는 클래스 외부에서 이 멤버에 접근하지 못하도록 멤버를 `private`으로 표시하는 방법이 있습니다. 예: 

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // 오류: 'name'은 비공개로 선언되어 있습니다;
```

TypeScript는 구조적인 타입 시스템입니다. 두개의 다른 타입을 비교할 때 어디서 왔는지에 상관없이 모든 멤버의 타입이 호환이 된다면, 그 타입들 자체가 호환 가능하다고 말합니다.

그러나 `private` 및 `protected` 멤버가 있는 타입들을 비교할 때, 이러한 타입들은 다르게 처리합니다. 호환된다고 판단되는 두 개의 타입 중 한 쪽에서 `private` 멤버를 가지고 있다면, 다른 한 쪽도 무조건 동일한 선언에 `private` 멤버를 가지고 있어야 합니다. 이것은 `protected` 멤버에도 적용됩니다.

실제로 어떻게 작동하는지 잘 알아보기 위해 다음 예제를 살펴보겠습니다.

```ts
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

class Rhino extends Animal {
    constructor() { super("Rhino"); }
}

class Employee {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

let animal = new Animal("Goat");
let rhino = new Rhino();
let employee = new Employee("Bob");

animal = rhino;
animal = employee; // 오류: 'Animal'과 'Employee'은 호환될 수 없음.
```

이 예제에서는 `Animal`과 `Animal`의 하위클래스인 `Rhino`가 있습니다. `Animal`과 형태가 같아보이는 `Employee`라는 새로운 클래스도 있습니다. 이러한 클래스들의 인스턴스를 생성하여 할당하고 어떻게 작동하는지 살펴보겠습니다. `Animal`과 `Rhino`는 `Animal`의 `private name:string`이라는 동일한 선언으로부터 `private` 부분을 공유하기 때문에 호환이 가능합니다. 하지만 `Employee` 경우는 그렇지 않습니다. `Employee`를 `Animal`에 할당할 때, 타입이 호환되지 않다는 오류가 발생합니다. `Employee`는 `name`이라는 `private` 멤버를 가지고 있지만, `Animal`에서 선언한 것이 아니기 때문입니다.

## `protected` 이해하기 (Understanding protected)

`protected` 지정자도 `protected`로 선언된 멤버를 파생된 클래스 내에서 접근할 수 있다는 점만 제외하면 `private`지정자와 매우 유사합니다. 예를 들면,

```ts
class Person {
    protected name: string;
    constructor(name: string) { this.name = name; }
}

class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
console.log(howard.getElevatorPitch());
console.log(howard.name); // 오류
```

`Person` 외부에서 `name`을 사용할 수 없지만, `Employee`는 `Person`에서 파생되었기 때문에 `Employee`의 인스턴스 메서드 내에서는 여전히 사용할 수 있습니다.

생성자 또한 `protected`로 표시될 수도 있습니다. 이는 클래스를 포함하는 클래스 외부에서 인스턴스화 할 수 없지만 확장 할 수 있음을 의미합니다. 예를 들면,

```ts
class Person {
    protected name: string;
    protected constructor(theName: string) { this.name = theName; }
}

// Employee는 Person 확장합니다.
class Employee extends Person {
    private department: string;

    constructor(name: string, department: string) {
        super(name);
        this.department = department;
    }

    public getElevatorPitch() {
        return `Hello, my name is ${this.name} and I work in ${this.department}.`;
    }
}

let howard = new Employee("Howard", "Sales");
let john = new Person("John"); // 오류: 'Person'의 생성자는 protected 입니다.
```

# 읽기전용 지정자 (Readonly modifier)

`readonly`키워드를 사용하여 프로퍼티를 읽기전용으로 만들 수 있습니다. 읽기전용 프로퍼티들은 선언되거나 생성자에서 초기화해야 합니다.

```ts
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 오류! name은 읽기전용 입니다.
```

## 매개변수 프로퍼티 (Parameter properties)

마지막 예제의 `Octopus` 클래스 내에서 `name`이라는 읽기전용 멤버와 `theName`이라는 생성자 매개변수를 선언했습니다. 이는 `Octopus`의 생성자가 수행된 후에 `theName`의 값에 접근하기 위해서 필요합니다. *매개변수 프로퍼티*를 사용하면 한 곳에서 멤버를 만들고 초기화할 수 있습니다. 다음은 매개변수 프로퍼티를 사용한 더 개정된 `Octopus`클래스입니다.

```ts
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```

생성자에 짧아진 `readonly name: string` 파라미터를 사용하여 `theName`을 제거하고 `name` 멤버를 생성하고 초기화했습니다. 선언과 할당을 한 곳으로 통합했습니다.

매개변수 프로퍼티는 접근 지정자나 `readonly` 또는 둘 모두를 생성자 매개변수에 접두어로 붙여 선언합니다. 매개변수 프로퍼티에 `private`을 사용하면 비공개 멤버를 선언하고 초기화합니다.마찬가지로, `public`, `protected`, `readonly`도 동일하게 작용합니다.

`작업중 ... `




