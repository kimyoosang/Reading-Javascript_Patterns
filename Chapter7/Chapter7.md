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

## **7.3 반복자 (Iterator)**

- 반복자 패턴에서, 객체는 일종의 집합적인 데이터를 가진다. 데이터가 저장된 내부구조는 복잡하더라도 개별 요소에 쉽게 접근할 방법이 필요할 것이다. 객체의 사용자는 데이터가 어떻게 구조화되었는지 알 필요가 없고 개별 요소로 원하는 작업을 할 수만 있으면 된다
- 반복자 패턴에서, 객체는 next() 메서드를 제공한다. next()를 연이어 호출하면 반드시 다음 순서의 요소를 반환해야 한다. 데이터 구조 내에서 '다음 순서'가 무엇을 의미하는지는 여러분에게 달려있다
- agg라는 객체가 있다고 할때, 다음과 같이 단순히 루프 내에서 next()를 호출하여 개별 데이터 요소에 접글할 수 있다

  ```javascript
  var element;

  while ((element = agg.next())) {
    //element로 어떤 작업을 수행한다
    console.log(element);
  }
  ```

- 반복자 패턴에서 객체는 보통 hasNext()라는 편리한 메서드도 제공한다. 객체의 사용자는 이 메서드로 데이터의 마지막에 다다랐는지 확인할 수 있다

  ```javascript
  while (agg.hasNext()) {
    //다음 요소로 어떤 작업을 수행한다
    console.log(agg.next());
  }
  ```

- 반복자 패턴을 구현할 때 데티너는 물론 다음에 사용할 요소를 가리키는 포인터도 비공개로 저장해두는 것이 좋다
- 데이터에 좀더 쉽게 접그느하고 여러 차례 반복해 순회할 수 있도록, 다음과 같은 편의 메서드를 추가로 제공할 수 있다
  1. rewind(): 포인터를 처음으로 되돌린다
  2. current(): 현재의 요소를 반환한다. next()는 포인터를 전진시키기 때문에 이 작업을 할 수 없다

```javascript
var agg = (function () {
  //생략

  return {
    //생략
    rewind: function () {
      index = 0;
    },
    current: function () {
      return data[index];
    },
  };
})();

//테스트
//이 루프는 1,3,5를 찍을 것이다
while (agg.hasNext()) {
  console.log(agg.next());
}

//처음으로 되돌린다
agg.rewind();
console.log(agg.current); //1
```

## **7.4 장식자 (Decorator)**

- 장식자 패턴을 이용하면 런타임시 부가적인 기능을 객체에 동적으로 추가할 수 있다
- 장식자 패턴의 편리한 특징은 기대되는 행위를 사용자화하거나 설정할 수 있다는 것이다. 처음에는 기보적인 몇가지 기능을 가지는 평범한 객체로 시작한다. 그런 다음 사용 가능한 장식자들의 풀(pool)에서 원하는 것을 골라 갹채애ㅔ 가눙울 덧붙여간다

**구현**

