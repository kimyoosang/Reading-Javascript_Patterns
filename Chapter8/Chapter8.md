# Chapter8. DOM과 브라우저 패턴

- 코어 자바스크립트가 아닌 가장 일반적인 자바스크립트 프로그램의 실행 환경인 브라우저에 특화된 여러 가지 패턴에대해 알아본다
- 브라우저 스크립팅은 사실 많은 개발자들이 자바스크립트 중에서 싫어하는 부분이기도 한데, 브라우저별로 일관성이 없는 호스트 객체와 DOM 구현들의 불편함은 클라이언트 스크립팅을 도와주는 훌륭한 패턴들을 사용한다면 이런 어려움을 덜 수 있다

## **8.1 관심사의 분리**

- 웹 애플리케이션 개발에서의 주요 관심사는 다음의 세 가지로 나누어 볼 수 있다
  - 내용
    - HTML 문서
  - 표현
    - CSS스타일: 문서가 어떻게 보여질 것인지 지정한다
  - 행동
    - 자바스크립트: 사용자 인터렉션과 문서의 동적인 변경을 처리한다
- 세 가지 관심사를 최대한 분리할수록, 좀더 광범위한 사용자 에이전트 (예를 들어 그래픽 브라우저, 텍스트만 지원하는 브라우저, 장애인을 위한 보조공학기기, 모바일 기기) 에 애플리케이션을 탑재하기가 용이해진다
- 관심사의 분리는 실무에서 다음과 같은 의미다
  1. CSS 끈 상태에서 페이지를 테스트한다. 이 상태로도 사용 가능하고 내용이 표시되며 읽을 수 있어야 한다
  2. 자바스크립트를 끈 상태에서 페이지를 테스트한다. 여전히 주 목적에 맞게 제대로 동작하고, 모든 링크가 작동하며, 홈 또한 제대로 동작하고 전송할 수 있어야 한다
  3. onclick과 같은 인라인 이벤트 핸들러 또는 인라인 style 속성은 '내용'에 속하지 않으므로 사용하지 않는다
  4. 시맨틱하고 의미에 맞는 HTML 엘리먼트를 사용한다
- '행동'에 속하는 자바스크립트는 **무간섭적**이어야 한다. 즉 자바스크립특 사용자를 방해하거나, 자바스크립트를 지원하지 않는 브라우저에서 페이지를 사용할 수 없게 만들어서는 안되며, 페이지 동작시 필수 요건이 되어서는 안된다는 뜻이다. 자바스크립트는 페이지를 향상시키기만 해야 한다
- **기능 탐지**는 브라우저간의 차이점을 우아하게 다루는 일반적인 기술 중 하나다. 사용자 에이전트를 감지해 코드를 분기하는 대신에, 사용하려는 메서드나 프로퍼티가 현재의 실행 환경에 존재하는지 확인하는 기술을 말한다. 사용자 에이전트 감지는 대체로 안티패턴이라 할 수 있다. 때로는 사용자 에이전트 감지를 쓸 수밖에 없는 경우도 있지만, 기능 탐지로 확실한 결과를 얻을 수 없거나 성능상 심각한 문제를 초래하는 경우에만 최후의 선택 사항으로 고려해야 한다

  ```javascript
  //안티패턴
  if (navigator.userAgent.indexOf("MSIE") !== 1) {
    document.attachEvent("onclick", console.log);
  }

  //더 좋은 방법
  if (document.attachEvent) {
    document.attachEvent("onclick", console.log);
  }

  //조금 더 정확한 방법
  if (typeof document.attachEvent !== "undefined") {
    document.attachEvent("onclick", console.log);
  }
  ```

- 관심사를 분리하면 개발 및 유지보수 그리고 기존의 웹애플리케이션을 업데이트하기 또한 용이해진다. 문제가 생겼을 때 어느 부분을 확인하면 되는지 알 수 있기 때문이다. 자바스크립트 오류가 발생하면, 문제를 해결하기 위해 HTML이나 CSS를 찾아볼 필요가 없다

## **8.2 DOM 스크립팅**

