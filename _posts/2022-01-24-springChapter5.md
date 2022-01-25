---
title:  "[Spring] 배워서 바로쓰는 스링 프레임워크 5.3.5~5.4.1"
excerpt: ""

categories:
  - Spring
tags:
  - [Spring]

toc: true
toc_sticky: true
 
date: 2022-01-24
last_modified_at: 2022-01-24
---

# 5.3.5 DestructionAwareBeanPostProcessor

---

- 빈이 제거되기 전에 제거될 빈 인스턴스와 상호작용하고 싶은 경우가 있다
- DestructionAwareBeanPostProcessor 인터페이스를 구현한 빈을 xml 파일에서 설정해야한다
- DestructionAwareBeanPostProcessor는 BeanPostProcessor의 하위 인터페이스로 다음 메소드를 정의함

---

```java
void postProcessBeforeDestruction(Object bean, String beanName)
//Object bean => 요 자리는 스프링 컨테이너가 제거하려는 대상 빈 인스턴스
//String beanName => 요 자리는 제거하려는 대상 빈 이름
//요 두가지를 인수로 받음
```

- 스프링 컨테이너는 싱글턴 빈 인스턴스를 없애기 전에 위 메소드를 호출
- 일반적으로 이 메소드는 빈 인스턴스에 정의된 커스텀 정리 메서드를 호출
- 🔥 **if 프로토타입 빈..호출안됨ㅋ**

# 5.4 BeanFactoryPostProcessor를 사용해 빈 정의 변경하기

---

빈 정의를 변경하고 싶은 클래스가 있다?

→ 스프링 BeanFactoryPostProcessor 인터페이스를 구현한다

- 스프링 컨테이너가 빈 정의를 로드한 다음
- 빈 인스턴스를 만들기 전에 BeanFactoryPostProcessor 실행
- 요거슨 xml파일에 정의된 다른 모든 빈이 생성되기 전에 생성
- 그렇기 때문에 다른 빈의 정의를 변경할 기회가 주어짐
- 얘도 xml에서 정의하면 됨ㅋ

BeanFactoryPostProcessor 인터페이스에는..

 postProcessBeanFactory 메소드만 들어있다.

이 메소드는 ConfigurableListableBeanFactory타입 인수를 받는댄다.

요 인수로 스프링 컨테이너가 로드한 빈 정의를 얻어서 변경할 수 있다.

postProcessBeanFactory 메소드 안에

ConfigurableListableBeanFactory의 getBean 메소드를 호출해서

빈 인스턴스를 만들 수도 있지만,,,,,,,,

**postProcessBeanFactory 메소드 안에서**

**빈 만드는거 권장 안한다~ 하지마라~**

postProcessBeanFactory 메소드안에서 생성된 빈 인스턴스에 대해서는

BeanPostProcessors(5.3절)의 메소드들이 호출 안된다는 사실 잊지말자

### ConfigurableListableBeanFactory에 대하여..

---

- 🔥 **ApplicationContext와 마찬가지로 스프링 컨테이너에 접근하게 해준다**
- 스프링 컨테이너를 설정하고 모든 빈을 이터레이션하며,
- 빈 정의를 변경할 수 있게 해줌
- 예를들어..
    
    ConfigurableListableBeanFactory 객체를 사용하면
    
    PropertyEditorRegistrars 등록(3.6절), BeanPostProcessors 등록 등의 일을 할 수 있다
    

## 5.4.1 BeanFactoryPostProcessor 예제

---

빈을 자동 연결에 사용하지 못하게 막고(4.6절 <bean>엘리먼트의 autowire-candidate 속성)

싱글턴 빈이 프로토타입 빈에 의존하는 경우

오류메시지를 로그에 남기는 BeanFactoryPostProcessor 구현을 살펴보자

<aside>
💡 스프링 BeanFactoryPostProcessor 인터페이스를 구현하는 빈 → 특별한 유형의 빈

</aside>

<aside>
💡 스프링 컨테이너는 BeanFactoryPostProcessor 빈을 자동으로 감지해서 실행해줌ㅎㅎ

</aside>

예제 5-22

MyBank에서 BeanFactoryPostProcessor 인터페이스를 구현하는 ApplicationConfigurer 클래스

