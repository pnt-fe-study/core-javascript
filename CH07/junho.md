# 07 클래스

## 01 클래스와 인스턴스의 개념 이해
![스크린샷 2025-05-20 오후 9 40 31](https://github.com/user-attachments/assets/20bcfa77-a7d9-4ffd-aadd-a60f3335d7dd)
- 음식, 과일은 모두 집단, 클래스이다.
- 음식은 과일보다 상위의(superior) 개념이고, 과일은 음식보다 하위의(subordinate) 개념이다.
  
![스크린샷 2025-05-20 오후 9 43 05](https://github.com/user-attachments/assets/b5adc5f4-b8d4-4fd6-a3a8-c67d14f4b172)
- 현실 세계에서는 이미 존재하는 개체를 다양한 기준(성별, 직업, 국적 등)에 따라 분류하기 위해 클래스를 도입하며, 하나의 개체가 여러 클래스의 인스턴스가 될 수 있다.
- 그러나 프로그래밍에서는 반대로 클래스(설계도)를 먼저 정의하고, 이를 바탕으로 인스턴스를 생성하는 방식이다. 즉, 현실은 객체 → 클래스, 프로그래밍은 클래스 → 객체 순서로 접근한다는 점에서 정반대다.

## 02 자바스크립트의 클래스
- 자바스크립트의 클래스는 내부적으로 프로토타입으로 만들어진 syntactic sugar 이다.
  
![스크린샷 2025-05-20 오후 10 25 07](https://github.com/user-attachments/assets/443f774a-ab16-4696-8f28-e34ee19018ff)
![스크린샷 2025-05-20 오후 10 29 50](https://github.com/user-attachments/assets/5e912517-ccc9-4ef0-9b35-bd7cebd7b7af)

```js
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
Rectangle.isRectangle = function(instance) {
  return (
    instance instanceof Rectangle && instance.width > 0 && instance.height > 0
  );
};

var rect1 = new Rectangle(3, 4);
console.log(rect1.getArea()); // 12 (O)
console.log(rect1.isRectangle(rect1)); // Error (X)
console.log(Rectangle.isRectangle(rect1)); // true
```
- getArea 함수는 프로토타입 메서드이다.
- isRectangle 함수는 스태틱 메서드이다.

## 03 클래스 상속
### 7-3-1 기본 구현
- 7-3절에서는 실제 적용을 위해 코드를 하나하나 분석하며 머리에 담고자 애쓰기보다는 이해를 목표로 '예전에는 이런 다양한 방식으로 고군분투해왔구나'라는 마음으로 읽어보자.

```js
var Grade = function() {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = [];
var g = new Grade(100, 80);
```
![스크린샷 2025-05-21 오후 11 19 27](https://github.com/user-attachments/assets/49a2ad1d-bf38-45cc-99e9-1da40e110b63)
```js
var Grade = function() {
  var args = Array.prototype.slice.call(arguments);
  for (var i = 0; i < args.length; i++) {
    this[i] = args[i];
  }
  this.length = args.length;
};
Grade.prototype = [];
var g = new Grade(100, 80);

g.push(90);
console.log(g); // Grade { 0: 100, 1: 80, 2: 90, length: 3}

delete g.length;
g.push(70);
console.log(g); // Grade { 0: 70, 1: 80, 2: 90, length: 1 }
```
- length 프로퍼티를 삭제하고 다시 push를 했더니, push한 값이 0번째 인덱스에 들어갔고, length가 1이 됐다.
- Grade 클래스의 인스턴스는 배열 메서드를 상속하지만 기본적으로 일반 객체의 성질을 그대로 지니므로 삭제가 가능해서 문제가 된다.
- 0번째 인덱스에 70이 들어가고 length가 1이 될 수 있었던 이유는 g.__proto__ 즉, Grade.prototype이 빈 배열을 가리키고 있기 때문이다.
- 그럼 만약 Grade.prototype에 요소를 포함하는 배열을 매칭시켰다면?
  ```js
  var Grade = function() {
    var args = Array.prototype.slice.call(arguments);
    for (var i = 0; i < args.length; i++) {
      this[i] = args[i];
    }
    this.length = args.length;
  };
  Grade.prototype = ['a', 'b', 'c', 'd'];
  var g = new Grade(100, 80);
  
  g.push(90);
  console.log(g); // Grade { 0: 100, 1: 80, 2: 90, length: 3 }
  
  delete g.length;
  g.push(70);
  console.log(g); // Grade { 0: 100, 1: 80, 2: 90, ___ 4: 70, length: 5 }
  ```
  - g.length가 없으니깐 g.__proto__.length를 찾고, 값이 4이므로 인덱스 4에 70을 넣고, 다시 g.length에 5를 부여하는 순서로 동작한다.

### 7-3-2 클래스가 구체적인 데이터를 지니지 안헥 하는 방법
첫 번째 방법은 일단 만들고 나서 프로퍼티를 일일이 지우고 더는 새로운 프로퍼티를 추가할 수 없게 하는 것이다.
![스크린샷 2025-05-22 오전 12 09 38](https://github.com/user-attachments/assets/a95bb35d-cb9e-4c72-bbb2-66e6957771a9)

두 번째는 SubClass의 prototype에 직접 SuperClass의 인스턴스를 할당하는 대신 아무런 프로퍼티를 생성하지 않는 빈 생성자 함수를 하나 더 만들어서 그 prototype이 SuperClass의 prototype을 바라보게끔 한 다음, SubClass의 prototype에는 빈 생성자 함수의 인스턴스를 할당하게 하는 것이다.
```js
var Rectangle = function (width, height) {
  this.width = width;
  this.height = height;
}
Rectangle.prototype.getArea = function () [
  return this.width * this.height;
}
var Square = function (width) {
  Rectangle.call(this, width, width);
}
var Bridge = function () {};
Bridge.prototype = Rectangle.prototype;
Square.prototype = new Bridge();
Object.freeze(Square.prototype);
```
![스크린샷 2025-05-22 오전 12 18 15](https://github.com/user-attachments/assets/47b4fc02-39c8-4a9b-97c8-e9215dcfdc98)

마지막 방법은 Object.create을 이용한 방법이다.
- 이 방법은 SubClass의 prototype의 __proto__가 SuperClass의 prototype을 바라보되, SuperClass의 인스턴스가 되지는 않으므로 앞서 소개한 두 방법보다 간단하면서 안전하다.
```js
var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var Square = function(width) {
  Rectangle.call(this, width, width);
};
Square.prototype = Object.create(Rectangle.prototype);
Object.freeze(Square.prototype);

var sq = new Square(5);
console.log(sq.getArea()); // 25
```

### 7-3-3 constructor 복구하기
- 위 세 가지 방법 모두 기본적인 상속에는 성공했지만, SubClass 인스턴스의 constructor는 여전히 SuperClass를 가리키는 상태이다.
```js
var extendClass3 = function(SuperClass, SubClass, subMethods) {
  SubClass.prototype = Object.create(SuperClass.prototype);
  SubClass.prototype.constructor = SubClass;
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};
```
- SubClass.prototype.constructor = SubClass를 통해 constructor를 복구한다.

### 7-3-4 상위 클래스에의 접근 수단 제공
- 때론 하위 클래스의 메서드에서 상위 클래스의 메서드 실행 결과를 바탕으로 추가적인 작업을 수행하고 싶을 때가 있다.
```js
var extendClass = function(SuperClass, SubClass, subMethods) {
  SubClass.prototype = Object.create(SuperClass.prototype);
  SubClass.prototype.constructor = SubClass;
  SubClass.prototype.super = function(propName) {
    // 추가된 부분 시작
    var self = this;
    if (!propName)
      return function() {
        SuperClass.apply(self, arguments);
      };
    var prop = SuperClass.prototype[propName];
    if (typeof prop !== 'function') return prop;
    return function() {
      return prop.apply(self, arguments);
    };
  }; // 추가된 부분 끝
  if (subMethods) {
    for (var method in subMethods) {
      SubClass.prototype[method] = subMethods[method];
    }
  }
  Object.freeze(SubClass.prototype);
  return SubClass;
};

var Rectangle = function(width, height) {
  this.width = width;
  this.height = height;
};
Rectangle.prototype.getArea = function() {
  return this.width * this.height;
};
var Square = extendClass(
  Rectangle,
  function(width) {
    this.super()(width, width); // super 사용 (1)
  },
  {
    getArea: function() {
      console.log('size is :', this.super('getArea')()); // super 사용 (2)
    },
  }
);
var sq = new Square(10);
sq.getArea(); // size is : 100
console.log(sq.super('getArea')()); // 100
```
- 필자가 만든 super

## 04 ES6의 클래스 및 클래스 상속
- ES6에서 본격적으로 클래스 문법이 도입됐다.
ES5와 ES6의 클래스 문법 비교
```js
var ES5 = function(name) {
  this.name = name;
};
ES5.staticMethod = function() {
  return this.name + ' staticMethod';
};
ES5.prototype.method = function() {
  return this.name + ' method';
};
var es5Instance = new ES5('es5');
console.log(ES5.staticMethod()); // es5 staticMethod
console.log(es5Instance.method()); // es5 method

var ES6 = class {
  constructor(name) {
    this.name = name;
  }
  static staticMethod() {
    return this.name + ' staticMethod';
  }
  method() {
    return this.name + ' method';
  }
};
var es6Instance = new ES6('es6');
console.log(ES6.staticMethod()); // es6 staticMethod
console.log(es6Instance.method()); // es6 method
```

ES6의 클래스 상속
```js
var Rectangle = class {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  getArea() {
    return this.width * this.height;
  }
};
var Square = class extends Rectangle {
  constructor(width) {
    super(width, width);
  }
  getArea() {
    console.log('size is :', super.getArea());
  }
};
```