- 페이지의 DOM 트리를 다루는 것은 클라이언트 측 자바스크립트에서 처리하는 가장 흔한 일 중 하나다. 동시에 DOM 메서드가 브라우저간에 일관성 없이 구현되어 있기 때문에 가장 골치아픈 작업이기도 하다
- 때문에 브라우저간의 차이점을 추상화한 훌륭한 자바슬크립트 라이브러리를 사용하면 개발 속도를 크게 향상시킬 수 있다

**DOM접근**

- DOM접근은 비용이 많이 드는 작업이다
- 자바스크립트이 성능에서 DOM접근은 가장 흔한 병목 지점이다. 일반적으로 DOM은 자바스크립트 엔진과 별개로 구현되었기 때문이다. 자바스크립트 애플리케이션에서 DOM을 전혀 사용하지 않을 수도 있기 때문에, 브라우저 입장에서 보면 이런 접근 방식이 타당하다. 또한, 이러한 분리된 구현을 통해 자바스크립트 외의 언어들도 DOM작업을 할 수 있게 된다
- **핵심을 말하자면 DOM접근을 최소화해야한다**
  1. 루프 내에서 DOM 접그느을 피한다
  2. DOM 참조를 지역 변수에 할당하여 사용한다
  3. 가능하면 셀렉터 API를 사용한다
  4. HTML 콜렉션을 순회할 때 length를 캐시하여 사용한다
- 예제 1: 두 번째 루프는 코드가 길어지기 했지만 브라우저에 따라 수십에서 수백 배 빠르다

  ```javascript
  //안티패턴
  for (var i = 0; i < 100; i += 1) {
    document.getElementById("result").innerHTML += i + ",";
  }

  //지역 변수를 활용하는 개선안
  var i,
    content = "";
  for (i = 0; i < 100; i += 1) {
    content += i + ",";
  }
  document.getElementById("result").innerHTML += conetent;
  ```

- 예제 2: 지역 변수를 활용하는 첫 예제보다 한줄이 더 길고, 변수가 하나 더 필요하지만 더 좋은 코드다

  ```javascript
  //안티패턴
  var padding = document.getEmementById("result").style.padding;
  margin = document.getEmementById("result").style.margin;

  //개선안
  var style = document.getEmementById("result").style;
  padding = style.padding;
  margin = style.margin;
  ```

- 예제 3: 셀렉터 API 사용

  ```javascript
  document.querySelector("ul .sekected");
  document.querySelectorAll("#widget .class");
  ```

  - 이 메서드들은 CSS 셀렉터 문자열을 받아 그에 해당하는 DOM 모드의 목록을 반환한다. 셀렉터 메서드들은 IE8 이상을 포함한 대부분의 최신 브라우버에서 사용가능하며 다른 DOM 메서드를 사용한 선택 방식보다 항상 빠르다
  - 다중적인 자바스크립트 라이브러리들도 최신 버전에서 셀렉터 API를 활용하고 있으므로, 사용중인 라이브러리가 최신 버전인지 반드시 확인해보아야 한다
  - 자주 접근하는 엘리먼트에 id 속성을 추가하는 것도 성능 향상에 도움이 될 수 있다. document.getElementById(myid)가 노드를 찾는 가장 쉽고 빠른 방법이기 때문이다

**DOM조작**

- DOM 엘리먼트 접근 이외에도, DOM 엘리먼트를 변경 또는 제거라거나 새로운 엘리먼트를 추가하는 작업도 자주 필요하다. DOM 업데이트시 브라우저는 화면을 다시 그리고(repaint), 다시 재구조화(reflow)하는데, 이 또한 비용이 많이 드는 작업니다
- 원칙적으로 DOM 업데이트를 최소화하는게 좋다. 이를 위해서는 변경 사항들을 일괄 처리하거나, 실제 문서 트리 외부에서 변경 작업을 수행해야 한다
- 비교적 큰 서브 트리를 만들어야 한다면, 서브 트리를 완전히 생성한 다음에 문서에 추가해야한다. 이를 위해 문저 조각(document fragment)에 모든 하위 노드를 추가하는 방법을 사용할 수 있다
- 예제 1: 문서에 노드를 붙일 때 피해야할 안티패턴

  ```javascript
  //안티패턴
  //노드를 만들고 곧바로 문서에 붙인다

  var p, t;

  p = document.createElement("p");
  t = document.createTextNode("first paragraph");
  p.appendChild(t);

  document.body.appendChild(p); //곧바로 문서에 붙인다

  p = document.createElement("p");
  t = document.createTextNode("second paragraph");
  p.appendChild(t);

  document.body.appendChild(p); //곧바로 문서에 붙인다
  ```

