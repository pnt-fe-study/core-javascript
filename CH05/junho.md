# 05 클로저

## 01 클로저의 의미 및 원리 이해
- 클로저는 여러 함수형 프로그래밍 언어에서 등장하는 보편적인 특성이다.
- 가비지 컬렉터는 어떤 값을 참조하는 변수가 하나라도 있다면 그 값은 수집 대상에 포함시키지 않는다.
- **`클로저란 어떤 함수 A에서 선언한 변수 a를 참조하는 내부함수 b를 외부로 전달할 경우 A의 실행 컨텍스트가 종료된 이후에도 변수 a가 사라지지 않는 현상을 말한다.`**
```ts
var outer = function () {
  var a = 1;
  var inner = function () {
    return ++a;
  };
  return inner();
};
var outer2 = outer();
console.log(outer2); // 2
console.log(outer2); // 2
```

```ts
var outer = function() {
  var a = 1;
  var inner = function() {
    return ++a;
  };
  return inner;
};
var outer2 = outer();
console.log(outer2()); // 2 
console.log(outer2()); // 3
```

- return 없이도 클로저가 발생하는 경우
    ```ts
    // (1) setInterval/setTimeout
    (function() {
      var a = 0;
      var intervalId = null;
      var inner = function() {
        if (++a >= 10) {
          clearInterval(intervalId);
        }
        console.log(a);
      };
      intervalId = setInterval(inner, 1000);
    })();
    ```

    ```ts
    // (2) eventListener
    (function() {
      var count = 0;
      var button = document.createElement('button');
      button.innerText = 'click';
      button.addEventListener('click', function() {
        console.log(++count, 'times clicked');
      });
      document.body.appendChild(button);
    })();
    ```
    
## 02 클로저와 메모리 관리
- 메모리 누수라는 표현은 개발자의 의도와 달리 어떤 값의 참조 카운트가 0이 되지 않아 가비지 컬렉터의 수거 대상이 되지 않는 경우를 칭한다.
- 참조 카운트를 0으로 만드는 방법은 식별자에 참조형이 아닌 기본형 데이터(보통 null 또는 undefined)를 할당하면 된다.
  ```ts
  // (1) return에 의한 클로저의 메모리 해제
  var outer = (function() {
    var a = 1;
    var inner = function() {
      return ++a;
    };
    return inner;
  })();
  console.log(outer());
  console.log(outer());
  outer = null; // outer 식별자의 inner 함수 참조를 끊음
  ```

  ```ts
  // (2) setInterval에 의한 클로저의 메모리 해제
  (function() {
    var a = 0;
    var intervalId = null;
    var inner = function() {
      if (++a >= 10) {
        clearInterval(intervalId);
        inner = null; // inner 식별자의 함수 참조를 끊음
      }
      console.log(a);
    };
    intervalId = setInterval(inner, 1000);
  })();
  ```

  ```ts
  // (3) eventListener에 의한 클로저의 메모리 해제
  (function() {
    var count = 0;
    var button = document.createElement('button');
    button.innerText = 'click';
  
    var clickHandler = function() {
      console.log(++count, 'times clicked');
      if (count >= 10) {
        button.removeEventListener('click', clickHandler);
        clickHandler = null; // clickHandler 식별자의 함수 참조를 끊음
      }
    };
    button.addEventListener('click', clickHandler);
    document.body.appendChild(button);
  })();
  ```

## 03 클로저 활용 사례
### 5-3-1 콜백 함수 내부에서 외부 데이터를 사용하고자 할 때
- 콜백 함수를 내부함수로 선언해서 외부변수를 직접 참조하는 방법
  ```ts
  var fruits = ['apple', 'banana', 'peach'];
  var $ul = document.createElement('ul'); // (공통 코드)
  
  fruits.forEach(function(fruit) {
    // (A)
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', function() {
      // (B)
      alert('your choice is ' + fruit);
    });
    $ul.appendChild($li);
  });
  document.body.appendChild($ul);
  ```
