# 객체를 생성할 때 사용할 수 있는 다양한 생성 패턴

## 목차 레퍼런스

- https://www.inflearn.com/course/ts-js-%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-canvas-%EC%A0%9C%EB%A1%9C%EC%B4%88/dashboard

* **Creational Pattern**
* **참조한 예제 코드는 모두 예시 및 수도코드 입니다.**

## 1. 싱글턴(Singleton) - 앱 내에서 단 하나만 존재해야 할 때

### 왜 이런 패턴이 쓰이는가?

- 앱 전역에서 “하나의 인스턴스만 존재”해야 하는 경우에 사용
- 예) 애플리케이션 전체에서 공유되는 설정(config), 전역 캐시, 로그 관리자, 데이터베이스 연결 등
- 인스턴스를 여러 개 생성하면 안 되는 상황에서 이를 강제하기 위함

### 핵심 아이디어

- 생성자를 `private`으로 감추고, 유일한 객체 접근점을 제공하는 `getInstance()`(또는 유사 메서드)를 둠
- 이미 인스턴스가 만들어져 있으면 그 인스턴스를 그대로 반환하고, 없으면 새로 만들어서 반환

### 장점

- 전역적으로 인스턴스가 단 하나만 보장되므로 일관된 상태를 유지할 수 있음 (“Single Source of Truth”).
- 인스턴스화가 여러 번 일어나는 것을 방지하므로 메모리 낭비를 줄일 수 있음 (상황에 따라).

### 단점

- 전역 상태가 생기기 때문에, 테스트하기 까다로워짐 (테스트 격리가 어려움)
- 모듈 간 결합도가 높아질 수 있음

### 예제 코드

```tsx
class EditorConfig {
  private static instance: EditorConfig;
  private config: Record<string, any> = {};

  // 생성자를 private으로 막아서 외부에서 new EditorConfig() 못 하게 함
  private constructor() {
    // 예: 초기 설정 로딩
    this.config = { theme: "dark", language: "ko" };
  }

  public static getInstance(): EditorConfig {
    if (!EditorConfig.instance) {
      EditorConfig.instance = new EditorConfig();
    }
    return EditorConfig.instance;
  }

  public get(key: string): any {
    return this.config[key];
  }

  public set(key: string, value: any) {
    this.config[key] = value;
  }
}

// 사용 예
const config1 = EditorConfig.getInstance();
const config2 = EditorConfig.getInstance();
console.log(config1 === config2); // true (항상 같은 인스턴스)
```

---

## 2. SOLID 원칙

SOLID는 ‘클린 코드’를 작성하기 위한 5가지 원칙의 약어

1. **단일 책임 원칙(Single Responsibility Principle, SRP)**
   - 클래스(또는 모듈)는 오직 하나의 책임만 가져야 함
   - 예) “그림 그리기” 클래스는 그림을 그리는 기능에만 집중, “도형 관리” 클래스는 도형 데이터 관리에만 집중
2. **개방-폐쇄 원칙(Open-Closed Principle, OCP)**
   - 소프트웨어 요소는 **확장**에는 열려 있고 **수정**에는 닫혀 있어야 함
   - 기능을 확장할 때 기존 코드를 수정하기보다는(=폐쇄), 상속·인터페이스·추가 클래스를 통해 확장(=개방)하는 방식을 권장
3. **리스코프 치환 원칙(Liskov Substitution Principle, LSP)**
   - 자식 클래스는 부모 클래스(또는 인터페이스)로 완전히 대체 가능해야 함
   - 부모 타입을 기대하는 곳에 자식 타입을 넣어도 아무 문제 없이 동작해야 한다는 뜻
4. **인터페이스 분리 원칙(Interface Segregation Principle, ISP)**
   - 사용하는 메서드만 가진 **작은 인터페이스** 여러 개로 쪼개서 제공하라는 원칙
   - “한 클래스(또는 모듈)가 사용하지 않는 메서드는 인터페이스에 넣지 말자”라는 의미
5. **의존 역전 원칙(Dependency Inversion Principle, DIP)**
   - 고수준(High-level) 모듈이 저수준(Low-level) 구현에 의존하지 않도록, 추상화(인터페이스)에 의존하게 만들라는 원칙
   - 예) 그림판이 DB 직접 의존하지 말고, “데이터 저장” 인터페이스를 두고 DB 구현체가 이를 구현하도록(의존 역전)

