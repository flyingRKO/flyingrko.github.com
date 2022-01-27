---
title:  "[Spring] 배워서 바로쓰는 스프링 프레임워크 6.3"
excerpt: "@Autowired에 대해 알아보자"

categories:
  - Spring
tags:
  - [Spring, Autowired]

toc: true
toc_sticky: true
 
date: 2022-01-27
last_modified_at: 2022-01-27
---

# 6.3 @Autowired - 객체의 타입으로 의존 관계 자동 연결하기

---

- 속성(field), setter method, constructor(생성자)에서 사용하며 **Type에 따라 알아서 Bean을 주입 해준다.**
- 무조건적인 객체에 대한 의존성을 주입시킨다.
- **이 애노테이션을 사용할 시, 스프링이 자동적으로 값을 할당**한다.
- **Controller 클래스에서 DAO나 Service에 관한 객체들을 주입 시킬 때 많이 사용**한다.
- 필드, 생성자, 입력 파라미터가 여러 개인 메소드(@Qualifier는 메소드의 파라미터)에 적용 가능하다.
- **Type을 먼저 확인한 후 못 찾으면 Name에 따라 주입**한다.
- *Name으로 강제하는 방법: @Qualifier을 같이 명시*

예제 6-5

```java
package sample.spring.chapter06.bankapp.service;

import java.util.Date;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import sample.spring.chapter06.bankapp.dao.AccountStatementDao;
import sample.spring.chapter06.bankapp.domain.AccountStatement;

@Service(value="accountStatementService")
@Qualifier("service")
public class AccountStatementServiceImpl implements AccountStatementService {
	
	@Autowired //이 부분
	private AccountStatementDao accountStatementDao;
	
	@Override
	public AccountStatement getAccountStatement(Date from, Date to) {
		return accountStatementDao.getAccountStatement(from, to);
	}
}
```

- BeanPostProcessor를 구현한 AutowiredAnnotationBeanPostProcessor가 스프링 컨테이너에서 AccountStatementDao 타입 빈을 얻어서 accountStatementDao 필드에 자동으로 대입한다
- @Autowired를 설정한 필드나 필드에 대응하는 세터 메서드가 꼭 public일 필요는 없다

예제 6-6

```java
package sample.spring.chapter06.bankapp.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Service;

import sample.spring.chapter06.bankapp.dao.CustomerRegistrationDao;
import sample.spring.chapter06.bankapp.domain.CustomerRegistrationDetails;

**@Service("customerRegistrationService")**
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Qualifier("service")
public class CustomerRegistrationServiceImpl implements
		CustomerRegistrationService {

	private CustomerRegistrationDetails customerRegistrationDetails;

	@Autowired
	private CustomerRegistrationDao customerRegistrationDao;

	**@Autowired**
	public void obtainCustomerRegistrationDetails(   // 여기 이 부분
			**CustomerRegistrationDetails customerRegistrationDetails**) {
		**this.customerRegistrationDetails = customerRegistrationDetails**;
	}

	public CustomerRegistrationDetails getCustomerRegistrationDetails() {
		return customerRegistrationDetails;
	}

	public CustomerRegistrationDao getCustomerRegistrationDao() {
		return customerRegistrationDao;
	}

	@Override
	public void setAccountNumber(String accountNumber) {
		**customerRegistrationDetails.setAccountNumber(accountNumber)**; // 여기도
	}

	@Override
	public void setAddress(String address) {
		customerRegistrationDetails.setAddress(address);
	}

	@Override
	public void setDebitCardNumber(String cardNumber) {
		customerRegistrationDetails.setCardNumber(cardNumber);
	}

	@Override
	public void register() {
		customerRegistrationDao.registerCustomer(customerRegistrationDetails);
	}

}
```

- 메서드에 @Autowired를 설정하면 메서드 인수가 자동 연결된다
- 여기서 @Autowired를 설정한 메서드가 public일 필요는 없다

예제 6-7

