# 함수의 구분

## 26.1 함수의 구분

ES6 이전의 모든 함수는 일반 함수 및 생성자 함수로서 호출이 가능하다.

```javascript
var foo = function () {
  return 1;
};

// 일반적인 함수로서 호출
foo(); // -> 1

// 생성자 함수로서 호출
new foo(); // -> foo {}

// 메서드로서 호출
var obj = { foo: foo };
obj.foo(); // -> 1
```

즉, callable (호출할 수 있는 함수 객체)이면서 동시에 constructor (인스턴스를 생성할 수 있는 함수 객체) 이다.

```javascript
var foo = function () {};

// ES6 이전의 모든 함수는 callable이면서 constructor다.
foo(); // -> undefined
new foo(); // -> foo {}

// 메서드 (객체에 바인딩된 함수) 역시 callable 이며 동시에 constructor이다.
var obj = {
  x: 10,
  f: function () { return this.x; }
};

// 프로퍼티 f에 바인딩된 함수를 메서드로서 호출
console.log(obj.f()); // 10

// 프로퍼티 f에 바인딩된 함수를 일반 함수로서 호출
var bar = obj.f;
console.log(bar()); // undefined

// 프로퍼티 f에 바인딩된 함수를 생성자 함수로서 호출
console.log(new obj.f()); // f {}
```

이러한 특성은 문법적으로도 성능면에서도 문제가 있다. 생성자 함수로 호출하지 않아도 프로토타입 객체를 생성하기 때문에 실수를 유발할 수도 있고 성능면에서도 좋지 않다.

## 26.2 메서드

ES6 부터 메서드는 메서드 축약 표현으로 정의된 함수만을 의미한다.

```javascript
const obj = {
  x: 1,
  // foo는 메서드이다.
  foo() { return this.x; },
  // bar에 바인딩된 함수는 메서드가 아닌 일반 함수이다.
  bar: function() { return this.x; }
};

console.log(obj.foo()); // 1
console.log(obj.bar()); // 1
```

ES6 메서드는 인스턴스를 생성할 수 없는 non-constructor이다.

```javascript
new obj.foo(); // -> TypeError: obj.foo is not a constructor
new obj.bar(); // -> bar {}

// obj.foo는 constructor가 아닌 ES6 메서드이므로 prototype 프로퍼티가 없다.
obj.foo.hasOwnProperty('prototype'); // -> false

// obj.bar는 constructor인 일반 함수이므로 prototype 프로퍼티가 있다.
obj.bar.hasOwnProperty('prototype'); // -> true
```

표준 빌트인 객체가 제공하는 프로토타입 메서드 및 정적 메서드 역시 non-constructor이다.

```javascript
String.prototype.toUpperCase.prototype; // -> undefined
String.fromCharCode.prototype;          // -> undefined

Number.prototype.toFixed.prototype; // -> undefined
Number.isFinite.prototype;          // -> undefined

Array.prototype.map.prototype; // -> undefined
Array.from.prototype;          // -> undefined
```

ES6 메서드는 자신을 바인딩한 객체를 가리키는 `[[HomeObject]]` 내부 슬롯을 갖는다. super 참조는 이 내부 슬롯을 사용하여 수퍼 클래스의 메소드를 참조한다. 따라서 이 내부 슬롯을 갖는 ES6 메서드는 super 키워드를 사용할 수 있다.

```javascript
const base = {
  name: 'Lee',
  sayHi() {
    return `Hi! ${this.name}`;
  }
};

const derived = {
  __proto__: base,
  // sayHi는 ES6 메서드다. ES6 메서드는 [[HomeObject]]를 갖는다.
  // sayHi의 [[HomeObject]]는 sayHi가 바인딩된 객체인 derived를 가리키고
  // super는 sayHi의 [[HomeObject]]의 프로토타입인 base를 가리킨다.
  sayHi() {
    return `${super.sayHi()}. how are you doing?`;
  }
};

console.log(derived.sayHi()); // Hi! Lee. how are you doing?
```