---

## 3. 심플 팩토리(Simple Factory) - 크롬 그림판과 IE 그림판

### 왜 이런 패턴이 쓰이는가?

- 객체를 생성(또는 초기화)하는 로직을 한 곳에서 관리하고, 그 외의 로직에서는 '무엇을 생성할지'에 대한 고민을 줄이기 위해
- "간단한 분기"로 어느 클래스를 생성할지 결정할 때 사용

### 핵심 아이디어

- `Factory` 클래스(또는 함수)에 “switch/if” 분기를 몰아넣고, 필요한 객체를 생성·반환하게 함
- 클라이언트에서는 `new` 대신 팩토리만 사용하여 객체를 얻음

### 장점

- 객체 생성 로직을 한 곳에 모아 가독성과 유지보수가 편해짐
- 생성 로직을 수정할 때, 클라이언트 코드를 직접 고칠 필요가 적음

### 단점

- 팩토리에 분기가 많아지면, 팩토리의 책임이 무거워짐 (클래스 추가될 때마다 팩토리 수정 필요)
- 확장성이 제한적 (새 제품이 추가될 때마다 `switch`/`if` 수정 필요)

### 예제 코드

```tsx
abstract class Editor {
  abstract draw(): void;
}

class ChromeEditor extends Editor {
  draw() {
    console.log("크롬 에디터로 그림을 그립니다.");
  }
}

class IEEditor extends Editor {
  draw() {
    console.log("IE 에디터로 그림을 그립니다.");
  }
}

// 심플 팩토리
class EditorFactory {
  createEditor(type: string): Editor {
    switch (type) {
      case "chrome":
        return new ChromeEditor();
      case "ie":
        return new IEEditor();
      default:
        throw new Error("지원하지 않는 Editor 타입입니다.");
    }
  }
}

// 사용 예
const factory = new EditorFactory();
const chromeEditor = factory.createEditor("chrome");
chromeEditor.draw(); // "크롬 에디터로 그림을 그립니다."
```

---

## 4. 팩토리 메서드(Factory Method) - 사파리 그림판이 추가된다면?

### 왜 이런 패턴이 쓰이는가?

- "생성 로직"을 서브클래스 쪽으로 넘겨, 새로운 클래스가 추가되거나 변경될 때 “심플 팩토리”보다 유연하게 확장할 수 있도록
- 심플 팩토리와 달리 **각 서브클래스가 어떤 구체 객체를 생성할지 결정**하게 함

### 핵심 아이디어

- 부모 클래스(추상 클래스)에 “팩토리 메서드”라는 추상 메서드를 정의
- 실제 객체 생성은 자식 클래스에서 오버라이딩하여 결정

### 장점

- 새로운 종류의 제품(예: SafariEditor) 클래스가 추가될 때, 기존 팩토리 로직을 직접 수정하지 않고 자식 클래스를 하나 더 만들면 됨 (개방-폐쇄 원칙 준수)
- 생성 로직을 필요한 만큼 다양하게 오버라이드할 수 있어 확장성이 좋음

### 단점

- 클래스 수가 많아질 수 있음 (서브클래스가 늘어남)
- 단순 분기만 원하는 경우에는 오히려 코드가 복잡해질 수 있음

### 예제 코드

```tsx
abstract class Editor {
  abstract draw(): void;
}

class ChromeEditor extends Editor {
  draw() {
    console.log("크롬 에디터로 그림 그리기");
  }
}
class IEEditor extends Editor {
  draw() {
    console.log("IE 에디터로 그림 그리기");
  }
}
class SafariEditor extends Editor {
  draw() {
    console.log("사파리 에디터로 그림 그리기");
  }
}

// 팩토리 메서드를 가지고 있는 추상 Creator
abstract class EditorCreator {
  // 팩토리 메서드 (어떤 Editor를 생성할지 자식 클래스가 결정)
  abstract createEditor(): Editor;

  // 공통 로직 예시
  public renderEditor() {
    const editor = this.createEditor();
    editor.draw();
    // ... 기타 에디터 초기화 로직 ...
  }
}

// 구체 Creator들
class ChromeEditorCreator extends EditorCreator {
  createEditor(): Editor {
    return new ChromeEditor();
  }
}

class IEEditorCreator extends EditorCreator {
  createEditor(): Editor {
    return new IEEditor();
  }
}

class SafariEditorCreator extends EditorCreator {
  createEditor(): Editor {
    return new SafariEditor();
  }
}

// 사용 예
function client(creator: EditorCreator) {
  creator.renderEditor();
}

client(new ChromeEditorCreator()); // "크롬 에디터로 그림 그리기"
client(new SafariEditorCreator()); // "사파리 에디터로 그림 그리기"
```

