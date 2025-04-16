<!-- 학습한 내용은 여기에 정리해주세요. -->
## 실행 컨텍스트
- 실행할 코드에 제공할 환경 정보드를 모아놓은 객체
- ![image](https://github.com/user-attachments/assets/2dfffa01-7366-41ea-9437-4128deffafef)
```js
var a = 1;
function outer() {
  function inner() {
    console.log(a); // undefined
    var a = 3;
  }
  inner();
  console.log(a); // 1
}
outer();
console.log(1); // 1
```
- 자바스크립트를 실행하는 순간 전역컨텍스트가 콜스택에 담김
- 코드를 순차적으로 진행하다가 자바스크립트 엔진은 outer에 대한 환경 정보를 수집하여 outer 실행 컨텍스트를 생성한 후 콜스택에 담음
- 다시 inner함수의 실행 컨텍스트가 콜스택 가장 위에 담기면 inner함수 내부의 코드를 순서대로 진행함

### VariableEnvionment
- ![image](https://github.com/user-attachments/assets/2791e0d3-20ce-46f4-a578-be36f89b3a3e)
- 실행 컨텍스트를 생성할 때 VariableEnvionment에 정보를 먼저 담고 그대로 복사해서 LexicalEnvironment를 만듦

### LexicalEnviroment
- 자바스크립트 코드에서 변수나 함수 등의 식별자를 정의하는데 사용하는 객체
  
#### environmentRecord와 호이스팅 
- 식별자와 식별자에 바인딩된 값을 기록해두는 객체
- 현재 컨텍스트와 관련된 코드의 식별자 정보들이 저장됨(매개변수의 이름, 함수선언, 변수명 등)
- 컨텍스트 내부 전체를 처음부터 끝까지 쭉 훑어나가며 순서대로 수집함
- 호이스팅: 선언문이 마치 최상단에 끌어올려진듯한 현상
  ```js
  // 변수 호이스팅
  function a(x) { // 수집대상1 매개변수
    console.log(x); 
    var x;  // 수집대상2 (변수선언)
    console.log(x);
    var x = 2;  // 수집대상3 (변수선언)
    console.log(x);
  }
  a(1)
  ```
  ```js
  function a() {
    var x;  // 수집대상1의 변수 선언 부분
    var x;  // 수집대상2의 변수 선언 부분
    var x;  // 수집대상3의 변수 선언 부분

    x = 1;  // 수집대상 1의 할당 부분
    console.log(x);
    console.log(x);
    x = 2;  // 수집대상 3의 할당 부분
    console.log(x);
  }
  ```
  - 변수x 선언 후 확보한 공간의 주솟값에 변수x 연결
  - x에 1할당 숫자1을 별도 메모리에 담은 후 x와연결된 메모리 공간에 주솟값 입력
  - x에 2할당 숫자2를 별도 메모리에 담은 후 숫자1을 가르키는 주솟값을 2의 주솟값으로 대치
  - 마지막줄 출력 후 함수 내부의 모든 코드 실행되었으니 실행 컨텍스트 콜택에서 제거
    
- 자바스크립트 엔진이 먼저 전체 코드를 스캔하면서 변수같은 정보를 실행컨텍스트 어딘가에 기록해놓기 때문에 호이스팅이 발생함
- 이때 기록해놓는곳이 EnvironmentRecord(환경 레코드)
  ```js
  function a () {
    console.log(b);
    var b = 'bbb';  // 수집대상1 변수선언
    console.log(b);
    function b () { }  // 수집대상2 함수선언
    console.log(b);
  }
  a();
  ```
  ```js
  funtion a () {
    var b;  // 수집대상1 변수는 선언부만 끌어올림
    function b() { };   //수집대상2 함수선언은 전체를 끌어올림

    console.log(b);
    b = 'bbb'; // 변수의 할당부
    console.log(b);
    console.log(b);
  }
  a();
  ```
    - 변수 b선언, 메모리에 변수 b 주솟값 연결
    - 함수 b에 변수 b할당, 메모리에서 이제 변수b는 함수b를 가르키게 됨
    - 다시 문자열 'bbb'를 할당하고 주솟값 덮어씀
    - 함수 내부의 모든 코드가 실행됐으므로 실행컨텍스트가 콜스택에서 제거됨

#### 함수 선언문과 함수 표현식
```js
console.log(sum(1,2));
console.log(multiply(3,4));

function sum(a, b) {  // 함수 선언문
  return a + b;
}

var multiply = function (a, b) {  // 함수 표현식
  return a * b;
}
```
```js
// 호이스팅 후
var sum = function sum(a,b) { 
  return a + b;  
}
var multiply;  // 변수는 선언부만 호이스팅
console.log(sum(1,2));
console.log(multiply(3,4));

multiply = function(a,b) {
  return a * b;
}
```
- 함수선언문은 전체를 호이스팅한 반면 함수 표현식은 변수 선언부만 호이스팅
- 함수도 하나의 값으로 취급할 수 있기 때문에 이런 차이가 발생함 (함수를 다른 변수에 값으로써 '할당'한것이 곧 함수표현식)

### 스코프, 스코프 체인, outerEnvironmentReference
- 스코프 체인: 식별자의 유효범위를 안에서부터 바깥으로 차례로 검색해나가는 것
- outerEnvironmentReference는 현재 호출된 함수가 선언될 당시의 LexicalEnvironment를 참조하는데, '선언하다'라는 행위가 일어날 수 있는 시점이란 실행 컨텍스트가 활성화 상태일 때뿐이다.
- outerEnvironmentReference는 오직 자신이 선언된 시점의 LexicalEnvironment만 참조하므로 여러 스코프에서 동일한 식별자를 선언한 경우 무조건 스코프 체인 상에서 가장 먼저 발견된 식별자에만 접근 가능 -> 식별자 결정
```js
var a = 1;
var outer = function () {
  var inner = function() {
    console.log(a);
    var a = 3;
  };
  inner();
  console.log(a);
};
outer();
console.log(a);
```
![image](https://github.com/user-attachments/assets/4f0b5e9f-bbf8-462f-9afc-4f8268acced3)
- inner함수 내부에서 a에 접근하려고 하면 스코프 체인상의 첫번째 인자부터 검색한다.
- 식별자가 존재하면 스코프체인 검색을 멈춘다.
- 전역 공간에서 선언한 동일한 이름의 a 변수에는 접근할 수 없다 -> 변수 은닉화
