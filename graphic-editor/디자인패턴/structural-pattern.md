# 다양한 객체를 조합하여 활용하는 구조 패턴

## 목차 레퍼런스

- https://www.inflearn.com/course/ts-js-%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-canvas-%EC%A0%9C%EB%A1%9C%EC%B4%88/dashboard

* **Structural Pattern**
* **참조한 예제 코드는 모두 예시 및 수도코드 입니다.**

## 1. 퍼사드(Facade) - 복잡한 건 숨기고 간단한 것만 드러내기

### 왜 쓰이는가?

- 내부적으로 복잡한 서브시스템(클래스들)이 있을 때, 단순한 인터페이스를 제공하여 사용하기 쉽게 만들고 싶을 때
- 예) 에디터 내부 로직(렌더링, 파일 IO 등)은 복잡하지만, 외부에는 간단한 메서드(`initEditor()`, `save()`)만 노출

### 핵심 아이디어

- Facade 클래스가 내부 서브시스템 객체들을 가지고, 복잡한 작업을 간단한 API로 묶어서 제공
- 클라이언트는 Facade의 간단한 메서드만 호출 → 내부 동작은 Facade가 처리

### 장점

- 서브시스템과 클라이언트 간 결합도 감소
- 코드 가독성과 유지보수성 향상(복잡성 캡슐화)

### 단점

- Facade가 너무 많은 기능을 갖게 되면, 결국 복잡해질 수 있음
- 너무 추상화해버리면, 세부 제어가 어려워질 수도 있음

### 예제 코드

```tsx
class ComplexSystemA {
  public init() {
    console.log("SystemA init");
  }
  public doWorkA() {
    console.log("SystemA work");
  }
}

class ComplexSystemB {
  public setup() {
    console.log("SystemB setup");
  }
  public doWorkB() {
    console.log("SystemB work");
  }
}

class EditorFacade {
  private sysA: ComplexSystemA;
  private sysB: ComplexSystemB;

  constructor() {
    this.sysA = new ComplexSystemA();
    this.sysB = new ComplexSystemB();
  }

  public initializeEditor() {
    this.sysA.init();
    this.sysB.setup();
  }

  public renderSomething() {
    this.sysA.doWorkA();
    this.sysB.doWorkB();
  }
}

// 사용 예
const editor = new EditorFacade();
editor.initializeEditor(); // 내부적으로 SystemA.init, SystemB.setup
editor.renderSomething(); // 내부적으로 SystemA.doWorkA, SystemB.doWorkB
```

---

## 2. 어댑터(Adapter) - 호환이 안 되는 두 개 호환시키기

### 왜 쓰이는가?

- 기존 코드(클라이언트)가 기대하는 인터페이스와, 실제 사용하는 서비스의 인터페이스가 다를 때 “중간 변환기(어댑터)”를 둬서 호환시키기 위해
- 예) 그림판에서 외부 라이브러리(이미지 로더)가 우리 코드와 인터페이스가 다를 때, Adapter를 만들어 일관성 있게 사용

### 핵심 아이디어

- 클라이언트가 기대하는 인터페이스(Target)를 정의
- 어댑터가 “Target 인터페이스를 구현”하면서, 내부에서 “실제 서비스(Adaptee)”를 호출하여 포맷/메서드명을 변환

### 장점

- 기존 코드 수정 없이, 새/기존 라이브러리를 연결 가능
- 점진적 마이그레이션에 유용

### 단점

- 성능상 한 단계 더 거치므로 약간의 오버헤드(보통 미미함)
- 어댑터가 많아지면 복잡해질 수 있음

### 예제 코드

```tsx
// 1) 클라이언트가 기대하는 인터페이스
interface ImageLoader {
  loadImage(path: string): void;
}

// 2) 실제 라이브러리는 이런 식이라고 가정(Adaptee)
class ExternalImageLib {
  public load(path: string, callback: () => void) {
    console.log(`External Lib: loading image from ${path}`);
    callback();
  }
}

// 3) 어댑터
class ImageLoaderAdapter implements ImageLoader {
  private externalLib: ExternalImageLib;

  constructor(externalLib: ExternalImageLib) {
    this.externalLib = externalLib;
  }

  loadImage(path: string): void {
    // 클라이언트가 원하는 메서드 시그니처 → 실제 라이브러리 호출
    this.externalLib.load(path, () => {
      console.log("이미지 로드 완료 via Adapter");
    });
  }
}

// 사용 예
const extLib = new ExternalImageLib();
const adapter = new ImageLoaderAdapter(extLib);

// 클라이언트 코드
function clientCode(loader: ImageLoader) {
  loader.loadImage("test.png");
}

clientCode(adapter);
// 출력:
//   "External Lib: loading image from test.png"
//   "이미지 로드 완료 via Adapter"
```

