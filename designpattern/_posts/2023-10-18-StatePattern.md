---
layout: post
title: State 패턴
description: >
  2023.10.18 디자인 패턴 스터디 - State Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---

## 목차

[State 패턴이란?](#State-패턴이란?)

[예제 State 클래스 다이어그램](#예제-State-클래스-다이어그램)

[예제 코드 구현](#예제-코드-구현)
- [State 인터페이스](#State인터페이스)
- [DayState 클래스](#DayState-클래스)
- [NightState 클래스](#NightState-클래스)
- [Context 인터페이스](#Context-인터페이스)
- [SafeFrame 클래스](#SafeFrame-클래스)
- [Main 클래스](#Main-클래스)

[State 패턴의 클래스 다이어그램](#State-패턴의-클래스-다이어그램)

[존재 이유](#존재-이유)

---

## State 패턴이란
![image](https://github.com/SWM-REPL/gifthub-was/assets/62206617/d7891591-c2fa-4a53-9925-aa7759c8b136)

- 상태를 클래스로 표현
- 상태가 바뀌면 클래스가 바뀐다.
## 예제 State 클래스 다이어 그램

![image](https://github.com/SWM-REPL/gifthub-was/assets/62206617/857e5cc7-18f7-4bc4-931a-f558ae778ed3)


## 예제 코드 구현

### State 인터페이스
```java
public interface State {
	public abstract void doClock(Context context, int hour);
	public abstract void doUse(Context context);
	public abstract void doAlarm(Context context);
	public abstract void doPhone(Context context);

}

```

### DayState 클래스
```java
public class DayState implements State{
	private static DayState singleton = new DayState();
	private DayState() {

	}
	public static State getInstance() {
		return singleton;
	}
	@Override
	public void doClock(Context context, int hour) {
		if(hour < 9 || 17 <= hour) {
			context.changeState(DayState.getInstance());
		}
	}
	@Override
	public void doUse(Context context) {
		context.recordLog("금고사용(주간)");
	}
	@Override
	public void doAlarm(Context context) {
		context.callSecurityCenter("비상벨(주간)");
	}
	@Override
	public void doPhone(Context context) {
		context.callSecurityCenter("일반통화(주간)");
	}
	@Override
	public String toString() {
		return "[주간]";
	}
}

```

### NightState 클래스
```java
public class NigntState implements State{
	private static NigntState singleton = new NigntState();
	private NigntState() {

	}
	public static State getInstance() {
		return singleton;
	}
	@Override
	public void doClock(Context context, int hour) {
		if(9 <= hour && hour < 17) {
			context.changeState(DayState.getInstance());
		}
	}
	@Override
	public void doUse(Context context) {
		context.callSecurityCenter("비상 : 야간금고 사용!");
	}
	@Override
	public void doAlarm(Context context) {
		context.callSecurityCenter("비상벨(야간)");
	}
	@Override
	public void doPhone(Context context) {
		context.recordLog("야간통화 녹음");
	}
	@Override
	public String toString() {
		return "[야간]";
	}
}

```

### Context 인터페이스
```java
public interface Context {
	public abstract void setClock(int hour);
	public abstract void changeState(State state);
	public abstract void callSecurityCenter(String msg);
	public abstract void recordLog(String msg);

}

```
### SafeFrame 클래스
```java
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class SafeFrame extends Frame implements ActionListener,Context{
	private TextField textClock = new TextField(60);
	private TextArea textScreen = new TextArea(10,60);
	private Button buttonUse = new Button("금고사용");
	private Button buttonAlarm = new Button("비상벨");
	private Button buttonPhone = new Button("일반통화");
	private Button buttonExit = new Button("종료");

	private State state = DayState.getInstance();
	public SafeFrame(String title){
		super(title);
		setBackground(Color.lightGray);
		setLayout(new BorderLayout());
		add(textClock,BorderLayout.NORTH);
		textClock.setEditable(false);
		add(textScreen,BorderLayout.CENTER);
		textScreen.setEditable(false);
		Panel panel = new Panel();
		panel.add(buttonUse);
		panel.add(buttonAlarm);
		panel.add(buttonPhone);
		panel.add(buttonExit);
		add(panel,BorderLayout.SOUTH);
		pack();
		show();
		buttonUse.addActionListener(this);
		buttonAlarm.addActionListener(this);
		buttonPhone.addActionListener(this);
		buttonExit.addActionListener(this);
	}

	@Override
	public void actionPerformed(ActionEvent e) {
		System.out.println(e.toString());
		if(e.getSource() == buttonUse) {
			state.doUse(this);
		}else if(e.getSource() == buttonAlarm) {
			state.doAlarm(this);
		}else if(e.getSource() == buttonPhone) {
			state.doPhone(this);
		}else if(e.getSource() == buttonExit) {
			System.exit(0);
		}else {
			System.out.println("?");
		}
	}
	@Override
	public void setClock(int hour){
		String clockstring = String.format("현재시간은 %02d:00",hour);
		System.out.println(clockstring);
		textClock.setText(clockstring);
		state.doClock(this,hour);
	}

	@Override
	public void changeState(State state) {
		System.out.println(this.state + "에서" + state + "로 상태가 변화했습니다.");
		this.state = state;
	}

	@Override
	public void callSecurityCenter(String msg) {
		textScreen.append("call! " + msg + "\n");
	}
	@Override
	public void recordLog(String msg) {
		textScreen.append("record ... " + msg + "\n");
	}
}

```

### Main 클래스
```java
public class Main {
	public static void main(String[] args) {
		System.out.println("Start.");
		SafeFrame frame = new SafeFrame("State Sample");
		while(true) {
			for(int hour = 0; hour < 24; hour++) {
				frame.setClock(hour);
				try {
					Thread.sleep(1000);
				}catch(InterruptedException e) {

				}
			}
		}
	}
}

```

## State 패턴의 클래스 다이어 그램

![image](https://github.com/SWM-REPL/gifthub-was/assets/62206617/cff6013c-1b85-4973-b4f4-2ac91db5142c)

## 존재 이유
- 상태가 많을 때 (복잡한 조건의 분기처리가 많아질 때)
