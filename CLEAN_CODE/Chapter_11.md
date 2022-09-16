# Chapter 11 - 시스템

높은 추상화 수준, 즉 시스템 수준에서도 깨끗함을 유지하는 방법

## 시스템 제작과 시스템 사용을 분리하라

---

제작과 사요은 아주 다르다.

```java
public Service getService() {
	if (service == null)
		servide = new MyServiceImpl();
	return service;
```

이것은 초기화 지연 혹은 계산 지연이라는 기법이다. null 포인터를 어떤 경우에도 반환하지 않고, 필요할 때까지 객체를 생성하지 않는다는 장점이 있다.

그러나 생성자 인수에 명시적으로 getService 메서드가 의존하게 되고, 런타임 로직에서 MyServiceImpl 객체를 전혀 사용하지 않더라도 의존성을 해결하지 않으면 컴파일이 안된다. 또한 테스트도 의존성을 가지게 되고, 결론적으로 단일 책임 원칙을 깨게 된다.

### 생성 로직을 Main으로 분리

생성과 사용을 분리하는 가장 간단한 방법은 모든 생성과 관련된 로직을 main으로 옮기는 것이다. 어플리케이션에서는 사용할 모등 객체들이 main에서 잘 생성되었을 것이라 여기고 나머지 디자인에 집중할 수 있다.

### 팩토리

factory 객체를 만들어서 던져줄 수 있다. 만약 자세한 구현을 숨기고 싶다면 Abstract Factory 패턴을 사용

### 의존성 주입

의존성 주입은 제어 역전 기법을 의존성 관리에 적용한 메커니즘이다. 제어 역전에서는 한 객체가 맡은 보조 책임을 새로운 객체에게 전적으로 떠넘긴다. 새로운 객체는 넘겨받은 책임만 맡으므로 단일 책임 원칙을 지키게 된다.

## 확장

먼저, 관심사를 적절히 분리하지 못하는 아키텍처 예를 소개한다. 원래 EJB1과 EJB2 아키텍처는 관심사를 적절히 분리하지 못했기에 유기적인 성장이 어려웠다.

```java
package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public interface BankLocal extends java.ejb.EJBLocalObject {
    String getStreetAddr1() throws EJBException;
    String getStreetAddr2() throws EJBException;
    String getCity() throws EJBException;
    String getState() throws EJBException;
    String getZipCode() throws EJBException;
    void setStreetAddr1(String street1) throws EJBException;
    void setStreetAddr2(String street2) throws EJBException;
    void setCity(String city) throws EJBException;
    void setState(String state) throws EJBException;
    void setZipCode(String zip) throws EJBException;
    Collection getAccounts() throws EJBException;
    void setAccounts(Collection accounts) throws EJBException;
    void addAccount(AccountDTO accountDTO) throws EJBException;
}

package com.example.banking;
import java.util.Collections;
import javax.ejb.*;

public abstract class Bank implements javax.ejb.EntityBean {
    // Business logic...
    public abstract String getStreetAddr1();
    public abstract String getStreetAddr2();
    public abstract String getCity();
    public abstract String getState();
    public abstract String getZipCode();
    public abstract void setStreetAddr1(String street1);
    public abstract void setStreetAddr2(String street2);
    public abstract void setCity(String city);
    public abstract void setState(String state);
    public abstract void setZipCode(String zip);
    public abstract Collection getAccounts();
    public abstract void setAccounts(Collection accounts);

    public void addAccount(AccountDTO accountDTO) {
        InitialContext context = new InitialContext();
        AccountHomeLocal accountHome = context.lookup("AccountHomeLocal");
        AccountLocal account = accountHome.create(accountDTO);
        Collection accounts = getAccounts();
        accounts.add(account);
    }

    // EJB container logic
    public abstract void setId(Integer id);
    public abstract Integer getId();
    public Integer ejbCreate(Integer id) { ... }
    public void ejbPostCreate(Integer id) { ... }

    // The rest had to be implemented but were usually empty:
    public void setEntityContext(EntityContext ctx) {}
    public void unsetEntityContext() {}
    public void ejbActivate() {}
    public void ejbPassivate() {}
    public void ejbLoad() {}
    public void ejbStore() {}
    public void ejbRemove() {}
}
```