---

## 3. 데코레이터(Decorator) - 기존 클래스에 기능 추가하기 1

### 왜 쓰이는가?

- 기존 객체(컴포넌트)에 **동적으로 새로운 기능(책임)을 덧씌우고** 싶을 때(상속 대신 조합 이용)
- 예) “그림 그리기” 컴포넌트에 “테두리 그리기”, “배경색 채우기” 기능을 선택적으로 붙이고 싶다면?

### 핵심 아이디어

- 대상 객체에 대한 기능 확장이나 변경이 필요할때 객체의 결합을 통해 서브클래싱 대신 쓸수 있는 유연한 대안 구조 패턴
- “데코레이터” 클래스가 원본 객체(컴포넌트)를 래핑(wrap)하고, 필요한 기능을 추가한 뒤, 다시 원본 객체의 메서드를 호출
- 중첩해서 여러 데코레이터를 쌓아 올릴 수도 있음

### 장점

- 상속보다 유연하게, 필요한 기능만 조합해 넣을 수 있음
- 런타임에 객체에 기능을 추가/제거 가능

### 단점

- 너무 많은 데코레이터가 중첩되면, 디버깅과 구조 파악이 어렵다
- 객체 식별이 어렵고, 데코레이터가 많이 쌓이면 스택 추적 복잡

### 예제 코드

```tsx
// Component
interface Shape {
  draw(): void;
}

// Concrete Component
class Circle implements Shape {
  draw() {
    console.log("원 그리기");
  }
}

// 데코레이터
class ShapeDecorator implements Shape {
  protected shape: Shape;
  constructor(shape: Shape) {
    this.shape = shape;
  }
  draw(): void {
    this.shape.draw();
  }
}

// 구체 데코레이터 (1) - 색칠
class ColorDecorator extends ShapeDecorator {
  draw(): void {
    super.draw();
    this.setColor();
  }
  private setColor() {
    console.log("색칠: 빨간색");
  }
}

// 구체 데코레이터 (2) - 테두리
class BorderDecorator extends ShapeDecorator {
  draw(): void {
    super.draw();
    this.setBorder();
  }
  private setBorder() {
    console.log("테두리: 두껍게");
  }
}

// 사용 예
const circle = new Circle();
const coloredCircle = new ColorDecorator(circle);
const borderedColoredCircle = new BorderDecorator(coloredCircle);

borderedColoredCircle.draw();
// 출력:
//   "원 그리기"
//   "색칠: 빨간색"
//   "테두리: 두껍게"
```

---

## 4. 믹스인(Mixin) - 기존 클래스에 기능 추가하기 2

### 왜 쓰이는가?

- 여러 클래스에서 공유하고 싶은(다중 상속처럼) 기능이 있을 때, 자바스크립트/타입스크립트의 “mixin” 기법으로 주입
- 예) “이벤트 발행 기능”을 다양한 클래스에 섞어서 추가하는 식

### 핵심 아이디어

- 간단히 말해, 함수를 통해 클래스를 ‘확장(augment)’하는 방법
- TypeScript에서는 인터페이스 & “extends” & “Object.assign” 형태로 구현
- **_Mixin_**은 상속 없이 어떤 객체나 클래스에 재사용 가능한 기능을 추가할 수 있는 객체

### 장점

- 다중 상속을 대신해, 간단히 공통 기능만 섞어넣을 수 있음
- 기존 코드 일부 수정으로 다양한 기능을 재사용 가능

### 단점

- 혼합되는 기능이 많아지면, 클래스가 점점 무거워질 수 있음
- 믹스인 충돌(동일 메서드명) 주의

### 간단 예제

```tsx
// 믹스인 함수
function WithTimestamp<TBase extends new (...args: unknown[]) => {}>(
  Base: TBase
) {
  return class extends Base {
    timestamp: Date = new Date();
    printTimestamp() {
      console.log("timestamp:", this.timestamp.toISOString());
    }
  };
}

// 기본 클래스
class BasicLogger {
  log(message: string) {
    console.log("Log:", message);
  }
}

// 믹스인을 적용한 새 클래스
const TimestampedLogger = WithTimestamp(BasicLogger);

const logger = new TimestampedLogger();
logger.log("안녕하세요");
logger.printTimestamp();
```

---

## 5. 대리인(Proxy) - 접근제어, 로딩, 캐싱 너한테 맡길게!

### 왜 쓰이는가?

- 원본 객체 접근 전(또는 대신)에, “접근제어, 로깅, 지연로딩, 캐싱” 등의 부가 기능을 수행하고 싶을 때
- 예) “이미지 로딩” 시, 실제 이미지는 필요할 때만(스크롤 등) 불러오고, 나머지는 프록시가 썸네일 등으로 대체

