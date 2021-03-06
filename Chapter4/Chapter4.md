# Chapter4. 함수

## **4.1 배경지식**

- 자바스크립트의 함수를 특별하게 만드는 두 가지 중요한 특징
  1. 함수는 일급객체다
  2. 함수는 유효범위(scope)를 제공한다
- 함수는 다음과 같은 특징을 가지는 객체다
  - 런타임, 즉 프로그램 실행 중에 동적으로 생성할 수 있다
  - 변수에 할당할 수 있고, 다른 변수에 참조를 복사할 수 있으며, 확장 가능하고, 몇몇 특별한 경우를 제외하면 삭제할 수 있다
  - 다른 함수의 인자로 전달할 수 있고, 다른 함수의 반환값이 될 수 있다
  - 자기 자신의 프로퍼티와 메서드를 가질 수 있다
- 자바스크립트에서 함수는 하나의 객체라고 생각하면 된다. 다만 이 객체는 호출하여 실행할 수 있는 특별한 기능을 가지고 있다

```javascript
// 안티패턴
// 테도믜 목적으로만 사용한다
var add = new Function("a,b", "return a + b");
add(1, 2); // 3을 반환한다
```

- 이 코드에서 add()는 생성자를 통해 만들었기 때문에 객체라는 사실이 자명하다. 그러나 Function()생성자의 사용은 eval()만큼이나 좋지않은 방법이다. 코드가 문자열로 전달되어 평가되기 때문이다. 또한 따옴표를 이스케이프해야하고 가독성을 높이기 위해 함수 본문을 들여쓰기 하려면 별도로 신경써야하기 때문에 읽고 쓰기에 불편하다
- 두 번째로 중요한 기능은 함수가 유효범위를 제공한다는 것이다
- 자바스크립트에서는 중괄호 지역 유효범위가 없다. 달리 말해서 블록이 유효범위를 만들지 않는다. 단지 함수 유효범위가 있을 뿐이다. 어떤 변수이건 함수 내에서 var로 정의되면 지역변수이고 함수 밖에서는 참조할 수 없다
- 변수는 해당 블럭을 감싸는 함수가 있을 때만 지역 변수가 된다. 감싸는 함수가 없으면 전역 변수가 된다

**용어정리**

```javascript
//기명 함수 표현식
var add = function add(a, b) {
  return a + b;
};

//함수 표현식 (또는 익명함수)
var add = function (a, b) {
  return a + b;
};
```

- 이름을 생략함 함수 표현식을 무명 함수 표현식, 간단하게 함수 표현식이라고도 하며 익명함수라는 말로도 널리 쓰인다

```javascript
//함수 선언문
function foo() {
  //함수본문
}
```

- 문법적인 면에서, 함수 표현식의 결과를 변수에 할당하지 않을경우 기명 함수 표현식과 함수 선언식은 비슷해 보인다
- 함수 선언문에는 세미콜론이 필요하지 않지만 함수 표현식에는 필요하다

**선언문 VS 표현식: 이름과 호이스팅**

- 하마수 선언문은 전역 유효범위나 다른 함수의 본문 내부, 즉 '프로그램 코드'에서만 쓸 수 있다
- 변수나 프로퍼티에 할당할 수 없고, 함수 호출시 인자로 함수를 넘길 때도 사용할 수 없다

**함수의 name 프로퍼티**

- 함수를 정의하는 패턴을 선택할 때는 읽기 전용인 name프로퍼티를 쓸 일이 있는지도 고려해보아야한다
- 함수 선언문과 기명 함수 표현식을 사용하면 name프로퍼티가 정의된다. 반면 무명 함수 표현식의 name 프로퍼티 값은 경우에 따라 다르다. IE에서는 undefined가 되고, 파이어폭스와 웹킷에서는 빈 문자열로 정의된다
- name프로퍼티는 파이어버그나 다른 디버거에서 코드를 디버깅할 때 유용하다
- 함수 선언문보다 함수 표현식을 선호하는 이유는, 함수 표현식을 사용하면 함수가 다른 객체들과 마찬가지로 객체의 일종이며 어떤 특별한 언어 구성요소가 아니라는 사실이 좀더 드러나기 때문이다

**함수 호이스팅**

- 함수 선언문을 사용하면 변수 선언뿐 아니라 함수 정의 자체도 호이스팅되기 때문에 자칫 오류를 만들어내기 쉽다

