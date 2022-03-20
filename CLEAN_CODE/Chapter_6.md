# Chapter 6 - 객체와 자료구조

---

변수를 비공개(private)로 정의하는 이유가 있다. 남들이 변수에 의존하지 않게 만들고 싶어서다. 그렇다면 어째서 수많은 프로그래머가 조회(get) 함수와 설정(set) 함수를 당연하게 공개(public)해 비공개 변수를 외부에 노출할까??

## 자료 추상화

---

아래의 두 클래스는 모두 2차원 점을 표현한다. 그런데 한 클래스는 구현을 외부로 노출하고 다른 클래스는 구현을 완전히 숨긴다.

```java
// 6-1 구체적인 Point 클래스
public class Point {
	public double x;
	public double y;
}

// 6-2 추상적인 Point 클래스
public interface Point {
	double getX();
	double getY();
	void setCartesian(double x, double y);
	double getR();
	double getTheta();
	void setPolar(double r, double theta);
}
```

6-2는 클래스 메서드가 접근 정책을 강제한다. 좌표를 읽을 때는 각 값을 개별로 읽어야 하며, 좌표를 설정할 때는 두 값을 한꺼번에 설정해야 한다. 반면 6-1은 구현을 노출하고, 개별적으로 좌표값을 읽고 설정하게 강제한다. 변수를 private로 설정하더라도 각 값마다 조회(get) 함수와 설정(set) 함수를 제공한다면 구현을 외부로 노출하는 셈이다.

변수 사이에 함수라는 게층을 넣는다고 구현이 감춰지는 것은 아니다. 추상화가 필요하다. 추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스이다.

## 자료/객체 비대칭

---

객체는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다. 자료구조는 자료를 그대로 공개하며 별다른 함수를 제공하지 않는다. 두 정의는 본질적으로 상반된다.

```java
// 6-3 절차적인 도형
public class Square {
  public Point topLeft;
  public double side;
}

public class Rectangle {
  public Point topLeft;
  public double height;
  public double width;
}

public class Circle {
  public Point center;
  public double radius;
}

public class Geometry {
  public final double PI = 3.141592653589793;

  public double area(Object shape) throws NoSuchShapeException {
    if (shape instanceof Square) {
      Square s = (Square)shape;
      return s.side * s.side;
    } else if (shape instanceof Rectangle) {
      Rectangle r = (Rectangle)shape;
      return r.height * r.width;
    } else if (shape instanceof Circle) {
      Circle c = (Circle)shape;
      return PI * c.radius * c.radius;
    }
    throw new NoSuchShapeException();
  }
}

// 6-4 다형적인 도형
public class Square implements Shape {
  private Point topLeft;
  private double side;

  public double area() {
    return side * side;
  }
}

public class Rectangle implements Shape {
  private Point topLeft;
  private double height;
  private double width;

  public double area() {
    return height * width;
  }
}

public class Circle implements Shape {
  private Point center;
  private double radius;
  public final double PI = 3.141592653589793;

  public double area() {
    return PI * radius * radius;
  }
}
```

6-3에서 만약 Geometry 클래스에 둘래 길이를 구하는 perimeter() 함수를 추가하고 싶다면 도형 클래스는 아무 영향도 받지 않는다. 물론 도형 클래스에 의존하는 다른 클래스도 마찬가지다. 반대로 새 도형을 추가하고 싶다면 Geometry 클래스에 속한 함수를 모두 고쳐야 한다. 앞서와 마찬가지로 두 조건은 완전히 정반대이다.

반면에 6-4 에서는 새 도형을 추가해도 기존 함수에는 아무런 영향을 미치지 않는다. 반면 새 함수를 추가하고 싶다면 도형 클래스를 전부 고쳐야 한다.

앞서 말했듯이, 6-3과 6-4는 상호 보완적인 특질이 있다. 사실상 반대다! 그래서 객체와 자료구조는 근본적으로 양분된다.

> (자료구조를 사용하는) 절차적인 코드는 기존 자료 구조를 변경하지 않으면서 새 함수를 추가하기 쉽다. 반면, 객체 지향 코드는 기존 함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

반대쪽도 참이다.

> 절차적인 코드는 새로운 자료 구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야 한다. 객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야 한다.

다시 말해, 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉬우며, 절차적인 코드에서 어려운 변경은 객체 지향 코드에서 쉽다!

