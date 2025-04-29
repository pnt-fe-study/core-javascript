### 콜백 함수란?
call 호출하다, back 뒤돌아오다 = 되돌아 호출해달라

즉, 함수 X를 호출하면서 특정 조건일 때 함수 Y를 실행하서 나에게 알려달라는 요청을 함께 보내는 것
-> 함수 또는 메서드에게 인자로 넘겨줌으로써 제어권도 함께 위임한 함수

### 제어권
```js
var count = 0;
var cbFunc = setInterval(function() {
  console.log(count);
  if(++count > 4) clearInterval(timer);
}, 300);

var timer = setInterval(cbFunc, 300);
// 0.3초마다 각 0, 1, 2, 3, 4 호출
```
- setInterval을 호출할 때 두개의 매개변수 전달(익명함수, 300)
- cbFunc 함수의 호출주체와 제어권은 모두 사용자에게 있음
- setInterval(cbFunc, 300) 함수의 호출주체와 제어권은 모두 setInterval에게 있음
- 콜백 함수의 제어권을 넘겨받은 코드는 콜백 함수 호출 시점에 대한 제어권을 가짐

#### 인자
```js
var newArr = [10, 20, 30].map(function (currentValue, index) {
  console.log(currentValue, index);
  return currentValue + 5;
});
console.log(newArr);

// 결과
// 10 0 , 20 1, 30 2, [15, 25, 35]
```
- map 메서드의 첫 번째 매개변수로 익명 함수를 전달

```js
// map 메서드의 구조
Array.prototype.map(callback[, thisArg])
callback: function(currentValue, index, array)
```
- 첫 번째 인자로 callback 함수를 받고 생략 가능한 두 번째 인자로 콜백함수 내부에서 this로 인식할 대상 특정 가능
- thisArg를 생략할 경우 전역객체가 바인딩됨
- 메서드의 대상이 되는 배열의 모든 요소를 하나씩 꺼내어 콜백함수를 반복 호출하고 모아서 새로운 배열을 만듦
- 콜백 함수의 인자에는 각 현재값, 현재값의 인덱스, 대상이 되는 배열 자체가 담김
- 즉, 순서에 의해서만 각각을 구분하고 인식함

#### this
제어권을 넘겨받은 코드가 콜백을 call/apply로 실행할 때, this가 원하는 객체로 명시적으로 바뀌게 됨
```js
setTimeout(function() { console.log(this); }, 300);  // 1) window {...}

[1, 2, 3, 4, 50].forEach(funtion (x) {
  console.log(this);  // 2) window {...} 
});

document.body.innerHTML += '<button id='a'>button</button>'
document.body.querySelector('#a')
  .addEventListener('click', function(e) {
    console.log(this, e);
    // 3) <button id='a'>button</button>, MouseEvent {isTrusted: true, ...}
  }
);
```
  1) call 메서드의 첫 번째 인자에 전역객체를 넘기기 때문에 콜백 함수 내부에서의 this가 전역객체를 가리킴
  2) 별도의 인자로 this를 넘겨주지 않았기 때문에 전역객체를 가리킴
  3) call 메서드의 첫번째 인자에 addEventListener메서드의 this를 그대로 넘기도록 정의되어있으므로 콜백함수 내부에서의 this가 addEventListener를 호출한 주체인 HTML엘리먼트를 가리킴


### 콜백 함수는 함수다
콜백 함수로 어떤 객체의 메서드를 전달하더라도 그 메서드는 메서드가 아닌 함수로서 호출됨
메서드? - 객체의 프로퍼티(속성) 중 값이 함수인 것을 의미
```js
var obj = {
  vals: [1, 2, 3];
  logValues: function(v, i) {
    console.log(this, v, i);
  }
};
obj.logValues(1, 2);  // {vals: [1, 2, 3], logValues: f} 1 2
[4, 5, 6].forEach(obj.logValues);  // window {...} 4 0, window {...} 5 1, window {...} 6 2
}
```
- obj객체의 logValue는 메서드로 정의됨
- this는 obj를 가르키고 인자로 넘어온 1, 2 출력
- forEach는 콜백함수를 호출할 때 this를 지정해주지 않음
- 콜백 함수는 일반 함수처럼 호출되기 때문에 logValues안의 this는 obj가 아님