```javascript
// 전역 함수
function foo() {
  alert("blobal foo");
}
function bar() {
  alert("blobal bar");
}
function hoistMe() {
  console.log(typeof foo); //"function"
  console.log(typeof bar); //"undefined"

  foo(); // "local foo"
  bar(); //"TypeError: bar is not a function

  //함수 선언문
  //변수 'foo'와 정의된 함수 모두 호이스팅 된다
  function foo() {
    alert("local foo");
  }
  //함수 표현식
  //변수 'bar'는 호이스팅 되지만 정의된 함수는 호이스팅 되지 않는다
  var bar = function () {
    alert("local bar");
  };
}
```

- 지역변수 foo()는 나중에 정의되어도 상단으로 호이스팅되어 정상 동작하는 반면, bar()의 정의는 호이스팅되지 않고 선언문만 호이스팅 된다
- 함수 호이스팅은 함수를 사용하기 전에 반드시 함수를 선언해야 한다는 규칙을 무시하므로 함수 표현식을 권장한다

## **4.2 콜백 패턴**

- 함수는 객체다. 즉 함수를 다른 함수에 인자로 전달할 수 있다. 이를 콜백함수, 혹은 콜백이라고 부른다

**콜백 예제**

```javascript
var findNodes = finction () {
  var i = 10000, //긴 루프
      nodes = [], // 결과를 저장할 배열
      found; // 노드 탐색 결과
    while (i) {
      i -= 1;
      // 이 부분에 복잡한 로직이 들어간다
      ndoes.push(found)
    }
    return nodes;
}
```

- 이 함수는 findNodes()와 같은 형식으로 호출되며, DOM트리를 탐색해 필요한 엘리먼트의 배열을 반환한다

```javascript
var hide = function = (nodes) {
  var i = 0, max = ndoes.length;
  for(; i < max; i += 1) {
    nodes[i].style.display = "none";
  }
}
// 함수를 실행한다
hide(fincNodes());

```

- 이 함수는 이름에서 짐작할 수 있듯이 페이지에서 노드를 숨긴다. 이 구현은 findNodes()에서 반환된 노드의 배열에 대해 hide()가 다시 루프를 돌아야하기 때문에 비효율적이다. findNodes()에서 노드를 선택하고 바로 숨긴다면 재차 루프를 돌지 않아 더 효율적인 것이다
- 노드를 숨기는 로직의 실행을 콜백 함수에 위임하고 이 함수를 findNodes()에 전달한다

```javascript
var findNodes = function (callback) {
  var i = 10000, //긴 루프
    nodes = [], // 결과를 저장할 배열
    found; // 노드 탐색 결과
  // 콜백 함수를 호출할 수 있는지 확인한다
  if (typeof callback !== "function") {
    callback = false;
  }
  while (i) {
    i -= 1;
    // 이 부분에 복잡한 로직이 들어간다

    // 여기서 콜백을 실행한다
    if (callback) {
      callback(found);
    }
    ndoes.push(found);
  }
  return nodes;
};

//콜백함수
var hide = function = (nodes) {
  nodes[i].style.display = "none";
}
```

- findNodes()에는 콜백 함수가 추가되었는지 확인하고, 있으면 실행하는 작업 하나만 추가되었다
- hide()의 구현은 노드들을 순회할 필요가 없어서 더 간단해졌다

**콜백과 유효범위**

- 콜백이 일회성의 익명 함수나 전역 함수가 아니고 객체의 메서드인 경우도 많다

```javascript
var myapp = {};
myapp.color = "green";
myapp.paint = function (node) {
  node.style.color = this.color;
};

var findNodes = function (callback) {
  // ...
  if (typeof callback === "function") {
    callback(found);
  }
};
```

- findNodes(mypapp.paint)를 호출하면 this.color가 정의되지 않아 예상대로 동작하지 않는다. findNodes()가 전역 함수이기 때문에 객체 this는 전역 객체를 참조한다. findNodes()가 dom이라는 객체의 메서드라면, 콜백 내부의 this는 예상과는 달리 myapp이 아닌 dom을 참조하게 된다
- 이 문제를 해결하기 위해서는 콜백 함수와 함꼐 콜백이 속해 있는 객체를 전달하면 된다

```javascript
var findNodes = function (callback, callback_obj) {
  if (typeof callback === "string") {
    callback = callback_obj[callback];
  }
  // ...
  if (typeof callback === "function") {
    callback.call(callback_obj, found);
  }
};
```

