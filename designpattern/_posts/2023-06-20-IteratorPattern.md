---
layout: post
title: Iterator 패턴
description: >
  2023.06.25 디자인 패턴 스터디 - Iterator Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---

## 목차

[Iterator 패턴이란?](##Iterator-패턴이란?)

[예제 Iterator 클래스 다이어그램](##예제-Iterator-클래스-다이어그램)

[예제 코드 구현](##예제-코드-구현)
- [Aggregate 인터페이스](###Aggregate-인터페이스)
- [Iterator 인터페이스](###Iterator-인터페이스)
- [Book 클래스](###Book-클래스)
- [BookShelf 클래스](###BookShelf-클래스)
- [BookShelfIterator 클래스](###BookShelfIterator-클래스)
- [Main 클래스](###Main-클래스)

[이상적인 Iterator 클래스 다이어그램](##이상적인-Iterator-클래스-다이어그램)

[존재 이유](##존재-이유)

[느낀점](##느낀점)


---

## Iterator 패턴이란?

## 예제 Iterator 클래스 다이어그램


## 예제 코드 구현

- ### Aggregate 인터페이스
```java
public interface Aggregate {
    Iterator iterator();
}
```


- ### Iterator 인터페이스
```java
public interface Iterator {
    boolean hasNext();
    Object next();
}
```


- ### Book 클래스
```java
public class Book {
    private String name;

    public Book(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "Book{" +
                "name='" + name + '\'' +
                '}';
    }
}
```


- ### BookShelf 클래스
```java
public class BookShelf implements Aggregate {
    private Book[] books;
    private int last = 0;

    public BookShelf(int maxSize) {
        this.books = new Book[maxSize];
    }

    public Book getByindex(int index) {
        return books[index];
    }

    public void appendBook(Book book) {
        this.books[last] = book;
        last++;

    }
    public int getLength() {
        return last;
    }

    @Override
    public Iterator iterator() {
        return new BookShelfIterator(this);
    }
}
```


- ### BookShelfIterator 클래스
```java
public class BookShelfIterator implements Iterator{
    private BookShelf bookShelf;
    private int index;

    public BookShelfIterator(BookShelf bookShelf) {
        this.bookShelf = bookShelf;
        this.index = 0;
    }

    @Override
    public boolean hasNext() {
        if (index<bookShelf.getLength()){
            return true;
        }else{
            return false;
        }

    }
    @Override
    public Object next() {
        Book book = bookShelf.getByindex(index);
        index++;
        return book;
    }
}
```


- ### Main 클래스
```java
public class Main {
    public static void main(String[] args) {
        BookShelf bookShelf = new BookShelf(4);
        bookShelf.appendBook(new Book("hello-1"));
        bookShelf.appendBook(new Book("hello-2"));
        bookShelf.appendBook(new Book("hello-3"));
        bookShelf.appendBook(new Book("hello-4"));

        Iterator it = bookShelf.iterator();
        while (it.hasNext()) {
            Book book = (Book) it.next();
            System.out.println(book.toString());
        }
    }
}
```


## 이상적인 Iterator 클래스 다이어그램

## 존재 이유
만약 구현한 사람이 배열을 사용하지 않고, java.util.Vector을 사용하려고 할 때(Aggregate를 바꾸려고 할 때) `BookShelf`만 수정하여 손쉽게 바꿀 수 있다. 

```java
import java.util.Vector;

public class BookShelf implements Aggregate {
    private Vector books;

    public BookShelf() {
        this.books = new Vector();

    }
    public void appendBook(Book book){
        books.add(book);
    }
    public Book getByindex(int index){
        return (Book)books.get(index);
    }

    public int getLength(){
        return books.size();
    }
    @Override
    public Iterator iterator() {
        return new BookShelfIterator(this);
    }
}
```
## 느낀점

## 참고
- Java 언어로 배우는 디자인 패턴 입문(유키 히로시)