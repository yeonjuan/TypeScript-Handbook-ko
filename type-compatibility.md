# 소개 (Introduction)

TypeScript에서 타입 호환성은 구조적 서브 타이핑(subtyping)을 기반으로 합니다. 구조적 타이핑이란 오직 멤버만으로 타입을 관계시키는 방식입니다. 명목적 타이핑(nominal typing) 과는 반대입니다. 다음 코드를 살펴보겠습니다:

```ts
interface Named {
    name: string;
}

class Person {
    name: string;
}

let p: Named;
// ok, 구조적 타이핑이기 때문입니다.
p = new Person();
```

C#이나 Java와 같은 명목적 타입 언어에서는 `Person` 클래스는 `Named` 인터페이스를 명시적인 구현체로 기술하지 않았기 때문에 해당 코드는 오류를 발생시킵니다.

TypeScript의 구조적 타입 시스템은 JavaScript 코드의 일반적인 작성 방식에 따라서 설계되었습니다. JavaScript는 함수 표현식이나 객체 리터럴 같은 익명 객체를 광범위하게 사용하기 때문에 JavaScript에서 발견되는 관계의 타입을 명목적 타입 시스템보다는 구조적 타입 시스템을 이용하여 표현하는 것이 훨씬 더 자연스럽습니다.

## 건전성에 대한 참고사항 (A Note on Soundness) 

TypeScript의 타입 시스템은 컴파일 할 때 확인할 수 없는 특정 작업을 안전하게 수행할 수 있습니다. 타입 시스템이 이 프로퍼티를 갖고 있을 때, 건전하지 않다고 말합니다. TypeScript에서 건전하지 못한 곳을 허용하는 부분을 신중하게 고려했으며, 이 문서 전체에서 이러한 상황이 발생하는 곳과 유발하는 시나리오에 대해 설명합니다.

# 시작하기 (Starting out)

TypeScript의 구조적 타입 시스템의 기본 규칙은 `y`가 최소한 `x`와 동일한 멤버를 가지고 있다면 호환된다는 것입니다. 예를 들어:

```ts
interface Named {
    name: string;
}

let x: Named;
// y의 추론된 타입은 { name: string; location: string; } 입니다.
let y = { name: "Alice", location: "Seattle" };
x = y;
```

`y`를 `x`에 할당할 수 있는지 검사하기 위해, 컴파일러는 `x`의 각 프로퍼티를 검사하여 `y`에서 상응하는 호환 가능한 프로퍼티를 찾습니다. 이 경우, `y`는 `name`이라는 문자열 멤버를 가지고 있어야 합니다. 그러므로 할당이 허용됩니다.

함수 호출 인수(function call arguments)를 검사할 때 동일한 할당 규칙이 적용됩니다.

```ts
function greet(n: Named) {
    console.log("Hello, " + n.name);
}
greet(y); // OK
```

`y`는 `location` 프로퍼티를 추가적으로 가지고 있지만 오류를 발생시키지 않는 점에 유의합니다. 호환성을 검사할 때는 오직 대상 타입의 멤버(이 경우는 `Named`)만 고려됩니다.

이 비교하는 과정은 재귀적으로 각 멤버와 하위 멤버의 타입을 탐색하면서 진행됩니다.

`작업중...`


