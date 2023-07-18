---
layout: post
title: Builder 패턴
description: >
  2023.07.18 디자인 패턴 스터디 - Builder Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---

## 목차

[Builder 패턴이란?](#Builder-패턴이란?)
- 

[예제 Builder 클래스 다이어그램](#예제-Builder-클래스-다이어그램)

[예제 코드 구현](#예제-코드-구현)
- [Builder 클래스](#Builder-클래스)
- [Director 클래스](#Director-클래스)
- [TextBuilder 클래스](#TextBuilder-클래스)
- [HTMLBuilder 클래스](#HTMLBuilder-클래스)
- [Main 클래스](#Main-클래스)

[Builder 패턴의 클래스 다이어그램](#Builder-패턴의-클래스-다이어그램)

[Builder 패턴의 시퀀스 다이어그램](#Builder-패턴의-시퀀스-다이어그램)


[장점](#장점)

[의존성 주입](#의존성-주입)

## Builder 패턴이란
- 객체를 생성하는 클래스와 표현하는 클래스를 분리하여, 동일한 절차에서도 서로 다른 표 현을 생성하는 방법 제공

## 예제 Builder 패턴 클래스 다이어 그램

![1](https://github.com/inh2613/inh2613.github.io/assets/62206617/dc8436a5-c27b-45da-a950-b28114e4bdc5)


## 예제 코드 구현

### Builder 클래스

```java
public abstract class Builder {
	public abstract void makeTitle(String title);
	public abstract void makeString(String str);
	public abstract void makeItems(String[] items);
	public abstract void close();
}
```

### Director 클래스

```java
public class Director {
	private Builder builder;

	public Director(Builder builder) {
		this.builder = builder;
	}

	public void construct() {
		builder.makeTitle("Greeting");
		builder.makeString("일반적인 인사");
		builder.makeItems(new String[]{
				"How are you?",
				"Good morning",
				"Good afternoon"
		});
		builder.makeString("시간대별 인사");
		builder.makeItems(new String[]{
				"Good evening",
				"Good night",
				"Goodbye"
		});
		builder.close();
	}
}
```

### TextBuilder 클래스

```java
public class TextBuilder extends Builder{
	private StringBuilder sb = new StringBuilder();
	public void makeTitle(String title) {
		sb.append("==============================\n");
		sb.append("『" + title + "』\n");
		sb.append("\n");
	}
	public void makeString(String str) {
		sb.append('■' + str + "\n");
		sb.append("\n");
	}
	public void makeItems(String[] items) {
		for (int i = 0; i < items.length; i++) {
			sb.append("　・" + items[i] + "\n");
		}
		sb.append("\n");
	}
	public void close() {
		sb.append("==============================\n");
	}
	public String getResult() {
		return sb.toString();
	}
}

```

### HTMLBuilder 클래스

```java
import java.io.FileWriter;
import java.io.Writer;

public class HTMLBuilder extends Builder{
	private String filename="untitled.html";
	private StringBuilder sb = new StringBuilder();

	@Override
	public void makeTitle(String title){
		filename = title + ".html";
		sb.append("<DOCTYPE html>\n");
		sb.append("<html lang=\"ko\">\n");
		sb.append("<head><title>\n");
		sb.append(title);
		sb.append("</title></head>\n");
		sb.append("<body>\n");
		sb.append("<h1>");
		sb.append(title);
		sb.append("</h1>\n");
	}
	@Override
	public void makeString(String str){
		sb.append("<p>");
		sb.append(str);
		sb.append("</p>\n");
	}
	@Override
	public void makeItems(String[] items){
		sb.append("<ul>\n");
		for(int i=0; i<items.length; i++){
			sb.append("<li>");
			sb.append(items[i]);
			sb.append("</li>\n");
		}
		sb.append("</ul>\n");
	}
	@Override
	public void close(){
		sb.append("</body>\n");
		sb.append("</html>\n");
		try{
			Writer writer=new FileWriter(filename);
			writer.write(sb.toString());
			writer.close();
		}catch (Exception e){
			e.printStackTrace();
		}
	}

	public String getHTMLResult(){
		return filename;
	}
}
```


### Main 클래스

```java
public class Main {
	public static void main(String[] args) {
		if(args.length != 1){
			usage();
			System.exit(0);
		}
		if(args[0].equals("text")){
			TextBuilder textBuilder = new TextBuilder();
			Director director = new Director(textBuilder);
			director.construct();
			String result = textBuilder.getResult();
			System.out.println(result);
		}else if(args[0].equals("html")){
			HTMLBuilder htmlBuilder = new HTMLBuilder();
			Director director = new Director(htmlBuilder);
			director.construct();
			String filename = htmlBuilder.getHTMLResult();
			System.out.println(filename + "가 작성되었습니다.");
		}else{
			usage();
			System.exit(0);
		}
	}
	public static void usage(){
		System.out.println("Usage: java Main text 일반 텍스트로 문서작성");
		System.out.println("Usage: java Main html HTML 파일로 문서작성");
	}
}
```


## Builder 패턴의 클래스 다이어 그램

![2](https://github.com/inh2613/inh2613.github.io/assets/62206617/2309c94d-4a55-4f16-aac2-70a5f45fec27)

- **Builder**
    - 인스턴스의 각 부분을 만드는 메소드를 결정하는 추상 클래스
    - Builder 클래스
- **ConcreteBuilder**
    - 추상 클래스 Builder 구현
    - TextBuilder, HTMLBuilder
- **Director**
    - ConcreteBuilder에 의존하는 프로그래밍은 X(Builder의 메소드만 사용)
    - Director
- **Client**
    - Builder 패턴 이용
    - Main


## Builder 패턴의 시퀀스 다이어 그램

![시퀀스 drawio](https://github.com/inh2613/inh2613.github.io/assets/62206617/405c3510-389e-420f-bfa0-615cc3b5c87e)

## 장점
- Director 클래스는 Builder 클래스의 하위 클래스를 **호출하지 않아 교체에 용이 하다**

## 의존성 주입
- Director 가 동작하려면 Builder의 구체적인 인스턴스가 필요하다
    - `이 인스턴스를 이용해 동작해 주세요` 라는 의미의 의존성 주입 `Main.java`에서 나타난다.