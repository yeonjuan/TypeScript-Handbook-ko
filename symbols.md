# 소개 (Introduction)

ECMAScript 2015부터, `symbol`은 `number`와 `string` 같은 기본 자료형입니다.

`symbol` 값들은 `Symbol`생성자를 호출해서 생성합니다.

```ts
let sym1 = Symbol();

let sym2 = Symbol("key"); // 선택적 문자열 키
```

심볼들은 불변하며 유일합니다.

```ts
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, 심볼은 유일합니다.
```

문자열처럼, 심볼은 객체 프로퍼티의 키로 사용될 수 있습니다.

```ts
const sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```

심볼은 계산된 프로퍼티 선언과 결합해 객체 프로퍼티와 클래스 멤버를 선언할 수도 있습니다.

```ts
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

# 잘 알려진 심볼들 (Well-known Symbols)

사용자가 정의한 심볼 이외에도, 잘 알려진 내장 심볼들이 있습니다.
내장된 심볼들은 내부 언어 동작을 나타내는 데 사용됩니다.

다음은 잘 알려진 심볼 목록입니다:

## `Symbol.hasInstance`

생성자 객체가 어떤 객체를 생성자의 인스턴스로 인식하는지 확인하는 메서드입니다. instanceof 연산자에 의해 호출됩니다.

## `Symbol.isConcatSpreadable`

객체가 자신의 배열 요소를 Array.prototype.concat()에 의해 직렬로 나타낼 수 있는지를 나타내는 불리언 값입니다.

## `Symbol.iterator`

객체의 기본 반복자를 반환하는 메서드입니다. for-of 문에 의해 호출됩니다.

## `Symbol.match`

정규식을 문자열과 비교하는 정규식 메서드입니다. `String.prototype.match` 메서드에 의해 호출됩니다.

## `Symbol.replace`

일치하는 부분 문자열을 대체하는 정규식 메서드입니다. `String.prototype.replace` 메서드에 의해 호출됩니다.

## `Symbol.search`

정규식과 일치하는 문자열의 인덱스를 반환하는 정규식 메서드입니다. `String.prototype.search` 메서드에 의해 호출됩니다.

## `Symbol.species`

파생된 객체를 만드는 생성자 함수 프로퍼티 값입니다.

## `Symbol.split`

정규식과 일치하는 인덱스들에 위치한 문자열을 분할하는 정규식 메서드입니다.
`String.prototype.split` 메서드에 의해 호출됩니다.

## `Symbol.toPrimitive`

객체를 해당 기본 값으로 변환하는 메서드입니다.
`ToPrimitive` 추상 연산에 의해 호출됩니다.

## `Symbol.toStringTag`

객체의 기본 문자열 설명을 만드는 데 사용되는 문자열 값입니다.
내장 메소드`Object.prototype.toString`에 의해 호출됩니다.

## `Symbol.unscopables`

고유한 프로퍼티 이름들이 연관된 객체의 with 환경 바인딩에서 제외된 객체입니다.