- bind 메서드를 활용하는 방법
  ```ts
  var fruits = ['apple', 'banana', 'peach'];
  var $ul = document.createElement('ul');
  
  var alertFruit = function(fruit) {
    alert('your choice is ' + fruit);
  };
  fruits.forEach(function(fruit) {
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', alertFruit.bind(null, fruit));
    $ul.appendChild($li);
  });
  document.body.appendChild($ul);
  ```
  - 인자로 넘어오는 순서가 바뀌는 점과 함수 내부에서의 this가 원래의 그것과 달라지는 점을 감안해야 한다.
- 콜백 함수를 고차함수로 바꿔서 클로저를 적극적으로 활용한 방법
  ```ts
  var fruits = ['apple', 'banana', 'peach'];
  var $ul = document.createElement('ul');
  
  var alertFruitBuilder = function(fruit) {
    return function() {
      alert('your choice is ' + fruit);
    };
  };
  fruits.forEach(function(fruit) {
    var $li = document.createElement('li');
    $li.innerText = fruit;
    $li.addEventListener('click', alertFruitBuilder(fruit));
    $ul.appendChild($li);
  });
  document.body.appendChild($ul);
  ```

### 5-3-2 접근 권한 제어(정보 은닉)
- 정보 은닉은 어떤 모듈의 내부 로직에 대해 외부로의 노출을 최소화해서 모듈 간의 결합도를 낮추고 유연성을 높이고자 하는 현대 프로그래밍 언어의 중요한 개념 중 하나이다.
- 접근 권한에는 public, private, protected의 세 종류가 있다.
- 클로저를 이용하면 함수 차원에서 public한 값과 private한 값을 구분할 수 있다.
- return한 변수들은 공객 멤버가 되고, 그렇지 않은 변수들은 비공개 멤버가 된다.
  ```ts
  // 간단한 자동차 객체
  var car = {
    fuel: Math.ceil(Math.random() * 10 + 10), // 연료(L)
    power: Math.ceil(Math.random() * 3 + 2), // 연비(km/L)
    moved: 0, // 총 이동거리
    run: function() {
      var km = Math.ceil(Math.random() * 6);
      var wasteFuel = km / this.power;
      if (this.fuel < wasteFuel) {
        console.log('이동불가');
        return;
      }
      this.fuel -= wasteFuel;
      this.moved += km;
      console.log(km + 'km 이동 (총 ' + this.moved + 'km)');
    },
  };

  car.fuel = 10000;
  car.power = 100;
  car.moved = 1000;
  ```
  - car.fuel, car.power 변수에 직접 접근이 가능하다.
 
  ```ts
  // 간단한 자동차 함수
  var createCar = function() {
    var fuel = Math.ceil(Math.random() * 10 + 10); // 연료(L)
    var power = Math.ceil(Math.random() * 3 + 2); // 연비(km / L)
    var moved = 0; // 총 이동거리
    return {
      get moved() {
        return moved;
      },
      run: function() {
        var km = Math.ceil(Math.random() * 6);
        var wasteFuel = km / power;
        if (fuel < wasteFuel) {
          console.log('이동불가');
          return;
        }
        fuel -= wasteFuel;
        moved += km;
        console.log(km + 'km 이동 (총 ' + moved + 'km). 남은 연료: ' + fuel);
      },
    };
  };
  var car = createCar();
  ```

### 5-3-3 부분 적용 함수
- 부분 적용 함수란 n개의 인자를 받는 함수에 미리 m개의 인자만 넘겨 기억시켰다가, 나중에 (n-m)개의 인자를 넘기면 비로소 원래 함수의 실행 결과를 얻을 수 있게끔 하는 함수이다.
- bind 메서드를 활용한 부분 적용 함수
  ```ts
  var add = function() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
      result += arguments[i];
    }
    return result;
  };
  var addPartial = add.bind(null, 1, 2, 3, 4, 5);
  console.log(addPartial(6, 7, 8, 9, 10)); // 55
  ```
