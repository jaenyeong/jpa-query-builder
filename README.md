# jpa-query-builder

## Step0: TDD 실습

### 프로그래밍 요구사항
* 모든 로직에 단위 테스트를 구현한다. 단, 테스트하기 어려운 UI 로직은 제외
  * 핵심 로직을 구현하는 코드와 UI를 담당하는 로직을 구분한다.
  * UI 로직을 InputView, ResultView와 같은 클래스를 추가해 분리한다.
* 자바 코드 컨벤션을 지키면서 프로그래밍한다.
  * 이 과정의 Code Style은 `intellij idea Code Style. Java`을 따른다.
  * `intellij idea Code Style. Java`을 따르려면 `code formatting` 단축키(Windows : `Ctrl + Alt + L`. Mac : `⌥ (Option) + ⌘ (Command) + L`.)를 사용한다.
* 규칙 : 객체지향설계의 규칙을 다 지켜본다.

### 기능 목록 및 commit 로그 요구사항
* 기능을 구현하기 전에 `README.md` 파일에 구현할 기능 목록을 정리해 추가한다.
* git의 commit 단위는 앞 단계에서 `README.md` 파일에 정리한 기능 목록 단위로 추가한다.
  * [AngularJS Commit Message Conventions](https://gist.github.com/stephenparish/9941e89d80e2bc58a153)

### String 클래스의 학습 테스트
* `src>test>java>persistence>study` 하위에 `StringTest` 클래스를 하나 생성한다.

### 요구사항 1 - `123` 이라는 숫자를 문자열로 반환
```java
@Test
void convert() {
// todo: 테스트 코드 만들어보기

}
```
---

## Step1: Reflection

### 요구사항 1 - 클래스 정보 출력
* `src/test/java/persistence/study` > `Car` 클래스의 모든 필드, 생성자, 메소드에 대한 정보를 출력한다.

### 요구사항 2 - test로 시작하는 메소드 실행
* `src/test/java/persistence/study` > `Car` 객체의 메소드 중 `test`로 시작하는 메소드를 자동으로 실행한다.
* 이와 같이 `Car` 클래스에서 test로 시작하는 메소드만 Java `Reflection`을 활용해 실행하도록 구현한다.
* 구현은 `src/test/java/persistence/study` > `ReflectionTest` 클래스의 `testMethodRun()` 메소드에 한다.

### 요구사항 3 - @PrintView 애노테이션 메소드 실행
* `@PrintView` 애너테이션이 설정되어 있는 메소드를 자동으로 실행한다.
* 이와 같이 `Car` 클래스에서 `@PrintView` 애노테이션으로 설정되어 있는 메소드만 Java `Reflection`을 활용해 실행하도록 구현한다.
* 구현은 `src/test/java/persistence/study` > `ReflectionTest` 클래스의 `testAnnotationMethodRun()` 메소드에 한다.

```java
public class Car {
    // ...

    @PrintView
    public void printView() {
        System.out.println("자동차 정보를 출력 합니다.");
    }
    // ...
}

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PrintView {
}
```

### 요구사항 4 - private field에 값 할당
* 자바 `Reflection API`를 활용해 다음 `Car` 클래스의 `name`과 `price` 필드에 값을 할당한 후 getter 메소드를 통해 값을 확인한다.
* 구현은 `src/test/java/persistence/study` > `ReflectionTest` 클래스의 `privateFieldAccess()` 메소드에 한다.

```java
public class Car {
    private String name;
    private int price;

    public Car() {
    }

    public Car(String name, int price) {
        this.name = name;
        this.price = price;
    }

    public String getName() {
        return name;
    }

    public int getPrice() {
        return price;
    }
    // ...
}

public class ReflectionTest {
    private static final Logger logger = LoggerFactory.getLogger(ReflectionTest.class);
    
    @Test
    public void privateFieldAccess() {
        Class<Car> clazz = Car.class;
        logger.debug(clazz.getName());

    }
}
```

### 요구사항 5 - 인자를 가진 생성자의 인스턴스 생성
* `Car` 클래스의 인스턴스를 자바 `Reflection API`를 활용해 `Car` 인스턴스를 생성한다.
* 구현은 `src/test/java/persistence/study` > `ReflectionTest` 클래스의 `constructorWithArgs()` 메소드에 한다.
---

## Step2: QueryBuilder DDL

### 쿼리 실행 방법
* `src/main/java/persistence/Application` 을 참고해 실행시켜 보자.

```java
final DatabaseServer server = new H2();
server.start();

final JdbcTemplate jdbcTemplate = new JdbcTemplate(server.getConnection());

server.stop();
```

### 요구사항 1 - 아래 정보를 바탕으로 create 쿼리 만들어보기
* 구현은 `src/main/java/persistence` > `sql/ddl` > 하위에 구현한다.
```java
@Entity
public class Person {
    @Id
    private Long id;
    
    private String name;
    
    private Integer age;
}
```

### 요구사항 2 - 추가된 정보를 통해 create 쿼리 만들어보기
* 구현은 `src/main/java/persistence` > `sql/ddl` > 하위에 구현한다.
* 아래의 정보를 통해 `Person` 클래스의 정보를 업데이트 해준다.
```java
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nick_name")
    private String name;

    @Column(name = "old")
    private Integer age;
    
    @Column(nullable = false)
    private String email;
}
```

### 요구사항 3 - 추가된 정보를 통해 create 쿼리 만들어보기2
* 구현은 `src/main/java/persistence` > `sql/ddl` > 하위에 구현한다.
* 아래의 정보를 통해 `Person` 클래스의 정보를 업데이트 해준다.

```java
@Table(name = "users")
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "nick_name")
    private String name;

    @Column(name = "old")
    private Integer age;

    @Column(nullable = false)
    private String email;

    @Transient
    private Integer index;
}
```

### 요구사항 4 - 정보를 바탕으로 drop 쿼리 만들어보기
* 구현은 `src/main/java/persistence` > `sql/ddl` > 하위에 구현한다.
* `@Entity`, `@Table`를 고려해서 잘 작성해보자.