- 장식자 패턴을 구현하기 위한 한 가지 방법은 모든 장식자 객체에 특정 메서드를 포함시킨 후, 이 메서드를 덮어쓰게 만드는 것이다
- 각 장식자는 사실 이전의 장식자로 기능이 추가된 객체를 상속한다. 장식 기능을 담당하는 메서드들은 uber(상속된 객체)에 이쓴ㄴ 동일한 메서드를 호출하여 값을 가져온다음 추가 작업을 덧붙이는 방식으로진행한다

  - 먼저 생성자와 프로토타입 메서드부터 시작해보자

  ```javascript
  function Sale(price) {
    this.price = price || 100;
  }
  Sale.prototype.getPrice = function () {
    return this.price;
  };
  ```

  - 장식자 객체들은 생성자 프로퍼티 Sale.decorators의 프로퍼티로 구현된다

  ```javascript
  Sale.decorators = {};
  ```

  - 장식자 객체는 처음에는 부모의 메서드로부터 값을 가져온 다음 그 값을 변경한다는 점에 주의한다

  ```javascript
  Sale.decorators.fedtax = {
    getPrice: function () {
      var price = this.uber.getPrice();
      price += (price * 5) / 100;
      return price;
    },
  };
  Sale.decorators.quebec = {
    getPrice: function () {
      var price = this.uber.getPrice();
      price += (price * 7.5) / 100;
      return price;
    },
  };
  Sale.decorators.money = {
    getPrice: function () {
      return "$" + this.uber.getPrice().toFixed(2);
    },
  };
  Sale.decorators.cdn = {
    getPrice: function () {
      return "CDN$" + this.uber.getPrice().toFixed(2);
    },
  };
  ```

  - 모든 조각들을 짜맞추는 decorate() 메서드를 구현해보자
  - 자식 객체가 부모 객체에 접근할 수 있도록 newobj에 uber프로퍼티를 저장한다. 그러고 나서 모든 장식자들의 추가 프로퍼티들을 새로 꾸며진 newobj 객체로 복사한다. 마지막으로 newobj가 반환된다. 이 예제에서는 newobj가 새롭게 갱신된 sale객체다

  ```javascript
  Sale.prototype.decorate = function (decorator) {
    var F = function () {},
      overrides = this.constructor.decorators[decorator],
      i,
      newobj;
    F.prototype = this;
    newobj = new F();
    newobj.uber = F.prototype;
    for (i in overrides) {
      if (overrides.hasOwnProperty(i)) {
        newobj[i] = overrides[i];
      }
    }
    return newobj;
  };
  ```

**목록을 사용한 구현**

- 이 방법은 자바스크립트의 동적 특성을 최대한 활용하며 상속은 전혀 사용하지 않는다. 각각의 꾸며진 메서드가 체인 안에 있는 이전의 메서드를 호출하는 대신에, 간단하게 이전 메서드의 결과를 다음 메서드에 매개변수로 전달한다
- 이 구현 방법을 사용하면 장식을 취소하거나 제거하기 쉽다
- decorate()에서 반환된 값을 다시 객체에 할당하지 않기 때문에 사용방법이 조금 더 간단하다. decorate()는 목록에 장식자를 추가하기만 할 뿐, 객체에는 아무런 일도 하지 않는다

  ```javascript
  var sale = new Sale(100); //가격은 100달러이다
  sale.decorate("fedtax"); //연방세를 추가한다
  sale.decorate("quebec"); //지방세를 추가한다
  sale.decorate("money"); //통화형식을 지정한다
  sale.getPrice(); // $112.88

  function Sale(price) {
    this.price = price > 0 || 100;
    this, (decorators_list = []);
  }
  ```

- 사용가능한 장식자들은 여기서도 Sale.decorators의 프로퍼티로 구현된다
- getPrice()메서드 역시 더 간단해졌음을 확인할 수 있다. 중간 결과 값을 가져오기 위해 부모의 getPrcie()를 호출할 필요가 없어졌기 때문이다. 결과 값은 매개변수로 그들에게 전달된다

  ```javascript
  Sale.decorators = {};

  Sale.decorators.fedtax = {
    getPrice: function (price) {
      return price + price * 5 / 100;
    },
  };
  Sale.decorators.quebec = {
    getPrice: function (price) {
      return price + price * 7.5 / 100;
    },
  Sale.decorators.money = {
    getPrice: function (price) {
      return '$' + price.toFixed(2)
    },
  ```

- decorate()는 단지 장식자를 목록에 추가할 뿐이고 getPrice()가 모든 일을 한다. 즉 현재 추가된 장식자들의 목록을 조사하고, 각각의 getPrice() 메서드를 호출하면서 이전 반환 값을 전달한다

  ```javascript
  Sale.prototype.decorate = function (decorator) {
    this.decorators_list.push(decorator);
  };
  Sale.prototype.getPrice = function () {
    var price = this.price,
      i,
      max = this.decorators_list.length,
      name;
    for (i = 9; i < max; i += 1) {
      name = this.decorators_list[i];
      price = Sale.decorators[name].getPrice(price);
    }
    return price;
  };
  ```

- 두 번째 장식자 패턴 구현 방법이 더 간단하고 상속과 관련이 없다

## **7.6 전략**

- 전략 패턴은 런타임에 알고리즘을 선택할 수 있게 해준다. 사용자는 동일한 인터페이스를 유지하면서, 특정한 작업을 처리할 알고리즘을 여러 가지 중에서 상화엥 맞게 선택할 수 있다

