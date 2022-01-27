---
title:  "[Spring] 배워서 바로쓰는 스프링 프레임워크 6.5"
excerpt: "@Inject와 @Named에 대해 알아보자"

categories:
  - Spring
tags:
  - [Spring, Inject, Named]

toc: true
toc_sticky: true
 
date: 2022-01-27
last_modified_at: 2022-01-27
---

# 6.5 JSR 330 @Inject와 @Named 애너테이션

---

JSR 330은 @Inject와 @Named 애너테이션을 지원한다

6-19

```java
package sample.spring.chapter06.bankapp.service;

import javax.inject.Inject;
import javax.inject.Named;

import sample.spring.chapter06.bankapp.dao.FixedDepositDao;
import sample.spring.chapter06.bankapp.domain.FixedDepositDetails;

@Named(value="fixedDepositService")
public class FixedDepositServiceImpl implements FixedDepositService {
	
	@Inject
	@Named(value="myFixedDepositDao")
	private FixedDepositDao myFixedDepositDao;
	
	...
}
```

- @Service와 @Qualifier 대신 @Named
- @Autowired 대신 @Inject ( 둘의 뜻이 같다. 객체 타입으로 의존 관계를 자동으로 연결)
- 작동방식이 같다
- <component-scan>엘리먼트도 @Named를 @Component 취급한다

@Inject와 @Named 애너테이션을 사용하려면 JSR 330 JAR 파일을 프로젝트에 포함해야함

```java
<dependency>
			<groupId>javax.inject</groupId>
			<artifactId>javax.inject</artifactId>
			<version>1</version>
		</dependency>
```

pom.xml 파일에 위 코드 추가

## 6.5.1 자바 8 Optional 타입

---

스프링은 Optional 타입인 필드, 생성자 인수, 메서드 파리미터를 자동 연결 가능

@Autowired의 required를 false로 설정하면 의존 관계가 **선택적**이 된다

@Inject에는 @Autowired의 required속성같은 부분은 없다!

하지만 자바 8의 Optional타입을 사용한다면?

똑같은 기능 수행가능

6-20

```java
import java.util.Optional
...

@Named(value="myService")
public class MyService{

@Inject
private Optional<ExternalService> externalServiceHolder;

public void doSomething(Data data){
	if(externalServiceHolder.isPresent()) {
		//- 외부 서비스를 사용해 데이터를 저장
		externalServiceHolder.get().save(data);
}
	else{
//--데이터를 로컬에 저장
		saveLocally(data);
}
}

private void saveLocally(Data data){
...}
```

위 클래스는 ExternalService빈을 사용해 데이터를 원격에 저장하는 빈을 표현함

ExternalService빈을 찾을 수 없으면 데이터를 로컬에 저장

ExternalService가 선택적 의존관계( 의존관계가 없을 수도 있는)이므로

ExternalService의 의존관계를 Optional<ExternalService>타입의 externalServiceHolder필드를 사용

Optional 타입은 null이 아니어야함

스프링이 ExternalService빈을 찾으면 externalServiceHolder에 저장

isPresent()메서드는 

- 값이 null이면 false
- 값이 있으면 true를 반환

true 반환하면 get()으로 안에 저장된 ExternalService 인스턴스를 가져옴

그후 save메서드로 원격 저장

false면 로컬에 저장