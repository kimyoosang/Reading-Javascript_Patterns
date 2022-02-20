# Chapter6. 코드 재사용 패턴

## **6.1 클래스 방식 VS 새로운 방식의 상속 패턴**

- 자바스크립트의 생성자 함수와 new 연산자 문법은 클래스를 사용하는 문법과 매우 닮아있다

  ```javascript
  //자바에서는 다음과 같이 쓸 수 있다
  Person adam = new Person()
  //자바스크립트에서는 이렇게 쓸 수 있다
  var adam = new Person()

  ```

- 문법상의 유사성 때문에 많은 개발자들이 자바스크립트를 클래스관점에서 생각하고 클래스를 전제한 상속 패턴을 반전시켜왔다. 이러한 구현 방법을 '클래스 방식'이라고 부를 수 있고, 클래스에 대해 생각할 필요가 없는 나머지 모든 패턴은 '새로운 방식'이라고 부를 수 있다

## **6.2 클래스 방식의 상속을 사용할 경우 예상되는 산출물**

- 클래스 방식의 상속을 구현할 때의 목표는 Child()라는 생성자 함수로 생성된 객체들이 다른 생성자 함수인 Parent()의 프로퍼티를 가지도록 하는 것이다
- Parent() 생성자와 Child() 생성자를 정의한 예제는 다음과 같다

  ```javascript
  //부모 생성자
  function Parent(name) {
    this.name = name || "Adam";
  }
  //생성자의 프로토타입에 기능을 추가한다
  Parent.prototype.say = function () {
    return this.name;
  };
  //아무 내용이 없는 자식 생성자
  function Child(name) {}

  //여기서 상속이 일어난다
  inherit(Child, Parent);
  ```

  - 여기서 상속을 처리하는 inherit함수는 언어에 내장되어 있지 않기 때문에 직접 구현해야 한다
