# Variable Declarations

`let`과 `const`는 JavaScript에서 비교적 새로운 두 가지 유형의 변수 선언입니다.
[앞에서 언급했듯이](./basic-types.md#a-note-about-let), `let`은 `var`와 어느 정도 유사하지만, 사용자가 JavaScript에서 자주 사용하는 결함을 피할 수 있게 해줍니다.
`const`는 `let`의 기능이 강화된 것으로 변수에 재할당을 방지합니다.

TypeScript는 JavaScript의 상위 집합이므로, 당연히 `let`과 `const`를 지원합니다.
여기서는 새로운 선언 방식들과 왜 그 방식들이 `var`보다 선호되는지를 더 자세히 설명하겠습니다.

만약, JavaScript를 아무렇게나 사용하고 있었다면, 다음 섹션이 기억을 새로 고치도록 도와줄 것입니다.
JavaScript에서 `var` 선언의 단점들에 대해 모두 알고 있다면 쉽게 넘어갈 수 있을 것입니다.

# `var` declarations

전통적으로 JavaScript에서 변수 선언을 할 때는 `var` 키워드를 사용하였습니다.

```ts
var a = 10;
```

알다시피, `a`라는 변수를 `10`이라는 값으로 선언했습니다.

또한, 변수를 함수 내부에 선언할 수도 있습니다.

```ts
function f() {
    var message = "Hello, world!";

    return message;
}
```

그리고, 같은 변수를 다른 함수 안에서 접근할 수도 있습니다.

```ts
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // '11'을 반환
```

위 예제에서, `g`는 `f` 안에 선언된 `a`를 잡아 둡니다.
언제든 `g`가 호출될 때, `a`의 값은 `f` 안의 `a` 값을 가리킵니다.
`f`가 실행되면서 `g`가 한번 호출된 후에도, `a`에 접근해 수정할 수 있습니다.

```ts
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // '2' 반환
```

## Scoping rules

`var` 선언은 다른 언어에서 와는 다른 몇몇의 이상한 스코프 규칙을 가지고 있습니다.
아래 예제를 살펴보겠습니다:

```ts
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // '10' 반환
f(false); // 'undefined' 반환
```

이 예제에서, 어떤 분들은 머뭇거릴 수도 있습니다.
변수 `x`는 `if` 블록 안에 선언되어 있지만, 블록의 바깥에서도 이 변수에 접근할 수 있습니다.
이 이유는 `var`선언은 이를 감싸고 있는 블록에 관계없이 이를 감싸고 있는 함수, 모듈, 네임스페이스, 전역 스코프에서 접근할 수 있기 때문입니다.
어떤 이는 이를 *`var`-스코프* 혹은 *함수 스코프*라고 부릅니다.
매개 변수 또한 함수 스코프입니다.

이런 스코프 규칙은 몇 가지 실수를 유발할 수 있습니다.
더욱 문제를 심각하게 하는 것은 변수를 여러 번 선언하는 것이 에러가 아니라는 것입니다.

```ts
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

아마 쉽게 찾을 수 있겠지만, `i`가 같은 함수 스코프의 변수를 참조하고 있기 때문에 `for`-loop 안에서 실수로 변수 `i`를 덮어쓸 수도 있습니다
경험 많은 개발자는 바로 알아차리겠지만, 비슷한 종류의 버그는 코드 리뷰를 거치며 좌절의 원인이 되기도 합니다.

## Variable capturing quirks

다음 코드의 출력 결과를 예상해 보세요:
```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

익숙하지 않은 분들을 위해 말씀드리자면, `setTimeout`은 특정 밀리 초 후에 함수를 실행하려고 할것입니다.(다른 작업의 실행이 멈추는 것을 기다리며)

준비됐나요? 살펴보겠습니다:
```text
10
10
10
10
10
10
10
10
10
10
```

많은 JavaScript 개발자들은 이런 동작에 익숙한 편이지만, 만약 놀랐더라도 당신 혼자만 놀란 것은 아닙니다. 
많은 사람들이 출력 결과가 다음과 같을 거라고 생각합니다.

```text
0
1
2
3
4
5
6
7
8
9
```

앞서 변수 캡쳐링에 대해 언급했던 부분을 기억하나요?
`setTimout`에 전달하는 모든 함수 표현식은 사실 같은 스코프에서 같은 `i`를 참조합니다.

잠시, 이게 무슨 뜻인지 생각해 보세요.
`setTimeout`은 함수를 몇 밀리 초 후에 실행 시키겠지만. *항상*`for`루프가 실행을 멈추고 난 뒤에 실행됩니다.
`for` 루프가 실행을 중지했을 때, `i`의 값은 `10`입니다.
따라서 매번 주어진 함수가 호출될 때마다 `10`을 출력할 것입니다.

일반적으로 이를 동작하게 하는 방법은 즉시 실행 함수(IIFE - an Immediately Invoked Function Expression)를 사용해 매 반복마다 `i`를 잡아두는 것입니다:

```ts
for (var i = 0; i < 10; i++) {
    // 현재 값으로 함수를 호출시켜
    // 현재 상태의 'i'를 잡아둔다.
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

이런 이상해 보이는 패턴이 사실 일반적인 패턴입니다.
매개변수에 `i`가 `for` 루프의 `i`를 감춰 버립니다. 하지만 이름을 같게 했기 때문에 루프의 실행 부를 크게 수정할 필요가 없습니다.

# `let` declarations

이제, `var`에  몇 가지 문제점에 대해 알게 되었는데, 이런 이유 때문에 `let`이 도입되게 되었습니다.
사용되는 키워드를 빼고는 `let` 문은 `var`와 동일한 방법으로 작성됩니다.

```ts
let hello = "Hello!";
```

주요한 차이점은 구문에 있다기 보단, 의미에 있는데, 이제 이 내용을 살펴볼 것입니다.

## Block-scoping

변수가 `let`을 이용해 선언되었을 때, 이는 *렉시컬 스코핑(lexical-scoping)* 혹은 *블록 스코핑(block-scoping)* 이라 불리는 것을 사용합니다.
`var`로 선언된 변수가 이를 포함한 함수까지 흘러나오는 것과 달리, 블록-스코프  변수들은 이를 가장 가깝게 감싸고 있는 블록 혹은 `for`-루프 밖에서 접근할 수 없습니다.

```ts
function f(input: boolean) {
    let a = 100;

    if (input) {
        // 'a'를 참조할 수 있습니다.
        let b = a + 1;
        return b;
    }

    // Error: 'b'는 여기서 존재하지 않습니다.
    return b;
}
```

여기, 두 지역 변수 `a`와 `b`가 있습니다.
`a`의 스코프는 `f`의 몸체로 한정되지만, `b`는 이를 감싸고 있는 `if`문의 블록까지로 한정됩니다.

`catch` 문에 선언된 변수 또한 비슷한 스코프 규칙을 가집니다.

```ts
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e'는 여기서 존재하지 않습니다.
console.log(e);
```

또 다른 블록-스코프 변수의 특징은 이 변수들이 선언되기 전에 읽거나, 쓰는 것이 불가능하다는 것입니다.
이 변수들은 스코프에 걸쳐 "존재"하지만, 선언되는 부분 전까지 모든 부분들이 *templaral dead zone*입니다.
이것은 `let`문 이전에 변수들에 접근할 수 없다는 정교한 방식이며, 다행히 TypeScript가 알려줍니다.

```ts
a++; // `a`가 선언되기 전에 잘못된 사용.
let a;
```

주의할 점은 여전히 선언되기 전에 블록-스코프 변수를 *잡아둘* 수 있다는 것입니다.
선언되기 전에 함수를 실행하는 것이 안된 다는 것만 알아두면 됩니다. 
ES2015를 대상으로한, 현대 런타임은 에러를 던질 것입니다; 하지만 현재 TypeScript에서는 허용되며, 에러를 보고하지 않습니다.

```ts
function foo() {
    // okay to capture 'a'
    return a;
}

// `a`가 선언되기 전에 `foo` 를 호출
// 런타임에 에러를 던질 것 입니다.
foo();

let a;
```

temporal dead zone에 더 자세한 설명은 [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let)를 살펴보세요.

`...작업중`
