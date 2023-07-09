---
layout: post
title: Iterator 패턴
description: >
  2023.07.09 디자인 패턴 스터디 - FactoryMethod Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---

## 목차

[FactoryMethod 패턴이란?](#FactoryMethod-패턴이란?)

[예제 FactoryMethod 클래스 다이어그램](#예제-FactoryMethod-클래스-다이어그램)

[예제 코드 구현](#예제-코드-구현)
- [Product 클래스](#Product-클래스)
- [Factory 클래스](#Factory-클래스)
- [IDCard 클래스](#IDCard-클래스)
- [IDCardFactory 클래스](#IDCardFactory-클래스)
- [Main 클래스](#Main-클래스)

[FactoryMethod 패턴의 클래스 다이어그램](#FactoryMethod-패턴의-클래스-다이어그램)

[장점](#장점)

## FactoryMethod 패턴이란

- 객체를 생성할 때 어떤 클래스의 인스턴스를 만들 지 서브 클래스에서 결정하는 패턴

## 예제 FactoryMethod 클래스 다이어그램

![facorymethod-Page-1 drawio](https://github.com/inh2613/inh2613.github.io/assets/62206617/167069b2-fe0a-4031-9cf3-6e7110dc63f9)


## 예제 코드 구현

- ### Product 클래스
```java
public abstract class Product {
	public abstract void use();
}

```
- ### Factory 클래스
```java
public abstract class Factory {
	public final Product create(String owner) {
		Product p = createProduct(owner);
		registerProduct(p);
		return p;
	}
	protected abstract Product createProduct(String owner);
	protected abstract void registerProduct(Product product);
}
```
- ### IDCard 클래스
```java
public class IDCard extends Product{
	private String owner;
	IDCard(String owner){
		System.out.println(owner + "의 카드를 만듭니다.");
		this.owner = owner;
	}
	public void use(){
		System.out.println(owner + "을 사용합니다.");
	}
	public String getOwner(){
		return owner;
	}
	@Override
	public String toString(){
		return "[IDCard:"+owner+"]";
	}
}
```
- ### IDCardFactory 클래스
```java
public class IDCardFactory extends Factory{
	@Override
	protected Product createProduct(String owner){
		return new IDCard(owner);
	}
	@Override
	protected void registerProduct(Product product){
		System.out.println(product+ "을 등록합니다.");
	}
}
```
- ### Main 클래스
```java
public class Main {
	public static void main(String[] args) {
		Factory factory = new IDCardFactory();
		Product card1 = factory.create("홍길동");
		Product card2 = factory.create("이순신");
		Product card3 = factory.create("강감찬");
		card1.use();
		card2.use();
		card3.use();
	}
}
```
## FactoryMethod 패턴의 클래스 다이어그램
![facorymethod-페이지-2 drawio](https://github.com/inh2613/inh2613.github.io/assets/62206617/818ca7cd-1da8-43a9-8fac-8722c5d8f538)

- **Product**
    - 생성되는 인스턴스가 가져야 할 인터페이스를 결정하는 추상 클래스
    - Product 클래스
- **Creator**
    - Product 역할을 생성하는 추상 클래스
    - new를 사용해 실제 인스턴스를 생성하지 않고, 인스턴스를 생성하는 메소드를 호출
    - 구체적인 클래스 이름에 의한 속박으로부터 상위 클래스를 자유롭게 함
    - Factory 클래스
- **ConcreteProduct**
    - 구체적인 제품 결정
    - IDCard 클래스
- **ConcreteCreator**
    - 구체적인 제품을 만들 클래스 결정
    - IDCardFactory 클래스

## 장점
- framework 패키지 안에 제품 페키지를 import 하지 않기 때문에 수정이 필요 없음

## 참고
- Java 언어로 배우는 디자인 패턴 입문(유키 히로시)