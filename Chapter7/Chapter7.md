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