### 핵심 아이디어

- 대상 원본 객체를 대리하여 대신 처리하게 함으로써 로직의 흐름을 제어하는 패턴
- Proxy가 “원본 객체(Real Subject)”와 동일한 인터페이스를 구현
- 클라이언트는 Proxy를 통해서만 Subject에 접근 → Proxy가 부가 로직 추가 후, 필요 시 원본 객체에 위임

### 장점

- 원본 객체를 직접 수정하지 않고 부가기능(권한 체크, 로깅 등) 추가 가능
- 지연 로딩, 캐싱 등에 유리

### 단점

- 호출이 많이 일어나면, 프록시에서도 성능 손실(추가 레이어)
- 구조가 복잡해질 수 있음(프록시 개수가 늘어나면)

### 예제 코드

```tsx
interface Image {
  display(): void;
}

class RealImage implements Image {
  private fileName: string;

  constructor(fileName: string) {
    this.fileName = fileName;
    this.loadFromDisk();
  }

  private loadFromDisk() {
    console.log("디스크에서 이미지 로딩:", this.fileName);
  }

  public display(): void {
    console.log("화면에 이미지 표시:", this.fileName);
  }
}

class ProxyImage implements Image {
  private realImage: RealImage | null = null;
  private fileName: string;

  constructor(fileName: string) {
    this.fileName = fileName;
  }

  public display(): void {
    // 실제로 필요할 때까지 RealImage를 생성하지 않음 (지연로딩)
    if (!this.realImage) {
      this.realImage = new RealImage(this.fileName);
    }
    this.realImage.display();
  }
}

// 사용 예
const image = new ProxyImage("test.jpg");
// 아직 RealImage 생성 안 됨
image.display(); // 여기서 RealImage 생성 + 디스크 로딩 후 display
image.display(); // 두 번째 호출 땐 이미 로드됨
```

---

## 6. 플라이웨이트(Flyweight) - 메모리 낭비를 줄여요

### 왜 쓰이는가?

- 대량의 ‘비슷한’ 객체가 있을 때, **공유할 수 있는 부분(내부 상태)** 을 공유해서 메모리를 절약하고 싶을 때
- 예) 에디터에서 수천 개의 문자를 각각 객체로 관리하되, 글자 모양, 스타일 등은 공유
- 재사용 가능한 객체 인스턴스를 공유시켜 메모리 사용량을 최소화하는 구조 패턴, 간단히 말하면 캐시 개념을 코드로 패턴화한 것

### 핵심 아이디어

- 객체를 “내부 상태(공유)”와 “외부 상태(개별)”로 나누어, 내부 상태는 플라이웨이트 객체로 재활용
- 팩토리(또는 풀)에서 이미 생성된 플라이웨이트를 꺼내서 사용하는 방식

### 장점

- 동일한 속성을 가진 객체들이 내부 상태를 공유 → 메모리 사용 감소
- 대규모 객체를 다룰 때 효율적

### 단점

- 코드가 복잡해짐(내외부 상태 분리)
- 플라이웨이트끼리 공유되는 부분을 잘 설계해야 함

### 간단 예제

```tsx
// 공유할 상태: 도형 컬러, 스타일 등
class ShapeFlyweight {
  constructor(public color: string, public style: string) {}
  draw(x: number, y: number) {
    console.log(
      `색(${this.color}) 스타일(${this.style})로 [${x}, ${y}]에 그림`
    );
  }
}

// Flyweight Factory
class ShapeFactory {
  private static flyweights: Record<string, ShapeFlyweight> = {};

  public static getFlyweight(color: string, style: string): ShapeFlyweight {
    const key = color + "_" + style;
    if (!this.flyweights[key]) {
      this.flyweights[key] = new ShapeFlyweight(color, style);
    }
    return this.flyweights[key];
  }
}

// 사용 예
const shapes = [
  { x: 10, y: 20, color: "red", style: "bold" },
  { x: 15, y: 25, color: "red", style: "bold" },
  { x: 30, y: 40, color: "blue", style: "dashed" },
];

shapes.forEach((s) => {
  const flyweight = ShapeFactory.getFlyweight(s.color, s.style);
  flyweight.draw(s.x, s.y);
});
// 동일한 color/style 조합이면 메모리를 공유
```

---

## 7. 브릿지(Bridge) - 프리미엄 그림판이 추가된다면? 추상과 구현의 분리

### 왜 쓰이는가?

- 추상(Abstraction)과 구현(Implementation)을 분리하여, **서로 독립적으로 확장**할 수 있도록
- 예) “그림판” 기능(추상)과 “렌더링 엔진(캔버스, WebGL 등)”(구현)이 서로 독립적으로 바뀔 수 있게

