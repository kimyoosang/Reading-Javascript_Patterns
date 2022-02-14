# Chapter3. 리터럴과 생성자

## **3.1 객체 리터럴**

- 자바스크립트에서 '객체'리고 하면 단순히 아름-값 쌍의 해시케이블을 생각하면 된다
- 원시 테이터 타입과 객체 모두 값이 될 수 있다. 함수도 값이 될 수 있으며 이런 함수는 메서드라고 부른다
- 자바스크립트에서 생성한 객체는 언제라도 변경할 수 있으며, 내장 네이티브 객체의 프로퍼티들도 대부분 변경이 가능하다
- 빈 객체를 정의해놓고 기능을 추가해나갈수도 있다
  ```javascript
  //빈 개개체에서 시작한다
  var dog = {};
  // 프로퍼티 하나를 추가한다
  dog.name = "Benji";
  //이번에는 메서드를 추가한다
  deg.getname = finction() {
    return dog.name
  }
  ```
- 객체 리터럴 표기법을 쓰면 다음 예제처럼 생성 시점에 객체에 기능을 추가할 수 있다
  ```javascript
  var dog = {
    name: "Benji",
    getName: function () {
      return this.name;
    },
  };
  ```
  **객체 리터럴 문법**

1. 객체를 중관호로 감싼다
2. 객체 내의 프로퍼티와 세머드를 쉼표로 분리한다. 마지막 이름 값 쌍 뒤에 쉬미표가 들어가면 IE에서는 에러가 발생하므로 마지막에는 사용하지 말아야 한다
3. 프로퍼티명과 프로퍼티 값은 콜론으로 분리한다
4. 개개체를 변수에 할당할 때는 닫는 중괄호 뒤에 세미콜론을 빼먹지 않도록 하라

**생성자 함수로 객체 생성하디**

- 자바스크립트에도 자바같은 클래스 기반 객체 생성과 비슷한 문법을 가지는 생성자 함수가 존재한다
- 다음 예제는 동일한 객체를 생성하는 두 가지 방법을 보여준다
  ```javascript
  //첫 번째 방법 - 리터럴 사용
  var car = {goes:v"fat"};
  //다른 방법 - 내장 생성자 사용
  //경고: 이 방법은 안티패턴이다
  var car = new Object();
  car.goes = "far";
  ```
- 객체 리터럴 표기법의 명백한 이점은 더 짧다. 리터럴 표기법을 사용하면 유효범위 판별 작업도 발생하지 않는다. 생성자 함수를 사용했다면 지역 유효범위에 동일한 이름의 생성ㅇ자가 있을 수 있기 때문에 Object()를호출한 위치에서부터 전역 Object 생성자까지 인터프리터가 쭉 거슬러 올라가며 유효범위를 검색해야 한다

**객체 생성자의 함정**

- Object()생성자는 인자를 받을 수 있다. 인자로 전달되는 값에 따라 생성자 함수가 다른 내장 생성자에 객체 생성을 위임할 수 도 있고, 따라서 기대한 것과는 다른 객체가 반환되기도 한다
- Object() 생성자의 이 같은 동작 방식 때문에, 런타임에 결정하는 동적인 값이 생성자에 인자로 전달될 경우 예기치 않은 결과가 반환될 수 있다
- 걀론적으로 new Object()가 아니라 더 간단하고 안정적인 객체 리터럴을 사용하는 것이 좋다

## **3.2 사용자 정의 생성자 함수**

- 객체 리터럴 패턴이나 생성자 함수를 쓰지 않고, 직접 생성자 함수를 만들어 객체를 생성할 수 도 있다

  ```javascript
  var Person = function (name) {
    this.name = name;
    this.say = function () {
      return "I am " + this.name;
    };
  };

  var adam = new PErson("Adam");
  adam.say()l // "I am Adam"
  ```

- new와 함께 생성자 함수를 호출하면 함수 안에서 다음과 같은 일이 일어난다
  1. 빈 객체가 생성된다. 이 객체는 this라는 변수로 참조할 수 있고, 해당 함수의 프로토타입을 상속받는다
  2. this로 참조되는 객체에 프로퍼티와 메서드가 추가된다
  3. 마지막에 다른 객체가 명시적으로 반환되지 않을 경우, this로 참조된 이 객체가 반환된다

**생성자의 반환값**