- this에 관여하지 않는 별도의 부분 적용 함수
  ```ts
  var partial = function() {
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    if (typeof func !== 'function') {
      throw new Error('첫 번째 인자가 함수가 아닙니다.');
    }
    return function() {
      var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
      var restArgs = Array.prototype.slice.call(arguments);
      return func.apply(this, partialArgs.concat(restArgs));
    };
  };
  
  var add = function() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
      result += arguments[i];
    }
    return result;
  };
  var addPartial = partial(add, 1, 2, 3, 4, 5);
  console.log(addPartial(6, 7, 8, 9, 10)); // 55
  
  var dog = {
    name: '강아지',
    greet: partial(function(prefix, suffix) {
      return prefix + this.name + suffix;
    }, '왈왈, '),
  };
  dog.greet('입니다!'); // 왈왈, 강아지입니다.
  ```

  ```ts
  Object.defineProperty(window, '_', {
    value: 'EMPTY_SPACE',
    writable: false,
    configurable: false,
    enumerable: false,
  });
  
  var partial2 = function() {
    var originalPartialArgs = arguments;
    var func = originalPartialArgs[0];
    if (typeof func !== 'function') {
      throw new Error('첫 번째 인자가 함수가 아닙니다.');
    }
    return function() {
      var partialArgs = Array.prototype.slice.call(originalPartialArgs, 1);
      var restArgs = Array.prototype.slice.call(arguments);
      for (var i = 0; i < partialArgs.length; i++) {
        if (partialArgs[i] === _) {
          partialArgs[i] = restArgs.shift();
        }
      }
      return func.apply(this, partialArgs.concat(restArgs));
    };
  };
  
  var add = function() {
    var result = 0;
    for (var i = 0; i < arguments.length; i++) {
      result += arguments[i];
    }
    return result;
  };
  var addPartial = partial2(add, 1, 2, _, 4, 5, _, _, 8, 9);
  console.log(addPartial(3, 6, 7, 10)); // 55
  
  var dog = {
    name: '강아지',
    greet: partial2(function(prefix, suffix) {
      return prefix + this.name + suffix;
    }, '왈왈, '),
  };
  dog.greet(' 배고파요!'); // 왈왈, 강아지 배고파요!
  ```
- 실무에서 사용하는 디바운스
  ```ts
  var debounce = function(eventName, func, wait) {
    var timeoutId = null;
    return function(event) {
      var self = this;
      console.log(eventName, 'event 발생');
      clearTimeout(timeoutId);
      timeoutId = setTimeout(func.bind(self, event), wait);
    };
  };
  
  var moveHandler = function(e) {
    console.log('move event 처리');
  };
  var wheelHandler = function(e) {
    console.log('wheel event 처리');
  };
  document.body.addEventListener('mousemove', debounce('move', moveHandler, 500));
  document.body.addEventListener(
    'mousewheel',
    debounce('wheel', wheelHandler, 700)
  );
  ```

### 5-3-4 커링 함수
- 커링 함수란 여러 개의 인자를 받는 함수를 하나의 인자만 받는 함수로 나눠서 순차적으로 호출될 수 있게 체인 형태로 구성한 것을 말한다.
- 커링은 한 번에 하나의 인자만 전달하는 것을 원칙으로 한다.
- 중간 과정상의 함수를 실행한 결과는 그다음 인자를 받기 위해 대기만 할 뿐, 마지막 인자가 전달되기 전까지는 원본 함수가 실행되지 않는다.
- 당장 필요한 정보만 받아서 전달하고 또 필요한 정보가 들어오면 전달하는 식으로 하면 결국 마지막 인자가 넘어갈 때까지 함수 실행을 미루는 셈이 된다. 이를 함수형 프로그래밍에서는 지연실행이라고 칭한다.
```ts
var curry3 = function(func) {
  return function(a) {
    return function(b) {
      return func(a, b);
    };
  };
};

var getMaxWith10 = curry3(Math.max)(10);
console.log(getMaxWith10(8)); // 10
console.log(getMaxWith10(25)); // 25

var getMinWith10 = curry3(Math.min)(10);
console.log(getMinWith10(8)); // 8
console.log(getMinWith10(25)); // 10
```