위와 같은 코드는 전형적인 EJB2 객체 구조의 코드이다.

- 비지니스 논리는 EJB2 애플리케이션 컨테이너에 강력하게 결합되어있다. 클래스를 생성할 때는 컨테이너에서 파생해야 하며 컨테이너가 요구하는 다양한 생명주기 메서드를 제공해야 한다.
- 이런 결합탓에 독자적인 단위 테스트가 어렵다.
- 결국 객체 지향 프로그래밍이라는 개념이 흔들리게된다.

### 횡단 관심사

횡단 관심사란 이론적으로는 독립된 형태로 구분될 수 있지만 실제로는 코드에 산재하기 쉬운 부분들을 뜻한다. ex) 트랜잭션, 보안, 영속정인 동작, 로깅

AOP에서 관점이라는 모듈 구성 개념은 `특정 관심사를 지원하려면 시스템에서 특정 지점들이 동작하는 방식을 일관성 있게 바꿔야한다`고 명시한다.

자바에서 사용하는 관점 혹은 관점과 유사한 메커니즘 세개를 살펴보자.

### 자바 프록시

자바 프록시는 단순한 상황에 적합하다. JDK에서 제공하는 동적 프록시는 인터페이스만 지원한다. 클래스 프록시를 이용하려면 바이트 코드 처리 라이브러리가 필요하다.

```java
// Bank.java
import java.utils.*;

// 은행 추상화
public interface Bank {
    Collection<Account> getAccounts();
    void setAccounts(Collection<Account> accounts);
}

// BankImpl.java
import java.utils.*;

// 추상화를 위한 POJO 구현
public class BankImpl implements Bank {
    private List<Account> accounts;

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = new ArrayList<Account>();
        for (Account account: accounts) {
            this.accounts.add(account);
        }
    }
}
// BankProxyHandler.java
import java.lang.reflect.*;
import java.util.*;

// 프록시 API가 필요한 “InvocationHandler”
public class BankProxyHandler implements InvocationHandler {
    private Bank bank;

    public BankHandler (Bank bank) {
        this.bank = bank;
    }

    // InvocationHandler에 정의된 메서드
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String methodName = method.getName();
        if (methodName.equals("getAccounts")) {
            bank.setAccounts(getAccountsFromDatabase());

            return bank.getAccounts();
        } else if (methodName.equals("setAccounts")) {
            bank.setAccounts((Collection<Account>) args[0]);
            setAccountsToDatabase(bank.getAccounts());

            return null;
        } else {
            ...
        }
    }

    // 세부사항은 여기에 이어진다
    protected Collection<Account> getAccountsFromDatabase() { ... }
    protected void setAccountsToDatabase(Collection<Account> accounts) { ... }
}

// 다른 곳에 위치하는 코드
Bank bank = (Bank) Proxy.newProxyInstance(
    Bank.class.getClassLoader(),
    new Class[] { Bank.class },
    new BankProxyHandler(new BankImpl())
);
```

위에서는 프록시로 감쌀 인터페이스 Bank와 비지니스 논리를 구현하는 POJO BankImpl을 정의했다.

프록시 API에는 InvocationHandler를 넘겨 줘야 한다. 넘긴 InvocationHandler는 프록시에 호출되는 Bank 메서드를 구현하는데 사용된다. BankProxyHandler는 자바 리플렉션 API를 사용해 제네릭스 메서드를 상응하는 BankImpl 메서드로 매핑한다.

### 자바 순수 AOP 프레임워크

위의 자바 프록시 API의 단점은 Spring, JBoss와 같은 순수 자바 AOP 프레임워크를 통해 해결할 수 있다.

