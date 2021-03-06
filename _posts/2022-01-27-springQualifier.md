---
title:  "[Spring] 배워서 바로쓰는 스프링 프레임워크 6.4.1~"
excerpt: "@Qualifier에 대해 알아보자"

categories:
  - Spring
tags:
  - [Spring, Qualifier]

toc: true
toc_sticky: true
 
date: 2022-01-27
last_modified_at: 2022-01-27
---

# 6.4 @Qualifier-빈 이름으로 의존 관계 자동 연결하기

---

- **@Autowired와 같이 쓰이며**, **같은 타입의 Bean 객체가 있을 때 해당 아이디를 적어 원하는 Bean이 주입될 수 있도록 하는 Annotation**이다. *같은 타입이 존재하는 경우 ex) 동물 = 원숭이, 닭, 개, 돼지*
- **같은 타입의 Bean이 두 개 이상이 존재하는 경우**에 **스프링이 어떤 Bean을 주입해야 할지 알 수 없어서 Spring Container를 초기화하는 과정에서 예외를 발생**시킨다.
- **이 경우 @Qualifier을 @Autowired와 함께 사용**하여 **정확히 어떤 bean을 사용할지 지정하여 특정 의존 객체를 주입할 수 있도록 한다.**

6-10

```java
package sample.spring.chapter06.bankapp.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import sample.spring.chapter06.bankapp.dao.FixedDepositDao;
import sample.spring.chapter06.bankapp.domain.FixedDepositDetails;

@Service(value="fixedDepositService")
@Qualifier("service")
public class FixedDepositServiceImpl implements FixedDepositService {
	
	@Autowired
	**@Qualifier(value="myFixedDepositDao") // value= 이거 안 써도 됨**
	private FixedDepositDao myFixedDepositDao;
	
	@Override
	public void createFixedDeposit(FixedDepositDetails fdd) throws Exception {
		// -- create fixed deposit
		myFixedDepositDao.createFixedDeposit(fdd);
	}
}
```

스프링은 먼저 @Autowired를 설정한 필드, 생성자 인수, 메서드 인수의 객체 타입으로 후보 빈을 찾는다

그 후, **@Qualifier**를 사용해 자동 연결 후보에서 유일한 빈을 구별한다

6-11

```java
public class Sample{
	
	@Autowired
	public Sample(@Qualifier("aBean") ABean bean){...}

	@Autowired
	public void doSomething(@Qualifier("bBean") BBean bean, CBean cBean){...}
```

이 예제에서는 @Qualifier를 생성자 인수와 메서드 인수에 설정한다

- 위 클래스를 생성할때 스프링은 aBean타입의 aBean 빈을 찾아서 생성자에게 넘겨준다
- doSomething 메서드를 호출할때 스프링은 BBean타입의 이름은 bBean인 빈과 CBean 타입의 빈을 찾아서 메서드 인수로 넘긴다
    - BBean 의존관계는 **빈 이름으로** 자동 연결되지만 CBean 의존 관계는 객체 타입으로 자동 연결된다

## 6.4.1 지정자를 사용해 빈 자동 연결하기

---

지정자란?

- @Qualifier를 사용해 빈을 연결할 때 사용하는 문자열
- 스프링 컨테이너에 등록된 빈 사이에서 유일할 필요는 없다

6-12

```java
package sample.spring.chapter06.bankapp.dao;

...

@Repository(value = "txDao")
@Qualifier("myTx")
public class TxDaoImpl implements TxDao {

	...
}
```

이 예제는 @Qualifier이용해서 빈에 지정자를 연관시키는 방법이다

- myTx는 txDao의 지정자
- @Repository로 지정한 txDao는 빈 이름

6-13

```java
package sample.spring.chapter06.bankapp.service;

...

@Service("txService")
@Qualifier("service")
public class TxServiceImpl implements TxService {
	@Autowired
	@Qualifier("myTx")
	private TxDao txDao;

	...
```

지정자 값을 사용해 txDao 빈을 자동 연결하는 방법이다

@Qualifier는 txDao를 txDao 빈에 연결하는 대신, 지정자 값이 myTx인 빈에 연결하라고 지정

6-14

```java
package sample.spring.chapter06.bankapp.service;

...

@Component
public class Services {
	@Autowired
	@Qualifier("service")
	private Set<MyService> services;

	...
```

타입 지정 컬렉션으로 지정자와 연관된 모든 빈을 자동연결하는 방법이다