**비동기 이벤트 리스너**

- 대부분의 클라이언트 측 브라우저 프로그래밍은 이벤트 구동 방식이다. 페이지의 로딩이 끝나면 load 이벤트를 발생시킨다
- 자바스크립트가 이벤트 구동형 프로그래밍에 특히 적합한 이유는 프로그램이 비동기적으로 달리 말하면 무작위로 동작할 수 있게 하는 콜백 패턴 덕분이다
- 어떤 이벤트는 영영 발생하지 않을 수도 있기 때문에 때로는 콜백 함수가 필요 이상으로 많을 수 도 있다

**타임아웃**

- 브라우저의 window 객체에 의해 제공되는 타임아웃 메서드인 setTimeout()과 setInterval()은 콜백함수를 받아서 실행시킨다

```javascript
var thePiotThickens = function () {
  console.log("500ms later,,,");
};
setTimeout(thePiotThickens, 500);
```

- 이 함수는 곧바로 실항하지 않고 setTimeout()이 나중에 호출할 수 있도록 함수를 가리키는 포인터만을 전달하고 있다

**라이브러리에서의 콜백**

- 콜백은 라이브러리를 설계할 때 유용한 간단하고 강력한 패턴이다. 소프트웨어 라이브러리에 들어갈 코드는 가능한 범용적이고 재사용할 수 있어야 한다. 콜백이 이런 일반화에 도움이 될 수 있다
- 핵심 기능에 집중하고 콜백의 형태로 '연결고리'를 제공하라. 콜백 함수를 활용하면 조금 더 쉽게 라이브러리 메서드를 만들고 확장하고 가다듬을 수 있다

## **4.3 함수 반환하기**

- 함수는 객체이기 때문에 반환 값으로 사용할 수 있다. 즉 함수의 실행 결과로 꼭 어떤 데이터 값이나 배열을 반환할 필요는 없다는 뜻이다

```javascript
var setup = function () {
  alert(1);
  return function () {
    alert(2);
  };
  //setup함수를 사용
  var my = setup(); //alert으로 1이 출력된다
  my(); ///alert로 2가 출력된다
};
```

- setup()은 반환도니 함수를 감싸고 있기 때문에 클로저를 생성한다. 클로저는 반환되는 함수에서는 접근할 수 있지만 코드 외부에서는 접근할 수 없기 때문에, 비공개 데이터 저장을 위해 사용할 수 있다

## **4.4 자기 자신을 정의하는 함수**

- 함수는 동적으로 정의할 수 있고 변수에 할당할 수 있다. 새로운 함수를 만들어 이미 다른 함수를 가지고 있는 변수에 할당한다면, 새로운 함수가 이전 함수를 덮어쓰게 된다

```javascript
var scareMe = function () {
  alert("Boo!");
  scareMe = function () {
    alert("Double Boo!");
  };
};

// 자기 자신을 정의하는 함수를 사용
scareMe(); // Boo!
scareMe(); // Dounle Boo!
```

- 이 패턴은 함수가 어떤 초기화 준비 작업을 단 한번만 수행할 경우에 유용하다. 함수가 자기 자신을 재정의하여 구현 내용을 갱신할 수 있다
- 재정의된 함수의 작업량이 적기 때문에 이 패턴은 애플리케이션 성능에 확실히 도움이 된다
- 이 패턴의 단점은 자기 자신을 재정의한 이유에는 이전에 우너본 함수에 추가했던 프로피트들을 모두 찾을 수 없게 된다는 점이다. 또한 함수가 다른 이름으로 사용된다면, 예를 들어 다른 변수에 할당되거나, 객체의 메서드로써 사용되면 재정의된 부분이 아니라 원본 함수의 본문이 실행된다

```javascript
// 1. 새로운 프로퍼티가 추가된다
// 2. 함수 객체가 새로운 변수에 할당된다
// 3. 함수는 메서드로써는 사용된다

//1. 새로운 프로퍼티를 추가한다
scareMe.property = "properly";

//2. 다른 이름으로 할당한다
var prank = scareMe;

//3. 메서드로 사용한다
var spooky = {
  boo: scareMe,
};

// 새로운 이름으로 호출된다
prank(); //Boo!
prank(); //Boo!
console.log(prank.property); //"properly"

//메서드로 호출한다
spooky.boo(); //Boo!
sooky.boo(); //Boo!
console.log(spooky.boo.property); //properly

//자기 자신을 재정의한 함수를 사용한다
scareMe(); //Double boo!
scareMe(); //Double boo!
console.log(scareMe.property); //undefined
```