**데이터 유효성 검사 예제**

- 웹페이지상의 폼에서 가져온 다음과 같은 데이터가 있다고 하자. 유효성 검사기를 통해 데이터가 유효한지 검증하려고 한다

  ```javascript
  var data = {
    first_name: "Super",
    last_name: "Man",
    age: "unknown",
    username: "o_0",
  };
  ```

- 설정을 통해 어떤 데이터를 유효한 데이터로 받아들일지 규칙을 지정해야한다

  ```javascript
  validator.config = {
    first_name: "isNonEmpty",
    age: "isNumber",
    username: "usAlphaNum",
  };
  ```

- 유효성 검사기(validator)객체가 데이터를 처리할 수 있도록 설정되었으니, validate() 메서드를 호출하여 검증 오류가 발생하는지 콘솔에 출력해보자

  ```javascript
  validator.validate(data);
  if (validator.hasErrors()) {
    console.log(validator.messages, join("\n"));
  }
  //'나이'값이 유효하지 않습니다. 숫자만 사용할 수 있습니다. 예" 1, 3.14, 2010
  //'사용자명'값이 유효하지 않습니다. 특수 문자를 제외한 글자와 숫자만 사용할 수 있습니다
  ```

- 이제 유효성 검사기(validator) 객체가 어떻게 구현되는지 살펴보자. 검사에 사용되는 알고리즘 객체들은 사전에 정의된 인터페이스를 가진다. 이 객체들은 validate() 메서드와 에러 메시지에서 사용될 한 줄 짜리 도움말 정보인 instructions 프로퍼티를 제공한다

  ```javascript
  //값을 가지는지 확인한다
  validator.types.isNinEmpty = {
    validate: function(value) {
      return value !== ""
    },
    instructions: "이 값은 필수입니다"
  }
  //숫자 값인지 확인한다
  validator.types.isNumber = {
    validate: function (value) {
      return !isNaN(value)
    },
    instructions:"숫자만 사용할 수 있습니다. 예: 1, 3.14, 2010"
  }
  //값이 문자와 숫자로만 이루어졌는지 확인한다
  validator.types.isAlphaNum = {
    validate: function(value) {
      return !\[^a-z0-9/i.test(value)]
    },
    instructions:"특수 문자를 제외한 글자와 숫자만 사용할 수 있습니다"
  }

  ```

- 마지막으로 validator 객체의 핵심 부분을 살펴보자

  ```javascript
  var validator = {
    // 사용할 수 있는 모든 검사 방법들
    types = {}.

    //현재 유효성 검사 세션의 에러 메시지들
    messages: [],

    //현재 유효성 검사 설정
    //'데이터 필드명: 사용할 검사 방법'의 형식
    config:{},

    //인터페이스 메서드
    //'data'는 이름 => 값 쌍이다
    validate: function (data) {
      var i, msg, type, checker, result_ok

      //모든 메시지를 초기화한다
      this.messages = []

      for(i in data) {
        if(data.hasOwnProperty(i)){

          type = this.config[i]
          checker = this.types[type]

          if(!type) {
            continue //서어정된 검사 방법이 없을경우 검증할 필요가 없으므로 건너뛴다
          }
          if(!checker) { //설정이 존재하나 해당하는 검사 방법을 찾을 수 없을 경우 오류 발생
            throw {
              name:"ValicationError",
              message: type + '값을 처리할 유효성 검사기가 존재하지 않습니다'
            }
          }
          result_ok = checker.valudate(data[i])
          if(!result_ok) {
            msg = "/'" + i + "\'값이 유효하지 않습니다." +checker.instructions
          }
        }
      }
      return this.hasErrors()
    },

    //도우미 메서드
    hasErrors: function() {
      return this.messages.length !== 0
    }
  }

  ```

- validator 객체는 범용적인 형태이기 때문에 모든 유효성 검사 사례에 사용할 수 있다
- 새로운 사례가 생길 때마다 단지 validator 객체에 설정을 추가하고 validatr() 메서드를 호출하기만 하면 된다

## **7.6 퍼사드(Facade)**