- 개선안은 문서 조각을 생성해 외부에서 수정한 후 , 처리가 완전히 끝난 다음에 실제 DOM에 추가하는 것이다
- 편리하게도, DOM 트리에 문서 조각을 추가하면, 조각 자체는 추가되지 않고 그 내용만 추가된다. 즉 문서 조각은 별도의 부모 노드 없이도 여러 개의 노드를 감쌀 수 있는 훌륭한 방법이다 (여러 개의 p태그를 div안에 넣지 않고도 한꺼번에 처리할 수 있다)

- 예제 2: 문서조각을 사용한 개선안

  ```javascript
  var p, t, frag;

  frag = document.createDocumentFragment();

  p = document.createElement("p");
  t = document.createTextNode("first paragraph");
  p.appendChild(t);
  frag.appentChild(p);

  p = document.createElement("p");
  t = document.createTextNode("second paragraph");
  p.appendChild(t);
  frag.appentChild(p);

  document.body.appendChild(frag);
  ```

- 앞서 본 안티패턴 코드예제와 달리 p 엘리먼트를 생성할 때마다 문서를 변경하지 않고 마지막에 단 한번만 변경한다. 화면을 다시 그리고 재계산 하는 과정도 한 번만 이루어진다
- 문서 조각은 새로운 노드를 트리에 추가할 때 유용하다. 하지만 문서에 이미 존재하는 트리를 변경할 때는 변경하려는 서브 트리의 루트를 복제해서 변경한 뒤 노드와 복제한 노드를 맞바꾸면 된다
- 예제 3: 이미 존재하는 트리를 변경

  ```javascript
  var oldnode = document.getElementById("result");
  clone = oldnode.cloneNode(true);

  //복제본을 가지고 변경작업을 처리한다
  //변경이 끝나고 나면 원래의 노드와 교체한다
  oldnode.parentNode.replaceChild(clone, oldnode);
  ```

## **8.3 이벤트**

- 브라우저 스크립팅의 또 다른 영역은 click, mouseover와 같은 이벤트 처리다
- 브라우저 이벤트 처리는 최악의 일관성을 가지고 있어서 숱한 좌절의 원인이 되곤 한다. 자바스크립트 라이브러리를 사용하면 W3C를 준수하는 브라우저와 IE(9버전 미만)를 모두 지원하기 위해 두 벌로 작업하는 수고를 덜 수 있다
- 간단한 페이지에서는 라이브러리를 사용하지 않을 수도 있고, 라이브러리를 직접 만들 수도 있기 때문에 이벤트 처리를 위한 핵심 부분을 살펴보도록 하자

**이벤트 처리**

