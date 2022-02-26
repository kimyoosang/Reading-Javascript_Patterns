# Chapter7. 디자인 패턴

## **7.1 싱글톤 (Singleton)**

- 싱글톤 패턴은 특정 클래스의 인스턴스를 오직 하나만 유지한다
- 동일한 클래스를 사용하여 새로운 객체를 생성하면, 두 번재부터는 처음 만들어진 객체를 얻게 된다
- 자바스크립트에서는 클래스가 없고 오직 객체만 있다. 새로운 객체를 만들면 실제로 이 객체는 다른 어떤 객체와도 같지 않기 때문에 이미 싱글톤이다. 객체 리터럴로 만든 단순한 객체또한 싱글톤의 예다

  ```javascript
  var obj = {
    myprop: "my value",
  };
  ```

- 자바스크립트에서 객체들은 동일한 개개체가 아니고서는 절대로 같을 수 없다. 완전히 같은 멤버를 가지는 똑같은 객체를 만들더라도, 이전에 만들어진 객체와 동일하지는 않다
- 따라서 객체 리터럴을 이용해 객체를 생성할 때마다 사실은 싱글톤을 만드는 것이고, 싱글톤을 만들기 위한 별도의 문법이 존재하지 않는다고 할 수 있다

**new사용하기**

- 자바스크립트에는 클래스가 없다. 따라서 말 그대로의 싱글톤이라는 정의는 엄밀히 말하면 이치에 맞지 않다. 그러나 자바스크립트에는 생성자 함수를 사용해 객체를 만드는 new구문이 있고, 때로는 이 구문을 사용해서 싱글톤을 구현하고자 할 수도있다
- 동일한 생성자로 new를 사용하여 여러개의 객체를 만들 경우, 실제로는 동일한 객체에 대한 새로운 포인터만 반환하도록 구현하는 것이다

  ```javascript
  var uni = new Universe();
  var uni2 = new Universe();
  uni === uni2; //true
  ```

- 자바스크립트에서는 객체의 인스턴스인 this가 생성되면 Universe 생성자가 이를 캐시한 후, 그 다음번에 생성자가 호출되었을 때 개시된 인스턴스를 반환하게 하면된다
- 여기에는 몇가지 선택사항이 있다
  1. 인슿턴스를 저장하기 위해 전역 변수를 사용한다. 일반적인 원칙상 전역변수 선언은 좋지 않기 때문에 이 방법은 추천하지 않는다. 뿐만 아니라, 누군가 실수로 이 전역 변수를 덮어 쓸 수도 있다. 따라서 이 선택 사항은 더 이상 거론하지 않겠다
  2. 생성자의 스태틱 프로퍼티에 인스턴스를 저장한다. 자바스크립트에서 함수는 객체이므로, 프로퍼티를 가질 수 있다. Universe.instance와 같은 프로퍼티에 인스턴스를 저장할 수 있다. 깔끔하고 괜찮은 방법이지만 instance 프로퍼티가 공개되어 있기 때문에, 외부 코드에서 값을 변경하면 인스턴스를 잃어버릴 수 있다는 한 가지 단점이 있다
  3. 인스턴스를 클로저로 감싼다. 이 방법은 인스턴스를 비공개로 만들어 생서앚 외부에서 수정할 수 없게 해준다. 추가적인 클로저가 필요한 단점이 있다

**스태틱 프로퍼티에 인스턴스 저장하기**

- Universee생성자의 스태틱 프로퍼티 내부에 단일 인스턴스를 저장하는 예제

  ```javascript
  function Universe() {
    //이미 instance가 존재하는가?
    if (typeof Universe.instance === "object") {
      return Universe.instance;
    }
    //정상적으로 진행한다
    this.start_time = 0;
    this.bang = "Big";

    //인스턴스를 캐시한다
    Universe.instance = this;

    //함축적인 반환
    return this;
  }
  ```

  - instance가 공개되어 있다는게 유일한 단점이다

**클로저에 인스턴스 저장하기**

- 클로저를 사용해 단일 인스턴스를 보호하는 방법이 있다

  ```javascript
  function Universe() {
    //캐싱된 인스턴스
    var instance = this;

    //정상적으로 진행한다
    this.start_time = 0;
    this.bang = "Big";

    //생성자를 재작성한다
    Universe = function () {
      return instance;
    };
  }

  //테스트
  var uni = new Universe();
  var uni2 = new Universe();
  uni === uni2; //true
  ```

- 원본 생서자가 최초로 호출되면 일반적인 방식대로 this를 반환한다. 두 번째, 세번째 혹은 그 이상 호출되면 재작성된 생성자가 실행된다. 재작성된 생성자는 클로저를 통해 비공개 instance 변수에 접근하여 이 인스턴스를 반환하기만 한다
- 단점은 재정의 시점 이전에 원본 생성자에 추가된 프로퍼티를 잃어버린다는 점이다. Universe() 프로토타입에 무언가를 추가해도 원본 생성자로 생성된 인스턴스와 연결되지 않는다
- 몇 가지 테스트로 이 문제를 확인해보자

  ```javascript
  //프로토타입에 추가한다
  Universe.prototype.nothing = true

  var uni = new Unerverse()

  //첫 번째 객체가 만들어진 이후
  //다시 프로토타입에 추갛나다
  Universe.prototype.everything = true

  ver uni2 = new Universe()

  //테스트

  uni.nothing //true
  uni2.nothing //true
  uni.everything //undefined
  uni2.everything //undefined

  //이 구문은 문제가 없어보인다
  uni.constructor.name //'Universe"

  //그러나 이상하다
  uni.constructor === Universe //false

  ```

  - uni.constructor가 더이상 Universe() 생성자와 같지 않은 이유는 uni.constructor가 재정의된 생서자가 아닌 원본 생성자를 가리키고 있기 때문이다
  - 프로토타입가 생성자 포인터가 제대로 동작해야 하는 것이 요구사항이라면, 몇가지 간단한 수정으로 이를 만족시킬 수 있다

  ```javascript
  function universe() {
    //캐싱된 인스턴스
    var instance = this;

    //생성자를 재작성한다
    Universe = function () {
      return instance;
    };

    //prototype 프로퍼티를 변경한다
    Universe.prototype = this;

    //instance
    instance = new Universe();

    //생성자 포인터를 재작성한다
    instance.start_time = 0;
    instance.bang = "Big";

    return instance;
  }
  ```

  - 또 다른 대안으로 생성자와 인스턴스를 즉시 실행 함수로 감싸는 방법이 있다. 생성자가 최초로 호출되면 생성자는 객체를 생성하고 비공개 instance를 가리킨다. 두 번째 호출부터는 단순히 비공개 변수를 반환한다

  ```javascript
  var Universe;

  (function () {
    var instance;

    Universe = function Universe() {
      if (instance) {
        return instance;
      }
      instance = this;

      //일반적으로 진행한다
      instance.start_time = 0;
      instance.bang = "Big";
    };
  })();
  ```