- 이 패턴은 객체의 대체 인터페이스를 제공한다. 메서드를 짧게 유지하고 하나의 메서드가 너무 많은 작업을 처리하지 않게 하는 방법은 설계상 좋은 습관이다
- 하지만 이렇게 하ㄷ보면 메서드 숫자가 엄청나게 많아지거나 uber 메서드에 엄청나게 많은 매개변수를 전달하게 될 수 있다. 두 개 이상의 메서드가 함게 호출되는 경우가 많다면, 이런 메서드 호출들을 하나로 묶어주는 새로운 메서드를 만드는게 좋다
- 예를 들어 브라우저 이벤트를 처리할 때 사용하는 stopPropagation()과 preventDefault()는 한꺼번에 호출되는 일이 많다. 따라서 두 개의 메서드 호출을 애플리케이션 여기저기에서 반복하기 보다는, 이 둘을 함께 호출하는 퍼사드 메서드를 생성하는게 좋다

  ```javascript
  var myevent = {
    //...
    stop: function (e) {
      e.preventDefault();
      e.stopPropagation();
    },
    //....
  };
  ```

- 퍼사드 패턴은 브라우저 스크립팅에도 적합하다. 브라우저 간의 차이점을 퍼사드 뒤편에 숨길 수 있기 때문이다
- IE에서의 이벤트 API 차이를 처리하는 코드를 추가해보자

  ```javascript
  var myevent = {
    //...
    stop: function (e) {
      //IE 이외의 브라우저
      if (typeof e.preventDefault === "function") {
        e.preventDefault();
      }
      if (typeof e.stopPropagation === "function") {
        e.stopPropagation();
      }
      //IE
      if (typeof e.returnValue === "boolean") {
        e.returnValue = false;
      }
      if (typeof e.canceBubble === "boolean") {
        e.canceBubble = true;
      }
    },
    //....
  };
  ```

- 퍼사드 패턴은 설계 변경과 리팩터링의 수고를 덜어준다. 복잡한 객체의 구현 내용을 교체하는 데는 상단한 시간이 걸리는데, 이와 동시에 이 객체를 사용하는 새로운 코드가 계속해서 작성되고 있을 것이다
- 이런 경우, 우선 새로운 객체의 API를 생각해보고, 기존 객체 앞에 이 API의 역할을 하는 퍼사드를 생성해 적용해볼 수 있다. 이렇게 기존 객체를 완전히 교체하기 전에 최신 코드가 새로운 API를 사용하게 하면, 최종 교체시 변경폭을 줄일 수 있다

## **7.7 프록시 (Proxy)**

- 프록시 디자인 패턴에서는 하나의 객체가 다른 객체에 대한 인터페이스로 동작한다. 퍼사드 패턴이 메서드 호출 몇개를 결합시켜 편의를 제공하는 것에 불과하다면, 프록시는 클라이언트 객체와 실제 대상 객체 사이에 존재하면서 접근을 통제한다
- 이 패턴은 비용이 증가하는 것처럼 보일 수도 있지만 실제로는 성능 개선에 도움을 준다. 프록시는 실제 대상 객체를 보호하야, 되도록 일을 적게 시키기 대문이다
- 프록시 패턴의 한 예로, 게으른 초기화(lazy initialization) 를 들 수 있다. 객체를 초기화하는 데 많은 비용이 들디만, 실제로 초기화한 후에는 한번도 사용하지 않는다고 해보다. 이런경우에 실제 대상 객체에 대한 인터페이스로 프록시를 사용하면 도움이 된다. 프록시는 초기화 요청을 대신 받지만, 실제 대상 객체가 정말로 사용되기 전까지는 이 요청을 전달하지 않는다

**예제**

- 프록시 패턴은 실제 대상 객체가 비용이 많이 드는 작업을 할 때 유용하다. 네트워크 요청은 애플리케이션에서 가장 비용이 많이 드는 작업중 하나다. 따라서 가능한 많은 HTTP 요청들을 하나로 결합하는 게 효과적이다
- HTTP 요청을 결합하는 예제
  - 가수를 선택하면 동영상을 재생해주는 간단한 애플리케이션
  - 동영상 정보와 URL은 페이지 내에 있지 않고 웹서비스를 호출하여 가져와야한다. 웹서비스는 다수의 동영상 ID를 받을 수 있기 때문에, 가능한 많은 수의 동영상 정보를 한꺼번에 가져오면 HTTP 요청 횟수가 줄어들어 애플리케이션 속도를 개선할 수 있다
  - 이 애플리케이션은 여러 개 또는 모든 동영상을 동시에 선택해 정보를 조회할 수 있므르로, 이 웹서비스 요청을 결합시키면 좋을 것이다