```java
package sample.spring.chapter06.bankapp.service;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Service;

import sample.spring.chapter06.bankapp.dao.CustomerRequestDao;
import sample.spring.chapter06.bankapp.domain.CustomerRequestDetails;

**@Service(value = "customerRequestService")**
@Qualifier("service")
public class CustomerRequestServiceImpl implements CustomerRequestService {
	private static Logger logger = LogManager
			.getLogger(CustomerRequestServiceImpl.class);
	private CustomerRequestDetails customerRequestDetails;
	private CustomerRequestDao customerRequestDao;

	@Autowired
	public CustomerRequestServiceImpl(
			**CustomerRequestDetails customerRequestDetails, // 이부분
			CustomerRequestDao customerRequestDao**) {
		logger.info("Created CustomerRequestServiceImpl instance");
		this.customerRequestDetails = customerRequestDetails;
		this.customerRequestDao = customerRequestDao;
	}

	@Override
	public void submitRequest(String requestType, String requestDescription) {
		// -- create an instance of UserRequestDetails and save it
		customerRequestDao.submitRequest(customerRequestDetails);
	}

}
```

- 이 예제는 @Autowired를 이 클래스의 생성자에 설정한다
- 생성자 인수가 자동 연결된다
- **여기서도 생성자가 public일 필요는 없다**

<aside>
💡 스프링 4.3부터 빈 클래스에 생성자가 단 하나만 있는 경우, 유일한 생성자에 @Autowired할 필요 없다. 스프링 컨테이너는 이런 유일한 생성자에 대한 자동 연결을 수행함

</aside>

만약 @Autowired를 사용하여 자동연결에 필요한 타입과 일치한 빈이 **없다면** 예외가 발생한다

@Autowired의 required 속성은 의존관계가 필수적인지 여부를 지정한다

- required 값이 false면 의존관계가 선택적인 것으로 간주.
    - 이는 필요한 타입을 스프링 컨테이너 안에서 찾을 수 없어도 예외 발생 안한다는 뜻
- required 속성의 디폴트는 true. 따라서 의존 관계를 **반드시** 만족시켜야함
- required 속성이 true면 다른 생성자에 @Autowired할 수 없다.

6-8 

```java
**@Service(value = "customerRequestService")**
@Qualifier("service")
public class CustomerRequestServiceImpl implements CustomerRequestService {
	private static Logger logger = LogManager
			.getLogger(CustomerRequestServiceImpl.class);
	private CustomerRequestDetails customerRequestDetails;
	private CustomerRequestDao customerRequestDao;

	@Autowire(required=false)
	public CustomerRequestServiceImpl(
			**CustomerRequestDetails customerRequestDetails**) {...}

	@Autowire
		public CustomerRequestServiceImpl(
				**CustomerRequestDetails customerRequestDetails, 
				CustomerRequestDao customerRequestDao**) {...}
		
	}

```

생성자가 2개 있는 빈 클래스의 경우 

두 생성자 중 @Autowire(required=false)는 의존관계 자동 연결이 선택적이고

다른 생성자는 디폴트로 true여서 의존 관계 자동 연결이 필수이므로

스프링은 예외를 발생

6-9

```java
**@Service(value = "customerRequestService")**
@Qualifier("service")
public class CustomerRequestServiceImpl implements CustomerRequestService {
	private static Logger logger = LogManager
			.getLogger(CustomerRequestServiceImpl.class);
	private CustomerRequestDetails customerRequestDetails;
	private CustomerRequestDao customerRequestDao;

	@Autowire(required=false)
	public CustomerRequestServiceImpl(
			**CustomerRequestDetails customerRequestDetails** 
			) {...}

	@Autowire(required=false)
		public CustomerRequestServiceImpl(
				**CustomerRequestDetails customerRequestDetails, 
				CustomerRequestDao customerRequestDao**) {...}
		
	}
```

빈 클래스에 required 값이 false인 @Autowired 애너테이션을 설정한 생성자가 여럿 있어도 된다!

위 예제에서 두 생성자 모두 CustomerRequestServiceImpl 인스턴스를 생성 할 때 호출할 수 있는 후보로 간주한다!

만족하는 의존 관계 개수가 가장 큰 생성자를 선택!

두 의존 관계를 모두 찾을 수 없는 경우 디폴트 생성자를 호출한다