복잡한 시스템을 짜다 보면 새로운 함수가 아니라 새로운 자료 타입이 필요한 경우가 생긴다. 이때는 **클래스와 객체 지향 기법**이 가장 적합하다. 반면, 새로운 자료 타입이 아니라 새로운 함수가 필요한 경우도 생긴다. 이때는 **절차적인 코드와 자료구조**가 좀 더 적합하다.

분별 있는 프로그래머는 모든 것이 객체라는 생각이 미신임을 잘 안다. 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

## 디미터 법칙

---

디미터 법칙이란 모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다는 법칙이다. 객체는 조회 함수로 내부 구조를 공개하면 안 된다는 의미다. 그러면 내부 구조를 노출하는 셈이니까.

정확히 표현하자면, 디미터 법칙은 “클래스 C의 메서드 f는 다음과 같은 객체의 메서드만 호출해야 한다"고 주장한다.

- 클래스 C
- f가 생성한 객체
- f 인수로 넘어온 객체
- C 인스턴스 변수에 저장된 객체

### 기차 충돌

---

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

위의 코드는 디미터 법칙을 어기는 듯이 보인다. 흔히 위와 같은 코드를 기차 충돌이라고 부른다. 일반적으로 조잡하다 여겨지는 방식으로 피하는 것이 좋다. 위 코드는 다음과 같이 나누는 편이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

위 예제가 디미터 법칙을 위반하는지 여부는 ctxt, Options, ScratchDir이 객체인지 아닌지 자료 구조인지에 달렸다. 객체라면 내부 구조를 숨겨야 하므로 확실히 디미터 법칙을 위반한다. 반면, 자료 구조라면 당연히 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.

### 잡종 구조

---

때로는 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다. 잡종 구조는 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다. 이러한 잡종 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다. 양쪽 세상에서 단점만 모아놓은 구조다. 그러므로 잡종 구조는 최대한 피하는 것이 좋다.

### 구조체 감추기

---

만약 ctxt, options, scrathDir이 진짜 객체라면 위의 예제처럼 줄줄이 사탕으로 엮으면 안된다. 위의 예제 한참 아래로 내려가면 이런 코드가 있다.

```java
String outFile = outputDir + "/" + className.replace('.', '/') + ".class";
FileOutputStream fout = new FileOutputStream(outFile);
BufferedOutputStream bos = new BufferedOutputStream(fout);
```

추상화 수준을 뒤섞어 놓아 다소 불편하다. 파일 확장자, 점, 슬래시, File 객체를 부주의하게 뒤섞어 놓았다. 어찌 되었거나, 임시 디렉터리의 절대 경로를 얻으려는 이유가 임시 파일을 생성하기 위한 목적이라는 사실이 드러났다.

그렇다면 ctxt 객체에 임시 파일을 생성하려고 시키면 어떨까?

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

ctxt는 내부 구조를 드러내지 않고, 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다. 따라서 디미터 법칙을 위반하지 않는다.

## 자료 전달 객체

---

자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스이다. 이런 자료 구조체를 때로는 자료 전달 객체(DTO)라고 한다.

```java
public class Address {
  public String street;
  public String streetExtra;
  public String city;
  public String state;
  public String zip;
}
```

DTO는 특히 데이터베이스와 통신하거나 소켓에서 받은 메시지의 구문을 분석할 때 유용하다.

좀더 일반적인 형태는 빈(bean) 구조다. 빈은 비공개 변수를 조회/설정 함수로 조작한다.

```java
public class Address {
  public String street;
  public String streetExtra;
  public String city;
  public String state;
  public String zip;

	public Address(String street, String streetExtra,
			String city, String state, String zip) {
		this.street = street;
		this.streetExtra = streetExtra;
		this.city = city;
		this.state = state;
		this.zip = zip;
	}

	public String getStreet() {
		return street;
	}

	...
}
```

### 활성 레코드

---

활성 레코드는 DTO의 특수한 형태이다. 공개 변수가 있거나 비공개 변수에 조회/설정 함수가 있는 자료 구조지만, 대개 save나 find와 같은 탐색 함수도 제공한다. 활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.

불행히도 활성 레코드에 비지니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔하다. 하지만 이렇게 하면 잡종 구조가 된다.

해결책은 단순하다. 활성 레코드는 자료 구조로 취급하면 된다. 비즈니스 규칙을 담으면서 내부 구조를 숨기는 객체는 따로 생성한다.

## 결론

---

객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.

자료 구조는 별다른 동작 없이 자료를 노축한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.