- **프록시가 없을 경우**

  - 이 애플리케이션에서 주요 행위자는 다음의 두 객체다
    - videos
      - videos.getInfo() 메서드로 정보 영역을 펼치고 닫는다. videos.getPlayer() 메서드로 동영상을 재생한다
    - http
      - http.makeReauest()메서드를 통해 서버와 통신한다
  - 프록시가 없다면, 각각의 동영상이 videos.getInfo()를 호출할 때마다 http.makeRequest()도 한 번씩 호출될 것이다. 프록시를 추가하면 proxy라는 새로운 행위자가 videos와 http사이에 들어가 makeRequest()에 대한 요청을 위임받으면서, 가능하면 이 요청을 병합하게 된다

    ```javascript
    //videos 객체
    var videos = {
      getPlayer: function (id) {},
      updateList: function (data) {},
      getinfo: function (id) {
        var info = $("info" + id);
        if (!info) {
          http.makeRequest([id], "videos.updateList");
          return;
        }
        if (info.style.display === "none") {
          info.style.display = "";
        } else {
          info.style.display = "none";
        }
      },
    };
    ```

  **프록시 추가**

  - proxy 객체를 추가해 http객체와 videos 객체 간의 통신을 전담하게 만들 것이다. videos 객체는 HTTP 서비스를 직접 호출하는 대신 proxy를 호출한다. proxy는 요청을 전달하기 전에 잠시 기다린다. 대기중인 50밀리초 안에 videos로부터 다른 호출이 들어오면 하나로 병합한다
  - 50밀리초 대기 시간은 사용자가 인지하기 힘든 짧은 시간이지만 "Toggel Checked"를 클릭해서 한 번에 여러 개의 동영상을 펼쳐보려고 할 때 개별 요청들을 병합시킴으로써 사용자 경험의 속도를 개선하는데 도움이 된다. 웹서버 역시처리할 요청의 수가 줄어들기 때문에 부하를 상당히 덜 수 있다
  - 수정된 버전의 코드가 이전의 코드와 유일하게 다른 점은 다음과 갍이 videos.getinfo()가 http.makeRequest() 대신에 proxy.makeRequest()를 호출한다는 것이다

    ```javascript
    proxy.makeRequest(id, videos.updateList, videos);
    ```

  - 프록시는 최근 50밀리초 이내에 받아들인 도영상 ID를 대기열 (queue)에 모아놓았다가, http 객체를 호출하면서 대기열을 비운다(flush). 이때 자신의 콜백 함수도 전달한다

    ```javascript
    var proxt = {
      ids: [],
      delay: 50,
      timeout: null,
      callback: null,
      context:null,

      makeRequest: function(id, callback, context) {

        //큐에 추가한다
        this,ids.push(id)

        this.callback = callback
        this.context = context

        //timeout을 설정한다
        if(!this.timeout) {
          this.timeout = setTimeout(function() {
            proxy.flush()
          }, this.delay)
        }

      },
      flush: function() {
        http.makeRequest(this.ids, "proxy.handler")

        //timeout과 큐를 비운다
        this.timeout = null
        this.ids = []
      },
      handler: function(data) {
        var i, max
        //동영상이 한 개일 경우
        if(parseInt(data.query.count, 10) === 10) {
          proxy.callback.call(proxy.context,data.query.results.Video)
          return
        }
        //동영상이 여러 개일 경우
        for (i=0; max = data.query.results.Video.length; i < max; i += 1) {
          proxy.callback.call(proxy.context,data.query.results.Video[i])
        }
      }
    }

    ```

  - 프로시를 도입하면 간단한 수정을 통해 여러 개의 웹서비스 호출을 하나로 병합할 수 있게 된다

**프록시를 사용해 요청 결과 캐시하기**

- 프록시의 새로운 cache 프로퍼티에 이전 요청의 결과를 캐시해두면, 실제 http객체를 더욱 보호할 수 있다
- 망약 vidoes 객체가 도이일한 동영상 ID에 대한 정보를 다시 요청하면, 프록시는 캐시된 결과를 반환해서 네트워크 라운드트립을 줄인다