```java
<beans>
    ...
    <bean id="appDataSource"
        class="org.apache.commons.dbcp.BasicDataSource"
        destroy-method="close"
        p:driverClassName="com.mysql.jdbc.Driver"
        p:url="jdbc:mysql://localhost:3306/mydb"
        p:username="me"/>

    <bean id="bankDataAccessObject"
        class="com.example.banking.persistence.BankDataAccessObject"
        p:dataSource-ref="appDataSource"/>

    <bean id="bank"
        class="com.example.banking.model.Bank"
        p:dataAccessObject-ref="bankDataAccessObject"/>
    ...
</beans>
```

각 ‘빈’은 중첩된 ‘아시아 인형’의 일부분 같다. Bank 도메인 객체는 DAO로 프록시되었으며, DTO 객체는 JDBC 드라이버 자료 소스로 프록시 되었다. 클라이언트는 Bank에 접근하고 있다고 생각하지만 사실은 가장 바깥의 BankDataSource에 접근하고 있는 것이다.

애플리케이션에서 DI 컨테이너에게 시스템 내 최상위 객체를 요청하려면 다음과 같은 코드가 필요하다.

```java
XmlBeanFactory bf = new XmlBeanFactory(new ClassPathResource("app.xml", getClass()));
Bank bank = (Bank) bf.getBean("bank");
```

스프링과 독립적이다. EJB2 시스템이 지녔던 강한 결합 문제가 사라진다.

```java
package com.example.banking.model;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Collection;

@Entity
@Table(name = "BANKS")
public class Bank implements java.io.Serializable {
    @Id @GeneratedValue(strategy=GenerationType.AUTO)
    private int id;

    @Embeddable // Bank의 데이터베이스 행에 '인라인으로 포함된' 객체
    public class Address {
        protected String streetAddr1;
        protected String streetAddr2;
        protected String city;
        protected String state;
        protected String zipCode;
    }

    @Embedded
    private Address address;
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER, mappedBy="bank")
    private Collection<Account> accounts = new ArrayList<Account>();
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void addAccount(Account account) {
        account.setBank(this);
        accounts.add(account);
    }

    public Collection<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(Collection<Account> accounts) {
        this.accounts = accounts;
    }
}
```

위와 같이 EJB2보다 훨씬 깨끗해졌다. 일부 상세한 엔티티 정보들은 annotation내에 정의되어 있지만 모든 정보가 annotation 안에 있으므로 이전보다 더 깨끗하고 테스트하고 개선하기 쉬워졌다.

### AspectJ

관심사를 분리하는 가장 관력한 도구는 AspectJ 언어다.

## 테스트 주도 시스템 아키텍쳐 구축

코드 수준에서 아키텍처 관심사를 분리할 수 있다면, 진정한 테스트 주도 아키텍처 구축이 가능해진다. 그때그때 새로운 기술을 채택해 단순 아키텍처를 복잡한 아키텍처로 키워갈 수 있다. 해로운 BEUF를 추구할 필요가 없다.

### 의사 결정을 최적화하라

모듈을 나누고 관심사를 분리하면 지엽적인 관리와 결정이 가능해진다. 결정은 최대한 정보를 모으고 최선의 결정을 위해 마지막 순간까지 미루는것도 좋은 선택이다.

### 명백한 가치가 있을 때 표준을 현명하게 사용하라

EJB2는 단지 표준이라는 이유만으로 많은 팀이 사용했다. 표준을 사용하면 아이디어와 컴포넌트를 재사용하기 쉽고, 여러 장점이 있다. 하지만 과장되게 포장된 표준에 집착할 이유는 없다.

### 시스템은 도메인 특화 언어가 필요하다

DSL(Domain-Specific Language)가 새롭게 조명받기 시작했다. DSL은 간단한 스크립트 언어나 표준 언어로 구현한 API를 가리킨다. 좋은 DSL은 도메인 개념과 그 개념을 구현한 코드 사이에 존재하는 ‘의사소통 간극’을 줄여준다.

## 결론

시스템은 역시 깨끗해야 한다. 깨끗하지 못한 아키텍처는 도메인 논리를 흐리며 기민성을 떨어트린다. 모든 추상화 단계에서 의도는 명확히 표현해야 한다. 그러면서 POJO를 작성하고 관점 혹은 관점과 유사한 메커니즘을 사용해 각 구현 관심사를 분리해야 한다.
