# Chapter5. 객체 생성 패턴

## **5.1 네임스페이스 패턴**

- 네임스페이스는 프로그램에서 필요로하는 전역 변수의 개수를 줄이는 동시에 과도한 접두어를 사용하지 않고도 이름이 겹치지 않게 해준다
- 수많은 함수, 객체, 변수들로부터 전역 유효범위를 어지럽히는 대신, 애플리케이션이나 라이브러리를 위한 전역 객체를 하나 만들고 모든 기능을 이 객체에 추가하면 된다

```javascript

//수정 전: 전역 변수 5개
//경고. 안티패턴이다

//생성자 함수 2개
function Parent() {}
function Childe() {}

//변수 1개
var some_var = 1;

//객체 2개
var moduel1 = {};
moduel1.data = {a:1,b:2}
vare module2 = {};

```

- 위와 같은 코드를 리팩터링하기 위해서는 먼저 애플리케이션 전용 전역 객체, 이를 테면 MYAPP을 생선한다음 모든 함수와 변수들을 이 전역 객체의 프로퍼티로 변경한다

```javascript
//수정 후: 전역 변수 1개

//전역 객체
var MYAPP = {};

//생성자
MYAPP.Parent = function () {};
MYAPP.Child = function () {};

//변수
MYAAPP.some_var = 1;

//객체 컨테이너
MYAPP.modules = {};

//객체들을 컨테이너 안에 추가한다

//객체 2개
MYAPP.modules.module1 = {};
MYAPP.modules.module1.data = { a: 1, b: 2 };
MYAPP.modules.module2 = {};
```

- 네임스페이스 패턴의 장점
  1. 코드에 네임스페이스를 지정해주며, 코드내의 이름 충돌 뿐 아니라 이 코드와 같은 페이지에 존재하는 자바스크립트 라이브러리나 위짓 등 서브 파티 코드와의 이름 충동도 방지해준다
  2. 다양한 작업에 응용할 수 있으며, 매우 권장하는 패턴이다
- 네임스페이스 패턴의 단점
  1. 모든 변수와 함수에 접두어를 붙여야 하기 때문에 전체적으로 코드량이 약간 더 많아지고 다운로드해야 하는 파일 크기도 늘어난다
  2. 전역 인스턴스가 단 하나뿐이기 때문에 코드의 어느 한 부분이 수정되어도 전역 인스턴스를 수정하게 된다. 즉 나머지 기능들도 갱신된 상태를 물려받는다
  3. 이름이 중첩되고 길어지므로 프로퍼티를 판별하기위한 검색 작업도 길고 느려진다

**범용 네임스페이스 함수**

- 프로그램의 복잡도가 증가하고 코드의 각 부분들이 별개의 파일로 분리되어 선택적으로 문서에 포함되게 되면, 어떤 코드가 특정 네임스페이스나 그 내부의 프로퍼티를 처음으로 정의한다고 가정하기가 어려워진다
- 때문에 네임스페이스를 생성하거나 프로퍼티를 추가하기 전에 먼저 이미 존재하는지 여부를 확인해야 한다

```javascript
//위험하다
var MYAPP = {};
//개선안
if (typeof MYAPP === "undefined") {
  var MYAPP = {};
}
//또는 더 짧게 쓸 수 있다
var MYAPP = MYAPP || {};
```

- 다음은 네임스페이스 함수를 구현한 예제다. 다음과 같은 방식은 해당 네임스페이스가 존재하면 덮어쓰지 않기 때문에 기존 코드를 망가뜨리지 않는다

```javascript
var MYAPP = MYAPP || {}; //존재여부 확인

MYAPP.namespace = function (ns_string) {
  var parts = ns_string.split(","),
    parent = MYAPP,
    i;

  //처음에 중복되는 전역 객체명은 제거한다
  if (parts[0] === "MYAPP") {
    parts = parts.slice(1);
  }
  for (i = 0; i < parts.length; i += 1) {
    if (typeof parent[parts[i]] === "undefined") {
      parent[parts[i]] = {};
    }
    parent = parent[parts[i]];
  }
  return parent;
};
```

## **의존 관계 선언**

- 자바스크립트 라이브러리들은 대개 네임스페이스를 지정하여 모듈화되어 있기 때문에, 필요한 모듈만 골라서 쓸 수 있다
- 이 때 함수나 모듈 내 최상단에, 의존 관계에 있는 모듈을 선언하는 것이 좋다. 즉 지역 변수를 만들어 원하는 모듈을 가리키도록 선언하는 것이다

