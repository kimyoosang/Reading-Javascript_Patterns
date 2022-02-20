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

## **6.3 클래스 방식의 상속 패턴 #1 - 기본패턴**

- 재사용 가능한 inherit() 함수의 첫 번째 구현 예제는 다음과 같다

  ```javascript
  function inherit(C, P) {
    C.prototype = new P();
  }
  ```

- 여기서는 prototype 프로퍼티가 함수가 아니라 객체를 가리키게 하는 것이 중요하다. 즉 프로토타입이 부몸 생성자 함수 자체가 아니라 부모 생성자 함수로 생성한 객체 인스턴스를 가리켜야 한다
- 이렇게 구현한 후에 애플리케이션에서 new Child()를 사용해 객체를 생성하면, 프로토타입을 통해 Parent() 인스턴스의 기능을 물려받게 된다

**프로토타입 체인 추적**

- 이 패턴을 사용하면 부모 객체의 프로토타입에 추가된 프로퍼티와 메서드들과 함께 , 부모 객체 자신의 프로퍼티도 모두 물려받게 된다
- 이 상속 패턴에서 프로토타입의 체인이 어떻게 동작하는지 살펴보자
  1. new Parent()를 사용해 객체를 생성하면, 블록을 하나 만든다고 생각하자. 이 블록 안에는 데이터와 함께 다른 블록에 대한 참조를 담을 수 있다
  2. 이 블록에는 name 프로퍼티에 대한 데이터가 담겨져 있다. (new Parent).say() 등의 형태로 say() 메서드에 접근을 시도하면, new Parent() 블록에는 해당 메서드가 없다
  3. 하지만 Parent() 생서자 함수의 프로토타입 프로퍼티를 가리키는 **proto** 라는 숨겨진 링크를 통해, say() 메서드를 알고 있는 Parent.prototype 에 접근할 수 있다

**패턴 #1 의 단점**

- 단점
  1. 부모 객체의 this에 추가된 객체 자신의 프로퍼티와 프로토타입 프로퍼티를 모두 물려받게 된다. 대부분의 경우 객체 자신의 프로퍼티는 특정 인스턴스에 한정되어 재사용 할 수 없기 때문에 필요가 없다
  2. 범용 inherit() 함수는 인자를 처리하지 못하는 문제도 가지고 있다. 즉 자식 생성자에 인자를 넘겨도 부모 생서앚에게 전달하지 못한다. 자식 객체가 부모 생성자에 인자를 전달하는 방법도 있겠지만, 이 방법은 자식 인스턴스를 생성할 때마다 상속을 실행해야하기 때문에, 결국 부모 객체를 계속해서 재생성하는 셈이고, 따라서 매우 비효율적이다

## **6.4. 클래스 방식의 상속 패턴 #2 - 생성자 빌려쓰기**

- 이번에 살펴볼 패턴은 자식에서 부모로 인자를 전달하지 못했던 #1의 문제를 해결한다
- 이 패턴은 부모 생성자 함수의 this에 자식 객체를 바인딩한 다음, 자식 생성자가 받은 인자들을 모두 넘겨준다
  ```javascript
  function Child(a, c, b, d) {
    Parent.apply(this, arguments);
  }
  ```
- 이렇게 하면 부모 생성자 함수 내부의 this에 추가된 프로퍼티만 물려받게 된다. 프로토타입에 추가된 멤버는 상속되지 않는다
- 생성자 빌려쓰기 패턴을 사용하면, 자식 객체는 상속된 멤버의 복사본을 받게 된다. 패턴 #1 처럼 자식 객체가 상속된 멤버의 참조를 물려받은 것과는 다르다
- 예제를 통해 차이점을 확인해보자

  ```javascript
  //부모생성자
  function Article() {
    this.tags = ["js", "css"];
  }
  var article = new Article();

  //클래스 방식의 패턴 #1을 사용해 article 객체를 상속하는 blog 객체를 생성한다
  function BlogPost() {}
  BlogPost.prototype = article;
  var blog = new BlogPost();
  // 여기서는 이미 인스턴스가 존재하기 때문에 'new Article'을 쓰지 않았다

  //생성자 빌려쓰기 패턴을 사용해 article을 상속하는 page 객체를 생성한다
  function StaticPage() {
    Article.call(this);
  }
  var page = new StaticPage();

  alert(article.hasOwnProperty("tags")); //true
  alert(blog.hasOwnProperty("tags")); //false
  alert(page.hasOwnProperty("tags")); //true
  ```