## **7.8 중재자 (Mediator)**

- 크기에 상관 없이 애플리케이션은 독립된 객체들로 만들어진다. 객체간의 통신은 유지보수가 쉽고 다른 객체를 건드리지 않으면서, 애플리케이션의 일부분을 안전하게 수정할 수 있는 방식으로 이루어져야 한다
- 객체들이 서로에 대해 너무 많은 정보를 아는 상태로 직접 통신하게 되면 서로간에 결합도가 높아져 바람직하지 않다. 객체들이 강하게 결합되면, 다른 객체들에 영향을 주지 않고 하나의 객체를 수정하기가 어렵다. 매우 간단한 변경도 어려워지고, 수정에 필요한 시간을 예측하는 것이 사실상 불가능해진다
- 중재자 패턴은 결합도를 낮추고 유지보수를 쉽게 개선하여 이런 문제를 완화시킨다. 이 패턴에서 독립된 동료 객체들은 직접 통신하지 않고, 중재자 객체를 거친다. 동료 객체들은 자신의 상태가 변경되면 중재자에게 알리고, 중재자는 이 변경 사항을 아아야 하는 다른 동료 객체들에게 알린다

**중재자 패턴 예제**

- 두 명의 플레이어 중 주어진 30초 동안 버튼을 더 많이 누르는 사람이 이기는 게임 애플리케이션이 있다. 플레이어 1은 1키를 누르고 플레이어 2는 0키를 누른다. 점수판에서 현재의 점수를 표시한다
- 이 게임을 구성하는 객체들은 다음과 같다
  - 플레이어 1
  - 플레이어 2
  - 점수판
  - 중재자
- 중재자는 다른 모든 객체에 대해 알고 있다. 중재자는 입력 장치와 통신하며, keypress 이벤트를 처리하고, 어떤 플레이어의 차례인지 결정해서 알려준다. 플레이어는 게임을 하면서 1점을 딸 때마다 득점 사실을 중재자에게 알려준다. 중재자는 점수판 객체에 플레이어의 점수를 전달한다. 전달된 점수는 차례로 화면에 표시된다
- 중재자 이외의 객체들은, 다른 객체들에 대해 전혀 알지 못한다. 덕분에 게임에 플레이어를 추가하거나 게임의 남은 시간을 표기하는 등의 새로운 기능을 쉽게 추가할 수 있다
- 플레이어 객체들은 Player() 생성자로 만ㄷ르어지고 points와 name 프로퍼티를 가진다. 프로토타입에 추가된 play()메서드는 점수를 1점씩 올리고 점수 변화를 중재자(mediator)에게 알린다

  ```javascript
  function Player(name) {
    this.points = 0;
    this.name = name;
  }
  Player.prototype.play = function () {
    this.points += 1;
    mediator.played();
  };
  ```

- 점수판 객체(scoreboard)는 update() 메서드를 가진다. 플레이어의 차례가 바뀔 때마다 중재자 객체가 이 메서드를 호출한다. 중재자로부터 전달받은 점수를 표시만 할 뿐이다

  ```javascript
  var scoreboard = {
    //점수를 표시할 HTML 엘리먼트
    elemente: document.getClementById("results"),

    //점수 표시를 갱신한다
    update: function (score) {
      var i,
        msg = "";
      for (i in score) {
        if (score.hasOwnProperty(i)) {
          mag += "<p><strong>" + i + "</strong>";
          msg += score[i];
          mas += "</p>";
        }
      }
      this.element.innerHTML = msg;
    },
  };
  ```

