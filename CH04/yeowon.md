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
- 즉, 어떤 함수의 인자에 객체의 메서드를 전달하더라도 이는 결국 메서드가 아닌 함수이다.

### 콜백 함수 내부의 this에 다른 값 바인딩하기
콜백 함수 내부에서 this가 객체를 바라보게 하려면? 
1) 별도의 인자로 this를 받는 함수의 경우 여기에 원하는 값을 넘겨준다.
2) 그렇지 않은 경우엔 this의 제어권도 넘겨주게 되므로 사용자가 임의로 값을 바꿀 수 없다.
   1) 콜백 함수 내부의 this에 다른 값을 바인딩하는 방법 - 전통적인 방법
      ```js
      var obj = {
        name: 'obj1',
        func: function() {
          var self = this;
          return function() {
            console.log(self.name);
          }
        }
      };

      var callback = obj1.func();
      setTimeout(callback, 1000);
      ```
      - 실제로 this를 사용하지도 않을뿐더러 번거롭다. 차라리 this를 아예 안쓰는 편이 나은듯?
   3) 콜백 함수 내부에서 this를 사용하지 않은 경우
     ```js
     var obj1 = {
       name: 'obj1',
       func: function() {
         console.log(obj1.name);
       }
     };

     setTimeout(obj1.func, 1000);
     ```
     - 훨씬 간결하지만 작성한 함수를 this를 이용해 다양한 상황에 재활용 할 수 없게 되어버렸다.
   5) 콜백 함수 내부의 this에 다른 값을 바인딩하는 방법 - bind 메서드 활용
      ```js
      var obj1 = {
        name: 'obj1',
        func: function() {
          console.log(this.name);
        }
      };
      setTimeout(obj1.func.bind(obj1), 1000);

      var obj2 = { name: 'obj2' };
      setTimeout(obj1.func.bind(obj2), 1500);
      ```
      - bind: 함수를 복사해서 새로운 함수로 만들되, this를 특정 객체에 고정시키는 메서드
      - 첫번째 setTimeout에서 obj1.func은 원래 this가 obj1을 바라보는 메서드
      - bind메서드를 사용하여 this를 obj1에 고정시킴
      - 따라서 1초 후 this.name인 obj1을 출력함

### 콜백 지옥과 비동기 제어
콜백지옥은 함수를 익명 함수로 전달하는 과정이 반복되어 코드의 들여쓰기 수준이 감당하기 힘들 정도로 깊어지는 현상
- 동기적 코드 : CPU계산에 의해 즉시 처리가능한 대부분의 코드
- 비동기적 코드 : 별도의 요청, 실행 대기, 보류 등과 관련된 코드

```js
setTimeout(function (name) {
  var coffeeList = name;
  console.log(coffeeList);

  setTimeout(function (name) {
    coffeeList += ', ' + name;
    console.log(coffeeList);

    setTimeout(function (name) {
      coffeeList += ', ' + name;
      console.log(coffeeList);

      setTimeout(function (name) {
        coffeeList += ', ' + name;
        console.log(coffeeList);
      }, 500, '카페라떼');
    }, 500, '카페모카');
  }, 500, '아메리카노');
}, 500, '에스프레소');
```
- 가독성 문제와 아래에서 위로 향하고 있어 어색하게 느껴짐을 해결하는 가장 간단한 방법은 익명의 콜백 함수를 모두 기명함수로 전환하는 것
- 그러나 일회성 함수를 전부 변수에 할당함으로써 헷갈릴 소지가 있다.
- 비동기 작업의 동기적 표현
  1) Promise
     ```js
      new Promise(function (resolve) {
        setTimeout(function () {
          var name = '에스프레소';
          console.log(name);
          resolve(name);
        }, 500);
      }).then(function (prevName) {
        return new Promise(function (resolve) {
          setTimeout(function () {
            var name = prevName + ', 아메리카노';
            console.log(name);
            resolve(name);
          }, 500);
        });
      }).
        then(function (prevName) {
          return new Promise(function (resolve) {
            setTimeout(function () {
              var name = prevName + ', 카페모카';
              console.log(name);
              resolve(name);
            }, 500);
          });
        }).then(function (prevName) {
          return new Promise(function (resolve) {
            setTimeout(function () {
              var name = prevName + ', 카페라데';
              console.log(name);
              resolve(name);
            }, 500);
          });
        });
     ```
     - Promise : 비동기 작업을 캡슐화 한 객체
     - resolve() : 작업 성공 시 다음 then()에 값 전달
     - then(): 이전 resolve()값을 받아서 처리
  3) Generator
     ```js
      var addCoffee = function (prveName, name) {
        setTimeout(function () {
          coffeeMaker.next(prevName ? prevName + ', ' + name : name);
        }, 500);
      };
      var coffeeGenerator = function* () {
        var espresso = yield addCoffee('', '에스프레소');
        console.log(espresso);
        var americano = yield addCoffee(espresso, '아메리카노');
        console.log(americano);
        var mocha = yield addCoffee(americano, '카페모카');
        console.log(mocha);
        var latte = yield addCoffee(mocha, '카페라떼');
        console.log(latte);
      };
      var coffeeMaker = coffeeGenerator()
      coffeeMaker.next()
     ```
     - Generator 함수 실행 시 next메서드를 호출함
     - Generator가 실행되면서 yield 함수의 실행을 멈춤
     - 비동기 작업이 완료되는 시점마다 next메서드를 호출한다면 함수 내부의 소스가 순차적으로 진행됨 
  5) async/await
     ```js
       var addCoffee = function (name) {
        return new Promise(function (resolve) {
          setTimeout(function () {
            resolve(name);
          }, 500);
        });
      };
      var coffeeMaker = async function () {
        var coffeeList = '';
        var _addCoffee = async function (name) {
          coffeeList += (coffeeList ? ',' : '') + (await addCoffee(name));
        };
        await _addCoffee('에스프레소');
        console.log(coffeeList);
        await _addCoffee('아메리카노');
        console.log(coffeeList);
        await _addCoffee('카페모카');
        console.log(coffeeList);
        await _addCoffee('카페라떼');
        console.log(coffeeList);
      };
      coffeeMaker();
     ```
  - coffeeMaker()함수 실행 -> 비동기 작업이 필요한 함수 앞에 async 표기 후 실질적인 비동기 작업이 필요한 위치마다 await표기
  - coffeeList라는 문자열 변수 생성
  - _addCoffee함수에서 addCoffee(name)을 호출하고 0.5초 기다린 후 coffeeList에 이름 추가
 
### 최종 정리
 - 콜백함수는 다른 코드에 인자로 넘겨줌으로써 그 제어권도 함께 위임한 함수
 - 제어권을 넘겨받은 코드는 다음과 같은 제어권을 가짐
   - 콜백 함수를 호출하는 시점을 스스로 판단해서 실행
   - 콜백 함수를 호출할 때 인자로 넘겨줄 값들 및 그 순서가 정해져있음
   - 콜백 함수의 this가 무엇을 바라보도록 할지가 정해져 있는 경우도 있음, 정해져 있지 않은 경우 전역객체를 바라보고 임의로 this를 바꾸고 싶을 경우 bind메서드 사용
 - 어떤 함수에 인자로 메서드를 전달하더라도 이는 결국 함수로서 실행됨
 - 콜백지옥에서 벗어날 수 있는 방법 - Promise, Generator, async/await 등
