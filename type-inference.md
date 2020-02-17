# 소개

이번 장에서는 타입스크립트의 타입 추론(type inference)을 이야기합니다.
즉, 타입이 어디에서, 어떻게 추론되는지 설명합니다.

# 기초

타입스크립트는 명시적인 타입이 없는 경우에 타입 정보를 제공하기 위해 타입을 추론합니다.
예를 들어, 이 코드에서

```ts
let x = 3;
```

`x` 변수의 타입은 `number`로 추론됩니다. 이러한 종류의 추론은 변수와 멤버를 초기화하고, 매개변수의 디폴트 변수를 설정하고, 함수의 리턴 값을 결정할 때 발생합니다.

대부분의 경우에 타입 추론은 직관적입니다.
이어지는 내용에서는 어떻게 타입이 추론되는지에 좀 더 자세히 알아봅니다.


# Best common type

타입 추론이 여러 표현식에 의해 이루어질 때, 이러한 표현식들의 타입들은 best common type을 계산할 때 사용됩니다.

```ts
let x = [0, 1, null];
```

위 예제의 `x` 타입을 추론하려면 각각의 배열 요소의 타입에 대해 고려해야 합니다.
여기서 배열의 타입으로 고를 수 있는 두 가지 후보가 있습니다: `number`와 `null`입니다.
best common type 알고리즘은 각 후보 타입을 고려하여 그 중 모든 다른 후보 타입을 포함할 수 있는 타입을 선택합니다.

best common type은 후보 타입으로부터 선택하기에, 모든 후보 타입을 포함할 수 있는 super 타입이 존재해도 후보 타입 중 super 타입이 존재하지 않아 선택하지 못하는 경우가 있습니다.

예를 들어,

```ts
let zoo = [new Rhino(), new Elephant(), new Snake()];
```

이상적으로는 `zoo` 변수가 `Animal[]` 타입으로 추론되길 원하지만 배열 중 `Animal`과 완전 동일한 타입을 나타내는 객체가 없어 Animal을 배열 타입으로 추론하지 않습니다.
이를 해결하기 위해서는 명시적으로 모든 후보 타입을 포함하는 super 타입을 제공해야 합니다.

```ts
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

best common type이 존재하지 않는 경우 추론의 결과는 union 배열 객체 타입입니다. `(Rhino | Elephant |  Snake)[]`처럼요.


# Contextual Typing


Contextual typing은 표현식의 타입을 표현식의 위치에 따라 추론할 때 발생합니다. 예를 들어,

```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.button);   //<- OK
    console.log(mouseEvent.kangaroo); //<- 에러!
};
```

여기에서 타입 스크립트 타입 체커는 `Window.onmousedown` 함수 타입을 우항식에 있는 함수 표현식의 타입으로 추론합니다.
이렇게 했을 때 mouseEvent 매개변수의 [타입](https://developer.mozilla.org/en-US/docs/Web/API/MouseEvent)이 `button` 프로퍼티를 가지고 있지만,
  `kangaroo` 프로퍼티는 가지고 있지 않음을 추론할 수 있습니다.

타입스크립트는 다른 맥락에서도 타입 추론을 잘해냅니다.

```ts
window.onscroll = function(uiEvent) {
    console.log(uiEvent.button); //<- 오류!
}
```

위의 함수가 `Window.onscroll`에 할당된다는 사실을 기반으로, 타입스크립트는 `uiEvent`가 이전 예제의 `MouseEvent`가 아닌 [UIEvent](https://developer.mozilla.org/en-US/docs/Web/API/UIEvent) 임을 알고 있습니다.
`UiEvent` 객체는 `button` 프로퍼티를 가지지 않기에, 타입스크립트는 에러를 발생시킵니다.
만약 이 함수가 맥락적으로 타입이 추론되지 않는 위치에 있다면, 함수의 argument는 암묵적으로 `any` 타입을 가집니다. 그리고 별도의 에러를 보고하지 않습니다. (--noImplicitAny 옵션을 사용하지 않는다면)

```ts
const handler = function(uiEvent) {
    console.log(uiEvent.button); //<- 성공
}
```

우리는 명시적으로 함수의 argument 타입을 any 타입으로 재정의할 수 있습니다.

```ts
window.onscroll = function(uiEvent: any) {
    console.log(uiEvent.button);  //<- 이제 오류가 발생하지 않음
};
```

하지만 `uiEvent`는 `button`이라는 프로퍼티를 가지고 있지 않아 이 코드는 `undefined`을 출력합니다.

Contextual typing은 함수 호출 인수나, 우항식이나, 타입 단언, 오브젝트 / 어레이 리터럴의 멤버나, 리턴문을 포함하여 많은 상황에 적용됩니다.
Contextual type은 best common type의 후보 타입으로도 쓰입니다.
예를 들어:

```ts
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

이 예제에서 best common type은 4가지 후보 타입을 가집니다: `Animal`, `Rhino`, `Elephant`, and `Snake`.
이들 중, `Animal`이 best common type 알고리즘에 의해 선택됩니다.