- Set<MyService> 컬렉션은 MyService 인터페이스를 구현하는 모든 서비스를 표현
- @Qualifier는 지정자값과 연관된 모든 빈을 컬렉션에 넣어준다
- 같은 프로젝트에서 MyService 마커 인터페이스를 구현하는 모든 서비스에는 @Qualifier(”service”)애너테이션을 설정한다
- 이렇게 이 프로젝트에 정의된 모든 서비스는 Set<MyService>에 자동 연결

## 6.4.2 커스텀 지정자 애너테이션 만들기

---

![6-1](/assets/images/스프링 그림 6-1.jpeg)

- FundTransferProcessor는 이체 요청을 처리
- 수신 계좌가 같은 은행에 있는지, 자금 이체가 즉시 시행되어야 하는지에 따라 적절한 인터페이스 구현을 사용

6-15

```java
package sample.spring.chapter06.bankapp.annotation;

...

import org.springframework.beans.factory.annotation.Qualifier;

@Target({ ElementType.FIELD, ElementType.PARAMETER, ElementType.TYPE, ElementType.ANNOTATION_TYPE })
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface FundTransfer {
	TransferSpeed transferSpeed();

	BankType bankType();
}
```

TransferSpeed와 bankType 속성을 정의하는 @FundTransfer 커스텀 지정자 애너테이션

- 메타 애너테이션으로 @Qualifier가 지정됨
    - 이는 @FundTransfer가 커스텀 지정자 애너테이션이라는 뜻
    - 만약 @Qualifier로 지정안하면 CustomAutowiredConfigurer빈을 사용해 명시적으로 등록해야함
- @FundTransfer 목적은 @Qualifier와 같음
- TransferSpeed와 bankType 속성에 따라 빈을 자동연결함

6-16

```java
package sample.spring.chapter06.bankapp.service;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.stereotype.Service;

import sample.spring.chapter06.bankapp.annotation.BankType;
import sample.spring.chapter06.bankapp.annotation.FundTransfer;
import sample.spring.chapter06.bankapp.annotation.TransferSpeed;
import sample.spring.chapter06.bankapp.domain.Account;

@Service
@FundTransfer(transferSpeed = TransferSpeed.IMMEDIATE, bankType=BankType.SAME)
public class ImmediateSameBank implements FundTransferService {
	...}
```

이 ImmediateSameBank 클래스에서는 transferSpeed = TransferSpeed.IMMEDIATE 이고

bankType=BankType.SAME 인 @FundTransfer가 붙어있다

다른 빈 클래스 구현도 비슷함

6-17

```java
package sample.spring.chapter06.bankapp.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import sample.spring.chapter06.bankapp.annotation.BankType;
import sample.spring.chapter06.bankapp.annotation.FundTransfer;
import sample.spring.chapter06.bankapp.annotation.TransferSpeed;
import sample.spring.chapter06.bankapp.domain.Account;

@Component
public class FundTransferProcessor {
	@Autowired
	@FundTransfer(transferSpeed = TransferSpeed.IMMEDIATE, bankType = BankType.SAME)
	private FundTransferService sameBankImmediateFundTransferService;
	
	@Autowired
	@FundTransfer(transferSpeed = TransferSpeed.IMMEDIATE, bankType = BankType.DIFFERENT)
	private FundTransferService diffBankImmediateFundTransferService;

	public void sameBankImmediateFundTransferService() {
		sameBankImmediateFundTransferService.transferFunds(new Account(), new Account());
	}
	
	public void diffBankImmediateFundTransferService() {
		diffBankImmediateFundTransferService.transferFunds(new Account(), new Account());
	}
	
}
```

@FundTransfer 애노테이션에 맞게 인스턴스와 필드를 자동 연결해준다

(ImmediateSameBank인스턴스는 sameBankImmediateFundTransferService에

ImmediateDifferBank인스턴스는 diffBankImmediateFundTransferService)

6-18

```java
<bean class="org.springframework.beans.factory.annotation.CustomAutowireConfigurer">
	<property name="customQualifierTypes">//스프링 컨테이너에 등록할 커스텀 지정자 애너테이션을 받음
		<set>
			<value>sample.MyCustomQualifier</value>
		</set>
	</property>
</bean>
```

커스텀 지정자 애너테이션에 @Qualifier를 메타 애너테이션으로 지정하지 않았을 때 xml에 위와 같이 직접 등록해야함