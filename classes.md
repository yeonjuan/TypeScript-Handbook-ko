# 소개 (Introduction)

기존의 JavaScript는 재사용할 수 있는 컴포넌트를 만들기 위해 함수와 프로토타입 기반의 상속을 사용했지만, 함수를 상속받는 클래스에서 객체가 만들어질때 객체 지향 접근 방식에 익숙한 프로그래머의 입장에서는 다소 어색함을 느낄 수 있습니다. ECMAScript 6로 알려진 ECMAScript 2015를 시작으로 JavaScript 프로그래머들은 이런 객체 지향적 클래스 기반의 접근 방식을 사용해서 애플리케이션을 만들 수 있습니다. TypeScript에서는 다음 버전의 JavaScript를 기다릴 필요 없이 개발자들이 이러한 기법들을 사용하고 JavaScript로 컴파일하여 주요 브라우저와 플랫폼에서 동작하게 합니다.

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


마지막 줄에서, `new`를 사용하여 클래스 `Greeter`의 인스턴스를 생성합니다. 이 코드는 이전에 정의한 생성자를 호출하여 `Gretter` 형태의 새로운 객체를 만들고, 생성자를 이용하여 초기화합니다.
