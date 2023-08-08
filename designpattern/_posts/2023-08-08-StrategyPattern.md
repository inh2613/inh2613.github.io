---
layout: post
title: Strategy 패턴
description: >
  2023.08.08 디자인 패턴 스터디 - Strategy Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---

## 목차

[Strategy 패턴이란?](#Strategy-패턴이란?)

[예제 Strategy 클래스 다이어그램](#예제-Strategy-클래스-다이어그램)

[예제 코드 구현](#예제-코드-구현)
- [Hand 클래스](#Hand-클래스)
- [Strategy 클래스](#Strategy-클래스)
- [WinningStrategy 클래스](#WinningStrategy-클래스)
- [ProbStrategy 클래스](#ProbStrategy-클래스)
- [Player 클래스](#Player-클래스)
- [Main 클래스](#Main-클래스)

[Strategy 패턴의 클래스 다이어그램](#Strategy-패턴의-클래스-다이어그램)


[장점](#장점)

## Strategy 패턴이란

- 알고리즘과 메소드를 분리
- 실행 중에 알고리즘을 선택할 수 있게 하는 행위 디자인 패턴


## 예제 Strategy 패턴 클래스 다이어 그램

![strategy-Page-1 drawio](https://github.com/inh2613/inh2613.github.io/assets/62206617/bc403e0a-f8ee-40f6-9082-a8840b5dd80b)


## 예제 코드 구현

### Hand 클래스

```java
public enum Hand {
	ROCK("바위",0),
	SCISSORS("가위",1),
	PAPER("보",2);

	private String name;
	private int handValue;
	private static Hand[] hands = {ROCK,SCISSORS,PAPER};

	Hand(String name, int handValue) {
		this.name = name;
		this.handValue = handValue;
	}
	public static Hand getHand(int handValue) {
		return hands[handValue];
	}
	public boolean isStrongerThan(Hand h) {
		return fight(h) == 1;
	}
	public boolean isWeakerThan(Hand h) {
		return fight(h) == -1;
	}
	private int fight(Hand h){
		if (this== h) {
			return 0;
		} else if ((this.handValue + 1) % 3 == h.handValue) {
			return 1;
		} else {
			return -1;
		}
	}
	@Override
	public String toString() {
		return name;
	}
}
```
### Strategy 클래스
```java
public interface Strategy {
	public abstract Hand nextHand();
	public abstract void study(boolean win);
}
```
### WinningStrategy 클래스
```java
import java.util.Random;

public class WinningStrategy implements Strategy{
	private Random random;
	private boolean won = false;
	private Hand prevHand;

	public WinningStrategy(int seed) {
		random = new Random(seed);
	}

	@Override
	public Hand nextHand(){
		if(!won){
			prevHand = Hand.getHand(random.nextInt(3));
		}
		return prevHand;
	}
	@Override
	public void study(boolean win){
		won = win;
	}
}
```

### ProbStrategy 클래스

```java
import java.util.Random;

public class ProbStrategy implements Strategy{
	private Random random;
	private int prevHandValue = 0;
	private int currentHandValue = 0;
	private int[][] history = {
		{1,1,1,},
		{1,1,1,},
		{1,1,1,},
	};

	public ProbStrategy(int seed) {
		random = new Random(seed);
	}

	@Override
	public Hand nextHand(){
		int bet = random.nextInt(getSum(currentHandValue));
		int handValue = 0;
		if(bet < history[currentHandValue][0]){
			handValue = 0;
		} else if(bet < history[currentHandValue][0] + history[currentHandValue][1]){
			handValue = 1;
		} else {
			handValue = 2;
		}
		prevHandValue = currentHandValue;
		currentHandValue = handValue;
		return Hand.getHand(handValue);
	}
	private int getSum(int handvalue){
		int sum=0;
		for(int i=0;i<3;i++){
			sum += history[handvalue][i];
		}
		return sum;
	}
	@Override
	public void study(boolean win){
		if(win){
			history[prevHandValue][currentHandValue]++;
		} else {
			history[prevHandValue][(currentHandValue+1)%3]++;
			history[prevHandValue][(currentHandValue+2)%3]++;
		}
	}
}

```

### Player 클래스

```java
public class Player {
	private String name;
	private Strategy strategy;
	private int wincount;
	private int losecount;
	private int gamecount;
	public Player(String name, Strategy strategy) {
		this.name = name;
		this.strategy = strategy;
	}
	public Hand nextHand() {
		return strategy.nextHand();
	}
	public void win() {
		strategy.study(true);
		wincount++;
		gamecount++;
	}
	public void lose() {
		strategy.study(false);
		losecount++;
		gamecount++;
	}
	public void even() {
		gamecount++;
	}
	@Override
	public String toString() {
		return "[" + name + ":" + gamecount + " games, " + wincount + " win, " + losecount + " lose" + "]";
	}

}

```

### Main 클래스

```java
public class Main {
	public static void main(String[] args) {
		if(args.length != 2) {
			System.out.println("Usage: java Main randomseed1 randomseed2");
			System.out.println("Example: java Main 314 15");
			System.exit(0);
		}
		int seed1 = Integer.parseInt(args[0]);
		int seed2 = Integer.parseInt(args[1]);
		Player player1 = new Player("Taro", new WinningStrategy(seed1));
		Player player2 = new Player("Hana", new ProbStrategy(seed2));

		for(int i=0;i<1000;i++){
			Hand nextHand1 = player1.nextHand();
			Hand nextHand2 = player2.nextHand();

			if(nextHand1.isStrongerThan(nextHand2)){
				System.out.println("Winner:" +player1);
				player1.win();
				player2.lose();
			} else if(nextHand2.isStrongerThan(nextHand1)){
				System.out.println("Winner:" +player2);
				player1.lose();
				player2.win();
			} else {
				System.out.println("Even...");
				player1.even();
				player2.even();
			}
		}
		System.out.println("Total result:");
		System.out.println(player1.toString());
		System.out.println(player2.toString());
	}
}
```

## Strategy 패턴의 클래스 다이어그램

![strategy-페이지-2 drawio](https://github.com/inh2613/inh2613.github.io/assets/62206617/03243447-e3d9-4295-bbf6-23da2f7c0510)

## 장점

- 적용되는 알고리즘이 많은 경우 **위임**을 이용해 알고리즘 동적 전환 가능