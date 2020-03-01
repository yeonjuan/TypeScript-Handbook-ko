# 소개 (Introduction)

TypeScript는 공통 타입 변환을 용이하게 하기 위해 몇가지 유틸리티 타입을 제공합니다. 이런 유틸리티들은 전역으로 사용 가능합니다.

## 목차 (Table of contents)

* [`Partial<T>`](#partialt)
* [`Readonly<T>`](#readonlyt)
* [`Record<K,T>`](#recordkt)
* [`Pick<T,K>`](#picktk)
* [`Omit<T,K>`](#omittk)
* [`Exclude<T,U>`](#excludetu)
* [`Extract<T,U>`](#extracttu)
* [`NonNullable<T>`](#nonnullablet)
* [`Parameters<T>`](#parameterst)
* [`ConstructorParameters<T>`](#constructorparameterst)
* [`ReturnType<T>`](#returntypet)
* [`InstanceType<T>`](#instancetypet)
* [`Required<T>`](#requiredt)
* [`ThisParameterType`](#thisparametertype)
* [`OmitThisParameter`](#omitthisparameter)
* [`ThisType<T>`](#thistypet)

# `Partial<T>`

`T`의 모든 프로퍼티를 선택적으로 만드는 타입을 구성합니다. 이 유틸리티는 주어진 타입의 모든 하위 타입 집합을 나타내는 타입을 반환합니다.

##### Example

```ts
interface Todo {
    title: string;
    description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
    title: 'organize desk',
    description: 'clear clutter',
};

const todo2 = updateTodo(todo1, {
    description: 'throw out trash',
});
```

# `Readonly<T>`

`T`의 모든 프로퍼티를 `읽기 전용(readonly)`으로 설정한 타입을 구성합니다, 즉 생성된 타입의 프로티들은 재할당 될 수 없습니다.

##### Example

```ts
interface Todo {
    title: string;
}

const todo: Readonly<Todo> = {
    title: 'Delete inactive users',
};

todo.title = 'Hello'; // Error: cannot reassign a readonly property
```

이 유틸리티는 런타임에 실패할 할당 표현식을 나타낼 때 유용합니다.(예, [frozen 객체](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)의 프로퍼티에 재할당 하려고 하는 경우)

##### `Object.freeze`

```ts
function freeze<T>(obj: T): Readonly<T>;
```

# `Record<K,T>`

타입 `T`의 프로퍼티의 집합 `K`로 타입을 구성합니다. 이 유틸리티는 타입의 프로퍼티들을 다른 타입에 매칭시키는 데 사용될 수 있습니다.

##### Example

```ts
interface PageInfo {
    title: string;
}

type Page = 'home' | 'about' | 'contact';

const x: Record<Page, PageInfo> = {
    about: { title: 'about' },
    contact: { title: 'contact' },
    home: { title: 'home' },
};
```

# `Pick<T,K>`

`T`에서 프로퍼티 `K`의 집합을 선택해 타입을 구성합니다.

##### Example

```ts
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Pick<Todo, 'title' | 'completed'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

# `Omit<T,K>`

`T`에서 모든 프로퍼티를 선택한 다음 `K`를 제거한 타입을 구성합니다.

##### Example

```ts
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Omit<Todo, 'description'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

# `Exclude<T,U>`

`T` 에서 `U`에 할당 할수 있는 모든 속성을 제외한 타입을 구성합니다.

##### Example

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2 = Exclude<string | number | (() => void), Function>;  // string | number
```

# `Extract<T,U>`

`T`에서 `U`에 할당 할수 있는 모든 속성을 추출하여 타입을 구성합니다.

##### Example

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void
```

# `NonNullable<T>`

`T`에서 `null` 과 `undefined`를 제외한 타입을 구성합니다.

##### Example

```ts
type T0 = NonNullable<string | number | undefined>;  // string | number
type T1 = NonNullable<string[] | null | undefined>;  // string[]
```

# `Parameters<T>`

함수 타입 `T`의 매개변수 타입들의 튜플 타입을 구성합니다.

##### Example

```ts
declare function f1(arg: { a: number, b: string }): void
type T0 = Parameters<() => string>;  // []
type T1 = Parameters<(s: string) => void>;  // [string]
type T2 = Parameters<(<T>(arg: T) => T)>;  // [unknown]
type T4 = Parameters<typeof f1>;  // [{ a: number, b: string }]
type T5 = Parameters<any>;  // unknown[]
type T6 = Parameters<never>;  // never
type T7 = Parameters<string>;  // Error
type T8 = Parameters<Function>;  // Error
```

# `ConstructorParameters<T>`

`ConstructorParameters<T>` 타입은 생성자 함수 타입의 모든 매개변수 타입을 추출할 수 있게 해줍니다. 모든 매개변수 타입을 가지는 튜플 타입(`T`가 함수가 아닌 경우 `never`)을 생성합니다.

##### Example

```ts
type T0 = ConstructorParameters<ErrorConstructor>;  // [(string | undefined)?]
type T1 = ConstructorParameters<FunctionConstructor>;  // string[]
type T2 = ConstructorParameters<RegExpConstructor>;  // [string, (string | undefined)?]
```

# `ReturnType<T>`

함수 `T`의 반환 타입으로 구성된 타입을 만듭니다.

##### Example

```ts
declare function f1(): { a: number, b: string }
type T0 = ReturnType<() => string>;  // string
type T1 = ReturnType<(s: string) => void>;  // void
type T2 = ReturnType<(<T>() => T)>;  // {}
type T3 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T4 = ReturnType<typeof f1>;  // { a: number, b: string }
type T5 = ReturnType<any>;  // any
type T6 = ReturnType<never>;  // any
type T7 = ReturnType<string>;  // Error
type T8 = ReturnType<Function>;  // Error
```

# `InstanceType<T>`

생성자 함수 타입 `T`의 인스턴스 타입으로 구성된 타입을 만듭니다.

##### Example

```ts
class C {
    x = 0;
    y = 0;
}

type T0 = InstanceType<typeof C>;  // C
type T1 = InstanceType<any>;  // any
type T2 = InstanceType<never>;  // any
type T3 = InstanceType<string>;  // Error
type T4 = InstanceType<Function>;  // Error
```

# `Required<T>`

`T`의 모든 프로퍼티가 필수로 설정된 타입을 구성합니다.

##### Example

```ts
interface Props {
    a?: number;
    b?: string;
};

const obj: Props = { a: 5 }; // OK

const obj2: Required<Props> = { a: 5 }; // Error: property 'b' missing
```

# `ThisParameterType`

함수 타입의 `this` 매개변수의 타입, 혹은 함수 타입에 `this` 메개변수가 없을 경우 `unknown`을 추출합니다.

유의: 이 타입은 `--strictFunctionTypes`가 활성화 되었을 때만 올바르게 동작합니다. [#32964](https://github.com/microsoft/TypeScript/issues/32964)를 참고하세요.

##### Example

```ts
function toHex(this: Number) {
    return this.toString(16);
}

function numberToString(n: ThisParameterType<typeof toHex>) {
    return toHex.apply(n);
}
```

# `OmitThisParameter`

함수 타입에서 'this' 매개변수를 제거합니다.

유의: 이 타입은 `--strictFunctionTypes`가 활성화 되었을 때만 올바르게 동작합니다. [#32964](https://github.com/microsoft/TypeScript/issues/32964)를 참고하세요.

##### Example

```ts
function toHex(this: Number) {
    return this.toString(16);
}

// The return type of `bind` is already using `OmitThisParameter`, this is just for demonstration.
const fiveToHex: OmitThisParameter<typeof toHex> = toHex.bind(5);

console.log(fiveToHex());
```

# `ThisType<T>`

This utility does not return a transformed type. Instead, it serves as a marker for a contextual `this` type. Note that the `--noImplicitThis` flag must be enabled to use this utility.

##### Example

```ts
// Compile with --noImplicitThis

type ObjectDescriptor<D, M> = {
    data?: D;
    methods?: M & ThisType<D & M>;  // Type of 'this' in methods is D & M
}

function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
    let data: object = desc.data || {};
    let methods: object = desc.methods || {};
    return { ...data, ...methods } as D & M;
}

let obj = makeObject({
    data: { x: 0, y: 0 },
    methods: {
        moveBy(dx: number, dy: number) {
            this.x += dx;  // Strongly typed this
            this.y += dy;  // Strongly typed this
        }
    }
});

obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```

In the example above, the `methods` object in the argument to `makeObject` has a contextual type that includes `ThisType<D & M>` and therefore the type of `this` in methods within the `methods` object is `{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`. Notice how the type of the `methods` property simultaneously is an inference target and a source for the `this` type in methods.

The `ThisType<T>` marker interface is simply an empty interface declared in `lib.d.ts`. Beyond being recognized in the contextual type of an object literal, the interface acts like any empty interface.
