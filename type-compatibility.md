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

## 안전성에 대한 참고사항 (A Note on Soundness) 

TypeScript의 타입 시스템은 컴파일 할 때 확인할 수 없는 특정 작업을 안전하게 수행할 수 있습니다. 타입 시스템이 이 프로퍼티를 갖고 있을 때, 안전하지 않다고 말합니다. TypeScript에서 안전하지 못한 곳을 허용하는 부분을 신중하게 고려했으며, 이 문서 전체에서 이러한 상황이 발생하는 곳과 유발하는 시나리오에 대해 설명합니다.

`작업중...`
