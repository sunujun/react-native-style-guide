# react-native-style-guide
이 문서는 TypeScript 코드를 이해하기 쉽고 명확하게 작성하기 위한 스타일 가이드입니다.
스타일 가이드는 일관되고 유지보수가 가능한 코드를 만들기 위한 간결한 규칙과 모범 사례를 제공합니다.
구성원들의 의사결정에 따라 수시로 변경될 수 있습니다.

## 목차
- [About Guide](#about-guide)
  - [What](#what)
  - [Why](#why)
- [TLDR](#tldr)
- [타입](#타입)
  - [타입 추론](#타입-추론)
  - [데이터 불변성](#데이터-불변성)
  - [반환 타입](#반환-타입)
  - [Discriminated Union](#discriminated-union)
  - [템플릿 리터럴 타입](#템플릿-리터럴-타입)
  - [any & unknown 타입](#any--unknown-타입)
  - [Type & Non-nullability 단언](#type--non-nullability-단언)
  - [타입 오류](#타입-오류)
  - [타입 정의](#타입-정의)
  - [배열 타입](#배열-타입)
  - [서비스](#서비스)
- [함수](#함수)
  - [일반](#일반)
  - [단일 객체 인수](#단일-객체-인수)
  - [Required & Optional 인수](#required--optional-인수)
  - [Args as Discriminated Union](#args-as-discriminated-union)
- [변수](#변수)
  - [Const 단언](#const-단언)
  - [Enum과 Const 단언](#enum과-const-단언)
  - [타입 Union과 Boolean 플래그](#타입-union과-boolean-플래그)
  - [Null과 Undefined](#null과-undefined)
- [Naming](#naming)
  - [Named Export](#named-export)
  - [Naming 컨벤션](#naming-컨벤션)
  - [주석](#주석)
- [Source Organization](#source-organization)
  - [코드 배치](#코드-배치)
  - [Import](#import)
- [부록 - React](#부록---react)
  - [Required & Optional Props](#required--optional-props)
  - [Discriminated 타입으로서의 Props](#discriminated-타입으로서의-props)
  - [Props를 State로](#props를-state로)
  - [Props Type](#props-type)
  - [데이터 저장 및 전달](#데이터-저장-및-전달)

## About Guide
### What
"일관성이 핵심"이기 때문에, TypeScript 스타일 가이드는 ESLint, TypeScript, Prettier 등과 같은 자동화된 도구를 사용하여 대부분의 규칙을 적용하려고 합니다.
그럼에도 불구하고 아래에 설명된 특정 디자인 및 아키텍처 결정은 반드시 따라야합니다.

### Why
- 프로젝트의 규모와 복잡성이 증가함에 따라 코드 품질을 유지하고 일관된 관행을 보장하는 것이 점점 더 어려워집니다.
- TypeScript 애플리케이션을 작성하는 표준 방식을 정의하고 따르면 일관된 코드베이스와 더 빠른 개발 주기를 가져옵니다.
- 코드 리뷰에서 코드 스타일을 논의할 필요가 없습니다.
- 팀의 시간과 에너지를 절약할 수 있습니다.

## TLDR
- Const 단언을 활용하세요. [⭣](#const-단언)
- 데이터 불변성을 추구하세요. [⭣](#데이터-불변성)
- Discriminated Union을 활용하세요. [⭣](#discriminated-union)
- Type 단언을 피하세요. [⭣](#type--non-nullability-단언)
- 함수가 순수하고, 상태가 없으며, 단일 책임을 가지도록 하세요. [⭣](#함수)
- 대부분의 함수 인수는 필수 항목이어야 합니다(선택적 인수는 드물게 사용하세요). [⭣](#required--optional-인수)
- Naming 컨벤션은 일관되고 읽기 쉽게 유지하는 데 중점을 둡니다. [⭣](#naming-컨벤션)
- Named Export를 사용하세요. [⭣](#named-export)
- 코드는 기능별로 구성되고 그룹화됩니다. 코드를 관련된 곳에 최대한 가깝게 배치하세요. [⭣](#코드-배치)

## 타입
타입을 생성할 때 우리는 코드를 가장 잘 설명할 수 있는 방법을 생각하려고 합니다.
표현력을 높이고 타입을 가능한 한 좁게 유지하면 코드베이스에 다음과 같은 이점이 있습니다.

- 타입 안전성 향상 - 좁혀진 타입은 데이터의 형태와 동작에 대해 더 구체적인 정보를 제공하므로 컴파일 시간에 오류를 포착할 수 있습니다.
- 코드 명확성 향상 - 데이터에 대해 더 명확한 경계와 제약 조건을 제공하여 인지 부하를 줄이고 다른 개발자가 코드를 더 쉽게 이해할 수 있게 합니다.
- 쉬운 리팩터링 - 타입이 좁기 때문에 코드를 변경하는 데 따르는 위험이 줄어들어 자신감을 가지고 리팩터링할 수 있습니다.
- 최적화된 성능 - 경우에 따라 좁은 타입은 TypeScript 컴파일러가 더 최적화된 JavaScript 코드를 생성하는 데 도움이 될 수 있습니다.

### 타입 추론
경험상, 타입을 좁히는 데 도움이 되는 경우 명시적으로 타입을 선언하는 것이 좋습니다.

<i>타입을 추가할 필요가 없다고 해서 추가하지 않아도 된다는 뜻은 아닙니다. 경우에 따라 명시적인 타입 선언이 코드 가독성과 의도를 높일 수 있습니다.</i>
``` typescript
// ❌ Avoid - 추론될 수 있는 타입을 명시적으로 선언하지 마세요.
const userRole: string = 'admin';
const employees = new Map<string, number>([['Gabriel', 32]]);
const [isActive, setIsActive] = useState<boolean>(false);

// ✅ 타입 추론을 사용하세요.
const USER_ROLE = 'admin'; // Type 'string'
const employees = new Map([['Gabriel', 32]]); // Type 'Map<string, number>'
const [isActive, setIsActive] = useState(false); // Type 'boolean'


// ❌ Avoid - (넓은) 타입을 추론하지 마세요. 좁힐 수 있습니다.
const employees = new Map(); // Type 'Map<any, any>'
employees.set('Lea', 'foo-anything');
type UserRole = 'admin' | 'guest';
const [userRole, setUserRole] = useState('admin'); // Type 'string'

// ✅ 타입을 좁히기 위해 명시적 타입 선언을 사용하세요.
const employees = new Map<string, number>(); // Type 'Map<string, number>'
employees.set('Gabriel', 32);
type UserRole = 'admin' | 'guest';
const [userRole, setUserRole] = useState<UserRole>('admin'); // Type 'UserRole'
```

### 데이터 불변성
대부분의 데이터는 `Readonly`, `ReadonlyArray`를 사용하여 변경 불가능해야 합니다.

readonly 타입을 사용하면 의도하지 않은 데이터 변형을 방지하여 의도하지 않은 부작용과 관련된 버그 발생 위험을 줄일 수 있습니다.

데이터 처리를 수행할 때 항상 새 배열, 객체 등을 반환하세요. 향후 개발자들의 인지 부하를 낮추려면 데이터 객체를 작게 유지하세요.
변형은 복잡한 객체, 성능상의 이유 등 정말 필요한 경우에만 드물게 사용해야 합니다.

``` typescript 
// ❌ Avoid - 데이터를 변형하지 마세요.
const removeFirstUser = (users: Array<User>) => {
  if (users.length === 0) {
    return users;
  }
  return users.splice(1);
};

// ✅ 실수로 인한 변형을 방지하기 위해 readonly 타입을 사용하세요.
const removeFirstUser = (users: ReadonlyArray<User>) => {
  if (users.length === 0) {
    return users;
  }
  return users.slice(1);
  // users.splice(1)를 사용하면 오류 발생 - Function 'splice' does not exist on 'users'
};
```

### 반환 타입
반환 타입 주석을 포함하는 것은 필수는 아니자만 적극 권장됩니다. ([eslint rule](https://typescript-eslint.io/rules/explicit-function-return-type/)).

함수의 반환 값에 명시적으로 타입을 지정할 때의 이점을 고려해보세요.
- 반환 값은 호출 코드에 어떤 타입이 반환되는지 명확하고 이해하기 쉽게 만듭니다.
- 반환 값이 없는 경우, 호출 코드가 undefined 값을 사용하려고 하지 않습니다.
- 향후 코드 변경으로 함수의 반환 타입이 변경될 경우 잠재적인 타입 오류를 더 빨리 발견할 수 있습니다.
- 반환 값이 올바른 타입의 변수에 할당되도록 보장하므로 리팩터링이 더 쉬워집니다.
- 구현 전에 테스트를 작성하는 것(TDD)과 유사하게, 함수 인수와 반환 타입을 정의하면 구현 전에 feature의 기능성과 인터페이스에 대해 논의할 수 있습니다.
- 타입 추론은 매우 편리하지만, 반환 타입을 추가하면 TypeScript 컴파일러의 작업량을 많이 줄일 수 있습니다.

### Discriminated Union
Discriminated Union은 복잡한 데이터 구조를 모델링하고 타입 안전성을 향상시키는 강력한 개념으로, 더 명확하고 오류가 적은 코드로 이어집니다.

Discriminated Union의 장점
- [Args as Discriminated Union](#args-as-discriminated-union)에서 언급했듯이 Discriminated Union은 선택적 인수를 제거하여 함수 API의 복잡성을 줄여줍니다. 
- 완전성 검사 - TypeScript는 타입의 모든 가능한 변형이 구현되었는지 확인할 수 있어, 런타임에 정의되지 않거나 예상치 못한 동작의 위험을 제거합니다. ([eslint rule](https://typescript-eslint.io/rules/switch-exhaustiveness-check/))
``` typescript
type Circle = { kind: 'circle'; radius: number };
type Square = { kind: 'square'; size: number };
type Triangle = { kind: 'triangle'; base: number; height: number };

// 'kind' 속성으로 객체 타입을 구분하는 'Shape' discriminated union을 생성합니다.
type Shape = Circle | Square | Triangle;

// TypeScript는 calculateArea 함수에서 오류를 경고합니다.
const calculateArea = (shape: Shape) => {
 // Error - Switch is not exhaustive. Cases not matched: "triangle"
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.size * shape.width; // Error - Property 'width' does not exist on type 'square'
  }
};
```
- 플래그 변수로 인한 코드 복잡성을 피할 수 있습니다.
- 주어진 타입에 대해 가능한 경우를 명시적으로 표시하여 읽고 이해하기 쉬워지므로 코드 의도를 명확히 할 수 있으므로, 코드 의도가 명확해집니다.
- TypeScript는 유니온 타입을 좁혀 컴파일 시간에 코드 정확성을 보장합니다.
- Discriminated Union은 관련 타입의 중앙 집중식 정의를 제공하여 리팩터링과 유지보수를 쉽게 만듭니다. 유니온 내에서 타입을 추가하거나 수정할 때 컴파일러는 코드베이스 전체에 걸쳐 일관성이 없는 부분을 보고합니다.
- IDE는 Discriminated Union을 활용하여 더 나은 자동 완성 및 타입 추론을 제공할 수 있습니다.

### 템플릿 리터럴 타입
단순히 (넓은) 문자열 타입 대신 템플릿 리터럴 타입을 사용하세요.

템플릿 리터럴 타입은 API 엔드포인트, 라우팅, 국제화, 데이터베이스 쿼리, CSS 타이핑 등 많은 사용 사례에 적용될 수 있습니다.
``` typescript
// ❌ Avoid
const userEndpoint = '/api/usersss'; // Type 'string' - Since typo 'usersss', route doesn't exist and results in runtime error
// ✅ Use
type ApiRoute = 'users' | 'posts' | 'comments';
type ApiEndpoint = `/api/${ApiRoute}`; // Type ApiEndpoint = "/api/users" | "/api/posts" | "/api/comments"
const userEndpoint: ApiEndpoint = '/api/users';

// ❌ Avoid
const homeTitle = 'translation.homesss.title'; // Type 'string' - Since typo 'homesss', translation doesn't exist and results in runtime error
// ✅ Use
type LocaleKeyPages = 'home' | 'about' | 'contact';
type TranslationKey = `translation.${LocaleKeyPages}.${string}`; // Type TranslationKey = `translation.home.${string}` | `translation.about.${string}` | `translation.contact.${string}`
const homeTitle: TranslationKey = 'translation.home.title';

// ❌ Avoid
const color = 'blue-450'; // Type 'string' - Since color 'blue-450' doesn't exist and results in runtime error
// ✅ Use
type BaseColor = 'blue' | 'red' | 'yellow' | 'gray';
type Variant = 50 | 100 | 200 | 300 | 400;
type Color = `${BaseColor}-${Variant}` | `#${string}`; // Type Color = "blue-50" | "blue-100" | "blue-200" ... | "red-50" | "red-100" ... | #${string}
const iconColor: Color = 'blue-400';
const customColor: Color = '#AD3128';
```

### any & unknown 타입
`any` 데이터 타입은 문자 그대로 TypeScript가 기본값으로 사용하는 "모든" 값을 나타내며 타입을 추론할 수 없을 때 타입 검사를 건너뛰기 때문에 사용해서는 안 됩니다. 따라서 `any`는 심각한 프로그래밍 오류를 가릴 수 있으므로 위험합니다.

모호한 데이터 타입을 다룰 때는 `any`의 타입 안전 대응물인 `unknown`을 사용하세요.
`unknown`은 모든 프로퍼티를 역참조할 수 없습니다 (`unknown`에는 무엇이든 할당할 수 있지만, `unknown`은 다른 것에 할당할 수 없습니다).
``` typescript
// ❌ Avoid - any
const foo: any = 'five';
const bar: number = foo; // no type error

// ✅ Use - unknown
const foo: unknown = 5;
const bar: number = foo; // type error - Type 'unknown' is not assignable to type 'number'

// 역참조하기 전에 다음을 사용하여 타입을 좁히세요.
// 타입 가드
const isNumber = (num: unknown): num is number => {
  return typeof num === 'number';
};
if (!isNumber(foo)) {
  throw Error(`API provided a fault value for field 'foo':${foo}. Should be a number!`);
}
const bar: number = foo;

// 타입 단언
const bar: number = foo as number;
```

### Type & Non-nullability 단언
타입 단언 `user as User`과 non-nullability 단언 `user!.name`은 안전하지 않습니다. 둘 다 TypeScript 컴파일러를 무시하고 런타임에 애플리케이션이 충돌할 위험을 증가시킵니다.

이들은 코드베이스에 도입해야 하는 강력한 이유가 있는 예외(예: 서드파티 라이브러리 타입 불일치, `unknown` 역참조 등)에만 사용할 수 있습니다 .
``` typescript
type User = { id: string; username: string; avatar: string | null };
// ❌ Avoid - type 단언
const user = { name: 'Nika' } as User;
// ❌ Avoid - non-nullability 단언
renderUserAvatar(user!.avatar); // Runtime error

const renderUserAvatar = (avatar: string) => {...}
```

### 타입 오류
TypeScript 오류를 완화할 수 없는 경우, 최후의 수단으로`@ts-expect-error`를 사용하여 오류를 억제하세요 ([eslint rule](https://typescript-eslint.io/rules/prefer-ts-expect-error/)). 향후 억제된 줄이 오류 없는 상태가 되면 TypeScript 컴파일러가 이를 표시합니다.

`@ts-ignore`는 허용되지 않으며, `@ts-expect-error`를 설명과 함께 사용해야 합니다 ([eslint rule](https://typescript-eslint.io/rules/ban-ts-comment/#allow-with-description)).

``` typescript
// ❌ Avoid - @ts-ignore
// @ts-ignore
const newUser = createUser('Gabriel');

// ✅ 설명돠 함께 @ts-expect-error를 사용하세요.
// @ts-expect-error: 라이브러리 타입 정의가 잘못되었습니다. createUser는 문자열 인수를 받습니다.
const newUser = createUser('Gabriel');
```

### 타입 정의
TypeScript는 타입 정의를 위해 `type`과 `interface` 두 가지 옵션을 제공합니다. 이들은 몇 가지 기능적 차이가 있지만 대부분의 경우 서로 교체하여 사용할 수 있습니다. 우리는 구문 차이를 제한하고 일관성을 위해 하나를 선택하려고 합니다.

모든 타입은 `type` alias로 정의해야 합니다 ([eslint rule](https://typescript-eslint.io/rules/consistent-type-definitions/#type)).

``` typescript
// ❌ Avoid - interface 정의
interface UserRole = 'admin' | 'guest'; // invalid - interface can't define (commonly used) type unions

interface UserInfo {
  name: string;
  role: 'admin' | 'guest';
}

// ✅ type 정의를 사용하세요.
type UserRole = 'admin' | 'guest';

type UserInfo = {
  name: string;
  role: UserRole;
};
```

Declaration merging의 경우 (예: 서드파티 라이브러리 타입 확장) `interface`를 사용하고 린트 규칙을 비활성화하세요.
``` typescript
// types.ts
declare namespace NodeJS {
  // eslint-disable-next-line @typescript-eslint/consistent-type-definitions
  export interface ProcessEnv {
    NODE_ENV: 'development' | 'production';
    PORT: string;
    CUSTOM_ENV_VAR: string;
  }
}

// server.ts
app.listen(process.env.PORT, () => {...}
```

### 배열 타입
배열 타입은 제네릭 구문으로 정의해야 합니다 ([eslint rule](https://typescript-eslint.io/rules/array-type/#generic)).
``` typescript
// ❌ Avoid
const x: string[] = ['foo', 'bar'];
const y: readonly string[] = ['foo', 'bar'];

// ✅ Use
const x: Array<string> = ['foo', 'bar'];
const y: ReadonlyArray<string> = ['foo', 'bar'];
```

### 서비스
문서는 작성되는 순간 오래된 것이 되며, 잘못된 문서는 문서가 없는 것보다 나쁩니다. 앱이 상호 작용하는 모듈(API, 메시징 시스템, 데이터베이스)을 설명할 때 타입에 대해서도 같은 말을 할 수 있습니다.

외부 API 서비스 (예: REST, GraphQL 등)의 경우, 계약에서 타입을 생성하는 것이 중요합니다. Swagger, 스키마 또는 기타 소스 (예: openapi-ts, graphql-config...)에서 생성하세요. 수동으로 선언하고 유지 관리하는 것을 피하세요. 동기화가 쉽게 깨질 수 있습니다.

예외적으로 외부 서비스에서 제공하는 문서가 정말로 없는 경우에만 수동으로 타입을 선언하세요.

## 함수
함수 규칙은 가능한 한 많이 따라야 합니다(일부 규칙은 함수형 프로그래밍 기본 개념에서 파생된 것입니다).
### 일반
함수는
- 단일 책임을 가져야 합니다.
- 동일한 입력 인수가 매번 동일한 값을 반환하는 stateless이어야 합니다.
- 최소한 하나의 인수를 받고 데이터를 반환해야 합니다.
- 부작용이 없어야 하며 순수해야 합니다. 구현은 로컬 환경 외부의 변수 값을 수정하거나 접근해서는 안 됩니다 (전역 상태, fetching 등).

### 단일 객체 인수
함수를 가독성 있게 유지하고 나중에 쉽게 확장(인수 추가/제거)할 수 있도록 여러 개의 인수 대신 단일 객체를 함수 인수로 사용하세요.

예외적으로 하나의 원시 단일 인수만 있는 경우 (예: 간단한 함수 isNumber(value), 커링 구현 등)에는 적용되지 않습니다.
``` typescript
// ❌ Avoid - 여러 개의 인수
transformUserInput('client', false, 60, 120, null, true, 2000);

// ✅ 옵션 객체를 인수로 사용하세요.
transformUserInput({
  method: 'client',
  isValidated: false,
  minLines: 60,
  maxLines: 120,
  defaultInput: null,
  shouldLog: true,
  timeout: 2000,
});
```

### Required & Optional 인수
**대부분의 인수를 Required로 하고 Optional 인수는 드물게 사용하세요.**

함수가 너무 복잡해지면 더 작은 조각으로 나누어야 합니다.
과장된 예로, 50개의 Optional 인수를 받는 "모든 것을 할 수 있는" 함수를 구현하는 것보다 5개의 Required 인수를 가진 10개의 함수를 구현하는 것이 더 좋습니다.

### Args as Discriminated Union

적용 가능한 경우 discriminated union 타입을 사용하여 optional 인수를 제거하세요. 이는 함수 API의 복잡성을 줄이고 사용 사례에 따라 necessary/required 인수만 전달됩니다.
``` typescript
// ❌ Avoid - optional args는 함수 API의 복잡성을 증가시킵니다.
type StatusParams = {
  data?: Products;
  title?: string;
  time?: number;
  error?: string;
};

// ✅ 대부분의 인수를 필수로 하려고 노력하세요. 그렇지 않은 경우,
// 함수 사용에 대한 명확한 의도를 위해 discriminated union을 사용하세요.
type StatusParamsSuccess = {
  status: 'success';
  data: Products;
  title: string;
};

type StatusParamsLoading = {
  status: 'loading';
  time: number;
};

type StatusParamsError = {
  status: 'error';
  error: string;
};

type StatusParams = StatusSuccess | StatusLoading | StatusError;

export const parseStatus = (params: StatusParams) => {...
```
## 변수
### Const 단언
const 단언 `as const`을 사용하세요.
- 타입이 좁혀집니다.
- 객체는 `readonly` 속성을 갖게 됩니다.
- 배열은 `readonly` tuple이 됩니다.
``` typescript
// ❌ Avoid - const 단언 없이 상수 선언
const FOO_LOCATION = { x: 50, y: 130 }; // Type { x: number; y: number; }
FOO_LOCATION.x = 10;

const BAR_LOCATION = [50, 130]; // Type number[]
BAR_LOCATION.push(10);

const RATE_LIMIT = 25;
const RATE_LIMIT_MESSAGE = `Max number of requests/min is ${RATE_LIMIT}.`; // Type string



// ✅ const 단언을 사용하세요.
const FOO_LOCATION = { x: 50, y: 130 } as const; // Type '{ readonly x: 50; readonly y: 130; }'
FOO_LOCATION.x = 10; // Error

const BAR_LOCATION = [50, 130] as const; // Type 'readonly [10, 20]'
BAR_LOCATION.push(10); // Error

const RATE_LIMIT = 25;
const RATE_LIMIT_MESSAGE = `Max number of requests/min is ${RATE_LIMIT}.` as const; // Type 'Rate limit exceeded! Max number of requests/min is 25.'
```

### Enum과 Const 단언
const 단언은 enum 대신 사용해야 합니다.
enum이 const 단언과 같은 사용 사례를 다룰 수 있지만, 우리는 이를 피하는 경향이 있습니다. TypeScript 문서에서 언급된 이유들 중 일부는 다음과 같습니다: [Const enum의 함정](https://www.typescriptlang.org/docs/handbook/enums.html#const-enum-pitfalls), [객체 vs Enum](https://www.typescriptlang.org/docs/handbook/enums.html#objects-vs-enums), [역방향 매핑](https://www.typescriptlang.org/docs/handbook/enums.html#reverse-mappings) 등...

``` typescript
// .eslintrc.js
'no-restricted-syntax': [
  'error',
  {
    selector: 'TSEnumDeclaration',
    message: '대신 const assertion이나 문자열 유니온 타입을 사용하세요.',
  },
],
```
``` typescript
// ❌ Avoid - enum 사용
enum UserRole {
  GUEST,
  MODERATOR,
  ADMINISTRATOR,
}

enum Color {
  PRIMARY = '#B33930',
  SECONDARY = '#113A5C',
  BRAND = '#9C0E7D',
}

// ✅ const 단언을 사용하세요.
const USER_ROLES = ['guest', 'moderator', 'administrator'] as const;
type UserRole = (typeof USER_ROLES)[number]; // 타입 "guest" | "moderator" | "administrator"

// UserRole 타입이 이미 정의되어 있다면 satisfies를 사용하세요 - 예: 데이터베이스 스키마 모델
type UserRoleDB = ReadonlyArray<'guest' | 'moderator' | 'administrator'>;
const AVAILABLE_ROLES = ['guest', 'moderator'] as const satisfies UserRoleDB;

const COLOR = {
  primary: '#B33930',
  secondary: '#113A5C',
  brand: '#9C0E7D',
} as const;
type Color = typeof COLOR;
type ColorKey = keyof Color; // Type "PRIMARY" | "SECONDARY" | "BRAND"
type ColorValue = Color[ColorKey]; // Type "#B33930" | "#113A5C" | "#9C0E7D"
```

### 타입 Union과 Boolean 플래그
여러 개의 Boolean 플래그 변수 대신 타입 Union 변수를 사용하세요.
플래그는 시간이 지남에 따라 누적되어 유지 관리가 어려워지는 경향이 있으며, 실제 앱 상태를 숨기게 됩니다.

``` typescript
// ❌ Avoid - 여러 개의 Boolean 플래그 변수
const isPending, isProcessing, isConfirmed, isExpired;

// ✅ 타입 Union 변수를 사용하세요.
type UserStatus = 'pending' | 'processing' | 'confirmed' | 'expired';
const userStatus: UserStatus;
```

플래그 사용으로 인해 가능한 상태의 수가 빠르게 증가하는 코드를 유지 관리할 때는 거의 항상 처리되지 않은 상태가 있으므로  [discriminated union](#discriminated-union)을 활용하세요.

### Null과 Undefined
TypeScript에서 null과 undefined 타입은 많은 경우 서로 바꿔 사용할 수 있습니다.
다음을 지향하세요:
- 값이 없음을 명시적으로 나타내기 위해 null을 사용하세요 - 할당, 함수 반환 타입 등
- 값이 존재하지 않을 때 undefined를 할당하세요. 예: form에서 필드 제외, payload 요청, 데이터베이스 쿼리([Prisma 구분](https://www.prisma.io/docs/orm/prisma-client/special-fields-and-types/null-and-undefined)) 등

## Naming
다른 사람이 여러분이 작성한 코드를 유지 관리할 수 있으므로 naming 컨벤션을 일관성 있고 가독성 있게 유지하여 중요한 컨텍스트를 제공하세요.
### Named Export
Named Export를 사용하여 모든 import가 균일한 패턴을 따르도록 해야 합니다([eslint rule](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/no-default-export.md)).
이는 전체 코드베이스에서 변수, 함수 등의 이름을 일관되게 유지합니다.
Named Export는 import 문이 선언되지 않은 것을 가져오려고 할 때 오류를 발생시키는 이점이 있습니다.

예외의 경우(예: Next.js 페이지) 규칙을 비활성화하세요.
``` typescript
// .eslintrc.js
overrides: [
  {
    files: ["src/pages/**/*"],
    rules: { "import/no-default-export": "off" },
  },
],
```
### Naming 컨벤션
최상의 이름을 찾기 어려운 경우가 많지만, 다음 컨벤션을 따라 일관성과 향후 개발자를 위해 코드를 최적화하려고 노력하세요.
#### 변수
- Local

Camel case로 작성하세요.

예: `products`, `productsFiltered`
- Boolean

`is`, `has` 등으로 접두사를 붙이세요.

예: `isDisabled`, `hasProduct`

[Eslint rule](https://typescript-eslint.io/rules/naming-convention)
``` typescript
// .eslintrc.js
'@typescript-eslint/naming-convention': [
  'error',
  {
    selector: 'variable',
    types: ['boolean'],
    format: ['PascalCase'],
    prefix: ['is', 'should', 'has', 'can', 'did', 'will'],
  },
],
```
- Constant

대문자로 작성하세요.

예: `PRODUCT_ID`
- 객체 상수

단수형, 대문자로 작성하고 const 단언을 사용하세요.
``` typescript
const ORDER_STATUS = {
  pending: 'pending',
  fulfilled: true,
  error: 'Shipping Error',
} as const;
```
객체 타입이 이미 존재하는 경우, const 단언과 함께 `satisfies` 연산자를 사용하여 객체가 해당 타입과 일치하는지 확인하세요.

```typescript
// OrderStatus 타입이 이미 정의되어 있음 - 예: 데이터베이스 스키마 모델, API 등에서 생성됨
type OrderStatus = {
  pending: 'pending' | 'idle';
  fulfilled: boolean;
  error: string;
};

const ORDER_STATUS = {
  pending: 'pending',
  fulfilled: true,
  error: 'Shipping Error',
} as const satisfies OrderStatus;
```
#### 함수
Camel case로 작성하세요

`filterProductsByType`, `formatCurrency`
#### 타입

Pascal case로 작성하세요

`OrderStatus`, `ProductItem`

[Eslint rule](https://typescript-eslint.io/rules/naming-convention)
``` typescript
// .eslintrc.js
'@typescript-eslint/naming-convention': [
  'error',
  {
    selector: 'typeAlias',
    format: ['PascalCase'],
  },
],
```
#### 제네릭
제네릭 변수는 대문자 T로 시작하고 그 뒤에 설명적인 이름이 와야 합니다. 

예: `TRequest`, `TFooBar`

복잡한 타입을 만들 때 종종 제네릭을 포함하는데, 이로 인해 읽고 이해하기 어려워질 수 있습니다. 그래서 우리는 제네릭의 이름을 지을 때 최선의 노력을 기울입니다.
`T`, `K` 등의 한 글자로 제네릭의 이름을 짓는 것은 변수를 더 많이 도입할수록 혼동하기 쉬워지기에 허용되지 않습니다.
``` typescript
// ❌ Avoid - 한 글자로 제네릭 이름 짓기
const createPair = <T, K extends string>(first: T, second: K): [T, K] => {
  return [first, second];
};
const pair = createPair(1, 'a');

// ✅ 이름은 대문자 T로 시작하고 그 뒤에 설명적인 이름이 옵니다.
const createPair = <TFirst, TSecond extends string>(first: TFirst, second: TSecond): [TFirst, TSecond] => {
  return [first, second];
};
const pair = createPair(1, 'a');
```

[Eslint rule](https://typescript-eslint.io/rules/naming-convention)
``` typescript
// .eslintrc.js
'@typescript-eslint/naming-convention': [
  'error',
  {
    // Generic type parameter must start with letter T, followed by any uppercase letter.
    selector: 'typeParameter',
    format: ['PascalCase'],
    custom: { regex: '^T[A-Z]', match: true },
  },
],
```
#### Abbreviation & Acronym
Acronym을 전체 단어로 취급하고, 첫 글자만 대문자로 사용하세요.
``` typescript
// ❌ Avoid
const FAQList = ['qa-1', 'qa-2'];
const generateUserURL(params) => {...}

// ✅ Use
const FaqList = ['qa-1', 'qa-2'];
const generateUserUrl(params) => {...}
```
가독성을 높이기 위해 널리 통용되고 필요한 경우가 아니라면 약어를 사용하지 않도록하세요.
``` typescript
// ❌ Avoid
const GetWin(params) => {...}

// ✅ Use
const GetWindow(params) => {...}
```
#### React 컴포넌트
Pascal case로 작성하세요.

예: `ProductItem`, `ProductsPage`

#### Prop 타입
React 컴포넌트 이름 뒤에 "Props" 접미사를 붙이세요.

예: `[ComponentName]Props` - `ProductItemProps`, `ProductsPageProps`
#### Callback Props
이벤트 핸들러(callback) props는 `on*` 접두사를 사용합니다. 

예: `onClick`

이벤트 핸들러 구현 함수는 `handle*` 접두사를 사용합니다. 
예: `handleClick`

([eslint rule](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/jsx-handler-names.md))
``` typescript
// ❌ Avoid - 일관성 없는 콜백 prop 명명
<Button click={actionClick} />
<MyComponent userSelectedOccurred={triggerUser} />

// ✅ prop 접두사 'on*'과 핸들러 접두사 'handle*'를 사용하세요.
<Button onClick={handleClick} />
<MyComponent onUserSelected={handleUserSelected} />
```
#### React Hooks
Camel case를 사용하고, 'use' 접두사를 붙이세요 ([eslint rule](https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks)). 

`[value, setValue] = useState()` 컨벤션을 따르세요 ([eslint rule](https://github.com/jsx-eslint/eslint-plugin-react/blob/master/docs/rules/hook-use-state.md#rule-details)).
``` typescript
// ❌ Avoid - 일관성 없는 useState hook 명명
const [userName, setUser] = useState();
const [color, updateColor] = useState();
const [isActive, setActive] = useState();

// ✅ Use
const [name, setName] = useState();
const [color, setColor] = useState();
const [isActive, setIsActive] = useState();
```
커스텀 hook은 항상 객체를 반환해야합니다.
``` typescript
// ❌ Avoid
const [products, errors] = useGetProducts();
const [fontSizes] = useTheme();

// ✅ Use
const { products, errors } = useGetProducts();
const { fontSizes } = useTheme();
```
### 주석
일반적으로 표현력 있는 코드를 작성하여 주석을 피하고 있는 그대로의 이름을 지정하세요.

코드로 표현할 수 없는 컨텍스트를 추가하거나 선택 사항을 설명해야 할 때 주석을 사용하세요(예: config 파일).

주석은 항상 완전한 문장이어야 합니다. 주석에서는 어떻게(how)와 무엇을(what)이 아니라 왜(why)를 설명하려고 노력하세요.
``` typescript
// ❌ Avoid
// 분으로 변환
const m = s * 60;
// 분당 평균 사용자 수
const myAvg = u / m;

// ✅ 표현력 있는 코드와 있는 그대로의 이름을 사용하세요.
const SECONDS_IN_MINUTE = 60;
const minutes = seconds * SECONDS_IN_MINUTE;
const averageUsersPerMinute = noOfUsers / minutes;

// ✅ 왜 그렇게 했는지 설명하는 컨텍스트를 주석으로 추가합니다.
// TODO: API 변경사항이 릴리즈되면 필터링을 백엔드로 이동해야 합니다.
// Issue/PR - https://github.com/foo/repo/pulls/55124
const filteredUsers = frontendFiltering(selectedUsers);
// 정보 손실을 최소화하기 위해 Fourier transformation을 사용합니다. - https://github.com/dntj/jsfft#usage
const frequencies = signal.FFT();
```
## Source Organization
### 코드 배치
- 모노레포의 모든 애플리케이션 또는 패키지는 **기능별**로 프로젝트 파일/폴더를 구성하고 그룹화합니다.
- **코드를 관련성이 있는 곳에 최대한 가깝게 배치하세요.**
- 깊은 폴더 중첩은 문제가 되지 않습니다.
### Import
Import 경로는 상대 경로(`./ `또는 `../`로 시작) 또는 절대 경로(`@common/utils`)일 수 있습니다.

Import 문을 더 읽기 쉽고 이해하기 쉽게 만들기 위해

- **상대** Import `./sortItems`는 같은 기능 내에서 서로 '가까운' 파일을 가져올 때 사용해야 합니다. 이는 기능을 코드베이스 내에서 이동할 때 이러한 Import 변경을 주지 않고도 가능하게 합니다.
- **절대** Import `@common/utils`는 다른 모든 경우에 사용해야 합니다.
- `모든` Import는 도구(예: [prettier-plugin-sort-imports](https://github.com/trivago/prettier-plugin-sort-imports), [eslint-plugin-import](https://github.com/import-js/eslint-plugin-import/blob/main/docs/rules/order.md))에 의해 자동 정렬되어야 합니다.
``` typescript
// ❌ Avoid
import { bar, foo } from '../../../../../../distant-folder';

// ✅ Use
import { locationApi } from '@api/locationApi';

import { foo } from '../../foo';
import { bar } from '../bar';
import { baz } from './baz';
```

## 부록 - React
React 컴포넌트와 hook도 함수이므로 해당 [함수 컨벤션](#함수)이 적용됩니다.
### Required & Optional Props
**대부분의 props를 필수로 하고 선택적 props는 신중하게 사용하세요.**

특히 first/single 사용 사례에 대한 새 컴포넌트를 만들 때 대부분의 props는 필수여야 합니다. 컴포넌트가 더 많은 사용 사례를 다루기 시작하면 선택적 props를 도입하세요.
처음부터 선택적 props를 구현해야 하는 잠재적인 예외가 있습니다(예: 여러 사용 사례를 다루는 공유 컴포넌트, UI 디자인 시스템 컴포넌트 - 버튼 `isDisabled` 등).

컴포넌트/hook이 너무 복잡해지면 더 작은 조각으로 나누어야 할 가능성이 있습니다.
각각 5개의 필수 props를 가진 10개의 React 컴포넌트를 구현하는 것이 50개의 선택적 props를 받는 "만능" 컴포넌트 하나를 구현하는 것보다 낫다는 과장된 예시가 있습니다.

### Discriminated 타입으로서의 Props
적용 가능한 경우 Discriminated 타입을 사용하여 선택적 props를 제거하세요. 이는 컴포넌트 API의 복잡성을 줄이고 사용 사례에 따라 necessary/required props만 전달됩니다.
``` typescript
// ❌ Avoid - optional props는 컴포넌트 API의 복잡성을 증가시킵니다.
type StatusProps = {
  data?: Products;
  title?: string;
  time?: number;
  error?: string;
};

// ✅ 대부분의 props를 required로 만들도록 노력하세요. 그렇지 않은 경우
// 컴포넌트 사용에 대한 명확한 의도를 위해 discriminated union을 사용하세요.
type StatusSuccess = {
  status: 'success';
  data: Products;
  title: string;
};

type StatusLoading = {
  status: 'loading';
  time: number;
};

type StatusError = {
  status: 'error';
  error: string;
};

type StatusProps = StatusSuccess | StatusLoading | StatusError;

export const Status = (status: StatusProps) => {...
```

### Props를 State로
일반적으로 props를 state로 사용하는 것을 피하세요. 컴포넌트가 prop 변경 시 업데이트되지 않기 때문입니다. 이는 추적하기 어려운 버그, 의도하지 않은 부작용, 테스트의 어려움으로 이어질 수 있습니다. prop을 초기 state로 사용해야 하는 경우가 정말로 있다면, prop 이름 앞에 `initial`을 붙여야 합니다(예: `initialProduct`, `initialSort` 등).
``` typescript
// ❌ Avoid - props를 state로 사용
type FooProps = {
  productName: string;
  userId: string;
};

export const Foo = ({ productName, userId }: FooProps) => {
  const [productName, setProductName] = useState(productName);
  ...

// ✅ 합리적인 사용 사례가 있을 때 prop 접두사 `initial`을 사용하세요.
type FooProps = {
  initialProductName: string;
  userId: string;
};

export const Foo = ({ initialProductName, userId }: FooProps) => {
  const [productName, setProductName] = useState(initialProductName);
  ...
```
### Props Type
``` typescript
// ❌ Avoid - React.FC 타입 사용
type FooProps = {
  name: string;
  score: number;
};

export const Foo: React.FC<FooProps> = ({ name, score }) => {

// ✅ 타입이 있는 props 인수를 사용하세요.
type FooProps = {
  name: string;
  score: number;
};

export const Foo = ({ name, score }: FooProps) => {...
```
### 데이터 저장 및 전달
- 전체 객체를 전달하는 대신 자식 컴포넌트에 필요한 props만 전달하세요.
- 특히 필터링, 정렬 등을 위해 URL에 상태를 저장하는 것을 활용하세요.
- URL 상태를 로컬 상태와 동기화하지 마세요.
- props를 통해 데이터를 전달하거나, URL을 사용하거나, 자식을 구성하는 방식을 고려하세요. 전역 상태(Zustand, Context)는 마지막 수단으로 사용하세요.
- `menu`, `accordion`, `navigation`, `tabs`, `list` 등과 같이 컴포넌트가 소속되어 함께 작동해야 하는 경우 React 복합 컴포넌트를 사용하세요.

항상 복합 컴포넌트를 다음과 같이 내보내세요.
``` typescript
// PriceList.tsx
const PriceListRoot = ({ children }) => <ul>{children}</ul>;
const PriceListItem = ({ title, amount }) => <li>Name: {name} - Amount: {amount}<li/>;

// ❌
export const PriceList = {
  Container: PriceListRoot,
  Item: PriceListItem,
};
// ❌
PriceList.Item = Item;
export default PriceList;

// ✅
export const PriceList = PriceListRoot as typeof PriceListRoot & {
  Item: typeof PriceListItem;
};
PriceList.Item = PriceListItem;

// App.tsx
import { PriceList } from "./PriceList";

<PriceList>
  <PriceList.Item title="Item 1" amount={8} />
  <PriceList.Item title="Item 2" amount={12} />
</PriceList>;
```
- UI 컴포넌트는 파생된 상태를 보여주고 이벤트를 보내야 하며, 그 이상은 없어야 합니다(비즈니스 로직 없음).
- 많은 프로그래밍 언어에서 함수 인자를 다음 함수로, 그리고 다음 함수로 전달할 수 있는 것처럼, React 컴포넌트도 다르지 않습니다. prop drilling이 문제가 되어서는 안 됩니다. 앱 규모가 커짐에 따라 prop drilling이 진정한 문제가 된다면, render method, 부모 컴포넌트의 로컬 상태, 컴포지션 사용 등으로 리팩터링을 시도해 보세요.
- 데이터 fetching은 컨테이너 컴포넌트에서만 허용됩니다.
- 서버 상태 라이브러리 사용을 권장합니다([react-query](https://github.com/tanstack/query), [apollo client](https://github.com/apollographql/apollo-client) 등).
- 전역 상태를 위한 클라이언트 상태 라이브러리 사용은 권장하지 않습니다. 어떤 것이 애플리케이션 전체에서 진정으로 전역적이어야 하는지 재고해 보세요. 예를 들어, `themeMode`, `Permissions`, 또는 심지어 서버 상태에 넣을 수 있는 것들(예: 사용자 설정 - /me endpoint)이 있습니다. 그래도 전역 상태가 정말 필요하다면 [Zustand](https://github.com/pmndrs/zustand)나 Context를 사용하세요.