- 함수가 새로운 변수에 할당되면 예상과 달리 자기 자신을 정의하지 않는다. 이 모든 호출들은 계속해서 전역 scareMe() 포인터를 덮어쓴다
- 따라서 마지막에 전역 scareMe()가 호출되었을 때 비로소, "Double boo!"를 출력하도록 갱신된 부분이 처음으로 제대로 실행된다
- 또한 screMe property도 더 이상 참조할 수 없게 된다

## **4.5 즉시 실행 함수**

- 즉시 실행 함수 패턴은 함수가 선언되자마자 실행되도록 하는 문법이다

```javascript
(function () {
  alert("watch out!");
});
```

- 이 패턴은 사실상 함수 표현식을 , 생성한 직후 실행된다
- 즉시 실행 함수 패턴은 다음 부분들로 구성된다
  1. 함수를 함수 표현식으로 선언한다 (함수 선언문으로는 동작하지 않는다)
  2. 함수ㅏㄱ 즉시 실행될 수 있도록 마지막에 괄호쌍을 추가한다
  3. 전체 함수를 괄호로 감싼다(함수를 변수에 할당하지 않을 경우에만 필요하다)
- 페이지 로드가 완료된 후, 이벤트 핸들러를 등록하거나 객체를 생성하는 등의 초기 설정 작업을 해야한다. 이 모든 작업은 단 한번만 실행되기 때문에 재사용하기 위해 이름이 지정된 함수를 생성할 필요가 ㅇ벗다. 하지만 한편으로는 초기화 단계가 완료될때까지만 사용할 임시 변수들이 필요하다. 이 모든 변수를 전역으로 생성하는 것은 좋지 않은 생각이다
- 이럴 때 즉시 실행 함수가 필요하다. 즉시 실행 함수는 모든 코드를 지역 유효범위로 감싸고 어던 변수도 전역 유효범위로 새어나가지 않게 한다

**즉시 실행 함수의 매개변수**

- 즉시 실행 함수에 인자를 전달할 수도 있다
- 일반적으로 전역 객체가 즉시 실행 함수의 인자로 전달된다. 따라서 즉시 실행 함수 내에서 window를 사용하지 않고도 전역 객체에 접근할 수 있다
- 일반적으로는 즉시 실행 함수에 대한 인자를 너무 많이 전달하지 않는 것이 좋다. 코드의 동작을 이해하려고 계속해서 코드의 맨 윗부분과 아랫부분 사이를 오가며 스크롤하기가 부담스럽기 때문이다

**즉시 실행 함수의 반환 값**

- 다른 함수와 비슷하게, 즉시 실행 함수도 값을 반환할 수 있고 반환된 값은 변수에 할당될 수 있다

```javascript
var result = (function () {
  return 2 + 2;
})();
```

- 감싸고 있는 괄호를 생략해서 같은 동작을 구현할 수 있다. 즉시 실행 함수의 반환 값을 변수에 할당할 때는 괄호가 필요 없기 때문이다

```javascript
var result = (function () {
  return 2 + 2;
})();
```

- 즉시 실행 함수가 함수를 반환하고 이 반환 값이 getResult라는 변수에 할당된다. 이 함수는 즉시 실행 함수에서 미리 계산하여 클로저에 저장해둔 res라는 값을 반환한다

**장점과 사용방법**

- 즉시 실행 함수 패턴은 폭넓게 사용된다. 전역변수를 남기지 않고 상당량의 작업을 할 수 있게 해준다. 선언된 모든 변수는 스스로를 호출하는 함수의 지역 변수가 되기 때문에 임시 변수가 전역 공간을 어지럽힐 까봐 걱정하지 않아도 된다
- 즉시 실행 함수 패턴을 사용해 개별 기능을 독자적인 모듈로 감쌀 수도 있다
- 점진적인 개선의 측면에서 약간의 코드를 추가해 페이지에 어느 정도 기능을 추가하려고 할 때, 이 코드를 즉시 실행 함수로 감싸고 페이지에 추가된 코드가 있을 때와 없을 때 잘 동작하는 지 확인한다. 그러고 나서 더 많은 개선사항을 추가하거나 제거할 수 도 있고 개별로 테스트할 수도 있으며, 사용자가 비활성화할 수 있게 하는 등의 작업을 할 수 있다

