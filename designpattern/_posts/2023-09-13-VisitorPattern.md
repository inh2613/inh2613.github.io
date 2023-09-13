---
layout: post
title: Visitor 패턴
description: >
  2023.09.13 디자인 패턴 스터디 - Visitor Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---

## 목차

- [Visitor 패턴이란?](#Visitor-패턴이란?)
- [예제 Visitor 클래스 다이어그램](#예제-Visitor-클래스-다이어그램)
- [예제 코드 구현](#예제-코드-구현)
    - [Visitor 클래스](#Visitor-클래스)

    - [Element 인터페이스](#Element-인터페이스)

    - [Entry 클래스](#Entry-클래스)

    - [File 클래스](#File-클래스)

    - [Directory 클래스](#Directory-클래스)

    - [ListVistor 클래스](#ListVistor-클래스)

    - [Main 클래스](#main-클래스)

- [예제 Visitor 시퀀스 다이어그램](#예제-Visitor-시퀀스-다이어그램)
- [Visitor 패턴의 클래스 다이어그램](#Visitor-패턴의-클래스-다이어그램)
- [존재 이유](#존재-이유)


## Visitor 패턴이란

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/ad10adbd-cb5b-4ef2-9edf-6ed5d0196c80)
- 데이터 구조와 처리를 분리한다

## 예제 Visitor 패턴 클래스 다이어 그램
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/28d153c5-7c86-4a26-b78a-e0c453a97794)

## 예제 코드 구현

### Visitor 클래스
```java
public abstract class Visitor {
	public abstract void visit(File file);
	public abstract void visit(Directory directory); //오버로드
}

```
### Element 클래스
```java
public interface Element {
	public abstract void accept(Visitor v);
}

```
### Entry 클래스
```java
public abstract class Entry implements Element{
	public abstract  String getName();
	public abstract  int getSize();

	@Override
	public String toString() {
		return getName() + " (" + getSize() + ")";
	}
}

```
### File 클래스
```java
public class File extends Entry{
	private String name;
	private int size;

	public File(String name, int size) {
		this.name = name;
		this.size = size;
	}

	@Override
	public String getName() { return name; }
	@Override
	public int getSize() { return size; }
	@Override
	public void accept(Visitor v) { v.visit(this); }
}

```
### Directory 클래스
```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class Directory extends Entry implements Iterable<Entry>{
	private String name;
	private List<Entry> directory = new ArrayList<>();

	public Directory(String name) {
		this.name = name;
	}
	@Override
	public String getName() { return name; }

	@Override
	public int getSize() {
		int size = 0;
		for (Entry entry : directory) {
			size += entry.getSize();
		}
		return size;
	}
	public Entry add(Entry entry) {
		directory.add(entry);
		return this;
	}

	@Override
	public Iterator<Entry> iterator(){
		return directory.iterator();
	}

	@Override
	public void accept(Visitor v) { v.visit(this); }

}

```
### ListVisitor 클래스
```java
public class ListVisitor extends Visitor{
	private String currentdir = "";

	@Override
	public void visit(File file) {
		System.out.println(currentdir + "/" + file);
	}

	@Override
	public void visit(Directory directory){
		System.out.println(currentdir + "/" + directory);
		String savedir = currentdir;
		currentdir = currentdir + "/" + directory.getName();
		for (Entry entry : directory) {
			entry.accept(this);
		}
		currentdir = savedir;
	}
}

```
### Main 클래스
```java
public class Main {
	public static void main(String[] args) {
		System.out.println("Making root entries...");
		Directory rootdir = new Directory("root");
		Directory bindir = new Directory("bin");
		Directory tmpdir = new Directory("tmp");
		Directory usrdir = new Directory("usr");

		rootdir.add(bindir);
		rootdir.add(tmpdir);
		rootdir.add(usrdir);

		bindir.add(new File("vi", 10000));
		bindir.add(new File("latex", 20000));
		rootdir.accept(new ListVisitor());
		System.out.println();

		System.out.println("Making user entries...");
		Directory yuki = new Directory("yuki");
		Directory hanako = new Directory("hanako");
		Directory tomura = new Directory("tomura");
		usrdir.add(yuki);
		usrdir.add(hanako);
		usrdir.add(tomura);
		yuki.add(new File("diary.html", 100));
		yuki.add(new File("Composite.java", 200));
		hanako.add(new File("memo.tex", 300));
		tomura.add(new File("game.doc", 400));
		rootdir.accept(new ListVisitor());
	}
}
```
### 예제 Visitor 시퀀스 다이어그램
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/adcd0b1a-aaf5-4fad-86bb-7ce9f804340d)

### Visitor 패턴의 클래스 다이어 그램
![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/fa259798-232e-4403-b34a-3a9aa6bc27ac)

## 존재 이유

- 데이터 구조와 처리를 같이하게 되면, 데이터 구조 파일에 접근하여 계속 수정해야 한다. -> 확장에는 열고 수정에는 닫아야 하는 원칙을 지킬 수 없다!(개방 폐쇄의 원칙)

---