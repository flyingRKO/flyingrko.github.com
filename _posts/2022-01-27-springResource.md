---
title:  "[Spring] 배워서 바로쓰는 스프링 프레임워크 6.6"
excerpt: "@Resource에 대해 알아보자"

categories:
  - Spring
tags:
  - [Spring, Resource]

toc: true
toc_sticky: true
 
date: 2022-01-27
last_modified_at: 2022-01-27
---

# 6.6 JSR 250 @Resource 애너테이션

---

- 필드와 세터 메서드를 **빈 이름으로** 자동 연결함
- CommonAnnotationBeanPostProcessor가 @Resource 애너테이션 처리
- @Resource에는 name속성이 있는데 자동 연결한 빈 이름을 지정함
- 생성자 인수, 여러 인수 받는 메서드 인수를 자동연결할 경우 @Resource 사용 불가

6-21

```java
package sample.spring.chapter06.bankapp.service;

import javax.annotation.Resource;

@Named(value="fixedDepositService")
public class FixedDepositServiceImpl implements FixedDepositService {
	
	@Resource(name="myFixedDepositDao")
	private FixedDepositDao myFixedDepositDao;
	
	...
}
```

6-19(@Inject와 @Named쓴거)를 @Resource써서 다시 작성한 것

**빈 이름으로** 자동 연결해야 하는 경우 @Autowired와 @Qualifier 대신 @Resource써야함!

만약 @Autowired와 @Qualifier을 써서 빈 이름으로 자동 연결한다? 번거롭다

스프링이 필드타입과 일치하며 자동 연결할수있는 빈 찾고 @Qualifier로 지정한 이름을 사용해 후보를 반 줄여서 블라블라..

@Resource는 name속성으로 단 하나의 빈을 찾음ㅎ

@Resource쓰면 스프링은 필드타입 고려 하지 않음

빈 자체가 Map 타입이면 @Autowired 작동 안함..

이럴때 @Resource써서 자동연결한다

6-22

```java
@Named(value="mybean")
public class MyService{

	@Resource
	private myDao myDao; //이게 이 @Resource의 name 값 (필드이름이니까)
	private SomeService service;

	@Resource
	public void setOtherService(SomeService service) {
		//이 @Resource name 값은 OtherService(프로퍼티 이름)
		this.service = service;
}
}
```

@Resource는 name 속성 지정하지않으면 필드나 프로퍼티 이름을 디폴트로 사용함

빈 이름으로 못 찾으면 타입으로 찾음