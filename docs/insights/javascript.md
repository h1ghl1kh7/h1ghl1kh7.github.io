---
layout: default
title: Javascript
nav_order: 1
parent: Insights
last_modified_date: 2024-04-04
---
---

# background

## 자바 스크립트 엔진

자바스크립트 엔진은 코드를 실행하기 전에 Lexical Environment 내에 있는 메모리에 함수, 변수 선언을 추가한다.
1. 함수와 변수 선언 스캔
2. Lexical Environment라고 불리는 자바스크립트 데이터 구조 내 메모리에 추가
3. 코드 실행

### 실행 컨텍스트 (Execution context)

- 실행 코드에 제공할 정보들을 모아 놓은 객체
- **생성 단계**
	- 실행 컨텍스트 생성
	- 선언문만 실행해서 Environment Record에 기록 (실질적인 값 할당은 아닌 듯 하다/선언문의 실행과 값 할당은 다르다.)
	1. 환경 레코드에 식별자 정보 저장
	2. 외부 렉시컬 환경 참조 -> 스코프 체인 형성
	3. this에 바인딩 될 값 결정
- **실행 단계**
	- 선언문 외 나머지 코드 순차적 실행
	-  Environment Record에 미리 기록된 정보 참조 or 업데이트
- call stack에 쌓이며 전역 컨텍스트는 자동으로 먼저 생성되고, 함수 호출 시 함수 컨텍스트가 생성된다.
	![](../../../assets/images/Pasted image 20240406194934.png)
	이렇게 생성된다고 한다.
	실제 사용되는 정보는 렉시컬 환경, 환경 레코드와 아우터이다.
	- 렉시컬 환경 (어휘적 환경, Lexical Environment)
		- identifier-variable 매핑 정보를 가지고 있는 데이터 구조이다.
		- 환경 레코드 : 식별 지정자 저장
			- 호이스팅 : 인터프리터가 코드를 실행하기 전에 함수, 변수, 클래스 또는 import 선언문을 해당 범위(스코프)의 맨 위로 끌어롤리는 것처럼 보이는 현상 + 인터프리터가 변수와 함수의 메모리 공간을 선언 전에 미리 할당하는 것
				1. 자바 스크립트 엔진이 전역 실행 컨텍스트 생성 -> 콜 스택에 삽입
				2. 전체 코드 스캔, 선언
				3. Environment Record에 새로운 식별자 (변수) 기록
				4. 값 초기화 (var의 선언이 undefined로 초기화 되는 반면, let/cost의 선언은 uninitialized로 남아있다고 한다 -> 초기화 되지 않음 -> 참조 불가)
				   *Temporal Dead Zone(일시적 사각지대)* : `let/const`로 선언했을 때, 선언 이전에 식별자를 참조할 수 없는 구역

			- 자바스크립트는 오직 선언만을 hoist한다. 초기화(할당)을 hoist 하는건 아니다. (아마 추상적으로 이렇게 있다 만 해놓고 나머지는 나중에 코드 실행할 때 하는 듯) -> 함수를 위에서 먼저 실행할 때 사용됨 -> 미리 있다 라는 것만 저장? 하는 느낌?
			- 함수 선언식은 hoisting 되지만, 함수 표현식은 호이스팅 되지 않는다. -> (위의 자바스크립트는 선언만 hoist하기 때문. 다만, 함수 선언식의 경우, 여러 선언식이 같은 이름으로 존재할 경우, 가장 마지막으로 선언된 함수를 따와서 실행하기 때문에, 주의해야 한다.
			```javascript
			console.log(aa)
			var aa=2
			```
			- 위와 같은  코드가 있을 때는, aa가 hoist되었기에, 출력의 결과는 undefined로 나온다.
		- Outer Environment Reference : 실행 컨텍스트를 잇는 연결다리
			- 스코프 체인 [Scope](#scope)
			- 바깥 Lexical Environment를 가리킨다.

```js
실행 컨텍스트 (Execution Context) : {
  동적인 환경 (Variable Environment) : {…},
  어휘적 환경 (Lexica lEnviroment) : {
    환경 레코드(Environment Record): {
      객체적 환경 레코드(Object Environment Record),
      선언적 환경 레코드(Declarative Environment Record): {
        함수 환경 레코드(Function Environment Record),
        모듈 환경 레코드(Module Environment Record)
      },
      디스 바인딩(

`this` binding)
    },
    외부 환경에 대한 참조(Reference to the outer environment)
  }
}
```
위처럼 이해하면 될 것 같다.
- 동적 환경 : 변수(var) 바인딩 저장
- 어휘적 환경 : 함수 선언 및 변수 (let, const) 바인딩 저장
	- 환경 레코드 : Lexical 환경 안에 변수와 함수의 선언 저장 / 함수 컨텍스트의 경우 인수를 포함하여 저장
		- 객체적 환경 레코드 : 변수와 함수 선언과는 별도로 전역 바인딩 객체 (브라우저의 경우 window 객체) 저장.
		  따라서, 바인딩 오브젝트의 각 속성에 대해 레코드에 새로운 엔트리가 생성
		- 선언적 환경 레코드 : 변수와 함수 선언 저장
			- 함수 환경 레코드 : 함수의 최상위 스코프를 나타내는데 사용되는 선언적 환경 레코드.
			  화살표 함수가 아니라면 `this` 바인딩을 제공하며 화살표 함수가 아니고 super를 참조하는 경우 super 메서드를 실행하는데 필요한 state 제공 [[Javascript#함수]]
			  - 모듈 환경 레코드 : Module의 외부 스코프를 나타낼 때 사용하는 선언적 환경 레코드, 변경이 가능함, 변경 불가능한 바인딩과 더불어 변경 불가능한 import 바인딩 (immutable import binding) 제공
		  - this 바인딩 : this의 값이 결정되거나 설정
	- 외부 환경에 대한 참조 : 스코프 체인 결정

![](../../../assets/images/Pasted image 20240406175613.png)

### 콜스택
- 실행 컨텍스트를 저장하는 자료구조
- 원시 타입의 데이터 저장됨
- 실행 컨텍스트를 통해 변수 식별자(이름) 저장, 스코프 체인 및 `this` 관리, 코드 실행 순서 관리 등을 수행
- 원시 타입
	- 주소 / 값 형태로 이루어짐
	- 변수에 데이터 할당 시, 변수는 해당 데이터를 가진 주소를 가리키게 된다.
	- 데이터 타입이 원시 타입이 아닌경우, 원시 타입의 값은 그 데이터를 가진 메모리 힙에 주소를 가리키게 됨
	- 변수 식별자 자체는 콜스택 상의 실행컨텍스트의 렉시컬 환경이라는 곳에 저장된다.
- 변수에 할당된 값을 바꾸면, 실제 메모리의 값을 바꾸는 것이 아닌, 다른 변수에 값이 할당되어 있는지를 확인하고 해당 주소를 참조한다. (변수의 주소가 같아질 수도 있다.)
- 만약 없으면, 메모리 공간을 새로 생성한 후, 그 주소가 변수의 주소로 할당될 수도 있다.
- 아무 변수도 참조하고 있지 않은 원시타입이 존재할 경우, 가비지 컬렉터가 알아서 제거한다.
- const 타입은 위와 같은 방식으로 동작이 불가능하다. (초기값 할당이 필수이기 때문)

### 메모리 힙
- 참조타입(객체 등) 데이터가 저장되며 구조적이지 않다.
- 배열과 같은 참조 타입 데이터가 메모리 힙에 저장된다.
- 메모리 힙의 주소값이 콜 스택에 저장된다.
![](../../../assets/images/Pasted image 20240407164252.png)
- 배열의 값이 바뀌면 메모리 힙에 저장된 배열의 값이 바뀌는 것이다.

### Scope
scope라는 함수의 내부에서는 global variable의 값을 참조할 수 있다.
- 함수 레벨 스코프
	- 함수 안에 있으면 참조 가능 (var)
- 블록 레벨 스코프
	- `{ }` 안에 있어야 참조 가능 (let, const는 블록 레벨 스코프 ES6)
- 스코프는 필요한 영역에 한정하여 유효 범위가 좁을 수록 좋다.
스코프 체인
- 자신이 속해있는 지역의 변수들을 참조할 수 있게 되며, 해당 코드 레벨에 참조값이 없다면 상위 레벨의 스코프로 참조값을 찾아나가는 현상
- 변수 은닉화 (Variable Shadowing)
	- 스코프 체인에 따라 밖으로 슉슉 찾아가는 스코프 체인 과정 중, 동일한 변수가 현재 스코프에 존재함과 동시에, 상위 스코프에도 존재할 경우, 현재 스코프에 존재하는 값을 가져오기 때문에, 상위 스코프의 식별자 값은 가려진다.
	- 위 현상을 변수 은닉화라고 한다.
정적 / 렉시컬 스코프
- 함수를 호출한 곳이 아닌 선언한 곳을 기준으로 스코프를 결정하는 원칙

```js
let greet = 'Hello';

function sayHi() {
	let greet = 'Hi';
  	print();
}

function print() {
	console.log(greet);
}

sayHi(); // 예상: "Hi" | 출력값: "Hello"
print(); // 예상: "Hello" | 출력값: "Hello"
```
위 처럼, print 함수를 main / sayHi에서 각각 호출하고 있지만, 최종적인 결과값은 둘 다 전역 변수의 값으로 나온다.
- module level scope
	모듈 내부에서 정의한 변수나 함수는 다른 스크립트에서 접근할 수 없음
## 함수
> 바인딩 : 실별자와 값 연결
> 변수 선언 -> 변수 이름 + 메모리 공간의 주소 바인딩 (할당은 아닌 듯 하다)
> 객체 안에 선언된 함수 : 메소드 (전역에서 선언된 일반 함수도 결국 전역 객체의 메소드 -> 모든 함수는 객체 내부에 있음)

- `this`
	- features
		- javascript 예약어
		- 자신이 속한 객체 또는 자신이 생성할 인스턴스를 가리키는 자기 참조 변수 (self-reference variable)
		- 자신이 속한 객체 또는 자신이 생성할 인스턴스의 프로퍼티나 메서드를 참조할 수 있음
		- `this`는 js 엔진에 의해 암묵적으로 생성된다.
		- `this`는 코드 어디서는 참조할 수 있다.
		- 자기 참조 변수이므로, 객체의 메서드 내부 또는 생성자 함수 내부에서만 의미가 있다.
		- 함수를 호출하면, 인자와 `this`가 암묵적으로 함수 내부로 전달된다.
		- 함수 내부에서 인자를 지역 변수처럼 사용할 수 있는 것처럼, `this`도 지역 변수처럼 사용할 수 있다.
		- `this` 바인딩은 함수 호출 방식에 의해 동적으로 결정된다.
		- 크게 전역에서의 사용 / 함수 안에서의 사용으로 나뉠 수 있다.
	- 브라우저라는 자바스크립트 런타임의 경우, `this`는 항상 window라는 전역 객체를 참조
	- 전역 객체 : 전역 범위에 항상 존재하는 객체를 의미함 (Node.js에서 전역 객체는 global)
	- 브라우저라는 자바스크립트 런타임에서 모든 변수, 함수는 window라는 객체의 프로퍼티와 메소드이다.
	- `this`를 함수 내부에서 사용한 경우
		- 현재 함수를 실행하고 있는 그 객체 참조
		- 함수 내부의 `this` 값은 함수를 호출하는 방법에 따라 바뀜
		- 엄격 모드 여부에 따라 참조 값이 달라짐 (엄격 모드에서 일반 함수 내부의 `this`는 `undefined`가 바인딩 됨)
- super
	- 자식 클래스 내에서 부모 클래스의 생성자 역할
	- 자식 클래스에서 부모 클래스의 메소드 접근 역할
	- `super.`을 이용하여 부모 클래스의 메서드에 접근할 수 있다.
```js
class Score{
	constructor(math, english){
		`this`.math = math;
		`this`.english = english;
	}
	sum() {
		return `this`.math + `this`.english;
	}
	avg() {
		const sum=`this`.sum();
		console.log("평균 : "+sum/2)
	}
}
class Student extends Score {
	constructor(math, english, science){
		super(math, english);
		`this`.science = science;
	}
	avg(){
		const sum = super.sum() + `this`.science;
		console.log("평균 : "+sum/3)
	}
}
```
- 화살표 함수
	- 함수 표현식에 대한 **간결한** 대안 (안되는게 있음)
	- 익명 함수로만 사용할 수 있다.
	- 약간의 의미적 차이와 의도적인 사용상의 제한을 가지고 있다.
	- 자체 바인딩이 this에 없다.
	- 인수 또는 super로 사용행 한다.
	- 메서드로 사용하면 안된다.
	- 생성자로 사용할 수 없다.
	- new로 호출하면 TypeError가 반환된다.
	- new.target 키워드에 대한 엑세스 권한도 없다.
	- 화살표 함수는 함수 내부에서 yield를 사용할 수 없으며 제너레이터 함수로 생성할 수 없습니다.
	- `this`가 동적으로 바인딩 되지 않고, 선언할 때 바인딩할 객체가 정적으로 결정된다. (화살표 함수의 this는 언제나 상위 스코프의 `this`를 가리킨다. Lexical this)
- anonymous function (익명함수)
	- 사용자가 따로 함수를 만들 때 이름을 지정하지 않고 변수 혹은 그냥 호출만으로 선언할 수 있는 함수
	- 런타임에 동적으로 선언된다.
	```js
	let func = function (){
		let v2=1;
	}
	// func라는 변수에 할당되긴 했지만, func는 단순히 변수 이름일 뿐 함수의 이름이 아니다.
```
- named function (기명함수)
	- 이름이 붙여진 함수
	- 런타임 이전에 선언된다.
## 클래스 / 객체 / 인스턴스
- 클래스
	- 설계도
	- 어떠한 변수와 메소드를 가지는지 적어놓은 것
- 객체
	- 클래스로 구현할 어떤 것
	- 구현하고 싶은 것
- 인스턴스
	- 객체의 실체화
	- 객체의 생성자를 통해 인스턴스라는 것으로 실체화 시켜야 함
	- 객체의 복사된 내용 (e.g. new 연산자)
모든 객체는 Object의 인스턴스이다.
즉, Object는 객체 클래스 중 최상위 클래스이다.
Object는 기본적인 객체를 생성하기 위한 내장 생성자 함수이다. (모든 객체는 Object의 프로퍼티와 메서드를 상속함)
```js
obj instance of Class
// obj가 Class에 속하는지 아닌지 판단하는 연산자이다.
```
이 연산자는 오른쪽 피연산자로 생성자 함수를 사용한다. 하지만, 사용자가 직접 정의한 생성자 함수와 내장 생성자 함수 둘다 오른쪽 피연산자가 될 수 있으므로, 구분하기 어려울 수 있다.
## Promise
## 모듈
파일이 여러개 분리되어 있을 때 분리된 파일 각각을 모듈이라고 부름
자바스크립트 커뮤니티에서 특별한 라이브러리르 만들어 필요한 모듈을 언제든지 불러올 수 있게 해준다거나 코드를 모듈 단위로 구성해주는 방법을 만드는 등 다양한 시도름 함
- AMD - 오래된 모듈 시스템 중 하나 / require.js라는 라이브러리를 통해 처음 개발됨
- CommonJS - Node.js 서버를 위해 만들어진 모듈 시스템
- UMD - AMD와 CommonJS 간의 다양한 모듈 시스템을 함께 사용하기 위해 만들어짐
스크립트 하나는 모듈 하나
`export`, `import`를 적용하면 다른 모듈을 불러와 불러온 모듈에 있는 함수를 호출하는 것과 같은 기능 공유를 할 수 있음
- `export` 지시자 : 변수나 함수 앞에 붙이면 외부 모듈에서 해당 변수나 함수에 접근할 수 있다.
- `import` 지시자 : 외부 모듈의 기능을 가져올 수 있다.

항상 엄격 모드로 실행된다.

모듈 레벨 스코프 [Scope](#scope)

## 비동기 처리
> Web API : 웹 브라우저에서 제공하는 API로 AJAX나 Timeout등의 비동기 작업을 실행
> Task Queue : Callback Queue라고도 하며 Web API에서 넘겨받은 Callback함수를 저장(선입선출 방식)
> Event Loop : Call Stack이 비어있다면 Task Queue의 작업을 Call Stack으로 옮김

JS는 싱글 스레드 언어여서, 동기적 요청을 통해 코드를 하나하나 처리한다.
이러한 특성 때문에, 콜스택에 실행 컨텍스트가 남아있는 동안 브라우저는 아무것도 할 수 없다.
이러한 문제를 비동기 처리로 해결 가능함
- e.g.
	- A 함수가 있다.
	- A 함수는 생성 단계를 거쳐 실행 단계를 지나면, 다른 실행컨텍스트와 같이 콜스택에서 제거된다.
	- 이 때, A 함수 내부에서 다른 API에게 콜백 함수와 함께 타이머를 보낸다.
	- 타이머가 다 되면, 해당 API는 A함수의 콜백함수를 테스크 큐에 밀어넣는다.
	- 테스크큐는 콜스택이 비어있는 것을 보고 테스크 큐에 콜백 함수를 콜스택에 전달한다.
	- 콜스택은 함수를 실행한다.
## 변수
- var
	- var 선언은 전역 범위 혹은 함수 범위로 지정됨
	- 호이스팅 [실행 컨텍스트 (Execution context)](#실행%20컨텍스트%20(Execution%20context))
- let
	- 해당 블록 내에서만 사용이 가능하다. (중괄호)
	- let은 업데이트 될 수 있다.
	- 다만, 다시 선언할 수는 없다.
	```js
	let test=1;
	let test=2; // Uncaught SyntaxError: Identifier 'test' has already been declared
    ```
- const
	- 선언된 블록 범위 내에서만 접근 가능함
	- 업데이트, 재선언 둘 다 불가능하다.
	- 개체의 경우는 조금 다른데, 개체 자체는 업데이트 할 수 없지만, 개체의 속성은 업데이트 할 수 있다.
	- 콜 스택 주소값의 변경을 허락하지 않는다는 뜻

## Prototype

간단하게 설명하자면

```javascript
function Foo(x){
	this.x = x;
}
var a = new Foo("test");

a.__proto__ == Foo.prototype
> True
```

이걸로 설명이 될 것 같다.

prototype은 그냥 원본 그자체의 의미를 가지고 있고 `__proto__`는 상위 객체의 prototype property를 참조하고, 상위 객체는 prototype property를 가지고 있어서 다른 하위 객체가 참조하려고 할 때 자신의 복사본인 prototype을 준다.

### Prototype Chaining

객체가 자기 자신의 property 뿐만이 아니라, 자신의 부모 역할을 하는 prototype 객체의 property도 자신의 property처럼 사용할 수 있게 하는 것이다.

특정 객체의 property에 접근하는 경우, 만약 자기 자신에게 해당 property가 존재하지 않는다면, prototype 링크로 연결된 prototype 객체에서 해당 property를 검색하게 된다.


## Reference

[https://velog.io/@y_jem/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8](https://velog.io/@y_jem/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EC%8B%A4%ED%96%89-%EC%BB%A8%ED%85%8D%EC%8A%A4%ED%8A%B8)
<br>
[https://curryyou.tistory.com/276](https://curryyou.tistory.com/276)
<br>
기타 블로그들..

# tricks

## Node vm
node:vm 모듈은 security mechanism이 아니라고 한다. -> untrusted code를 쓰지 말아라..
주로 다른 context에서 코드를 실행하기 위해 사용한다고 한다.
simple usage

```js
const vm = require('vm')
const context = { x : 2 }

const code = 'var y = 3; x += y;'

vm.runInContext(code, context)

console.log(context.x); // 5
console.log(context.y) // 3
```
### DoS attack
nodejs는 싱글 스레드이고 Event Loop에 의존하기 때문에 event loop를 heavy/endless한 작업들로 막으면, 서비스를 멈출 수 있다.
```js
const vm = require('vm');

const code = 'while(true){}';

vm.runInNewContext(code,{});

console.log('Never gets executed.')
```
infinite loop가 event loop를 main process안에서 막기 때문에, 멈춘다.
새로운 V8 Virtual Machine context에서 동작하는 것은 맞지만, 같은 process와 같은 event loop를 사용하기 떄문에 멈춘다..
### Escaping the Sandbox
VM 모듈은 새롭게 실행될 코드의 context를 original code와 분리한다.
반? 격리된 context에서 실행된다고 봐도 될 것 같다.
하지만, 쉽게 탈출된다.
`this.constructor.constructor`이런 느낌으로 간단하다

```js
> const vm = require("vm");

> code = "var x = this.y"
> let context = {y:1}
> vm.runInNewContext(code,context)
> console.log(context.x)

> code = "var x = this.constructor"
> vm.runInNewContext(code,context);
> console.log(context.x);
[Function: Object]

> code = "var x = this.constructor.constructor"
> vm.runInNewContext(code,context);
> console.log(context.x);
[Function: Function]

> code = "var x = this.constructor.constructor()"
> vm.runInNewContext(code,context);
> console.log(context.x);
[Function: anonymous]
undefined
```

- `this`는 context를 참조한다.
- `this.constructor`는 context의 constructor를 참조한다. (native code를 참조한다. 아마 )
- `this.constructor.constructor` 는 다른 native code function 이다.
- `this.constructor.constructor()` anonymous wrapper를 가졌다.


## 화살표 함수

```javascript
() => expression;
(param1, paramN) => {
    statements
};
```

## Filtering

```javascript
`chi${""}ld_p${""}rocess`

require("child_process").execSync("ls -al").toString();
require("child_process")['exe'+'cSync']("ls -al").toString();
```