### 핵심 아이디어

- **브릿지**는 큰 클래스 또는 밀접하게 관련된 클래스들의 집합을 두 개의 개별 계층구조(추상화 및 구현)로 나눈 후 각각 독립적으로 개발할 수 있도록 하는 구조 디자인 패턴
- 추상 클래스(또는 인터페이스)가 구현 객체에 대한 레퍼런스를 포함하고, 구체 구현은 별도 계층으로 분리
- “추상”을 상속 확장해도, “구현”을 상속 확장해도 서로에게 큰 영향이 가지 않음

### 장점

- 여러 조합(추상 & 구현)을 자유롭게 조합 가능
- 상속의 복잡한 계층보다 더 유연

### 단점

- 구조가 늘어나서 코드가 복잡해질 수 있음(추상 + 구현 클래스 각각)

### 예제 코드

```tsx
// 조합에 대한 얘기

// 구현 (Implementation)
interface DrawingImplementor {
  drawShape(shape: string): void;
}

// 구체 구현 (예: 2D Canvas)
class CanvasImplementor implements DrawingImplementor {
  drawShape(shape: string): void {
    console.log(`[Canvas] ${shape} 그리기`);
  }
}

// 구체 구현 (예: WebGL)
class WebGLImplementor implements DrawingImplementor {
  drawShape(shape: string): void {
    console.log(`[WebGL] ${shape} 그리기`);
  }
}

// 추상 (Abstraction)
abstract class Shape {
  protected implementor: DrawingImplementor;
  constructor(implementor: DrawingImplementor) {
    this.implementor = implementor;
  }
  abstract draw(): void;
}

// 구체 추상
class Circle extends Shape {
  draw(): void {
    this.implementor.drawShape("원");
  }
}

class Rectangle extends Shape {
  draw(): void {
    this.implementor.drawShape("사각형");
  }
}

// 사용 예
const canvas = new CanvasImplementor();
const webgl = new WebGLImplementor();

const circleOnCanvas = new Circle(canvas);
circleOnCanvas.draw(); // [Canvas] 원 그리기

const rectOnWebGL = new Rectangle(webgl);
rectOnWebGL.draw(); // [WebGL] 사각형 그리기
```

---

## 8. 컴포지트(Composite)

### 왜 쓰이는가?

- 트리 구조를 만들 때, **개별 객체(Leaf)와 복합 객체(Composite)를 같은 인터페이스**로 다루고 싶을 때
- 예) “도형(원, 사각형)” 각각은 Leaf, “그룹(도형 묶음)”은 Composite로, 둘 다 `draw()` 인터페이스를 동일하게 가지게

### 핵심 아이디어

- Component 인터페이스 정의 → Leaf(개별 객체)와 Composite(그룹) 모두 이 인터페이스를 구현
- Composite는 내부에 여러 Component(Leaf 또는 또 다른 Composite)를 보관

### 장점

- 클라이언트 입장에서 Leaf와 Composite를 **동일하게** 취급 가능. 재귀적 구조
- 복합 객체(트리)를 다루기 쉬워짐

### 단점

- 너무 일반적인 구조여서, Composite가 자식 관리 로직을 제대로 갖추려면 구현이 복잡할 수 있음
- Leaf와 Composite 간의 차이점을 명확히 해야 헷갈리지 않음

### 예제 코드

```tsx
// Component
interface Graphic {
  draw(): void;
}

// Leaf
class Dot implements Graphic {
  draw(): void {
    console.log("점 그리기");
  }
}

// Leaf
class Circle implements Graphic {
  draw(): void {
    console.log("원 그리기");
  }
}

// Composite
class CompoundGraphic implements Graphic {
  private children: Graphic[] = [];

  public add(child: Graphic) {
    this.children.push(child);
  }

  public remove(child: Graphic) {
    this.children = this.children.filter((c) => c !== child);
  }

  draw(): void {
    console.log("== 복합 그래픽 그리기 시작 ==");
    this.children.forEach((child) => child.draw());
    console.log("== 복합 그래픽 그리기 끝 ==");
  }
}

// 사용 예
const dot = new Dot();
const circle = new Circle();

const compound = new CompoundGraphic();
compound.add(dot);
compound.add(circle);

compound.draw();
// 출력:
//   "== 복합 그래픽 그리기 시작 =="
//   "점 그리기"
//   "원 그리기"
//   "== 복합 그래픽 그리기 끝 =="
```

> 이렇게 컴포지트 패턴을 쓰면, Graphic 인터페이스만 이용해 “점(Leaf)”도 그릴 수 있고 “복합 그래픽(Composite)”도 그릴 수 있어 트리 구조를 통합적으로 다룰 수 있음