- 기본 패턴을 적용한 blog객체는 tags를 자신의 프로퍼티로 가진 것이 아니라 프로토타입을 통해 접근하기 때문에 hasOwnProperty()로 확인하면 false가 반환된다
- 그러나 생성자만 빌려쓰는 방식으로 상속받은 page 객체는 부모의 tags 멤버에 대한 참조를 얻는 것이 아니라 복사존을 얻게 되므로 자기 자신의 tags 프로퍼티를 가진다
- 상속된 tags 프로퍼티를 수정할 때의 차이점을 살펴보자

  ```javascript
  blog.tags.push("html");
  page.tags.push("php");
  alert(article.tags.join(", ")); //'js, css, html'
  ```

- blog객체가 tags 프로퍼티를 수정하면 동시에 부모의 멤버도 수정된다
- 그러나 page.tags에 적용된 변경사항은 부모인 article에 영향을 미치지 않는다. page.tags는 상속 관정에서 별개로 생성된 복사본이기 때문이다

**프로토타입 체인**

- 앞서 사용했던 Parent()와 Child() 생성자에 이 패턴을 적용했을 때 프로토타입 체인은 어떤 모습이 되는지 살펴보자

  ```javascript
  //부모 생성자
  function Parent(name) {
    this.name = name || "Adam";
  }

  //프로토타입에 기능을 추가한다
  Parent.prototype.say = function () {
    return this.name;
  };

  //자식 생성자
  function Child(name) {
    Parent.apply(this, arguments);
  }

  var kid = new Child("Patrick");
  kid.name; //'Patrick'
  typeof kid.say; //undefined
  ```

- 새로 생성된 Child 객체와 Parent 사이에 링크는 존재하지 않는다. 이 패턴을 적용하면 kid는 자기 자신의 name 프로퍼티를 가지지만 say() 메서드는 상속받을 수 ㅇ벗다
- 여기서의 상속은 부모가 가진 자신만의 프로퍼티를 자식의 프로퍼티로 복사해주는 일회성 동작이며 **proto** 라는 링크는 유지되지 않는다

**생성자 빌려쓰기를 적용한 다중 상속**

- 생성자 빌려쓰기 패턴을 사용하면, 생성자를 하나 이상 빌려쓰는 다중 상속을 구현할 수 있다

  ```javascript
  function Cat() {
    this.legs = 4;
    this.say = function () {
      return "meaowww";
    };
  }
  function Bird() {
    this.wings = 2;
    this.fly = true;
  }
  function CatWings() {
    Cat.apply(this);
    Bird.apply(this);
  }

  var jane = new CatWings();
  console.dir(jsne);
  ```

**생성자 빌려쓰기 패턴의 장단점**

- 장점
  1. 무보 생성자 자신의 멤버에 대한 복사존을 가져올 수 있다. 덕분에 자식이 실수로 부모의 프로퍼티를 덮어쓰는 위협을 방지할 수 있다
- 단점
  1. 프로토타입이 전혀 상속되지 않는다는 한계가 있다. 재사용되는 메서드와 프로퍼티는 인스턴스 별로 재생성되지 않도록 프로토타입에 추가해야 하기 때문이다

### **6.5 클래스 방식의 상속 패턴 #3 - 생성자 빌려쓰고 프로토타입 지정해주기**

- 앞선 두 패턴을 결합해본다. 먼저 부모 생성자를 빌려온 후, 자식의 프로토타입이 부모 생성자를 통해 생성된 인스턴스를 가리키도록 지정한다

  ```javascript
  function Child(a, c, b, d) {
    Parent.apply(this, arguments);
  }
  Child.prototype = new Parent();
  ```

- 이렇게 하면 자식 객체는 부모가 가진 자신만의 프로퍼티의 복사좀을 가지게 되는 동시에, 부모의 프로토타입 멤버로 구현된 재사용가능한 기능들에 대한 참조 또한 물려받게 된다
- 자식이 무도 생성자에 인자를 넘길 수도 있다
- 부모가 가진 모든 것을 상속하는 동시에, 부모의 프로퍼티를 덮어쓸 위험 없이 자신만의 프로퍼티를 마음놓고 변경할 수 있다
- 부모 생성자를 비효율적으로 두 번 호출하는 것은 단점일 수 있다. 우리가 살펴본 에제에서 name의 경우와 같이, 부모가 가진 자신만의 프로퍼티는 두 번 상속된다

  ```javascript
  //부모생성자
  function Parent(name) {
    this.name = name || "Adam";
  }

  //프로토타입에 기능을 추가한다
  Parent.prototype.say = function () {
    return this.name;
  };
  //자식 생성자
  function Child(name) {
    Parent.apply(this, arguments);
  }
  Child.prototype = new Parent();

  var kid = new Child('Patrick')
  kid.name //'Patrick'
  kid.say() //'Patrick'
  delete.kid.name;
  kid.say() //'Adam'
  ```
