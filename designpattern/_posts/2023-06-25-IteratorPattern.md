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

[Iterator 패턴이란?](#iterator-패턴이란?)

[예제 Iterator 클래스 다이어그램](#예제-iterator-클래스-다이어그램)

[예제 코드 구현](#예제-코드-구현)
- [Aggregate 인터페이스](#aggregate-인터페이스)
- [Iterator 인터페이스](#iterator-인터페이스)
- [Book 클래스](#book-클래스)
- [BookShelf 클래스](#bookshelf-클래스)
- [BookShelfIterator 클래스](#bookshelfiterator-클래스)
- [Main 클래스](#main-클래스)

[이상적인 Iterator 클래스 다이어그램](#이상적인-iterator-클래스-다이어그램)

[존재 이유](#존재-이유)

---

## Iterator 패턴이란

-  무엇인가 많이 모여있는 것들을 순서대로 지정하면서 전체를 검색하는 처리를 실행하기 위한 것
- **하나씩 나열하는 구조가 Aggregate 역할의 외부에 놓여있다**는 특징을 가지고 있음

## 예제 Iterator 클래스 다이어그램

![iterator-Page-1 drawio (2)](https://github.com/inh2613/inh2613.github.io/assets/62206617/e4c30062-f4b3-4b0c-b4ba-bd60ab280806)

## 예제 코드 구현

- ### Aggregate 인터페이스
  - 요소들이 나열되어 있는 **집합체**를 나타냄 
  - Aggregate 인터페이스를 구현하는 클래스는 배열처럼 뭔가가 많이 모여있음 (배열, ArrayList, Vector 등)
  - 집합체에 대응하는 Iterator 1개를 작성하기 위해 iterator 메소드 하나만 선언되어 있음
    ```java
    public interface Aggregate {
        Iterator iterator();
    }
    ```
- ### Iterator 인터페이스
    - 요소를 하나씩 나열하면서 루프 변수와 같은 역할 수행
  ```java
  public interface Iterator {
      boolean hasNext();
      Object next();
  }
  ```
- ### Book 클래스
  - 배열의 요소(책)을 나타내는 클래스
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
  - Aggregate 인터페이스를 구현
  - Book의 배열인 books를 가지고 있음
  - iterator는 BookShelf 클래스에 대응하는 Iterator로 BookShelfIterator라는 클래스의 인스턴스를 생성해서 반환함
  ```java
  public class BookShelf implements Aggregate {
      private Book[] books;
      private int last = 0;
  
      public BookShelf(int maxSize) {
          this.books = new Book[maxSize];
      }
  
      public Book getByIndex(int index) {
          return books[index];
      }
  
      public void appendBook(Book book) {
          this.books[last] = book;
          last++;
  
      }
      public int getLength() {
          return last;
      }
      
      public Iterator iterator() {
          return new BookShelfIterator(this);
      }
  }
  ```


- ### BookShelfIterator 클래스
  - Iterator 인터페이스 구현
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
          Book book = bookShelf.getByIndex(index);
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

![iterator-페이지-2 drawio (1)](https://github.com/inh2613/inh2613.github.io/assets/62206617/fa79c041-8458-4a91-be29-28c3116c9ca4)

- **Iterator**
  - 요소를 순서대로 검색해가는 인터페이스 결정

- **ConcreteIterator(구체적인 반복자)**
  - Iterator를 실제로 구현
  - 예제의 `BookShelfIterator`
  
- **Aggregate**
  - Iterator 역할을 만들어내는 인터페이스 결정
  
- **ConcreteAggregate(구체적인 집합체)**
  - Aggregate를 실제로 구현
  - ConcreteIterator역할의 인스턴스 생성 역할
  - 예제의 `BookShelf`


## 존재 이유

만약 구현한 사람이 배열을 사용하지 않고, `java.util.Vector`을 사용하려고 할 때(Aggregate의 종류를 바꾸려고 할 때) `BookShelf`만 수정하여 손쉽게 바꿀 수 있다. 

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
    public Book getByIndex(int index){
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

## 이상적인 Iterator 클래스 다이어그램

![iterator-페이지-2 drawio (1)](https://github.com/inh2613/inh2613.github.io/assets/62206617/fa79c041-8458-4a91-be29-28c3116c9ca4)

## 존재 이유

- for 문이 있는데 굳이 왜 사용할까?
  -  구현과 분리해서 하나씩 셀수 있기 때문
  - 사용자가 BookShelf를 수정하더라도 Main의 while문 동작에 영향을 주지 않음
      ```java
      Iterator it = bookShelf.iterator();
            while (it.hasNext()) {
                Book book = (Book) it.next();
                System.out.println(book.toString());
            }
     ```
    **클래스의 재이용화를 용이하게 할 수 있음**

## 참고
- Java 언어로 배우는 디자인 패턴 입문(유키 히로시)