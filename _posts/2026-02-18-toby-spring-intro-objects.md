---
title: "[토비의 스프링] 1장. 오브젝트와 의존관계 - 다시 자바의 기본으로"
date: 2026-02-18 16:00:00 +0900
categories: [Backend, Spring]
tags: [spring, java, oop, pojo]
image:
  path: /assets/img/posts/spring/toby-logo.png
---

> "스프링이 가장 관심을 많이 두는 대상은 오브젝트다."

저자는 스프링을 이해하려면 먼저 오브젝트에 깊은 관심을 가져야 된다고 한다.
그리고 다양한 목적을 위해 재활용 가능한 설계 방법인 디자인 패턴, 지속적으로 개선해나가는 작업인 리팩토링에 대해서 알아보자.

## 0. 왜 다시 초난감 DAO인가?
많은 백앤드 개발자들이 스프링 공부를 시작할 때 `초난감 DAO` 예제로 접한다는 걸 알았다.
예전에 책을 읽었을 땐 코드를 따라 작성하는 것에 중점을 뒀지만 이번엔 좀 더 다른 관점으로 읽어 보려고 한다.

## 1. 첫 번째 개선: 메소드 추출
1.2장에서는 `UserDao`를 안에 혼재된 관심사를 분리한다.\
`add`, `get` 메소드에서 중복되던 DB 연결 로직을 독립적인 메소드로 분리한다.

```java
public void add(User user) throws SQLException {

    Connection c = getConnection(); 
    
    // SQL 실행 로직...
    PreparedStatement ps = c.prepareStatement(
        "insert into users(id, name, password) values(?,?,?)"
    );

}

private Connection getConnection() throws SQLException {

    Class.forName("com.mysql.jdbc.Driver");
    return DriverManager.getConnection(
        "jdbc:mysql://localhost/test", "id", "pw"
    );

}
```

## 2. 디자인 패턴을 통한 확장
"DB 연결 방식이 매번 바뀐다면 어떻게 해야 될까?" 책에서는 **상속**을 활용한 두 가지 패턴을 소개한다.

### 템플릿 메소드 패턴 (Template Method Pattern)
"전체적인 실행 순서는 고정하고, 특정 단계의 구현만 자식에게 맡긴다."
- **핵심**: 일하는 방법(How), 즉 로직의 흐름을 표준화한다.
- **커피 예시**: ‘물 끓이기 -> **[재료 넣기]** -> 컵에 담기’라는 순서는 고정하고, 메뉴에 따라 재료만 하위 클래스에서 결정합니다.

```java
abstract class CoffeeTemplate {
    // 템플릿 메소드: 전체 흐름을 정의 (변경 불가)
    public final void makeCoffee() {
        boilWater();
        addIngredients(); // 이 단계만 하위 클래스가 결정
        pourInCup();
    }

    private void boilWater() { System.out.println("1. 물을 끓인다."); }
    private void pourInCup() { System.out.println("3. 컵에 담는다."); }

    // 하위 클래스가 구현할 추상 메소드
    protected abstract void addIngredients();
}

class Americano extends CoffeeTemplate {
    @Override
    protected void addIngredients() { System.out.println("2. 에스프레소를 넣는다."); }
}
```

### 팩토리 메소드 패턴 (Factory Method Pattern)
"나는 결과물(인터페이스)만 알면 되고 구체적으로 어떤 객체를 만들지는 자식이 결정한다."
- **핵심**: 생성할 대상(What), 즉 객체 생성의 책임을 하위 클래스에 위임하여 결합도를 낮춘다.
- **커피 예시**: 본사는 "커피를 내놓으라"고 명령하고, 실제 '어떤 원두의 커피 객체'를 만들지는 각 지점(하위 클래스)이 결정한다.

```java
abstract class CoffeeMachine {
    public void serve() {
        // 무엇이 나올지 모르지만, "생성(Factory)"을 요청함
        Coffee coffee = createCoffee(); 
        coffee.brew(); // 생성된 객체를 사용함
    }

    // 객체 생성을 하위 클래스에 위임하는 팩토리 메소드
    protected abstract Coffee createCoffee();
}

class StarbucksMachine extends CoffeeMachine {
    @Override
    protected Coffee createCoffee() {
        return new StarbucksAmericano(); // 구체적인 객체 생성을 결정
    }
}
```

## 3. 리팩토링에 대한 고찰: 어떤 패턴이 더 적합할까
`초난감 DAO` 구조를 다시 들여다보니, 현재 상황에서는 팩토리 메소드 패턴이 더 적합하다는 결론에 도달했다.

1. **업무 흐름의 단순함**: 템플릿 메소드 패턴은 부모 클래스에 고정된 복잡한 순서가 있을 때 유리하다. 하지만 현재 DAO는 로직 흐름보다 객체 생성 자체에 비중이 더 크다.

2. **확장의 포인트**: 현재의 핵심 고민은 “어떤 DB(N사, D사)를 쓸 것인가”이다. 즉, 상황에 맞는 커넥션 객체를 공급받는 것이 중요하므로 팩토리 메소드 패턴이 더 논리적이다.

---
## 마치며
“결국 백엔드 설계에서는 패턴의 이름이나 정의보다, ‘지금 내가 무엇을 분리하고 싶어 하는가’를 정확히 파악하는 것이 더 중요하다는 것을 알게 되었다.”