---

## 5. 추상 팩토리(Abstract Factory) - 여러 객체가 '세트'로 구성되어 있을 때

### 왜 이런 패턴이 쓰이는가?

- 서로 연관되거나(또는 호환되는) 여러 종류의 객체를 **‘세트’**로 생성해야 할 때 사용
- 예) 윈도우즈 스킨 UI(버튼, 체크박스, 스크롤바 등) 세트 vs. 맥 스킨 UI 세트

### 핵심 아이디어

- 팩토리 메서드를 여러 개 묶어서(버튼 생성, 체크박스 생성 등) 하나의 “추상 팩토리 인터페이스”로 제공
- 구체 팩토리는 이 인터페이스를 구현하여, 서로 호환되는 제품들의 구체 클래스 객체를 생성

### 장점

- 한 번에 관련 객체들이 호환성을 유지하면서 생성됩니다(테마, 플랫폼별 세트)
- 클라이언트 측에서 구체 클래스에 의존하지 않고, “팩토리”만 교체하면 다른 제품 세트를 쓸 수 있음

### 단점

- 새로운 메서드가 추가되면, 모든 팩토리 클래스가 변경돼야 함
- 팩토리 클래스와 제품 클래스 등, 클래스 구조가 많아질 수 있음

### 예제 코드

```tsx
// 제품 인터페이스: 버튼, 다이얼로그
interface Button {
  click(): void;
}
interface Dialog {
  open(): void;
}

// 구체 제품 (Windows 계열)
class WindowsButton implements Button {
  click() {
    console.log("윈도우 버튼 클릭");
  }
}
class WindowsDialog implements Dialog {
  open() {
    console.log("윈도우 다이얼로그 열기");
  }
}

// 구체 제품 (Mac 계열)
class MacButton implements Button {
  click() {
    console.log("맥 버튼 클릭");
  }
}
class MacDialog implements Dialog {
  open() {
    console.log("맥 다이얼로그 열기");
  }
}

// 추상 팩토리
interface UIFactory {
  createButton(): Button;
  createDialog(): Dialog;
}

// 구체 팩토리
class WindowsFactory implements UIFactory {
  createButton(): Button {
    return new WindowsButton();
  }
  createDialog(): Dialog {
    return new WindowsDialog();
  }
}

class MacFactory implements UIFactory {
  createButton(): Button {
    return new MacButton();
  }
  createDialog(): Dialog {
    return new MacDialog();
  }
}

// 사용 예
function clientCode(factory: UIFactory) {
  const button = factory.createButton();
  const dialog = factory.createDialog();
  button.click();
  dialog.open();
}

clientCode(new WindowsFactory()); // "윈도우 버튼 클릭" / "윈도우 다이얼로그 열기"
clientCode(new MacFactory()); // "맥 버튼 클릭" / "맥 다이얼로그 열기"
```

---

## 6. 빌더(Builder) - 객체를 생성하는 과정이 복잡할 때

### 왜 이런 패턴이 쓰이는가?

- 생성 과정이 단계별(순서 의존)로 복잡하게 이루어진 객체가 있을 때, “구현(생성) 단계”를 명시적으로 분리하고 캡슐화하기 위해
- “어떤 단계는 optional, 어떤 단계는 필수”와 같이 복잡한 초기화 로직이 있을 때

### 핵심 아이디어

- 단계별로 객체를 조립(build)하는 메서드들을 하나의 “Builder” 인터페이스/클래스에 정의
- 최종적으로 `getResult()` 등을 통해 완성된 객체를 얻음
- 복잡한 초기화를 캡슐화하고, 다양한 빌더 구현체를 통해 객체 생성 과정을 달리할 수 있음