```javascript
var myFunction = function () {
  //의존 관계에 있는 모듈들
  var event = YAHOO.util.Event,
    dom = YAHOO.util.Dom;

  //이제 event와 dom이라는 변수를 사용한다
};
```

- 이 패턴의 장점
  1. 의존 관계가 명시적으로 선언되어 있기 때문에 코드를 사용하는 사람이 페이지내에 반드시 포함시켜야 하는 스크립트 파일이 무엇인지를 알 수 있다
  2. 함수의 첫머리에 의존 관계가 선언되기 때문에 의존 관계를 찾아내고 이해하기가 쉽다
  3. dom과 같은 지역 변수를 YAHOO와 같은 전역 변수보다 언제나 더 빠르며 YAHOO.util.Dom처럼 전역 변수의 중첩 프로퍼티와 비교하면 더 말할 것도 없다. 의존 관계 선언 패턴을 잘 지키면 함수 안에서 전역 객체 판별을 단 한 번만 수행하고, 이 다음부터는 지역변수를 사용하기 때문에 훨씬 빠르다
  4. YUI 컴프레서나 구글 클로서 등 고급 압축 도구는 지역 변수명에 대해서는 event를 A라는 글자 하나로 바꾸는 식으로 축약해 코드를 줄여준다. 하지만 전역 변수명 변경은 위험하기 때문에 축약하지 않는다

### **5.3 비공개 프로퍼티와 메서드**

- 자바스크립트에서는 private, protected, public 같은 프로퍼티와 메서드를 나타내는 별도의 문법이 없다. 객체의 모든 멤버는 public, 즉 공개되어 있다
- 생성자 함수를 사용해 객체를 생성할 때도 마찬가지로 모든 멤버가 공개된다

**비공개(private)멤버**

- 비공개 멤버에 대한 별도의 문법은 없지만 클로저를 사용해서 구현할 수 있다. 생성자 함수 안에서 클로저를 만들면, 클로저 유효범위 안의 변순는 생성자 함수 외부에 노출되지 않지만 객체의 공개 메서드 안에서는 쓸 수 있다. 즉 생성자에서 객체를 반환할 때 객체의 메서드를 정의하면, 이 메서드 안에서는 비공개 변수에 접근할 수 있다

  ```javascript
  function Gadget() {
    //비공개 멤버
    var name = "iPod";
    //공개된 함수
    this.getName = function () {
      return name;
    };
  }
  var toy = new Gadget();
  //'name'은 비공개이므로 undefined가 출력된다
  console.log(toy.name); //undefined

  //공개 메서드에서는 'name'에 접근할 수 있다
  console.log(toy.getName); // 'iPod'
  ```

**특권(privileged)메서드**

- 특권 메서드라는 개념은 특정 문법과는 관련이 없다.단지 비공개 멤버에 접근권한을 가진 공개 메서드를 가리키는 이름을 뿐이다

**비공개 멤버의 허점**

- 비공개 멤버를 유지하는게 관건이라면, 다음과 같은 경우에 대해서 신경을 써야 한다
  1. 파이어폭스의 초기 버전 중 일부는 eval()함수에 두 번째 매개변수를 전달할 수 있게 되어있따. 이 매개변수는 함수의 비공개 유효범위를 들여다볼 수 있게 해주는 컨텍스트 객체다. 현재 널리 사용되는 브라우저에는 적용되지 않는 사례들이다
  2. 특권 메서드에서 비공개 변수의 값을 바로 반환할 경우 이 변수가 객체나 배열이라면 값이 아닌 참조가 반환되기 때문에 , 외부 코드에서 비공개 변수 값을 수정할 수 있다

```javascript
function Gadget() {
  //비공개 멤버
  var specs = {
    screen_width: 320,
    screen_hight: 480,
    color: "white",
  };

  //공개 함수
  this.getSpecs = function () {
    return specs;
  };
}
```

- 여기서 getSpec()메서드가 specs객체에 대한 참조를 반환한다는게 문제다
- 이를 해결하는 하나의 방법은 getSpecs()에서 아예 새로운 객체를 만들어 사용자에게 쓸모있을 만한 데이터 일부만 담아 반환하는 것이다. 이것을 **최조 권한의 법칙**이라고도 한다
- 모든 데이터를 넘겨야 한다면 객체를 복사하는 범용 함수를 사용하며 specs 객체의 복사본을 만드는 것도 방법이 될 수 있다

