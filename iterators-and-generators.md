# 반복자와 생성자 (Iterators and Generators)

## 반복 가능 (Iterables)

[`Symbol.iterator`](https://www.typescriptlang.org/docs/handbook/symbols.html#symboliterator) 속성에 대한 구현이 있는 객체는 반복 가능한 것으로 간주합니다.
`Array`, `Map`, `Set`, `String`, `Int32Array`, `Uint32Array` 등과 같은 일부 내장된(built-in) 타입에는 이미 `Symbol.iterator` 속성이 구현되어 있습니다.
객체의 `Symbol.iterator` 함수는 반복할 값 목록을 반환합니다.

### for .. of 문

for..of는 반복 가능한 객체를 반복하여 객체의 `Symbol.iterator` 속성을 호출합니다.
다음은 배열의 간단한 for..of 루프입니다.

```ts
let someArray = [1, "string", false];

for (let entry of someArray){
 console.log(entry); // 1, "string", false
}
```

#### _for..of vs. for..in 문_

`for..of` 및 `for..in` 문 모두 목록을 반복합니다.
반복되는 값은 다르지만 `for..in`은 반복되는 객체의 키 목록을 반환하고 `for..of`는 반복되는 객체의 숫자 속성값 목록을 반환합니다.

다음은 이러한 차이점을 보여주는 예입니다.

```ts
let list = [4, 5, 6];

for (let i in list){
 console.log(i); // "0", "1", "2"
}

for (let i of list){
 console.log(i); // "4", "5", "6"
}
```

또 다른 차이점은 `for..in`은 모든 객체에서 작동한다는 것입니다.
이 객체의 속성을 검사하는 방법으로 사용됩니다.
반면에 `for..of`는 반복 가능한 객체의 값에 주로 관심이 있습니다.
`Map` 및 `Set`과 같은 내장된 객체는 저장된 값에 액세스할 수 있는 `Symbol.iterator` 속성을 구현합니다.

```ts
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets){
 console.log(pet); // "species"
}

for (let pet of pets){
 console.log(pet); // "Cat", "Dog", "Hamster"
}
```

_코드 생성_
_ES5 및 ES3 타게팅_

ES5 또는 ES3 호환 엔진을 대상으로하는 경우, 반복자는 배열 유형의 값에만 허용됩니다.
이러한 배열이 아닌 값이 `Symbol.iterator` 속성을 구현하더라도 비 배열 값에서 `for..of` 루프를 사용하면 오류가 발생합니다.

컴파일러는 `for..of` 루프에 대한 간단한 `for` 루프를 생성합니다. 예를 들면 다음과 같습니다.

```ts
let numbers = [1, 2, 3];
for (let num of numbers){
 console.log(num);
}
```

는 다음과 같이 생성할 것입니다:

```ts
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++){
 var num = numbers[_i];
 console.log(num);
}
```

_ESMAScript 2015 및 상위 버전 타케팅_

ECMAScipt 2015-호환 엔진(compliant engine)을 타케팅하는 경우, 컴파일러는 엔진의 내장 반복자(ieterator) 구현을 대상으로하는 `for..of` 루프를 생성합니다.