- 중재자 객체(meiator)는 게임을 초기화한다
- setup() 메서드 안에서 player객체를 만들고, players 프로퍼티에 플레이어 객체들의 참조를 저장해준다
- player()메서드는 차례가 바뀔 때마다 각 츨레이어 객체에 의해 호출된다. 이 메서드는 score 해시를 업데이트한 다음 scoreboard 객체에 전달해 화면에 점수를 표시한다
- 마지막 메서드인 keypress()는 키보드 이벤트를 처리하고 어떤 플레이어의 차례인지 판단해 알려준다

  ```javascript
  var mediator = {
    //모든 player 객체들
    players: {},

    //초기화
    setup: function() {
      var players = this.players
      players.home = new Player('Home')
      players.guest = new Player('Guest')
    },

    //누군가 play하고 점수를 업데이트 한다
    played: function () {
      var players = this,.players,
          score = {
            Home: players.home.points,
            Guest: players.guest.points
          }
          scoreboard.update(score)
    },

    //사용자 인터렉션을 핸들링한다
    keypress: function() {
      e= e || window.event //IE
      if(e.which === 49) { // 키 "1"
        mediator.players.home.play()
        return
      }
      if(e.which === 48) { //키 "2"
        mediator.players.guest.play()
        return
      }
    }
  }

  ```

- 그리고 마지막으로 게임을 시작하고 종료시킨다

  ```javascript
  //시작!
  mediator.setup();
  window.onkeypress = mediator.keypress;

  //30초 후에 게임을 종료시킨다
  setTiumeout(function () {
    window.onkeypress = null;
    alert("Game over");
  }, 30000);
  ```

## **7.9 감시자 (Observer)**

- 감시자 패턴은 클라이언트 측 자바스크립트 프로그래밍에서 널리 사용되는 패턴이다. mouseover, keypress 와 같은 모든 브라우저 이벤트가 감시자 패턴의 예다. 감시자 패턴은 **커스텀 이벤트(custom event)**라고 부르기도 하는데, 이는 브라우저가 발생시키는 이벤트가 아닌 프로그램에 의해 만들어진 이벤트를 뜻한다. 또 다른 이름으로 구독자/발행자 패턴이라고도 한다
- 이 패턴의 주요 목적은 결합도를 낮추는 것이다. 어떤 객체가 다른 객체의 메서드를 호출하는 대신, 객체의 특별한 행동을 구독해 알림을 받는다. 구독자는 감시자 라고도 부르며, 관찰되는 객체는 발행자 또는 감시대상이라고 부른다. 발행자는 중요한 이벤트가 발생했을 때 모든 구독자에게 알려주며 주로 이벤트 객체의 형태로 메시지를 전달한다

**예제: 잡지구독**

- 일간 신문과 월간 잡지를 출판하는 paper라는 발행자가 있다. 구독자 joe는 출판될 때마다 알림을 받게 된다
- paper 객체에는 모든 구독자를 저장하는 배열인 subscribers 프로퍼티가 존재한다. 구독은 단지 이 배열에 구독자를 추가하는 것으로 이워진다. 이벤트가 발생하면 paper는 subscrubers의 목록을 순회하면서 각 구독자에게 알린다. '알림'이란 구독자 객체의 메서드를 호출한다는 뜻이다. 따라서, 구독자는 구독할 떄 자신의 메서드 중 하나를 paper의 subscribe() 메서드에 전달해야 한다
- paper는 unsubscribe() 메서드도 제공할 수 있다. unsubscribe에는 subscribers 배열에서 구독자를 제거한다는 뜻이다. 마지막으로 publish() 메서드 또한 중요하다 . 이 메서드는 subscribers의 메서드들을 호출한다. 요약하면 발행자 객체는 다음의 멤버들을 가져야 한다
  - subscribers : 배열
  - subscribe() : subscribers 배열에 구독자를 추가한다
  - unsubscribe() : subscribers 배열에서 구독자를 제거한다
  - publish() : subscribers를 순회하여 구독자들이 등록할 때 제공한 메서드들을 호출한다