**객체 리터럴과 비공개 멤버**

- 객체 리터럴로 객체를 생성한 경우에 비공개 멤버를 만들기 위해서는 비공개 데이터를 함수로 감싸기만 하면 된다. 따라서 객체 리터럴에서는 익명 즉시 실행 함수를 추가하여 클로저를 만든다

```javascript
var myobj;
(function () {
  //비공개 멤버
  var name = "my, oh my";

  //공개될 부분을 구현한다
  //var를 사용하지 않았다는 데 주의하라
  myobj = {
    //특권 메서드
    getName: function () {
      return name;
    },
  };
})();

myobj.getName(); // "my, oh my"
```

**프로토타입과 비공개 멤버**

- 생성자를 사용하여 비공개 멤버를 만들 경우, 생성자를 호출하여 새로운 객체를 만들 때마다 비공개 멤버가 매번 재생성 된다는 단점이 있다
- 이러한 중복을 없애고 메모리를 절약하려면 공통 프로퍼티와 메서드를 생성자의 protoype 프로퍼티에 추가해야 한다. 이렇게 하면 동일한 생성자로 생성한 모든 인스턴스가 공통된 부분을 공유하게 된다
- 감춰진 비공개 멤버들도 모든 인스턴스가 함께 쓸 수 있다
- 이를 위해서는 두 가지 패턴, 생성자 함수 내부에서 비공개 멤버를 만드는 패턴과 객체 리터럴로 멤버를 만드는 패턴을 함께 써야한다
- 그 이유는 prototype 프로퍼티도 결국 객체라서, 객체 리터럴로 생성할 수 있기 때문이다

```javascript
function Gadget() {
  //비공개 멤버
  var name = "iPod";
  //공개함수
  this.getName = function () {
    return name;
  };
}
Gedget.prototype = (function () {
  //비공개멤버
  var browser = "Mpbile Webkit";
  //공개된 프로토타입 멤버
  return {
    getBrowser: function () {
      return browser;
    },
  };
})();

var toy = new Gadget();
console.log(toy.getName()); //객체 인스턴스의 특권 메서드
console.log(tpy.getBrowser()); //프로토타입의 특권 메서드
```

\*_비공개 함수를 공개 메서드로 노출시키는 방법_

- 노출패턴(revelation parrern)은 비공개 메서드를 구현하면서 동시에 공개 메서드로도 노출하는 것을 말한다
- 객체의 모든 기능이 객체가 수행하는 작업에 필수불가결한 것들이라서 최대한의 보호가 필요한데, 동시에 이 기능들의 융용성 때문에 공개적인 접근도 허용하고 싶은 경우가 있을 수 있다
- 이럴때 객체 리터럴 안에서 비공개 멤버를 만드는 패턴을 사용하면 된다

```javascript
var myarray;
(function () {
  var astr = "[object Array]",
    toString = Object.prototype.toString;

  function isArray(a) {
    return toString.call(a) === astr;
  }
  function indexOf(haystack, needle) {
    var i = 0;
    max = haystack.length;
    for (; i < max; i += 1) {
      if (haystack[i] === needle) {
        return i;
      }
      return -1;
    }
  }
  myarray = {
    isArray: isArray,
    indexOf: indexOf,
    inArray: indexof,
  };
})();

//테스트
myarray.isArray([1, 2]); // true
myarray.isArray({ 0: 1 }); //false
myarray.indexOf(["a", "b", "z"], "z"); // 2
myarray.inArray(["a", "b", "z"], "z"); // 2
```

- 즉시 실행 함수의 마지막 부분을 보면, 공개적인 접근을 허용해도 괜찮겠다고 결정한 기능들이 myarray 객체에 채워진다
- 이제 공개된 메서드인 indexOf()에 예기치 못한 일이 일어나더라도, 비공개 함수인 indexOf()는 안전하게 보혿괴기 때문에 inArray()는 계속해서 잘 동작할 것이다

## **5.4 모듈 패턴**