## **4.6 즉시 객체 초기화**

- 전역 유효범위가 난잡해지지 않도록 보호하는 또 다른 방법은 앞서 설명한 즉시 실행 함수 패턴과 비슷한 즉시 객체 초기화 패턴이다
- 이 패턴은 개게가 생성된 즉시 init() 메서드를 실행해 객체를 사용한다. init() 함수는 모든 초기화 작업을 처리한다

```javascript
({

  // 여기에 설정 값(설정 상수)들을 정의할 수 있다
  maxwidth: 600,
  maxheight:400

  //유틸리티 메서드 또한 정의할 수 있다
  gimmeMax: function() {
    return this.maxwidth + "x" + this.maxheight
  }
  //초기화
  init: function(){
    console.log(this.gimmeMax())
    // 더 많은 초기화 작업들..
  }
})
```

- 객체만 괄호로 감싸는게 아니라 객체와 init() 호출 전체를 관호 안에 넣을 수도 있다
- 이 패턴의 장점은 즉시 실행 함수 패턴의 장점과 동일하다. 단 한번의 초기화 작업을 실행하는 동안 전역 네임스페이스를 보호할 수 있다. 코드를 익명 함수로 감싸는 것과 비교하면 이 패턴은 문법적으로 신경써야할 부분이 좀덤 많은 것처럼 보일 수도 있다. 그러나 초기화 작업이 더 복잡하다면 전체 초기화 절차를 구조화 하는데 도움이 된다
- 이 패턴의 단점은 대부분의 자바스크립트 압축 도구가 즉시 실행 함수 패턴에 비해 효과적으로 압축하지 못할 수 있다는 것이다. 비공개 프로퍼티와 메서드의 이름은 더 짧게 변경되지 않느데 압축 도구의 관점에서는 그런 방식이 안전하기 때문이다.
- 구글의 클로저 컴파일러의 고급 모드만이 즉시 초기화되는 객체의 프로퍼티명을 단축시켜준다

## **4.7 초기화 시점의 분기**

- 초기화 시점의 분기(로드타임 분기)는 최적화 패턴이다. 어떤 조건이 프로그램의 생명주기 동안 변경되지 않는게 확실할 경우, 조건을 단 한번만 확인하는 것이 바람직하다. 브라우저 탐지가 저형적인 예다
- DOM엘리먼트의 계산된 스타일을 확인하거나 이벤트 핸들러를 붙이는 작업도 초기화 시점 분기 패턴의 이점을 살릴 수 있는 도 다른 후보들이다

```javascript
//변경이전
var utils = {
  addlistener: function (el, type, fn) {
    if (typeof window.addEventListener === "function") {
      el.addEventListener(type, fn, false);
    } else if (typeof document.attachEvent === "function") {
      //IE
      el.attachEvent("on", +type, fn);
    } else {
      //구형의 브라우저
      el["on" + type] = fn;
    }
  },
  removeEventListener: function (el, type, fn) {
    //거의 동일한 코드
  },
};

//변경이후

//인터페이스
var utils = {
  addListener: null,
  removeListener: null,
};
//구현
if (typeof winodw.addEventListener === "function") {
  utils.addListener = function (el, type, fn) {
    el.addEventListener(type, fn, false);
  };
  utils.removeListener = function (el, type, fn) {
    el.removeEventListener(type, fn, flase);
  };
} else if (typeof document.attachEvent === "function") {
  //IE
  utils.addListener = function (el, type, fn) {
    el.attachEvent("on" + type, fn);
  };
  utils.removeListener = function (el, type, fn) {
    el.detachEvent("on" + type, fn);
  };
} else {
  //구형 브라우저
  utils.addListener = function (el, type, fn) {
    el["on" + type] = fn;
  };
  utils.removeListener = function (el, type, fn) {
    el["on" + type] = null;
  };
}
```

- 이 패턴을 사용할 때 브라우저의 기능을 섣불리 가정하지 말아야 한다. 예를 들어, 브라우저가 window.addEventListener를 지원하지 않는다고 해서 이 브라우저가 IE이고 SMLHTTPRequest도 지원하지 않을 것이라고 가정해서는 안된다는 얘기다
- 가장 좋은 전략은 초기화 시점의 분기를 사용해 기능을 개별적으로 탐지하는 것이다