- 세 매서드 모두 type 매개변수를 필요로 한다. 발행자는 신문 또는 잡지 출판 등 여러가지 이벤트를 동작시킬 수 있고, 구독자는 어떤 이벤트를 구독할지 선택할 수 있기 때문이다
- 이 멤버들은 어떤 발행자 객체에도 적용할 수 있기 때문에, 별도의 객체로 구현하는 것이 바람직 하다. 이렇게 하면 믹스인 패턴에 따라 이 멤버들을 복사해 어떤 객체든지 발행자 객체로 바꿀 수 있다
- **발행자의 기능**을 구현해보자

  ```javascript
  var publisher = {
    subscribers: {
      any: [], //'이벤트 타입: 구독자의 배열' 의 형식
    },
    subscribe: function (fn, type) {
      type = type || "any";
      if ((typeof this, subscribers[type] === "undefined")) {
        this, (subscribers[type] = []);
      }
      this.subscrubers[type.push(fn)];
    },
    unsubscribe: function (fn, type) {
      this.visitSubscibers("unsubscribe", fn, type);
    },
    publish: function (publication, type) {
      this.visitSubscibers("publish", publication, type);
    },
    visitSubscibers: function (action, arg, type) {
      var pubtype = type || "any",
        subscribers = this.subscribers[pubtype].i,
        max = subscribers.length;
      for (i = 0; i < max; i += 1) {
        if (action === "publish") {
          subscribers[i](arg);
        } else {
          if (subscribers[i] === arg) {
            subscribers.splce(i, 1);
          }
        }
      }
    },
  };
  ```

- 다음의 함수는 객체를 받아 발행자 객체로 바꿔준다. 단순히 해당 객체에 범용 발행자 메서드들을 복사해 넣ㄴ느다

  ```javascript
  function makePublisher(o) {
    var i;
    for (i in publisher) {
      if (publisher.hasOwnProperty(i) && typeof publisher[i] === "function") {
        o[i] = publisher[i];
      }
    }
    o.subscibers = { any: [] };
  }
  ```

- paper객체를 구현해보자
- paper객체는 일간 또는 월간으로 출판하는 일만 처리한다
  ```javascript
  var paper = {
    daily: function () {
      this.publish("big news today");
    },
    monthly: function () {
      this, publish("intersting analysis", "monthly");
    },
  };
  ```
- paper를 발행자로 만든다

  ```javascript
  makePublisher(paper);
  ```

- 발행자를 만들었으니, 이제 구독자 객체 joe를 살펴보자. joe는 두 개의 메서드를 가진다

  ```javascript
  var joe = {
    drinkCoffee: function (paper) {
      console.log(paper + "를 읽었습니다");
    },
    sundayPreNap: function (monthly) {
      console.log("잠들기 전에" + monthly + "를 읽고 있습니다");
    },
  };
  ```

- 그리고 paper 구독자 목록에 joe 추가한다.(다르게 말해서 joe가 paper를 구독한다)

  ```javascript
  paper.subscribe(joe.drinkCoffee);
  paper.subcribe(jow.sundayPreNap, "monthly");
  ```

- joe는 기본 이벤트 타입인 'any' 이벤트를 발생 시 호출될 메서드와, 'monthly' 타입의 이벤트 발생시 호출될 메서드를 전달했다. 이제 몇가지 이벤트를 발생시켜보다
  ```javascript
  paper.daily();
  paper.daily();
  paper.daily();
  paper.monthly();
  ```
- 모든 이벤트 발행은 각각에 대응하는 joe의 메서드를 호출하게 되고 콘솔에는 다음과 같은 결과가 출력된다
  ```
  big news today를 읽었습니다
  big news today를 읽었습니다
  big news today를 읽었습니다
  잠들기 전에 interesting analysis를 읽고있습니다
  ```
- paper객체 내에서 joe를 하드코딩하지 않았고, joe 객체 안에서도 역시 paper객체를 하드코딩하지 않았다는 점에서 이 코드는 훌륭하다. 모든 내용을 알고 있는 중재자 객체가 존재하지도 않는다. 객체들은 느슨하게 결합되었고, 이 객체들은 전혀 수정하지 않고 paper 에 수많은 수독자를 추가할 수 있다. 또한 joe는 언제든지 구독을 해지할 수 있다
- 이 예제를 한층 더 발전시켜보도 joe를 발행자로도 만들어보자
- joe가 발행자가 되어 트위터에 상태 업데이트를 포스팅한다

  ```javascript
  makePublisher(joe);
  joe.tweet = function (msg) {
    this, publish(msg);
  };
  ```

- paper 홍보 부서에서 readTweets() 메서드로 독자들의 트윗을 릭고 joe를 구독하기로 결정하였다고 하자

  ```javascript
  paper.readTweets = function (tweet) {
    alert("Call big meeting! Someone" + tweet);
  };
  joe.subscribe(paper.readTweets);
  ```

- 이제 joe가 트윗을 하자마자, paper는 알림을 띄우게 된다