### 장점

- 동일한 생성 절차로도 다른 형태의 객체(제품)를 생성할 수 있음 (예: JSON vs. XML 식 객체 생성)
- 생성 로직이 명확히 단계별로 나뉘어 있어서 가독성이 높음

### 단점

- 단계가 많으면, 빌더 클래스 자체가 복잡해질 수 있음
- 단순한 객체에 적용하기엔 오히려 코드만 늘어날 수 있음

### 예제 코드

```tsx
class ComplexObject {
  partA?: string;
  partB?: string;
  partC?: string;
}

// 빌더 인터페이스
interface Builder {
  reset(): void;
  buildPartA(): void;
  buildPartB(): void;
  buildPartC(): void;
  getResult(): ComplexObject;
}

// 구체 빌더
class ConcreteBuilder implements Builder {
  private product: ComplexObject;

  constructor() {
    this.product = new ComplexObject();
  }

  reset(): void {
    this.product = new ComplexObject();
  }

  buildPartA(): void {
    this.product.partA = "A";
  }

  buildPartB(): void {
    this.product.partB = "B";
  }

  buildPartC(): void {
    this.product.partC = "C";
  }

  getResult(): ComplexObject {
    return this.product;
  }
}

// 감독자(Director) - 빌더를 사용해 객체 생성 순서를 지시하는 역할(필요시)
class Director {
  constructSimple(builder: Builder) {
    builder.reset();
    builder.buildPartA();
  }

  constructComplex(builder: Builder) {
    builder.reset();
    builder.buildPartA();
    builder.buildPartB();
    builder.buildPartC();
  }
}

// 사용 예
const builder = new ConcreteBuilder();
const director = new Director();

director.constructSimple(builder);
const simpleObj = builder.getResult();
console.log(simpleObj); // { partA: 'A' }

director.constructComplex(builder);
const complexObj = builder.getResult();
console.log(complexObj); // { partA: 'A', partB: 'B', partC: 'C' }
```

### 빌더 패턴으로 나머지 버튼들 완성하기?

- 예컨대 “저장 버튼”, “취소 버튼”, “이미지 삽입 버튼” 등 다양한 버튼을 만들 때, 버튼 텍스트, 스타일, 이벤트 핸들러, 아이콘 등 여러 속성을 단계적으로 세팅할 수 있도록 빌더를 사용할 수 있음

---

## 7. 프로토타입(Prototype) - 기존 객체를 복붙해서 새 객체 만들기

### 왜 이런 패턴이 쓰이는가?

- “비슷한 객체를 자주 만들어야 하는데, 생성 비용(또는 과정)이 복잡한 경우” 기존 객체를 복제하여 성능 향상이나 편의성을 얻으려함
- JS 자체적으로는 `Object.assign` 또는 `...`(스프레드) 문법, `structuredClone()` 등이 있어 쉽게 복제할 수 있음

### 핵심 아이디어

- 생성자 호출 대신, “이미 존재하는 객체”를 복제하여(얕거나 깊은 복사) 새 객체를 생성
- 공통 속성은 그대로 복사하고, 필요한 부분만 변경해서 재사용

### 장점

- 객체 생성 비용(복잡한 초기화 로직)을 절감할 수 있음
- 런타임 시점에 객체를 복사하므로, 유연한 객체 생성이 가능

### 단점

- 얕은 복사 vs. 깊은 복사 문제(참조 타입 속성은 어떻게 할지 결정 필요)
- 상속 구조가 복잡할 경우, 복제 로직을 관리하기가 어려울 수 있음

### 예제 코드

```tsx
interface Cloneable {
  clone(): Cloneable; // 복제 메서드
}

class Shape implements Cloneable {
  public x: number;
  public y: number;
  public color: string;

  constructor(x: number, y: number, color: string) {
    this.x = x;
    this.y = y;
    this.color = color;
  }

  public clone(): Shape {
    // 간단한 얕은 복사 예시
    return new Shape(this.x, this.y, this.color);
  }
}

// 사용 예
const circle1 = new Shape(10, 20, "red");
const circle2 = circle1.clone();
circle2.x = 15;
console.log(circle1, circle2);
// circle1 => (10,20,'red'), circle2 => (15,20,'red') (서로 다른 인스턴스)
```