## **4.8 함수 프로퍼티 - 메모이제이션(Memoization)패턴**

- 언제든지 함수에 사용자 정의 프로퍼티를 추가할 수 있다. 함수에 프로퍼티를 추가하여 결과를 캐시하면 다음 호출 시점에 복잡한 연산을 반복하지 않을 수 있다. 이런 활용 방법을 메모이제이션 패턴이라고 한다

```javascript
var myFunc = function (param) {
  if (!myFunc.cache[param]) {
    var result = {};
    // 비용이 많이 드는 수행...
    myFunc.cache[param] = result;
  }
  return myfunc.cache9param;
};
//캐시 저장 공간
myFunc.cache = {};
```

## **4.9 설정 객체 패턴**

- 설정 객체 패턴은 좀더 깨끗한 API를 제공하는 방법이다. 라이브러리나 다른 프로그램에서 사용할 코드를 만들 떄 특히 유용하다

```javascript
//함수에 많은 수의 매개변수를 전달한다
addperson("Bruce", "Wayne", new Date(), null, null);
```

- 함수에 많은 수의 매개변수를 전달하기를 불편하다. 모든 매개변수를 하나의 객체로 만들어 대신 전달하는 방법이 낫다

```javascript
var conf = {
  username: "batman",
  fiest: "Bruce",
  last: "Wayne",
};

addperson(conf);
```

- 설정 객체의 장점
  1. 매개변수와 순서를 기억할 필요가 없다
  2. 선택적인 매개변수를 안전하게 생략할 수 있다
  3. 읽기 쉽고 유지보수하기 편하다
  4. 매개변수를 추가하거나 제거하기가 편하다
- 설정 객체의 단점
  1. 매개변수의 이름을 기억해야 한다
  2. 프로퍼티 이름은 압축되지 않는다
- 이 패턴은 함수가 DOM 엘리먼트를 생성할 때나 엘리먼트의 CSS 스타일을 지정 할 때 유용하다

## **4.10 커리(Curry)**

**함수적용**

- 함수는 불려지거나 호출된다고 표현하기보다 적용(apply)된다고 표현한다

```javascript
//함수를 정의한다
var sayHi = function (who) {
  return "Hello" + (who ? ", " + who : "") + "!";
};

//함수를 호출한다
sayHi();
sayHi("world");

//함수를 적용한다
sayHi.apply(null, ["hello"]);
```

- 함수를 적용하는 것과 호출하는 것 모두 결과는 동일하다
- apply는 두 개의 매개변수를 받는다. 첫 번째는 이 함수 내에 this와 바인딩할 객체이고, 두 번째는 배열 또인 인자로 함수 내부에서 배열과 비슷한 형태의 arguments 객체로 사용하게 된다. 첫 번째 매개변수가 null이면 this는 전역 객체를 가리킨다. 즉 함수를 특정 객체의 메서드로서가 아니라 일반적인 함수로 호출할 때와 같다
- 함수 호출이라는 것은 사실상 함수 적용을 가리키는 문법 설탕이나 다름 없다

**부분적인 적용**

- 함수의 호출이 실제로는 인자의 묶음을 함수에 적용하는 것임을 알게 되었다. 인자 전부가 아니라 일부 인자만 전달하는 것이 가능할까?

```javascript
// 다음 코드는 가상의 partialApply()메서드 사용법을 보여준다

var add = function (x, y) {
  return x + y;
};

//모든 인자를 적용한다
add.apply(null, [5, 4]);
//인자를 부분적으로만 적용한다
var newadd = add.partialApply(null, [5]);
//새로운 함수에 인자를 적용
newadd.apply(null, [4]); //9
```

- partialApply()메서드는 존재하지 않고, 자바스크립트의 함수는 기본적으로는 이렇게 동작하지 않는다. 그러나 자바스크립트는 굉장히 동적이기 때문에 이렇게 동작하도록 만들 수 있다
- 함수가 부분적인 적용을 이해하고 처리할 수 있도록 만드는 과정을 커링이라고 한다

**커링**

- 커링은 함수를 변형하는 과정이다. 자바스크립트에서는 add()함수를 수정하여 부분 적용을 처리하는 커링 함수롬 만들 수 있다

