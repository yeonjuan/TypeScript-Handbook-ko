# 소개 (Introduction)

ECMAScript 2015부터 JavaScript에는 모듈 개념이 있습니다. TypeScript는 이 개념을 공유합니다.

모듈은 전역 범위가 아닌 그 자체 범위 내에서 실행됩니다; 즉 모듈 내에서 선언된 변수, 함수, 클래스 등은 [`export` 양식]() 중 하나를 사용하여 명시적으로 내보내지 않는 한 모듈 외부에서 보이지 않습니다. 반대로 다른 모듈에서 내보낸 변수, 함수, 클래스, 인터페이스 등을 사용하기 위해서는 [`import` 양식]() 중 하나를 사용하여 가져와야 합니다.

모듈은 선언형입니다; 모듈 간의 관계는 파일 수준에서 imports와 exports 관점에서 지정됩니다.

모듈은 모듈 로더를 사용하여 다른 모듈을 가져옵니다. 모듈 로더는 런타임 시에 모듈을 실행하기 전에 모듈의 모든 의존성을 찾고 실행해야 합니다. JavaScript에서 사용하는 유명한 모듈 로더로는 [CommonJS](https://en.wikipedia.org/wiki/CommonJS) 모듈 용 Node.js의 로더와 웹 애플리케이션의 [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md) 모듈 용 [RequireJS](https://requirejs.org/) 로더가 있습니다.

ECMAScript 2015와 마찬가지로 TypeScript는 최상위 수준의 `import` 혹은 `export`가 포함된 모든 파일을 모듈로 간주합니다. 반대로 최상위 수준의  `import` 혹은 `export` 선언이 없는 파일은 전역 범위에서 사용할 수 있는 스크립트로 처리됩니다(모듈도 마찬가지).

`작업중...`