- 클릭할 때마다 카운터의 숫자를 증가시키는 버튼이 있다고 가정해보자. 인라인 onclick 속성을 추가하면 모든 브라우저에서 잘 동작하겠지만 관심사의 분리와 점진적인 개선의 원칙에 위배된다. 따라서 마크업을 건드리지 않고 항상 자바스크립트에서 이벤트 리스너를 처리해야 한다
- 예제1 : 이벤트 처리

  - 다음과 같은 마크업이 있다고 가정하다

  ```javascript
  <button id="clickme">Click me: 0</button>
  ```

  -간단하게 노드의 onclick 프로퍼티에 함수를 할당할 수도 있지만, 이 방법은 한번밖에 쓸 수 없다

  ```javascript
  var b = document.getElementByid("clickme"),
    count = 0;
  b.onclick = function () {
    count += 1;
    b.innerHTML = "Click me: " + count;
  };
  ```

  - 이 패턴으로는 하나의 클릭 이벤트에 여러 개의 함수가 실행되게 하면서 동시에 낮은 결합도를 유지하기 어렵다. 기술적으로 가능하기는 하다. onclick에 이미 함수가 할당되었는지 확읺하고, 할당되어 있다면 이미 존재하는 함수를 새로운 함수안에 추가하고 이를 onclick의 값으로 대체하면 된다
  - 그렇지만 addEventListener() 메서드를 사용하는 게 훨씬 깔끔한 해결책이다. 이 메서드는 IE8 버전까지는 존재하지 않기 때문에 IE8 버전 이하에서는 attachEvent() 메서드를 사용해야 한다

  ```javascript
  var b = document.getElementById("clickme");
  if (document.addEventListener) {
    //W3C
    b.addEventListener("click", myHandler, false);
  } else if (document.attachEvent) {
    //IE
    b.attachEvent("onclick", myHandler);
  } else {
    //최후의 수단
    b.onclick = myHandler;
  }
  ```

  - 이제 버튼을 클릭하면, myHandler() 함수가 실행될것이다. 이 함수가 버튼의 라벨에 쓰인 "Click me: 0"의 숫자를 증가시키도록 만들어보자
  - 조금더 흥미롭게, 여러 개의 버튼을 두고 모든 버튼이 myHandler() 함수 하나만 사용하게 만들것이다. 각 버튼 노드에 대한 참조와 카운터 숫자를 저장해두는 것은 비효율적이므로, 클릭할 때마다 생성되는 이벤트 객체로부터 필요한 정보를 구한다

  ```javascript
  function myHandler(e) {
    var src, parts;

    //이벤트 객체와 소스 엘리먼트를 가져온다
    e = e || window.event;
    src = e.target || t.srcElement;

    //버튼의 라벨을 변경한다
    parts = src.innerHTML.split(": ");
    parts[1] = parseInt(parts[i], 10) + 1;
    src.innerHTML = parts[0] + ": " + parts[1];

    //이벤트가 상위 노드로 전파되지 않게 한다
    if (typeof e.stopPropagation === "function") {
      e.stopPropagation();
    }
    if (typeof e.cancelBubble !== "undefined") {
      e.cancelBulle = true;
    }

    //기본 동작이 수행되지 않게 한다
    if (typeof e.preventDefault === "function") {
      e.preventDefault();
    }
    if (typeof e.returnValue !== "undefined") {
      e.returnValue = false;
    }
  }
  ```

  - 이벤트 핸들러 함수는 네 부분으로 구성되어 있다
    1. 우선 이벤트 객체를 가져온다. 이벤트 객체는 이벤트에 대한 정보와 이벤트가 발생한 엘리먼트에 대한 정보를 가지고 이싿. 이 이벤트 객체는 콜백 이벤트 핸들러에 인자로 전달되며 onclick 프로퍼티를 사용한 경우에는 전역 프로퍼티인 window.event를 통해 접근할 수 있다
    2. 두 번째 부분은 버튼의 이름을 변경하는 실제 작업을 수행한다
    3. 그 다음에는 이벤트가 상위 노드로 전파되지 않게 한다. 사실 이 예제에서는 필요하지 않지만, 일반적으로 이벤트 전파를 막지 않으면 문서의 최상단 또는 window 객체에까지 버블링되어 올라갈 수 있다. 여기서도 두 가지 방법, 즉 W3C 표준인 방법과 IE를 위한 방법을 사용한다
    4. 마지막으로, 기본 동작이 수행되지 않게 한다. 어떤 이벤트들은 지정된 기본 동작을 하는데, 필요에 따라 preventDefault() 를 사용해 이를 막을 수 있다
  - 중복 작업이 반복되기 때문에, 퍼사드 메서드로 이벤트 유틸리티를 만들어두는 게 좋다

**이벤트 위임**