- 모듈 패턴은 늘어나는 코드를 구조화하고 정리하는 데 도움이 되기때문에 널리 쓰인다
- 모듈 패턴을 사용하면 개별적인 코드를 느슨하게 결합시킬 수 있다. 따라서 각 기능들을 블랙박스처럼 다루면서도 소프트웨어 개발 중에 요구 사항에 따라 기능을 추가하거나 교체하거나 삭제하는 것도 자유롭게 할 수 있다
- 모듈 패턴은 다음 패턴들 여러개를 조합한 것이다
  1. 네임스페이스 패턴
  2. 즉시 실행 함수
  3. 비공개 멤버와 특권 멤버
  4. 의존 관계 선언
- 즉시 실행 함수의 비공개 유효범위를 사용하면, 비공개 프로퍼티와 메서드를 마음껏 선언할 수 있다. 모듈에 의존 관계가 있다면 즉시 실행 함수 상단에서 정의한다

```javascript

MYAPP.namespace("MYAPP.utilities.array")
MYAPP.utilities.array = (function () {

  //의존관계
  var uobj = MYAPP.utilities.object,
  var uobj = MYAPP
  ulang = .utilities.lang

  //비공개 프로퍼티
  array_string = "[object Array]"
  ops = Object.prototype.toString

  //비공개 메서드들
  //...

  //var선언을 마친다

  //필요하면 일회성 초기화 절차를 실행한다
  // ...

  //공개 API
  return{
    inArray: function (needle, haystack) {
      for(var i = 0; max = haystack.length; i += 1) {
        if(haystack[i] === needle) {
          return true
        }
      }
    }
    isArray: function (a) {
      return ops.call(a) === array.string
    }
    //... 더 필요한 메서드와 프로퍼티를 여기 추가한다
  };
}());

```

**모듈 노출 패턴**

- 모든 메서드를 비공개 상태로 유지하고, 최종적으로 공개 API를 갖출 때 공개할 메서드만 골라서 노출하는 패턴

```javascript
MYAPP.utilities.array = (function () {
  //비공개 프로퍼티
  array_string = "[object Array]";
  ops = Object.prototype.toString;

  //비공개 메서드들
  var inArray = function (needle, haystack) {
    for (var i = 0; (max = haystack.length); i += 1) {
      if (haystack[i] === needle) {
        return true;
      }
    }
  };
  var isArray = function (a) {
    return ops.call(a) === array.string;
  };

  //var선언을 마친다

  //공개 API 노출
  return {
    isArray: isArray,
    indexOf: inArray,
  };
})();
```

**생성자를 생성하는 모듈**

- 모듈 패턴을 사용하면서 생성자 함수를 사용해 객체를 만드는 패턴
- 모듈을 감싼 즉시 실행 함수가 마지막에 객체가 아니라 함수를 반환하게 하면 된다
- 다음 모듈 패턴 예제는 생성자 함수인 MYAPP.utilities.Array를 반환한다

```javascript

MYAPP.namespace("MYAPP.utilities.array")
MYAPP.utilities.array = (function () {

  //의존관계
  var uobj = MYAPP.utilities.object,
  var uobj = MYAPP
  ulang = .utilities.lang

  //비공개 프로퍼티와 메스드들을 선언한 후...
  Constr;
  //var 선언을 마친다

  //필요하면 일회성 초기화 절차를 실행한다
  // ...

  //공개 API - 생성자 함수
  Constr = function (o) {
    this.elements = this.utilities.Array(o)
  }
  //공개 API -프로토타입
  Consrt.prototype = {
    contructor: MYAPP.utilities.Array,
    version: "2.0",
    toArrray: function (obj) {
      for(var i; a = [], len = obj.length; i < len; i += 1) {
        a[i] = obj[i]
      }
      return a
    }
  }

  //생성자 함수를 반환한다
  //이 함수가 새로운 네임스페이스에 할당될 것이다
  return Constr
}());

//이 생성자 함수는 다음과 같이 사용한다
var arr  =  new MYAPP.utilities.Array(obj);

```

**모듈에 전역 변수 가져오기**

- 이 패턴의 흔한 변형 패턴으로는, 모듈을 감싼 즉시 실행 함수에 인자를 전달하는 형태가 있다
- 보통 전역 변수에 대한 참조 또는 전역 객체 자체를 전달한다. 이렇게 전역 변수를 전달하면 즉시 실행 함수 내에서 지역 변수로 사용할 수 있게 되기 때문에 탐색 작업이 좀더 빨라진다

```javascript
MYAPP.utilities.module = (function (app, global) {
  //전역 객체에 대한 참조와
  //전역 애플리케이션 네임스페이스 객체에 대한 참조가 지역 변수화된다
})(MYAPP, this);
```