- 생성자 함수를 new와 함께 호출하면 항상 객체가 반환된다. 기본값은 this로 참조되는 객체다. 생성자 함수 내에서 아무런 프로퍼티나 메서드를 추가하지 않았다면 '빈' 객체가 반환될 것이다
- 함수 내에 return문을 쓰지 않았더라도 생성자는 암묵적으로 this를 반환한다. 그러나 반환 값이 될 객체를 따로 정할 수 도 있다

  ```javascript
  var ObjectMaker = function () {
    //생성자가 다른 객체를 대신 반환하기로 결정했기 때문에 다음의 'name'프로퍼티는 무시된다
    this.name = "This is it";

    // 새로운 객체를 생성하여 반환한다
    var that = {};
    that.name = "And this's this";
    return that;
  };

  //테스트
  var o = new Objectmaker();
  console.log(0.name); // "And that's that"
  ```

- 이와 같이 생성자에서는 어떤 객체라도 반환할 수 있다. 객체가 아닌것을 반환하려고 시도하면, 에러가 발생하진 않지만 그냥 무시되고 this에 의해 참조된 객체가 대신 반환된다

## **3.3 new를 강제하는 패턴**

- 생성자를 호출할 때 new를 빼먹으면 생성자 내부의 this가 전역 객체를 가르치게 되기 때문에 논리적인 오류가 생겨 예기치 못한 결과가 나올 수 있다
- 생성자 내부에 this.member와 같은 코드가 있을 때 이 생성자를 new 없이 호출하면, 실제로는 전역 객체에 member라는 새로운 프로퍼티가 생성된다. 이 프로퍼티는 window.member 또는 그냥 member를 통해 접근할 수 있다
- 전역 네임스페이스는 항상 깨끗하게 유지해야 하기 때문에, 이런 동작 방식은 대단히 바람직하지 않다
- ECMAScript 5에서는 이러한 동작 방식의 문제에 대한 해결책으로, 스트릭트 모드에서는 this가 전역 객체를 가리키지 않도록 했다
- ES5를 쓸 수 없는 상황이라면, 생성자 함수가 new 없이 호출되어도 항상 동일하게 동작하도록 보장하는 방법을 써야한다

  1. 명명규칙

  - 가장 간단한 대안은 명명규칙을 사용하는 것이다. 생성자 함수명의 첫글자를 대문자로 쓰고 '일반적인'함수와 메서드의 첫글자는 소문자를 사용한다

  2. that 사용

  - 명명 규칙은 올바른 동작을 권고할 뿐 강제하지는 못한다
  - this에 모든 멤버를 추가하는 대신, that에 모든 멤버를 추가한 후 that을 반환한다
  - 이 패턴의 문제는 프로토타입과의 연결고리를 잃어버리게 된다는 점이다. 즉 프로토타입에 추가한 멤버를 객체에서 사용할 수 없다

  ```javascript
  function Waffle() {
    var that = {};
    that.tastes = "yummy";
    return that;
  }
  //간단한 객체라면 that이라는 지역변수를 만들 필요도 없이 객체 리터럴을 통해 객체를 반환해도 된다
  function Waffle() {
    return {
      tastes: "yummy",
    };
  }
  ```

  3. 스스로를 호출하는 생성자

  - 앞서 설명한 패턴의 문제점을 해결하고 인스턴스 객체에서 프로토타입의 프로터피들을 사용할 수 있게 하려면, 생성자 내부에서 this가 해당 생성자의 인스턴스인지를 확인하고, 그렇지 않은 경우 new와 함께 스스로를 재호출한다

  ```javascript
  function Waffle() {
    if (!(this instanceof Waffle)) {
      return new Waffle();
    }
    this.tastes = "yummy";
  }
  Waffle.prototype.wantAnother = true;
  //호출확인
  var first = new Waffle();
  second = Waffle();

  console.log(first.tastes); // "yummy"
  console.log(second.tastes); // "yummy"

  console.log(first.wantAnother); // true
  console.log(second.wantAnother); // true
  ```

  - 인스턴스를 판별하는 또다른 범용적인 방법은 생성자 이름을 하드코딩하는 대신 arguments.callee와 비교하는 것이다
  - 이것은 모든 함수가 호출될 때, 내부적으로 arguments라는 객체가 생성되며, 이 객체가 함수에 전달된 모든 인자를 담고있다는 점을 활용한 패턴이다. arguments의 callee라는 프로퍼티는 호출된 함수를 가리킨다. arguments.callee는 ES5의 스트릭트 모드에서는 허용되지 않는다는 점에 유의하라

  ```javascript
  if (!(this instanceof arguments.callee)) {
    return new arguments.callee();
  }
  ```