- 이벤트 위임 패턴은 이벤트 버블링을 이용해서 개별 노드에 붙는 이벤트 리스너의 개수를 줄여준다. div엘리먼트 내에 열개의 버튼이 있다면, 각 버튼 엘리먼트에 리스너를 붙이는 대신 div 엘리먼트에 하나의 이벤트 리스너만 붙인다
- 예제 1: div 안에 세 개의 버튼을 가지는 예제

  ```javascript
  //사용할 마크업
  <div id="click-wrap">
    <button>Click me: 0</button>
    <button>Click me top: 0</button>
    <button>Click me three: 0</button>
  </div>
  ```

  - 각 버튼에 이벤트 리스너를 붙이는 대신 "click-wrap" div 하나의 리스너만을 붙일 것이다. 그리고 불필요한 클릭을 걸러낼 수 있도록 이전 예제의 myHandler() 함수를 약간 수정해서 사용한다
  - 다음과 같이 이벤트가 발생한 노드의 nodeName이 "button" 인지 확인하도록 myHandler()를 변경하였다

    ```javascript
    //...
    //이벤트 객체와 이벤트가 발생한 엘리먼트를 가져온다
    e = e || window.event;
    src = e.target || e.srcElement;

    if (src.nodeName.toLowerCase() !== "button") {
      return;
    }
    //...
    ```

  - 이벤트 위임에는 불필요한 이벤트를 걸러내는 코드가 약간 추가된다는 단점이 있다. 그러나 성능상의 이점과 코드의 간결성으로 인한 장점이 단점보다 훨씬 크기 때문에 적극 추천하는 패턴이다

## \*\*8.4 장시간 수행되는 스크립트

- 스크립트가 너무 오래 수행되면 때때로 브라우저가 사용자에게 스크립트를 중단시킬지 물어보는 경우가 있다. 아무리 복잡한 작업이라도 애플리케이션에서 이런 현상이 발생하는 것은 바람직 하지 않다
- 과도한 스크립트 연산을 실행하면, 브라우저 UI는 응답불가능한 상태가 되어 사용자가 아무 것도 클릭할 수 없게 된다. 이런 상태느ㅡㄴ 사용자 경험에 해가 되므로 반드시 피해야 한다
- 자바스크립트에는 스레드가 없지만, setTimeout()이나 최신 브라우저에서 지원하는 웹워커(web-worker)를 사용해 스레드를 흉내낼 수 있다

**setTimeout()**

- 많은 양의 작업을 작은 덩어리로 쪼개고 각 덩어리를 setTimeout()을 이용해 1밀리초 간격의 타입아웃을 두고 실행하는 방법으로 스레드를 시물레이션할 수 있다. 1밀리초 간견의 타입아웃 덩어리를 사용하면 전체적인 작업 진행 시간이 늘어날 수 있지만, UI를 응답가능한 상태로 유지함으로서 사용자가 더 편하게 브라우저를 제어할 수 있게 해준다

**웹워커**

- 최신의 브라우저들은 장시간 수행되는 스크립트에 대한 또 다른 해결책을 제공한다. 바로 웹워커다
- 웹워커는 브라우저 내에서 백그라운드 스레드를 제공한다. 복잡한 계산을 분리된 파일에 두고 메인 프로그램(페이지)에서 다음과 같이 호출한다
  ```javascript
  var ww = new Worker("my_web_worker.js");
  ww.onmessage = function (event) {
    document.body.innerHTML +=
      "<p>백그라운드 스레드의 메시지: " + eevent.data + "</p>";
  };
  ```
- 웹 워커 소스는 다음과 같이 간단한 산훌 연산을 1억번(1e8) 반복한다

  ```javascript
  var end = 1e8,
    tmp = 1;

  postMessage("안녕하세요");

  while (end) {
    end -= 1;
    tmp += end;
    if (end === 5e7) {
      postMessage(
        '절반 정도 진행되었습니다 현재 " tmp 값"은 ' + tmp + "입니다"
      );
    }
  }

  postMessage("작업종료");
  ```

- 웹워커는 postMessage() 메서드로 호출자(메인페이지)와 통신하며 호출자는 onmessage 이벤트를 구독해 변경 내역을 받느다
- onmassage 콜백 함수는 이벤트 객체를 인자로 받는다. 이 이벤트 객체는 data 프로퍼티를 가지는데, 웹워커가 넘겨주는 어떤 데이터든지 그 값으로 지정될 수 있다
- 이와 비슷하게 호출자는 ww.postMessage()를 사용해 웹워커에게 데이터를 전달할 수 있고, 웹워커는 onmessage 콜백을 사용해서 그 메시지를 구독할 수 있다

