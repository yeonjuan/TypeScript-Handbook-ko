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

`...작업중`