```javascript
const derived = {
  __proto__: base,
  // sayHi는 ES6 메서드가 아니다.
  // 따라서 sayHi는 [[HomeObject]]를 갖지 않으므로 super 키워드를 사용할 수 없다.
  sayHi: function () {
    // SyntaxError: 'super' keyword unexpected here
    return `${super.sayHi()}. how are you doing?`;
  }
};
```

## 26.3 화살표 함수

function 키워드 대신 화살표를 사용하고 기존 함수보다 간단하게 동작하는 함수. 콜백 함수 내부에서 this가 전역 객체를 가리키는 문제의 대안으로 유용.

### 26.3.1 화살표 함수 정의

함수 정의

함수 표현식으로만 정의해야 한다.

```javascript
const multiply = (x, y) => x * y;
multiply(2, 3); // -> 6
```

매개변수 선언

```javascript
const arrow = (x, y) => { ... };
const arrow = x => { ... };
const arrow = () => { ... };
```

함수 몸체 정의

함수 몸체가 하나의 문으로 구성된다면 몸체를 감싸는 중괄호를 생략할 수 있다.

```javascript
// concise body
const power = x => x ** 2;
power(2); // -> 4

// 위 표현은 다음과 동일하다.
// block body
const power = x => { return x ** 2; };
```

표현식이 아닌데 중괄호를 생략하면 에러 발생.

```javascript
const arrow = () => const x = 1; // SyntaxError: Unexpected token 'const'

// 위 표현은 다음과 같이 해석된다.
const arrow = () => { return const x = 1; };
const arrow = () => { const x = 1; };
```

객체 리터럴 반환 시 소괄호로 감싸줘야 한다.

```javascript
const create = (id, content) => ({ id, content });
create(1, 'JavaScript'); // -> {id: 1, content: "JavaScript"}

// 위 표현은 다음과 동일하다.
const create = (id, content) => { return { id, content }; };

// { id, content }를 함수 몸체 내의 쉼표 연산자문으로 해석한다.
const create = (id, content) => { id, content };
create(1, 'JavaScript'); // -> undefined
```

함수 몸체가 여러 개라면 중괄호를 생략할 수 없다.

```javascript
const sum = (a, b) => {
  const result = a + b;
  return result;
};
```

즉시 실행 함수로 사용 가능.

```javascript
const person = (name => ({
  sayHi() { return `Hi? My name is ${name}.`; }
}))('Lee');

console.log(person.sayHi()); // Hi? My name is Lee.
```

고차 함수 (Array.prototype.map 등)에 인수로 전달 가능.

```javascript
// ES5
[1, 2, 3].map(function (v) {
  return v * 2;
});

// ES6
[1, 2, 3].map(v => v * 2); // -> [ 2, 4, 6 ]
```

### 26.3.2 화살표 함수와 일반 함수의 차이

화살표 함수는 인스턴스를 생성할 수 없는 non-constructor.

```javascript
const Foo = () => {};
// 화살표 함수는 생성자 함수로서 호출할 수 없다.
new Foo(); // TypeError: Foo is not a constructor

const Foo = () => {};
// 화살표 함수는 prototype 프로퍼티가 없다.
Foo.hasOwnProperty('prototype'); // -> false
```

중복된 매개변수 이름 선언 불가능.

```javascript
function normal(a, a) { return a + a; }
console.log(normal(1, 2)); // 4

'use strict';

function normal(a, a) { return a + a; }
// SyntaxError: Duplicate parameter name not allowed in this context

const arrow = (a, a) => a + a;
// SyntaxError: Duplicate parameter name not allowed in this context
```

화살표 함수는 함수 자체의 this, arguments, super, new.target 바인딩이 없다. 상위 스코프의 그것들을 따라서 참조한다.

### 26.3.3 this

일반 함수로서 호출되는 모든 함수 내부의 this는 전역 객체를 가리킨다. 동일한 조건에서 strict mode일 경우 내부의 this에는 undefined가 바인딩된다. 반면 화살표 함수는 함수 자체의 this 바인딩이 없고, 상위 스코프의 this를 그대로 참조한다. (lexical this)

