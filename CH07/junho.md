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