## **8.5 원격 스크립팅**

- 최신의 웹애플리케이션들은 현재 페이지를 다시 로드하지 않으면서 서버와 통신하기 위해 우너격 스크립팅을 자주 사용한다. 이를 통해 웹애플리케이션은 마치 데스크탑 애플리케이션처럼 빠르게 반응하게 된다

**XMLSttpRequest**

- XMLHttpRequest는 자바스크립트에서 HTTP 요청을 생성하는 특별한 객체(생성자 함수)로, 현재 대부분의 브라우저에서 사용가능하다. HTTP 요청을 만드는 과정은 세 단계로 이뤄진다
  1. XMLHttpRequest 객체를 설정한다
  2. 응답 객체의 상태가 변경될 때 알림을 받기 위한 콜백 함수를 지정한다
  3. 요청을 보낸다

**JSONP**

- 원격 요청을 생성하는 또 다른 방법은 JSONP(JSON with padding)를 사용하는 방법이다
- XHR과 달리 브라우저의 동일 도메인 정책의 제약을 받지 않는다. 따라서 서드파티 사이트에서 데이터를 로딩할 수 있으므로 보안 측면에서의 영향을 고려하여 신중하게 사용해야 한다
- XHR요청에 대한 응답이 될 수 있는 문서의 종류
  1. XML 문서(예전에 많이 사용되었다)
  2. HTML 조각(꽤 일반적으로 사용한다)
  3. JSON 데이터(가볍고 편리하다)
  4. 간단한 텍스트 파일 또는 다른 파일
- JSONP 요청에 대한 응답 데이터는 주로 JSON을 함수 호출로 감싼 형태다. 호출될 함수의 이름은 요청할 떄 함께 전달한다
- 예: JSONP 요청 URL의 일반적인 형태

  ```javascript
  http://example.org/getdata.php?callback=myHandler
  ```

  - getdata.php는 웹페이지가 될 수도 있고 스크립트가 될 수도 있다. callback 매개변수에는 응답을 처리할 자바스크립트 함수를 지정한다
  - 요청 URL은 다음과 같이 동적으로 생성된 스크립트 엘리먼트에 로드된다

  ```javascript
  var script = document.createElement("script");
  script.src = url;
  document.body.appendChild(script);
  ```

  - 서버는 JSON 데이터를 콜백 함수의 인자로 전달해 응답한다. 최종적으로 이 스크립트가 실제로 페이지에 삽인되면, 다음과 같이 콜백 함수가 호출된다

  ```javascript
  myHandler({ hello: "world" });
  ```

**프렝림과 이미지 비컨(image Beacons)**

- 원격 스크립팅을 위한 또 다른 방법으로 프레임을 사용하는 방법이 있다. 자바스크립트로 iframe을 생성하고 src에 URL을 지정하는 방식이다. 이 URL에는 데이터나 iframe외부의 부모 페이지를 업데이트 하는 함수 호출을 포함할 수 있다
- 원격 스크립팅의 가장 간단한 형태는 서버에 데이터를 보내기만 하고 응답을 필요로 하지 않는 것이다. 이런 경우에는 새로운 이미지를 만들고 이미지의 src를 서버의 스크립트로 지정하면 된다
  ```javascript
  new Image().src = "http://example.org/some/page.php";
  ```
- 이 패턴을 이미지 비컨이라 부른다. 이 패턴은 서버에 로그를 남길 목적으로 데이터를 전송할 때, 예를 들면 방문자 통계를 수집하고자 할 때 유용하다. 비컨에는 응답이 필요없기 때문에 일반적으로 서버는 1x1 픽셀 크기의 GIF 이미지를 보낸다. 이보자는 "204 No Content" HTTP 상태코드를 보내는 것이 더 좋다. 이 상태코드의 의미는 응답에 헤더만 있고 클라이언트에게 되돌려 줄 바디가 없다는 뜻이다