## **7.2 팩터리 (Factory)**

- 팩터리 패턴의 목적은 객체들을 생성하는 것이다. 팩터리 패턴은 흔히 클래스 내부에서 또는 클래스의 스태틱 메서드로 구현되며, 다음과 같은 목적으로 사용된다
  1. 비슷한 객체를 생성하는 반복작업을 수행한다
  2. 팩터리 패턴의 사용자가 컴파일 타임에 구체적인 타입(클래스)을 모르고도 객체를 생성할 수 있게 해준다
- 두 번째 목적은 정적 클래스 언어에서 특히 중요하다. 이런 언어에서는 클래스에 대한 정ㅇ보 없이 인스턴스를 생성하기 쉽지않다. 자바스크립트에서는 이를 구현하기가 상당히 쉽다
- 팩터리 메서드(또는 클래스)로 만들어진 객체들은 의도적으로 동일한 부모 객체를 상속하낟. 즉, 이들은 특화된 기능을 구현하는 구체적인 서브 클래스들이다. 어떤 경우에는 공통의 부모 클래스가 팩터리 메서드를 갖고있기도 하다
- 다음 항목들을 포함하는 구현 예제를 살펴보자

  1. CarMaker 생성자: 공통의 부모
  2. CarMaker.factory(): car 객체들을 생성하는 스태틱 메서드
  3. CarMaker.Compact, CarMaker.Suv, CarMaker.Convertible: CarMaker를 상속하는 특화된 생성자. 이 모두는 부모의 스태틱 프로퍼티로 정의되어 전역 네임스페이스를 깨끗하게 유지하며, 필요할 때 쉽게 찾을 수 있다

  ```javascript
  var carolla = CarMaker.factory("Compact");
  var solstice = CarMaker.factory("Convertible");
  var cherokee = CarMaker.factory("SUV");
  corolla.drive(); //"Vroom, I have 4 doors"
  solstice.drive(); //"Vroom, I have 2 doors"
  cherokee.drive(); //"Vroom, I have 17 doors"
  ```

- 이 부분을 눈여겨보자

  ```javascript
  var carolla = CarMaker.factory("Compact");
  ```

- 이 부분이 아마도 팩터리 패턴에서 가장 특징적인 부분일 것이다.이 메서드는 런타임시 문자열로 타입을 받아 해당 타입의 객체를 생성하고 반환한다. new와 함께 생성자를 사용하지 않고, 객체 리터럴도 보이지 않는다. 문잦열로 식별되는 타입에 기반하여 객체들을 생성하는 함수가 있을 뿐이다
- 다음은 이 코드가 동작하게 만드는 팩터리 패턴 구현 예제다

  ```javascript
  //부모 생성자
  function CarMaker() {}

  //부모의 메서드
  CarMaker.prototype.drive = function () {
    return "Vroom, I have " + this.doors + "doors";
  };

  //스태틱 factory 메서드
  CarMaker.factory = function (type) {
    var constr = type,
      newcar;

    //생성자가 존재하지 않으면 에러를 발생한다
    if (typeof CarMaker[constr] !== "function") {
      throw {
        name: "Error",
        message: constr + "doesn't exist",
      };
    }

    //생성자의 존재를 호가인했으므로 부모를 상속한다
    //상속은 단 한번만 실행하도록 한다
    if (typeof CarMaker[constr].prototype.drive !== "function") {
      CarMaker[constr].prototype = new CarMaker();
    }

    //새로운 인스턴스를 생성한다
    newCar = new CarMaker[constr]();

    //다른 메서들 호출이 필요하면 여기서 실행한 후, 인스턴스를 반환한다
    return newcar;
  };

  //구체적인 자동차 메이커들을 선언한다
  CarMaker.Compact = function () {
    this.doors = 4;
  };
  CarMaker.Convertible = function () {
    this.doors = 2;
  };
  CarMaker.SUV = function () {
    this.doors = 24;
  };
  ```

**내장 객체 팩터리**

- 팩터리 패턴의 실전 예제로 언어에 내장되어 있는 전역 Object() 생성자를 생각해 보자.이 생성자도 입력 값에 따라 다른 객체를 생성하기 때무넹 팩터리처럼 동작한다고 할 수 있다

  ```javascript
  var o = new Object(),
    n = new Object(1),
    s = Object("1"),
    b = Object(true);

  //테스트
  o.constructor = Object; //true
  n.constructor = Number; //true
  s.constructor = String; //true
  b.constructor = Boolean; //true
  ```