```javascript
// 화살표 함수는 상위 스코프의 this를 참조한다.
() => this.x;

// 익명 함수에 상위 스코프의 this를 주입한다. 위 화살표 함수와 동일하게 동작한다.
(function () { return this.x; }).bind(this);

// 중첩 함수 foo의 상위 스코프는 즉시 실행 함수다.
// 따라서 화살표 함수 foo의 this는 상위 스코프인 즉시 실행 함수의 this

를 가리킨다.
(function () {
  const foo = () => console.log(this);
  foo();
}).call({ a: 1 }); // { a: 1 }

// bar 함수는 화살표 함수를 반환한다.
// bar 함수가 반환한 화살표 함수의 상위 스코프는 화살표 함수 bar다.
// 하지만 화살표 함수는 함수 자체의 this 바인딩을 갖지 않으므로 bar 함수가 반환한
// 화살표 함수 내부에서 참조하는 this는 화살표 함수가 아닌 즉시 실행 함수의 this를 가리킨다.
(function () {
  const bar = () => () => console.log(this);
  bar()();
}).call({ a: 1 }); // { a: 1 }
```

화살표 함수가 전역 함수라면 this는 전역 객체를 가리킨다.

```javascript
// 전역 함수 foo의 상위 스코프는 전역이므로 화살표 함수 foo의 this는 전역 객체를 가리킨다.
const foo = () => console.log(this);
foo(); // window

// increase 프로퍼티에 할당한 화살표 함수의 상위 스코프는 전역이다.
// 따라서 increase 프로퍼티에 할당한 화살표 함수의 this는 전역 객체를 가리킨다.
const counter = {
  num: 1,
  increase: () => ++this.num
};

console.log(counter.increase()); // NaN

window.x = 1;

const normal = function () { return this.x; };
const arrow = () => this.x;

console.log(normal.call({ x: 10 })); // 10
console.log(arrow.call({ x: 10 }));  // 1
```

화살표 함수의 this는 call, apply, bind로 교체할 수 없고 언제나 상위 스코프의 this를 가리킨다.

```javascript
const add = (a, b) => a + b;

console.log(add.call(null, 1, 2));    // 3
console.log(add.apply(null, [1, 2])); // 3
console.log(add.bind(null, 1, 2)());  // 3
```

메서드로 사용할 때 역시 화살표 함수의 this는 상위 스코프를 가리키므로 사용하지 않는 것이 좋다. 대신 ES6 메서드 축약 표현을 사용하는 것이 좋다.

```javascript
// Good
const person = {
  name: 'Lee',
  sayHi() {
    console.log(`Hi ${this.name}`);
  }
};

person.sayHi(); // Hi Lee

// Bad
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = () => console.log(`Hi ${this.name}`);

const person = new Person('Lee');
// 이 예제를 브라우저에서 실행하면 this.name은 빈 문자열을 갖는 window.name과 같다.
person.sayHi(); // Hi
```

프로퍼티를 동적으로 추가할 경우는 ES6 메서드 정의를 사용할 수 없으므로 일반 함수를 사용한다.

```javascript
// Good
function Person(name) {
  this.name = name;
}

Person.prototype.sayHi = function () { console.log(`Hi ${this.name}`); };

const person = new Person('Lee');
person.sayHi(); // Hi Lee
```

클래스 필드에서 화살표 함수를 할당하면 프로토타입 메서드가 아닌 인스턴스 메서드가 되므로 ES6 메서드를 사용하는 것이 좋다.

```javascript
// Good
class Person {
  // 클래스 필드 정의
  name = 'Lee';

  sayHi() { console.log(`Hi ${this.name}`); }
}
const person = new Person();
person.sayHi(); // Hi Lee
```

### 26.3.4 super

화살표 함수의 super는 this와 마찬가지로 상위 스코프의 super를 참조한다.

```javascript
class Base {
  constructor(name) {
    this.name = name;
  }

  sayHi() {
    return `Hi! ${this.name}`;
  }
}

class Derived extends Base {
  // 화살표 함수의 super는 상위 스코프인 constructor의 super를 가리킨다.
  sayHi = () => `${super.sayHi()} how are you doing?`;
}

const derived = new Derived('Lee');
console.log(derived.sayHi()); // Hi! Lee how are you doing?
```

### 26.3.5 arguments

