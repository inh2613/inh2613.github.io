---
layout: post
title: Mediator 패턴
description: >
  2023.09.27 디자인 패턴 스터디 - Mediator Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---
## 목차

- [Mediator 패턴이란?](#Mediator-패턴이란?)
- [예제 Mediator 클래스 다이어그램](#예제-Mediator-클래스-다이어그램)
- [예제 코드 구현](#예제-코드-구현)
    - [Mediator 인터페이스](#Mediator-클래스)

    - [Colleague 인터페이스](#Colleague-인터페이스)

    - [ColleagueButton 클래스](#ColleagueButton-클래스)

    - [ColleagueTextField 클래스](#ColleagueTextField-클래스)

    - [ColleagueCheckbox 클래스](#ColleagueCheckbox-클래스)

    - [LoginFrame 클래스](#LoginFrame-클래스)

    - [Main 클래스](#main-클래스)

- [예제 Mediator 시퀀스 다이어그램](#예제-Visitor-시퀀스-다이어그램)
- [Mediator 패턴의 클래스 다이어그램](#Visitor-패턴의-클래스-다이어그램)
- [존재 이유](#존재-이유)


## Mediator 패턴이란?

행위패턴중 하나로, 객체들이 직접적으로 상호작용하지 않고 **중재자**를 통해서만 커뮤니케이션하도록 하는 패턴

## 예제 Mediator 클래스 다이어그램
![image](https://github.com/SWM-REPL/gifthub-was/assets/62206617/4f473af4-c445-4c1f-aa87-c1d5cf4b2fe9)

## 예제 코드 구현
### Mediator 인터페이스
```
public interface Mediator {
	public abstract void createColleagues();
	public abstract void colleagueChanged();
}

```

### Colleague 인터페이스
```java
public interface Colleague {
	public abstract void setMediator(Mediator mediator);
	public abstract void setColleagueEnabled(boolean enabled);
}

```

### ColleagueButton 클래스
```java
import java.awt.*;

public class ColleagueButton extends Button implements Colleague {
	private Mediator mediator;
	public ColleagueButton(String caption) {
		super(caption);
	}
	@Override
	public void setMediator(Mediator mediator) {
		this.mediator = mediator;
	}
	@Override
	public void setColleagueEnabled(boolean enabled) {
		setEnabled(enabled);
	}
}

```
### ColleagueTextField 클래스
```java
import java.awt.*;
import java.awt.event.TextEvent;
import java.awt.event.TextListener;

public class ColleagueTextField extends TextField implements TextListener, Colleague{
	private Mediator mediator;
	public ColleagueTextField(String text, int columns) {
		super(text, columns);
	}
	@Override
	public void setMediator(Mediator mediator) {
		this.mediator = mediator;
	}

	@Override
	public void setColleagueEnabled(boolean enabled) {
		setEnabled(enabled);
		setBackground(enabled ? Color.white : Color.lightGray);
	}
	@Override
	public void textValueChanged(TextEvent e) {
		mediator.colleagueChanged();
	}
}

```

### ColleagueCheckbox 클래스
```java
import java.awt.*;
import java.awt.event.ItemEvent;
import java.awt.event.ItemListener;

public class ColleagueCheckbox extends Checkbox implements ItemListener,Colleague {
	private Mediator mediator;
	public ColleagueCheckbox(String caption, CheckboxGroup group, boolean state) {
		super(caption, group, state);
	}
	@Override
	public void setMediator(Mediator mediator) {
		this.mediator = mediator;
	}
	@Override
	public void setColleagueEnabled(boolean enabled) {
		setEnabled(enabled);
	}
	@Override
	public void itemStateChanged(ItemEvent e) {
		mediator.colleagueChanged();
	}
}

```

### LoginFrame 클래스
```java
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class LoginFrame extends Frame implements ActionListener, Mediator {
	private ColleagueCheckbox checkGuest;
	private ColleagueCheckbox checkLogin;
	private ColleagueTextField textUser;
	private ColleagueTextField textPass;
	private ColleagueButton buttonOk;
	private ColleagueButton buttonCancel;

	public LoginFrame(String title) {
		super(title);
		setBackground(Color.lightGray);
		setLayout(new GridLayout(4, 2));
		createColleagues();
		add(checkGuest);
		add(checkLogin);
		add(new Label("Username:"));
		add(textUser);
		add(new Label("Password:"));
		add(textPass);
		add(buttonOk);
		add(buttonCancel);
		colleagueChanged();
		pack();
		show();
	}

	@Override
	public void createColleagues() {
		CheckboxGroup g = new CheckboxGroup();
		checkGuest = new ColleagueCheckbox("Guest", g, true);
		checkLogin = new ColleagueCheckbox("Login", g, false);

		textUser = new ColleagueTextField("", 10);
		textPass = new ColleagueTextField("", 10);
		textPass.setEchoChar('*');

		buttonOk = new ColleagueButton("OK");
		buttonCancel = new ColleagueButton("Cancel");

		//Mediator 설정
		checkGuest.setMediator(this);
		checkLogin.setMediator(this);
		textUser.setMediator(this);
		textPass.setMediator(this);
		buttonOk.setMediator(this);
		buttonCancel.setMediator(this);

		//Listener 설정
		checkGuest.addItemListener(checkGuest);
		checkLogin.addItemListener(checkLogin);
		textUser.addTextListener(textUser);
		textPass.addTextListener(textPass);
		buttonOk.addActionListener(this);
		buttonCancel.addActionListener(this);
	}

	//Colleague의 상태가 바뀌면 호출한다.
	@Override
	public void colleagueChanged() {
		if(checkGuest.getState()) {
			textUser.setColleagueEnabled(false);
			textPass.setColleagueEnabled(false);
			buttonOk.setColleagueEnabled(true);
		} else {
			textUser.setColleagueEnabled(true);
			userpassChanged();
		}
	}

	private void userpassChanged() {
		if(textUser.getText().length()>0){
			textPass.setColleagueEnabled(true);
			if(textPass.getText().length()>0){
				buttonOk.setColleagueEnabled(true);
			} else {
				buttonOk.setColleagueEnabled(false);
			}
		} else {
			textPass.setColleagueEnabled(false);
			buttonOk.setColleagueEnabled(false);
		}
	}

	@Override
	public void actionPerformed(ActionEvent e) {
		System.out.println(e.toString());
		System.exit(0);
	}
}

```
### Main 클래스
```java
public class Main {
	static public void main(String[] args) {
		new LoginFrame("Mediator Sample");
	}
}

```
## 예제 Mediator 시퀀스 다이어그램
![image](https://github.com/SWM-REPL/gifthub-was/assets/62206617/ddbf533d-8d64-4c51-b765-089237671080)


## Mediator 패턴의 클래스 다이어그램
![image](https://github.com/SWM-REPL/gifthub-was/assets/62206617/59f9e26b-6355-47f6-9374-d2cf390a60df)

## 존재 이유
- 복잡한 통신 프로토콜을 가진 시스템에서 유용
- 재사용성 높아짐 (ColleagueButton, ColleagueTextField..)
- 중재자가 모두 처리한다면 중재자의 유지보수가 어려워질 수 있다.