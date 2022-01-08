---
title:  "[java] 접근 제어 지시자(access modifier)와 정보은닉(infomation hiding)"
excerpt: "public, private, protected는 어떨 때 쓰고 getter와 setter메소드는 어떨 때 어떻게 쓰는 건지 알아보자."

categories:
  - 객체지향 프로그래밍
tags:
  - [public, private, getter, setter]

toc: true
toc_sticky: true
 
date: 2022-01-07
last_modified_at: 2022-01-07
---


## 접근 제어 지시자 (accesss modifier)

- 클래스 외부에서 클래스의 멤버 변수, 메서드, 생성자를 사용할 수 있는지 여부를 지정하는 키워드

- private : 같은 클래스 내부에서만 접근 가능 ( 외부 클래스, 상속 관계의 클래스에서도 접근 불가)

- 아무것도 안 쓸 경우 (default) : 같은 패키지 내부에서만 접근 가능 ( 상속 관계라도 패키지가 다르면 접근 불가)

- protected : 같은 패키지나 상속관계의 클래스에서 접근 가능하고 그 외 외부에서는 접근 할 수 없음

- public : 클래스의 외부 어디서나 접근 할 수 있음


## get()/ set() 메서드

- private 으로 선언된 멤버 변수 (필드)에 대해 접근, 수정할 수 있는 메서드를 public으로 제공

- get() 메서드만 제공 되는 경우 read-only 필드 말그대로 그냥 띄워서 읽게만 해주는거



### 뭔가 글만봐서는 감이 안 온다.. 예시를 들어보자
~~~
public class BirthDay {  
 private int day;  
 private int month;  
 private int year;  
  
 private boolean isValid;  
  
 public int getDay() {  
        return day;  
  }  
  
  public void setDay(int day) { 
	    this.day = day;  
    }  
  
  public int getMonth() {  
        return month;  
  }  
  
  public void setMonth(int month) {  
      if(month < 1 || month >12){  
           isValid = false;  
  }  
      else {  
            isValid = true;  
			this.month = month;  
		  }  
  }  
  
    public int getYear() {  
        return year;  
  }  
  
    public void setYear(int year) {  
        this.year = year;  
  }
~~~
위의 코드는 BirthDay 클래스를 만들어서 연도, 월, 일 변수의 접근 제어를  private로 설정하고 getter와 setter 메소드를 설정한 것이다.

그렇다면 이렇게 설정하면 뭐가 좋아서 하는 걸까?

getter와 setter메소드로 해당 클래스의 private 변수에 접근하는 것이 오류도 적고 디버깅도 쉽기 때문이다.

만약 BirthDay클래스에서 변수를 아래와 같이 private로 지정하지 않는다면..
~~~
int day;  
int month;  
int year;
~~~

main클래스에서 다음과 같은 일이 일어날 수 있다.
~~~
public class Main {  
  
public static void main(String[] args) {  
  BirthDay date = new BirthDay();  
  date.setYear(2022);  
  date.setMonth(13);  
  date.setDay(30);  
  date.month = 100;   
  }  
}
~~~
위 코드에서 setter 메소드를 통해 BirthDay클래스의 변수를 설정하고 있다.

여기서 문제의 코드를 보자

`date.month = 100;`

이게 뭐가 문제냐? 라고 한다면 먼저 BirthDay클래스의 setMonth메소드를 보자

~~~
public void setMonth(int month) {  
	if(month < 1 || month >12){  
          isValid = false;  
  }  
    else {  
          isValid = true;  
		  this.month = month;  
		  }  
}  
~~~
기껏 setter메소드로 변수 month의 범위를 정했는데 이걸 쌩까고 바꿔버렸다. 

이런 상황을 방지하기 위해 외부 클래스에서 지맘대로 접근하지 못하도록 변수를 private로 선언하는 것이다.

## 정보 은닉 

- private으로 제어한 멤버 변수도 public 메서드가 제공되면 접근 가능하지만 
- 변수가 public으로 공개되었을 때보다 private 일때 각 변수에 대한 제한을 public 메서드에서  잘 제어 할 수 있다.
- 정보 은닉이라기 보다는 정보 보호가 더 와 닿는다.
- 이렇게 접근 제어 지시자(public, private 등)을 사용해서 외부에서 접근 가능한 최소한의 정보를 오픈함으로써 객체의 오류를 방지하고 클라이언트 객체가 더 효율적으로 객체를 쓸 수 있게 해준다.