```java
package sample.spring.chapter05.bankapp.postprocessor;

import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;
import org.springframework.beans.BeansException;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.PropertyValue;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.config.RuntimeBeanReference;

public class ApplicationConfigurer implements BeanFactoryPostProcessor {
	private static Logger logger = LogManager
			.getLogger(ApplicationConfigurer.class);
	
	public ApplicationConfigurer() {
		logger.info("Created ApplicationConfigurer instance");
	}
	
	@Override
	public void postProcessBeanFactory( //요 메소드
			ConfigurableListableBeanFactory beanFactory) throws BeansException {
		String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames(); //1.
		// -- 모든 빈 정의를 얻는다
		for (int i = 0; i < beanDefinitionNames.length; i++) {
			String beanName = beanDefinitionNames[i];
			BeanDefinition beanDefinition = beanFactory //3.에서 말하는 객체
					.getBeanDefinition(beanName); //2.
			beanDefinition.setAutowireCandidate(false); //3.
			// -- 빈의 의존 관계를 얻는다
			if (beanDefinition.isSingleton()) { //4.
				if (hasPrototypeDependency(beanFactory, beanDefinition)) {
					logger.error("Singleton-scoped " + beanName
							+ " bean is dependent on a prototype-scoped bean.");
				}
			}
		}
	}

	.....
}
```

postProcessBeanFactory 메소드 작동 순서

1. ConfigurableListableBeanFactory의 getBeanDefinitionNames메소드 호출해서 스프링 컨테이너가 로드한 모든 빈 정의의 이름을 얻는다. 여기서 빈 정의의 이름은 <bean>엘리먼트의 id속성임
2. 모든 빈 정의의 이름을 얻으면 postProcessBeanFactory 메소드 ConfigurableListableBeanFactory의 getBeanDefinition 메소드를 호출해서 각 빈 정의에 해당하는 빈정의를 얻는다. getBeanDefinition 메소드는 1.에서 얻은 빈 정의 이름을 인수로 받는다
3. BeanDefinition 객체는 빈 정의를 표현, 이 객체를 변경하면 빈 설정을 변경할 수 있다. postProcessBeanFactory 메소드는 스프링 컨테이너가 로드한 모든 빈 정의에 대해 BeanDefinition의 setAutowireCandidate를 호출해서 모든 빈이 자동 연결 대상으로 되지 않게 한다.
4. BeanDefinition의 isSingleton 메소드는 해당 빈 정의가 싱글턴 빈의 빈 정의인 경우 true를 반환한다. 빈 정의가 싱글턴 빈 정의면  hasPrototypeDependency 메소드를 호출해서 **싱글턴 빈이 다른 프로토타입 빈에 의존하는지 검사한다.** 싱글턴 빈이 프로토타입 빈에 의존하면 로그에 오류 메시지를 남긴다.

예제 5-23

ApplicationConfigurer 클래스의 hasPrototypeDependency 메소드

```java
public class ApplicationConfigurer implements BeanFactoryPostProcessor {
.....

private boolean hasPrototypeDependency(
			ConfigurableListableBeanFactory beanFactory,
			BeanDefinition beanDefinition) {
		boolean isPrototype = false;
		MutablePropertyValues mutablePropertyValues = beanDefinition
				.getPropertyValues();  //1.
		PropertyValue[] propertyValues = mutablePropertyValues //2.
				.getPropertyValues();
		for (int j = 0; j < propertyValues.length; j++) {
			if (propertyValues[j].getValue() instanceof RuntimeBeanReference) { //3.
				String dependencyBeanName = ((RuntimeBeanReference) propertyValues[j]
						.getValue()).getBeanName();
				BeanDefinition dependencyBeanDef = beanFactory //4.
						.getBeanDefinition(dependencyBeanName);
				if (dependencyBeanDef.isPrototype()) { //5.
					isPrototype = true;
					break;
				}
			}
		}
		return isPrototype;
	}
}
```

hasPrototypeDependency 메소드는 BeanDefinition 인수가 표현하는 빈이 **프로토타입 빈에 의존하는지 검사한다.**

ConfigurableListableBeanFactory 인수는 스프링 컨테이너가 로드한 빈 정의에 접근할 수 있게 해준다.

hasPrototypeDependency 메소드 작동 순서