화살표 함수의 arguments 역시 상위 스코프의 arguments를 참조한다.

```javascript
(function () {
  // 화살표 함수 foo의 arguments는 상위 스코프인 즉시 실행 함수의 arguments를 가리킨다.
  const foo = () => console.log(arguments); // [Arguments] { '0': 1, '1': 2 }
  foo(3, 4);
}(1, 2));

// 화살표 함수 foo의 arguments는 상위 스코프인 전역의 arguments를 가리킨다.
// 하지만 전역에는 arguments 객체가 존재하지 않는다. arguments 객체는 함수 내부에서만 유효하다.
const foo = () => console.log(arguments);
foo(1, 2); // ReferenceError: arguments is not defined
```

## 26.4 Rest 파라미터

### 26.4.1 기본 문법

함수에 전달된 인수들의 목록을 배열로 전달받는다.

```javascript
function foo(...rest) {
  // 매개변수 rest는 인수들의 목록을 배열로 전달받는 Rest 파라미터다.
  console.log(rest); // [ 1, 2, 3, 4, 5 ]
}

foo(1, 2, 3, 4, 5);
```

일반 매개변수와 같이 사용할 수 있다.

```javascript
function foo(param, ...rest) {
  console.log(param); // 1
  console.log(rest);  // [ 2, 3, 4, 5 ]
}

foo(1, 2, 3, 4, 5);

function bar(param1, param2, ...rest) {
  console.log(param1); // 1
  console.log(param2); // 2
  console.log(rest);   // [ 3, 4, 5 ]
}

bar(1, 2, 3, 4, 5);
```

먼저 선언된 매개변수에 할당된 인수를 제외한 나머지 인수들이 들어온다. 따라서 제일 마지막에 선언되어야 한다.

```javascript
function foo(...rest) {}
console.log(foo.length); // 0

function bar(x, ...rest) {}
console.log(bar.length); // 1

function baz(x, y, ...rest) {}
console.log(baz.length); // 2
```

### 26.4.2 Rest 파라미터와 arguments 객체

arguments 객체는 함수 호출 시 전달된 인수들의 정보가 담겨있는 유사 배열 객체. 함수 내부에서 지역 변수처럼 사용 가능.

```javascript
// 매개변수의 개수를 사전에 알 수 없는 가변 인자 함수
function sum() {
  // 가변 인자 함수는 arguments 객체를 통해 인수를 전달받는다.
  console.log(arguments);
}

sum(1, 2); // {length: 2, '0': 1, '1': 2}
```

Rest 파라미터는 가변 인자 함수의 인수 목록을 배열로 직접 전달 가능.

```javascript
function sum(...args) {
  // Rest 파라미터 args에는 배열 [1, 2, 3, 4, 5]가 할당된다.
  return args.reduce((pre, cur) => pre + cur, 0);
}
console.log(sum(1, 2, 3, 4, 5)); // 15
```

화살표 함수는 함수 자체의 arguments가 없으므로 반드시 Rest 파라미터를 사용.

## 26.5 매개변수 기본값

ES6에서 도입. 매개 변수에 기본값을 설정해서 별도로 할당되지 않아도 에러 없이 사용 가능.

```javascript
function sum(x = 0, y = 0) {
  return x + y;
}

console.log(sum(1, 2)); // 3
console.log(sum(1));    // 1
```

매개변수에 인수를 전달하지 않았을 경우와 undefined를 전달한 경우에만 유효.

```javascript
function logName(name = 'Lee') {
  console.log(name);
}

logName();          // Lee
logName(undefined); // Lee
logName(null);      // null
```

Rest 파라미터는 기본값 지정이 불가능.

```javascript
function foo(...rest = []) {
  console.log(rest);
}
// SyntaxError: Rest parameter may not have a default initializer
```

매개변수 기본값은 함수 객체의 length 프로퍼티와 arguments 객체에 아무런 영향이 없다.

```javascript
function sum(x, y = 0) {
  console.log(arguments);
}

console.log(sum.length); // 1

sum(1);    // Arguments { '0': 1 }
sum(1, 2); // Arguments { '0': 1, '1': 2 }
```

