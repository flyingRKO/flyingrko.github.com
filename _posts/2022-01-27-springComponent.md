---
title:  "[Spring] 배워서 바로쓰는 스프링 프레임워크 6.2"
excerpt: "@Component에 대해 알아보자"

categories:
  - Spring
tags:
  - [Spring, Component]

toc: true
toc_sticky: true
 
date: 2022-01-27
last_modified_at: 2022-01-27
---

# 6.2 @Conponent - 스프링 빈 식별하기

---

- 타입 수준의 애너테이션
- 클래스가 스프링 빈(스프링 컴포넌트라고도 함)을 표현한다는 사실을 나타냄
    - **개발자가 직접 작성한 Class를 Bean으로 등록하기 위한 Annotation**이다.
- @Controller,  @Service, @Repository의 메타 애너테이션이 @Conponent
    - 메타 애너테이션이란.. : 애너테이션의 애너테이션
- 컨트롤러에는 @Controller, 서비스에는 @Service, DAO에는 @Repository를 설정하는데
- Spring에서 **@Component로 다 쓰지 않고 @Repository, @Service, @Controller등을 사용하는 이유**는, 예를들어 **@Repository는 DAO의 메소드에서 발생할 수 있는 unchecked exception들을 스프링의 DataAccessException으로 처리할 수 있기 때문- 담당일찐같은 느낌**

```java
package sample.spring.chapter06.bankapp.service;
import org.springframework.stereotype.Service;

@Service(value="fixedDepositService")// 이 부분 주목
@Qualifier("service")
public class FixedDepositServiceImpl implements FixedDepositService {...}
```

FixedDepositServiceImpl 클래스는 FixedDepositService 빈으로 스프링 컨테이너에 등록

value로 빈 이름을 지정! (@Controller,  @Service, @Repository, @Conponent도 이 방법임)

```java
@Service(value="fixedDepositService")// 이거나
@Service("fixedDepositService")// 이거나 둘 다 똑같음
```

위 코드말고도 빈 이름을 지정하지 않는 방법도 있다

이 경우 스프링은 클래스 이름에서 첫번째 글자를 소문자로 바꾼 이름을 빈 이름으로 사용함

스프링의 **클래스경로 스캐닝**을 활성화하면 스프링 컨테이너는 @Controller,  @Service, @Repository, @Conponent를 설정한 빈 클래스를 자동으로 등록함

활성화하려면 context 스키마에 <component-scan> 엘리먼트를 사용한다.

```java
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context 
        http://www.springframework.org/schema/context/spring-context.xsd">

	<context:component-scan base-package="sample.spring"/> // 이 부분
</beans>
```

<component-scan> 엘리먼트의 base-package 속성은 스프링 빈을 검색할 패키지를 콤마(,)로 나눠서??

나열한 목록이다.

base-package="sample.spring"이므로

sample.spring과 그 하위 패키지에서 스프링 빈을 검색한다.

예제 6-1의 FixedDepositServiceImpl 클래스에서 @Service를 설정하고, 패키지가 sample.spring.chapter06.bankapp.service이므로

sample.spring의 하위 패키지이다. 그래서 6-2의 <component-scan> 엘리먼트는 자동으로 6-1 클래스를 스프링 컨테이너에 자동으로 등록한다.

FixedDepositServiceImpl 클래스를 xml파일 안에서 다음과 같이 빈으로 정의하는 방법도 있다!

```java
<bean id= "fixedDepositService"
		class="sample.spring.chapter06.bankapp.service.FixedDepositServiceImpl"/>
```

스프링 컨테이너가 자동 등록 시 특정 빈 클래스를 걸러내고 싶다?

<component-scan> 엘리먼트의 resource-pattern속성을 사용해라

- resource-pattern속성의 디폴트 값은 **/*.class
    - base-package 속성으로 지정한 패키지 아래의 모든 클래스를 자동 등록 대상으로 한다는 뜻
- <component-scan> 엘리먼트의 하위 엘리먼트
    - <include-filter>
    - <exclude-filter>
- 이 두 가지를 사용하면 자동 등록시 사용할 컴포넌트 클래스와 제외할 컴포넌트 클래스를 간결하게 지정할 수 있다

```java
//<include-filter>와 <exclude-filter> 엘리먼트

<beans ....>
	<context:component-scan base-package="sample.example">
		<context:include-filter type="annotation" //include-filter
			expression="example.annotation.MyAnnotation"/>
		<context:exclude-filter type="regex" expression=".*Details"/> //exclude-filter
	</context:component-scan>
</beans>
```

- type은 빈 클래스를 걸러낼 때 사용할 전략을 지정한다
- expression은 걸러낼 때 사용할 식을 지정한다
    - include-filter는 MyAnnotation 타입 수준 애너테이션을 설정한 클래스를 자동 등록
    - exclude-filter는 Details로 끝나는 빈 클래스를 무시하도록 지정

<include-filter>와 <exclude-filter> 엘리먼트의 type 속성 값

| type 속성값 | 설명 |
| --- | --- |
| annotation | expression 속성을 빈 클래스에 설정한 애너테이션의 전체 클래스 이름을 지정한다 |
| assignable | expression 속성을 빈 클래스가 대입 가능한 클래스나 인터페이스 전체 이름을 지정한다 |
| aspectj | expression 속성을 빈 클래스를 걸러내기 위해 애스팩트J 식을 사용한다 |
| regex | expression 속성을 빈 클래스 이름을 걸러낼 때 사용할 정규식을 지정한다 |
| custom | 빈 클래스를 걸러내기 위해 or.springframework.core.type.TypeFillter 인터페이스에 대한 구현을 expression에 지정한다 |