1. BeanDefinition의 getPropertyValues 메소드 호출해서 <property>엘리먼트에 정의된 프로퍼티를 얻는다.  왜냐? BeanDefinition의 getPropertyValues는 MutablePropertyValues타입의 객체를 반환하는데, 이 객체를 변경하면 빈 프로퍼티를 변경할 수 있다.  예를 들어.. MutablePropertyValues의 addPropertyValue나 addPropertyValues를 사용하면 빈 정의에 프로퍼티를 추가 할 수 있다.
2. 모든 빈 프로퍼티에 대해 이터레이션하면서 프로토타입 빈을 참조하는 빈 프로퍼티가 있는지 찾기 위해 MutablePropertyValue의 getPropertyValues 메소드를 호출해서 PropertyValue 객체로 이뤄진 배열을 얻는다. PropertyValues 객체에는 빈 프로퍼티 정보가 담겨 있다.
3. 빈 프로퍼티가 스프링 빈을 가리키면 PropertyValue의 getValue 메소드를 호출해서 프로퍼티가 참조하는 빈의 이름이 들어있는 RuntimeBeanReference 객체를 얻는다. getValue 메소드가 반환하는 값이 RuntimeBeanReference 타입의 인스턴스인지 살펴보고 만약 맞다면 getValue가 반환한 객체를 RuntimeBeanReference 타입으로 타입 변환하고 RuntimeBeanReference의 getBeanName 메소드를 호출해서 참조 대상 빈의 이름을 얻는다.
4. 빈 프로퍼티가 참조하는 빈의 이름을 얻었으므로, ConfigurableListableBeanFactory의 getBeanDefinition을 호출하면 빈 참조에 해당하는 BeanDefinition 객체를 얻을 수 있다.
5. BeanDefinition의 isPrototype을 호출하면 참조되는 빈이 프로토타입 빈인지를 알 수 있다.

⭐참고하십쇼
![시퀀스 다이어그램](/assets/images/스프링 5-2찐.jpeg)

예제 5-24

applicationContext.xml - BeanFactoryPostProcessor 빈 정의

```java
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

	<bean ....>
	....

	<bean id="fixedDepositDao" //요거랑
		class="sample.spring.chapter05.bankapp.dao.FixedDepositDaoImpl"
		init-method="initializeDbConnection" destroy-method="releaseDbConnection">
		<property name="fixedDepositDetails" ref="fixedDepositDetails" />
	</bean>

	<bean id="fixedDepositDetails"//요거 봐라
		class="sample.spring.chapter05.bankapp.domain.FixedDepositDetails"
		scope="prototype" />

	<bean
		class="sample.spring.chapter05.bankapp.postprocessor.InstanceValidationBeanPostProcessor">
		<property name="order" value="1" />
	</bean>

	<bean
		class="sample.spring.chapter05.bankapp.postprocessor.ApplicationConfigurer" />
</beans>
```

fixedDepositDao 싱글턴 빈은 fixedDepositDetails 프로토타입 빈에 의존한다.

BankApp 클래스의 main 메서드를 실행하면 콘솔창에 다음 출력을 볼 수 있다.

```
Created **ApplicationConfigurer** instance
Singleton-scoped **fixedDepositDao** bean is dependent on a prototype-scoped bean.
Created **InstanceValidationBeanPostProcessor** instance
```

스프링 컨테이너가 ApplicationConfigurer를 만들고, 

InstanceValidationBeanPostProcessor 인스턴스를 생성하기전에

ApplicationConfigurer의 postProcessBeanFactory 메소드를 실행하는 것을 볼 수 있다.

🔥 **BeanFactoryPostProcessor 인터페이스를 구현하는 빈이** 

    **BeanPostProcessor 인터페이스를 구현하는 빈보다 먼저 처리된다!**

이로 인해 BeanPostProcessor를 사용해서 BeanFactoryPostProcessor 인스턴스를 변경할수 없다..

그니까..

---

- BeanFactoryPostProcessor를 사용하면 스프링 컨테이너가 로드한 빈 정의를 변경할 수 있다.
- BeanPostProcessor를 사용하면 새로 생성된 빈 인스턴스를 변경할 수 있다.

두 사이의 유사점으로는...

---

- xml 파일에서 여러 BeanFactoryPostProcessor를 설정할 수 있다.
    
    → 스프링 컨테이너가 BeanFactoryPostProcessor 실행 순서를 제어하기 위해 스프링 Ordered 인터페이스를 구현한다.(5.3절)
    
- 스프링 컨테이너에 BeanFactoryPostProcessor를 지연 초기화라고 지정해도, 스프링 인스턴스가 만들어질 때 BeanFactoryPostProcessor도 함께 만들어진다.