```javascript

//커링된 add()
//부분적인 인자의 목록을 받는다
function add(x,y) {
  var oldx = x, oldy = y
  if(typeof oldy === 'undefined') {
    return function (new) { //부분적인 적용
      return oldx + newy
    }
  }
  //전체 인자를 적용
  return x + y
}

//테스트
typeof add(5) //'function'
add(3)(4) //7

//새로운 함수를 만들어 저장
var add2000 = add(2000)
add2000(10) // 2010
```

- 이 코드에서 처음 add()를 호출할 때, add가 반환하는 내부 함수에 클로저를 만든다
- 어떤 함수라도 부분적인 매개변수를 받는 새로운 함수로 변형한다. 즉 범용 함수를 만드는 방법

```javascript
function schonfinkelize(fn) {
  var slice = Array.protytype.slice,
    stored_arg = slice.call(arguments, 1);
  return function () {
    var new_arg = slice.call(arguments),
      args = stred_args.concat(new_args);
    return fn.apply(null, args);
  };
}
```

- schonfinkelize()가 처음으로 호출될 때, 지역변수 slice에 slice() 메서드에 대한 참조를 저장하고, stored_args에 인자를 저장한다. 이 때 첫 번째 인자를 커링될 함수이기 때문에 떠어낸다. 그리고 새로운 함수를 반환한다. 반환된 새로운 함수는 호출되었을 때 클로저를 통해 이전에 비공개로 저장해 둔 stored_args와 slice 참조에 접근할 수 있다. 새로운 함수는 이미 일부 적용된 인자인 stored_args와 새로운 인자 new_args를 합친 다음, 클로저에 저장되어 있는 원래의 함수 fn에 적용하기만 하면 된다

**커링을 사용해야 할 경우**

- 어떤 함수를 호출할 때 대부분의 매개변수가 항상 비슷하다면, 커링의 적합한 후보라고 할 수 있다
- 매개변수를 적용하여 새로운 함수를 동적으로 생성하면 이 함수는 반복되는 매개변수를 내부적으로 저장하여, 매번 인자를 전달하지 않아도 원본 함수가 기대하는 전체 목록을 미치 채워놓을 것이다

## **4.11 요약**

- 자바스크립트에서 함수를 제대로 이해하고 적합하게 사용하는 일은 매우 중요하다
- 자바스크립트에서 중요한 두 가지 특징
  1. 함수는 일급 객체다. 값으로 전달될 수 있고, 프로퍼티와 메서드를 확장할 수 있다
  2. 함수는 지역 유효범위를 제공한다. 다른 중괄호 묶음은 그렇지 않다. 로컬 변수의 선언은 로컬 유효범위의 맨 윗부분으로 호이스팅된다는 점도 기억해두어야 한다
- 함수를 생성하는 문법
  1. 기명 함수 표현식
  2. 함수 표현식. 익명 함수라고도 한다
  3. 함수 선언문. 다른 언어의 함수 문법과 유사하다
- 함수의 여러가지 패턴
  1. API 패턴: 함수에 더 좋고 깔끔한 인터페이스를 제공할 수 있게 도와준다
  - 콜백 패턴: 함수를 인자로 전달한다
  - 설정 객체: 함수에 ㅁ낳은 수의 매개변수를 전달할 때 통제를 벗어나지 않도록 해준다
  - 함수 반환: 함수의 반환 값이 또 다시 함수일 수 있다
  - 커링: 원본 함수와 매개변수 일부를 물려받는 새로운 함수를 생성한다
  2. 초기화 패턴: 웹페이지와 애플리케이션에서 매우 흔히 사용되는 초기화와 설정 작업을 , 전역 네임스페이스를 어지럽히지 않고 임시 변수를 사용해 좀더 깨끗하고 구조화된 방법으로 수행할 수 있게 도와준다
  - 즉시 실행 함수: 정의되자마자 실행된다
  - 즉시 객체 초기화: 익명 객체 내부에서 초기화 작업을 구조화한 다음 즉시 호출할 수 있는 메서드를 제공한다
  - 초기화 시점의 분기: 최초 코드 실행 시점에 코드를 분기하여, 애플리케이션 생명 주기 동안 계속해서 분기가 발생하지 않도록 막아준다
  3. 성능패턴: 코드의 실행속도를 높이는데 도윰을 준다
  - 메모이제이션 패턴: 함수 프로퍼티를 사용해 계산된 값을 다시 계산되지 않도록 한다
  - 자기선언 함수:ㅣ 자기 자신을 덮었므으로써 두 번째 호출 이후부터는 작업량이 줄어들게 만든다
