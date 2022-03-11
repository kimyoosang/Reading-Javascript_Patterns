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
