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
