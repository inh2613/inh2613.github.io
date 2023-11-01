---
layout: post
title: Command 패턴
description: >
  2023.11.01 디자인 패턴 스터디 - Command Pattern 발표 자료 입니다.
sitemap: false
hide_last_modified: true
---

---


[Command 패턴이란?](#Command-패턴이란?)
- 객체의 행위(메서드)를 클래스로 만들어 캡슐화하는 행위 패턴

[예제 Command 클래스 다이어그램](#예제-Command-클래스-다이어그램)

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/d986f458-a9fc-46ce-a2dc-76cc84a927d7)

[예제 코드 구현](#예제-코드-구현)
- [Command 인터페이스](#Command-인터페이스)
```java
public interface Command {
	public abstract void execute();
}

```
- [MacroCommand 클래스](#MacroCommand-클래스)
```java
import java.util.ArrayDeque;
import java.util.Deque;

public class MacroCommand implements Command{
	private Deque<Command> commands=new ArrayDeque<Command>();

	@Override
	public void execute() {
		for(Command command:commands) {
			command.execute();
		}
	}

	public void addCommand(Command command) {
		if(command==this){
			throw new IllegalArgumentException("infinite loop");
		}
		commands.push(command);
	}
	public void undo() {
		if(!commands.isEmpty()) {
			commands.pop();
		}
	}
	public void clear() {
		commands.clear();
	}

}

```
- [DrawCommand 클래스](#DrawCommand-클래스)
```java
import java.awt.*;

public class DrawCommand implements Command{
	protected Drawable drawable;
	private Point position;

	public DrawCommand(Drawable drawable,Point position) {
		this.drawable=drawable;
		this.position=position;
	}

	@Override
	public void execute() {
		drawable.draw(position.x, position.y);
	}
}

```
- [Drawable 인터페이스](#Drawable-인터페이스)
```java
public interface Drawable {
	public abstract void draw(int x,int y);
}

```
- [DrawCanvas 클래스](#DrawCanvas-클래스)
```java
import java.awt.*;

public class DrawCanvas extends Canvas implements Drawable{
	private Color color=Color.red;
	private int radius=6;
	private MacroCommand history;

	public DrawCanvas(int width,int height,MacroCommand history) {
		setSize(width,height);
		setBackground(Color.white);
		this.history=history;
	}

	@Override
	public void paint(Graphics g) {
		history.execute();
	}
	@Override
	public void draw(int x,int y) {
		Graphics g=getGraphics();
		g.setColor(color);
		g.fillOval(x-radius, y-radius, radius*2, radius*2);
	}
}


```
- [Main 클래스](#Main-클래스)
```java
import java.awt.event.MouseMotionListener;
import java.awt.event.WindowListener;

import javax.swing.*;

public class Main extends JFrame implements MouseMotionListener, WindowListener {
	private MacroCommand history=new MacroCommand();
	private DrawCanvas canvas=new DrawCanvas(400,400,history);
	private JButton clearButton=new JButton("clear");

	public Main(String title) {
		super(title);

		this.addWindowListener(this);
		canvas.addMouseMotionListener(this);
		clearButton.addActionListener(e->{
			history.clear();
			canvas.repaint();
		});

		Box buttonBox=new Box(BoxLayout.X_AXIS);
		buttonBox.add(clearButton);
		Box mainBox=new Box(BoxLayout.Y_AXIS);
		mainBox.add(buttonBox);
		mainBox.add(canvas);
		getContentPane().add(mainBox);

		pack();
		setVisible(true);
	}
	@Override
	public void mouseMoved(java.awt.event.MouseEvent e) {
	}
	@Override
	public void mouseDragged(java.awt.event.MouseEvent e) {
		Command cmd=new DrawCommand(canvas,e.getPoint());
		history.addCommand(cmd);
		cmd.execute();
	}
	@Override
	public void windowClosing(java.awt.event.WindowEvent e) {
		System.exit(0);
	}
	@Override public void windowOpened(java.awt.event.WindowEvent e) {}
	@Override public void windowClosed(java.awt.event.WindowEvent e) {}
	@Override public void windowIconified(java.awt.event.WindowEvent e) {}
	@Override public void windowDeiconified(java.awt.event.WindowEvent e) {}
	@Override public void windowActivated(java.awt.event.WindowEvent e) {}
	@Override public void windowDeactivated(java.awt.event.WindowEvent e) {}

	public static void main(String[] args) {
		new Main("Command Pattern Sample");
	}
}

```

[Command 패턴의 클래스 다이어그램](#Command-패턴의-클래스-다이어그램)

![image](https://github.com/inh2613/inh2613.github.io/assets/62206617/9b7b9c6d-78bb-4a5f-8737-e40273f8c10c)

