# 객체가 자신의 역할을 깔끔하게 수행하게 만드는 행동 패턴

## 목차 레퍼런스

- https://www.inflearn.com/course/ts-js-%EB%94%94%EC%9E%90%EC%9D%B8%ED%8C%A8%ED%84%B4-canvas-%EC%A0%9C%EB%A1%9C%EC%B4%88/dashboard

* **Behavioral Pattern**
* **참조한 예제 코드는 모두 예시 및 수도코드 입니다.**

## 1. 명령(Command) - 모든 작업을 동일하게 규격화하기

### 왜 쓰이는가?

- “작업(행동)을 객체”로 만들어서, 실행(Invoke), 취소(Undo) 등을 일관된 방식으로 처리하고 싶을 때 사용
- 예) 그림판에서 ‘그리기 명령’, ‘지우기 명령’, ‘색 변경 명령’ 등을 각각 별도의 클래스로 만들어두면, 실행/취소를 쉽게 관리할 수 있음

### 핵심 아이디어

- **명령(Command) 인터페이스**를 정의: `execute()`, `undo()` 등의 메서드
- 각 ‘명령 클래스’가 실제 로직을 구현.
- 호출자(Invoker)는 명령 객체만 알고 실행/취소 호출 → 구체 로직이 캡슐화됨

### 장점

- 기능(행동)이 캡슐화되어, 새 명령을 쉽게 추가 가능
- 명령들을 모아두고 하나씩 실행/되돌리기(Undo/Redo) 등을 통합적으로 관리할 수 있음

### 단점

- 명령 클래스가 많아질 수 있음(기능별로 Command 클래스를 만들어야 하므로)

### 예제 코드

```tsx
// Command 인터페이스
interface Command {
  execute(): void;
  undo(): void; // undo가 필요한 경우
}

// 수신자(Receiver) - 실제 행동을 수행하는 대상
class DrawingBoard {
  private content: string = "";

  public draw(shape: string) {
    this.content += shape + " ";
    console.log("[DrawingBoard] Draw >>", this.content);
  }

  public eraseLast() {
    const shapes = this.content.trim().split(" ");
    shapes.pop(); // 마지막 도형 삭제
    this.content = shapes.join(" ") + " ";
    console.log("[DrawingBoard] Erase >>", this.content);
  }

  public getContent() {
    return this.content;
  }
}

// 구체 명령(1) : 도형 그리기
class DrawCommand implements Command {
  private receiver: DrawingBoard;
  private shape: string;

  constructor(receiver: DrawingBoard, shape: string) {
    this.receiver = receiver;
    this.shape = shape;
  }

  execute(): void {
    this.receiver.draw(this.shape);
  }

  undo(): void {
    this.receiver.eraseLast();
  }
}

// Invoker : 명령을 실행/취소 시키는 주체
class CommandInvoker {
  private history: Command[] = [];

  public executeCommand(command: Command) {
    command.execute();
    this.history.push(command);
  }

  public undoCommand() {
    const command = this.history.pop();
    if (command) {
      command.undo();
    }
  }
}

// 사용 예
const board = new DrawingBoard();
const invoker = new CommandInvoker();

// '원' 그리기
const drawCircle = new DrawCommand(board, "Circle");
invoker.executeCommand(drawCircle);

// '사각형' 그리기
const drawSquare = new DrawCommand(board, "Square");
invoker.executeCommand(drawSquare);

// Undo (사각형 지우기)
invoker.undoCommand();
```

> “명령 패턴으로 나머지 버튼 명령 추가하기”
>
> - 예를 들어, “선 그리기(LineCommand)”, “텍스트 삽입(TextCommand)” 등을 추가하려면 `Command` 인터페이스 구현 클래스를 각각 만든 뒤 `execute()`, `undo()` 로직만 넣으면 됨, Invoker는 동일한 인터페이스로 명령들을 호출할 수 있어 확장성이 좋아짐

---

## 2. 상태(State) - 현재 그림판의 모드 정하기 (if문 중복 제거법1)

### 왜 쓰이는가?

- 객체(또는 시스템)가 “여러 가지 상태”를 갖고 있으며, **상태에 따라 행동(메서드 로직)이 달라지는 경우**에,
  큰 `switch`/`if` 대신 각 “상태”를 별도 클래스로 분리하고 싶을 때 사용
- 예) 그림판이 “드로잉 모드, 선택 모드, 텍스트 모드” 등으로 구분되고, 버튼 클릭 로직이 상태별로 달라진다면?

### 핵심 아이디어

- “상태”를 나타내는 클래스를 여러 개 두고, Context가 이 상태 객체를 참조
- 요청이 들어오면, Context가 현재 상태 객체의 메서드를 호출 → 상태 객체마다 행동이 다르게 동작

### 장점

- 각 상태에 따른 로직이 “상태 클래스”로 깔끔히 분리되어, 조건문/분기문을 대폭 줄일 수 있음
- 새로운 상태 추가가 쉬워짐(기존 상태 로직 크게 손대지 않아도 됨)

### 단점

- 상태 클래스가 많아질 수 있음
- 로직이 매우 간단할 경우, 오히려 코드가 복잡해질 수도 있음

### 예제 코드

```tsx
// Context (그림판)
class Canvas {
  private state: State;

  constructor() {
    this.state = new DrawState(); // 초기 상태
  }

  public setState(state: State) {
    this.state = state;
    console.log("상태 변경:", state.constructor.name);
  }

  public clickEvent() {
    // 클릭 동작 -> 현재 상태에 따라 다르게 수행
    this.state.handleClick(this);
  }
}

// State 인터페이스
interface State {
  handleClick(canvas: Canvas): void;
}

// 구체 상태 (드로잉 상태)
class DrawState implements State {
  handleClick(canvas: Canvas): void {
    console.log("드로잉 상태: 도형을 그립니다.");
    // 상태 전환 가능: 그림을 그린 뒤 선택 모드로 바꾼다거나...
    canvas.setState(new SelectState());
  }
}

// 구체 상태 (선택 상태)
class SelectState implements State {
  handleClick(canvas: Canvas): void {
    console.log("선택 상태: 도형을 선택합니다.");
    // 선택 모드에서 다시 드로잉 모드로 바꾼다거나...
    canvas.setState(new DrawState());
  }
}

// 사용 예
const myCanvas = new Canvas();
myCanvas.clickEvent(); // "드로잉 상태: 도형을 그립니다." → SelectState로 변경
myCanvas.clickEvent(); // "선택 상태: 도형을 선택합니다." → DrawState로 변경
```

---

## 3. 전략(Strategy) - png, jpg, webp 저장 전략 (if문 중복 제거법2)

### 왜 쓰이는가?

- 알고리즘(행동)을 여러 가지로 교체 가능하게 만들고 싶을 때
- 예) “이미지 저장” 기능이 있고, PNG/JPG/WebP 등 여러 포맷을 선택할 수 있다면, 각 포맷마다 로직이 달라지는데, 조건문 대신 각 전략으로 분리

### 핵심 아이디어

- `Strategy` 인터페이스에 공통 메서드(`saveImage()`) 등을 정의하고, 구체 전략 클래스로 각 알고리즘을 구현
- 컨텍스트(에디터)는 전략 객체를 교체하는 식으로 알고리즘을 쉽게 바꿀 수 있음

### 장점

- 알고리즘을 ‘클래스’로 독립시켜 교체/추가가 용이
- 조건문/분기문이 사라지고, “전략 교체”로 로직 변경이 가능

### 단점

- 알고리즘마다 별도 클래스를 만들어야 하므로, 클래스 수가 많아짐
- 전략 간 공유 로직이 있을 경우 중복이 생길 수 있음(추상 클래스 활용 등으로 해결 가능)

### 예제 코드

```tsx
interface SaveStrategy {
  save(data: string): void;
}

class PNGSaveStrategy implements SaveStrategy {
  save(data: string) {
    console.log("PNG 형식으로 저장:", data);
  }
}

class JPGSaveStrategy implements SaveStrategy {
  save(data: string) {
    console.log("JPG 형식으로 저장:", data);
  }
}

class WebPSaveStrategy implements SaveStrategy {
  save(data: string) {
    console.log("WebP 형식으로 저장:", data);
  }
}

// Context
class ImageEditor {
  private strategy: SaveStrategy;

  constructor(strategy: SaveStrategy) {
    this.strategy = strategy;
  }

  public setStrategy(strategy: SaveStrategy) {
    this.strategy = strategy;
  }

  public saveImage(data: string) {
    this.strategy.save(data);
  }
}

// 사용 예
const editor = new ImageEditor(new PNGSaveStrategy());
editor.saveImage("이미지 데이터");
// PNG 형식으로 저장: 이미지 데이터

editor.setStrategy(new JPGSaveStrategy());
editor.saveImage("이미지 데이터");
// JPG 형식으로 저장: 이미지 데이터
```

---

## 4. 템플릿 메서드(Template Method) - 서식 마련해놓고 일부만 수정해서 쓰기

### 왜 쓰이는가?

- 알고리즘(작업) 전체 구조는 동일하지만, **일부 단계(훅 메서드)만 서브클래스에서 변경**하고 싶을 때
- 변하지 않는 기능(템플릿)은 상위 클래스에 만들어 두고 자주 변경되며 확장할 기능은 하위 클래스에서 만들도록 하여, 상위의 메소드 실행 동작 순서는 고정하면서 세부 실행 내용은 다양화 될 수 있는 경우에 사용
- 예) “이미지 처리” 큰 틀(파일 열기 → 처리 → 파일 닫기)은 같고, “처리” 부분만 다르다면, 그 부분만 오버라이딩하도록 만든다

### 핵심 아이디어

- 여러 클래스에서 공통으로 사용하는 메서드를 템플릿화 하여 상위 클래스에서 정의하고,하위 클래스마다 세부 동작 사항을 다르게 구현하는 패턴
- 부모 클래스에 “템플릿 메서드”를 만들고, 그 안에서 여러 단계 메서드를 순서대로 호출
- 서브클래스가 필요한 단계만 오버라이딩해 구현을 바꾸도록
- 극한의 상속(?)

### 장점

- 알고리즘 전체 구조가 한 곳(부모 클래스)에 명확히 정의
- 서브클래스는 수정이 필요한 단계만 오버라이딩해 재정의 가능

### 단점

- 상속을 사용하므로, 유연성(런타임 교체 등)은 전략 패턴보다 떨어질 수 있음
- 단계가 많아지면 부모/자식 간 결합도가 커질 수 있음

### 예제 코드

```tsx
abstract class ImageProcessor {
  // 템플릿 메서드
  public process() {
    this.openFile();
    this.applyFilter();
    this.saveFile();
  }

  protected openFile() {
    console.log("파일 열기");
  }

  protected saveFile() {
    console.log("파일 저장");
  }

  // 훅 메서드(서브클래스가 구현)
  protected abstract applyFilter(): void;
}

// 구체 구현(1) - 흑백 필터
class BlackWhiteProcessor extends ImageProcessor {
  protected applyFilter(): void {
    console.log("흑백 필터 적용");
  }
}

// 구체 구현(2) - 세피아 필터
class SepiaProcessor extends ImageProcessor {
  protected applyFilter(): void {
    console.log("세피아 필터 적용");
  }
}

// 사용 예
const bw = new BlackWhiteProcessor();
bw.process();
// 출력: 파일 열기 -> 흑백 필터 적용 -> 파일 저장

const sepia = new SepiaProcessor();
sepia.process();
// 출력: 파일 열기 -> 세피아 필터 적용 -> 파일 저장
```

---

## 5. 책임 연쇄(Chain of Responsibility) - 그림판 필터 연달아 적용하기

### 왜 쓰이는가?

- 책임 연쇄 패턴(Chain Of Responsibility Pattern, COR)은 클라이어트의 요청에 대한 세세한 처리를 하나의 객체가 몽땅 하는 것이 아닌, 여러 개의 처리 객체들로 나누고, 이들을 사슬(chain)처럼 연결해 집합 안에서 연쇄적으로 처리하는 행동 패턴
- 여러 핸들러(처리 객체)를 체인으로 엮어서, 요청이 들어오면 순차적으로 처리하거나 넘기고 싶을 때
- 예) “여러 필터를 순서대로 적용하는 로직”, “이벤트 버블링처럼 위로 전달” 등

### 핵심 아이디어

- 각 핸들러가 “자신이 처리할 수 있으면 처리”하고, 그렇지 않으면 다음 핸들러로 넘김.
- 핸들러들은 공통 인터페이스(또는 추상 클래스)와 “다음 핸들러” 참조를 가짐.

### 장점

- 핸들러 추가/제거가 쉬움(체인 구조만 수정)
- 각 핸들러가 자신의 책임만 처리하도록 분리

### 단점

- 요청이 끝까지 처리되지 않고 넘겨질 수도 있음(설계에 따라 달라질 수 있음)
- 디버깅 시 어떤 핸들러를 거쳤는지 추적이 복잡할 수 있음

### 예제 코드

```tsx
interface Filter {
  setNext(filter: Filter): Filter;
  handle(imageData: string): string;
}

abstract class AbstractFilter implements Filter {
  private nextFilter: Filter | null = null;

  public setNext(filter: Filter): Filter {
    this.nextFilter = filter;
    return filter;
  }

  public handle(imageData: string): string {
    const processed = this.applyFilter(imageData);
    if (this.nextFilter) {
      return this.nextFilter.handle(processed);
    }
    return processed;
  }

  protected abstract applyFilter(data: string): string;
}

// 구체 필터 (1) : 샤픈
class SharpenFilter extends AbstractFilter {
  protected applyFilter(data: string): string {
    console.log("샤픈 필터 적용");
    return data + "->Sharpened";
  }
}

// 구체 필터 (2) : 블러
class BlurFilter extends AbstractFilter {
  protected applyFilter(data: string): string {
    console.log("블러 필터 적용");
    return data + "->Blurred";
  }
}

// 사용 예
const sharpen = new SharpenFilter();
const blur = new BlurFilter();
sharpen.setNext(blur);

const rawData = "이미지";
const result = sharpen.handle(rawData);
console.log("최종 결과:", result);
// 출력 예:
//   "샤픈 필터 적용"
//   "블러 필터 적용"
//   "최종 결과: 이미지->Sharpened->Blurred"
```

---

## 6. 옵저버(Observer) - 구독하시면 저장 완료될 때 알림드릴게요~

### 왜 쓰이는가?

- 어떤 객체(주체, Subject) 상태가 바뀔 때, 그 변화(이벤트)를 여러 객체(옵저버)에게 알리고 싶을 때
- 예) 그림판에서 “저장 완료” 이벤트가 일어나면, 다른 UI(상태 표시바 등)도 자동 갱신.

### 핵심 아이디어

- Subject가 Observer 목록을 가지고 있다가, 상태 변화 시 전체 Observer에게 ‘알림(notification)’을 보냄
- Observer들은 동일한 인터페이스(`update()`)로 알림을 받음

### 장점

- 느슨한 결합: Subject는 Observer 인터페이스에만 의존하므로, Observer 교체/추가가 쉬움
- 1 대 다(one-to-many) 관계를 손쉽게 관리

### 단점

- Observer가 많아지면 알림이 너무 빈번해질 수 있음
- 순환 참조나 이벤트 폭주 등에 주의 필요 + 잘못 쓰면 디버깅이 매우 빡셈

### 예제 코드

```tsx
interface Observer {
  update(state: string): void;
}

interface Subject {
  attach(observer: Observer): void;
  detach(observer: Observer): void;
  notify(): void;
}

// 주체(Subject)
class SaveManager implements Subject {
  private observers: Observer[] = [];
  private saveState: string = "idle";

  public attach(observer: Observer): void {
    this.observers.push(observer);
  }

  public detach(observer: Observer): void {
    this.observers = this.observers.filter((o) => o !== observer);
  }

  public notify(): void {
    for (const obs of this.observers) {
      obs.update(this.saveState);
    }
  }

  public save() {
    console.log("저장 중...");
    this.saveState = "completed";
    this.notify();
  }
}

// 구독자(Observer)
class StatusBar implements Observer {
  update(state: string): void {
    console.log(`[StatusBar] 저장 상태: ${state}`);
  }
}

// 사용 예
const manager = new SaveManager();
const statusBar = new StatusBar();
manager.attach(statusBar);

manager.save();
// "저장 중..."
// "[StatusBar] 저장 상태: completed"
```

---

## 7. Pub/Sub - 옵저버를 좀 더 확장성 있게

### 왜 쓰이는가?

- Observer 패턴과 유사하지만, “주체(Publisher)와 구독자(Subscriber)가 서로 직접 연결되지 않도록 중간에 브로커(이벤트 버스)가 개입”하는 방식
- 대규모 애플리케이션에서, 특정 이벤트 채널(토픽)을 기반으로 메시지를 발행하고, 구독하는 구조

### 핵심 아이디어

- Publisher는 “이벤트 발행”만 하고, 누가 듣고 있는지는 신경 쓰지 않음
- Subscriber는 자신이 관심 있는 이벤트(토픽)를 구독만 함
- 중간에 EventBus(또는 MessageBroker) 같은 객체가 발행/구독을 중개

### 장점

- 발행자와 구독자가 서로 몰라도 되므로 결합도가 더욱 낮아짐
- 여러 이벤트 종류를 다루기 편리

### 단점

- 이벤트가 어디서 발행되고, 어디서 처리되는지 추적이 어려울 수 있음
- 잘못 설계 시 디버깅 복잡도 ↑ (그치만 누구나 잘못 설계하고 싶어하진 않지.. 잘 알고쓰자)

### 간단 예제 코드

```tsx
class EventBus {
  private subscribers: { [event: string]: ((data: unknown) => void)[] } = {};

  public subscribe(event: string, callback: (data: unknown) => void): void {
    if (!this.subscribers[event]) {
      this.subscribers[event] = [];
    }
    this.subscribers[event].push(callback);
  }

  public publish(event: string, data: unknown): void {
    if (this.subscribers[event]) {
      this.subscribers[event].forEach((cb) => cb(data));
    }
  }
}

// 사용 예
const bus = new EventBus();

// 구독자 A
bus.subscribe("SAVE_COMPLETE", (data) => {
  console.log("A - 저장완료 알림:", data);
});

// 구독자 B
bus.subscribe("SAVE_COMPLETE", (data) => {
  console.log("B - 저장완료 알림:", data);
});

// 퍼블리셔(발행)
bus.publish("SAVE_COMPLETE", { fileName: "test.png" });
// A - 저장완료 알림: { fileName: "test.png" }
// B - 저장완료 알림: { fileName: "test.png" }
```

---

## 8. 중재자(Mediator) - 모든 컨트롤을 중앙집중식으로!

### 왜 쓰이는가?

- 여러 객체가 서로 복잡하게 상호작용할 때, “중재자” 객체가 그 로직(의사소통)을 담당하게 하여 **객체 간 직접 의존을 줄이는** 패턴
- 중재자 패턴은 서로 상호작용하는 오브젝트들을 캡슐화함으로써 느슨한 결합을 유지하기 위해 사용
- 예) 복잡한 UI 폼에서 버튼, 체크박스, 드롭다운 등이 서로 영향을 미치면, 각 컨트롤이 직접 서로 호출하는 대신 Mediator에게 알려 중재하도록

### 핵심 아이디어

- “각 구성요소”는Mediator(중재자)만 알고, 서로 직접 통신하지 않음 → 변경 시 파급효과가 줄어듦
- 중재자가 이벤트, 상태 변화를 중간에서 받아 각자에게 적절히 전달·조정

### 장점

- 객체들 간의 **직접적인 복잡한 참조 관계**가 사라져, 유지보수가 편해짐
- 중재 로직을 한 곳에 모을 수 있어, 수정이 용이

### 단점

- Mediator 클래스가 너무 많은 기능을 담당하면 거대해질 수 있음(“God object” 주의)

### 예제 코드

```tsx
interface Mediator {
  notify(sender: Component, event: string): void;
}

abstract class Component {
  protected mediator: Mediator | null = null;

  setMediator(mediator: Mediator) {
    this.mediator = mediator;
  }
}

class Button extends Component {
  public click() {
    console.log("버튼 클릭");
    if (this.mediator) {
      this.mediator.notify(this, "click");
    }
  }
}

class TextBox extends Component {
  public setText(text: string) {
    console.log(`텍스트박스에 텍스트 설정: ${text}`);
  }
}

class ConcreteMediator implements Mediator {
  private button: Button;
  private textBox: TextBox;

  constructor(b: Button, t: TextBox) {
    this.button = b;
    this.textBox = t;
    this.button.setMediator(this);
    this.textBox.setMediator(this);
  }

  public notify(sender: Component, event: string): void {
    if (sender === this.button && event === "click") {
      console.log("중재자: 버튼이 클릭됨");
      // 버튼 클릭 시 텍스트박스에 뭔가 설정
      this.textBox.setText("버튼 클릭됨!");
    }
  }
}

// 사용 예
const btn = new Button();
const txt = new TextBox();
const mediator = new ConcreteMediator(btn, txt);

btn.click();
// 출력:
//   "버튼 클릭"
//   "중재자: 버튼이 클릭됨"
//   "텍스트박스에 텍스트 설정: 버튼 클릭됨!"
```

---

## 9. 메멘토(Memento) - 그림판 히스토리 만들기

### 왜 쓰이는가?

- 객체의 상태 정보를 가지는 클래스를 따로 생성하여, 객체의 상태를 저장하거나 이전 상태로 복원할 수 있게 해주는 패턴
- 객체의 “이전 상태”를 저장해두었다가 필요 시 복원(Undo)할 수 있게 하고 싶을 때, 원하는 시점의 상태 복원이 가능
- 예) 그림판에서 “작업 히스토리” 쌓아두고 되돌리기(Undo), 다시하기(Redo)를 구현

### 핵심 아이디어

- `Memento` 객체에 복원에 필요한 상태만 저장
- Caretaker(예: 히스토리 매니저)가 Memento를 보관하고, 필요할 때 Originator(실제 객체)에게 돌려줘서 이전 상태로 복원

### 장점

- 원본 객체의 캡슐화를 깨지 않고(내부 상태를 직접 노출하지 않고)도, 상태 스냅샷을 저장 가능
- Undo/Redo 구현이 쉬워짐

### 단점

- 상태가 클 경우, 메멘토가 많이 쌓이면 메모리 비용 증가
- 어떤 시점에 스냅샷을 찍을지 설계 필요

### 예제 코드

```tsx
// Originator
class EditorState {
  private content: string = "";

  public setContent(value: string) {
    this.content = value;
  }

  public getContent() {
    return this.content;
  }

  // 메멘토 생성
  public saveToMemento(): Memento {
    return new Memento(this.content);
  }

  // 메멘토 복원
  public restoreFromMemento(memento: Memento) {
    this.content = memento.getState();
  }
}

// Memento
class Memento {
  private state: string;

  constructor(state: string) {
    this.state = state;
  }

  public getState(): string {
    return this.state;
  }
}

// Caretaker
class HistoryManager {
  private mementos: Memento[] = [];

  public addMemento(m: Memento) {
    this.mementos.push(m);
  }

  public getMemento(index: number): Memento {
    return this.mementos[index];
  }
}

// 사용 예
const editor = new EditorState();
const history = new HistoryManager();

editor.setContent("버전1");
history.addMemento(editor.saveToMemento()); // 스냅샷 저장

editor.setContent("버전2");
history.addMemento(editor.saveToMemento());

editor.setContent("버전3");
console.log("현재:", editor.getContent()); // 버전3

// 되돌리기(Undo) - 1단계 전으로
editor.restoreFromMemento(history.getMemento(1));
console.log("되돌린 후:", editor.getContent()); // 버전2
```

---

## 10. 반복자(Iterator) - 이미 배열에 내장되어 있어요

### 왜 쓰이는가?

- 컬렉션(리스트, 트리, 해시 등)을 순회하는 방법을 “컬렉션 외부”로 추상화하여,**컬렉션 구조가 바뀌어도 순회 방식은 동일 인터페이스**로 쓰도록
- JS/TS에서는 이미 `Symbol.iterator` 등을 통해 내부적으로 Iterator 패턴을 사용
- 복잡하게 얽혀있는 자료 컬렉션(해시, 트리, 그래프…)들을 순회하는 알고리즘 전략을 정의하는 것

### 핵심 아이디어

- 일련의 데이터 집합에 대하여 순차적인 접근(순회)을 지원하는 패턴
- `Iterator` 인터페이스(`next()`, `hasNext()`) 정의 → 구체 컬렉션마다 구현
- 클라이언트는 컬렉션 구조에 상관없이 `Iterator`만 통해 반복

### 장점

- 컬렉션 구조가 다양해도, 반복 방법을 일관되게 쓸 수 있음
- 하나의 컬렉션에 대해 여러 종류의 이터레이터(역순, 조건부 등) 제공 가능

### 단점

- 내장 자료구조(배열, Map 등)를 이미 제공하는 언어 기능이 충분한 경우, 직접 구현 시 복잡도만 증가 가능

### 예제 코드

```tsx
interface Iterator<T> {
  current(): T;
  next(): T;
  hasNext(): boolean;
}

interface Aggregate<T> {
  createIterator(): Iterator<T>;
}

class MyArrayIterator<T> implements Iterator<T> {
  private collection: T[];
  private position: number = 0;

  constructor(collection: T[]) {
    this.collection = collection;
  }

  public current(): T {
    return this.collection[this.position];
  }

  public next(): T {
    this.position++;
    return this.collection[this.position];
  }

  public hasNext(): boolean {
    return this.position < this.collection.length - 1;
  }
}

class MyArray<T> implements Aggregate<T> {
  private items: T[] = [];

  public add(item: T) {
    this.items.push(item);
  }

  public createIterator(): Iterator<T> {
    return new MyArrayIterator<T>(this.items);
  }
}

// 사용 예
const arr = new MyArray<number>();
arr.add(1);
arr.add(2);
arr.add(3);

const it = arr.createIterator();
while (it.hasNext()) {
  console.log(it.current());
  it.next();
}
// 1 -> 2 -> 3
```

---

## 11. 방문자(Visitor) - 내 비즈니스 로직을 외부에서 실행하기

### 왜 쓰이는가?

- 데이터 구조(객체들)가 바뀌지 않고, “연산(로직)”만 자주 바뀌는 상황에서, “로직을 캡슐화한 Visitor”를 만들어 객체 구조를 순회하면서 메서드를 실행하고 싶을 때
- 예) “문서 객체 구조”가 고정되어 있고, “인쇄, 내보내기, 통계 계산” 등 다양한 기능을 외부 로직으로 추가하고 싶을 때

### 핵심 아이디어

- 방문자 패턴에서는 데이터 구조와 처리를 분리
- 데이터 구조 안을 돌아다니는 주체인 방문자를 나타내는 클래스를 준비해서 처리를 맡김. 새로운 처리를 추가하고 싶을 땐 새로운 방문자를 만들고 데이터 구조는 방문자를 받아들이면 됨
- 각 요소(Element)에 `accept(visitor: Visitor)` 메서드를 두고, `visitor.visitXxx(this)`를 호출
- Visitor는 요소별로 다른 메서드(`visitConcreteElementA()`, `visitConcreteElementB()`)를 가지고 있어, 객체의 타입에 따라 로직을 달리 처리

### 장점

- 요소 구조가 안정적일 때, 기능(Visitor) 추가가 쉬움(새 Visitor 클래스를 만들면 되니)
- 복잡한 if/switch 없이, “이중 디스패치(Double Dispatch)”를 구현

### 단점

- 요소 클래스가 많으면 Visitor에 visit 메서드가 늘어나서 복잡해짐
- 새로운 요소 클래스가 추가되면 Visitor 인터페이스도 바꿔야 함(요소 구조가 자주 바뀔 경우는 불리)

### 예제 코드

```tsx
// Visitor 인터페이스
interface Visitor {
  visitCircle(circle: Circle): void;
  visitRectangle(rect: Rectangle): void;
}

// Element 인터페이스
interface Shape {
  accept(visitor: Visitor): void;
}

class Circle implements Shape {
  public radius: number;
  constructor(radius: number) {
    this.radius = radius;
  }
  accept(visitor: Visitor) {
    visitor.visitCircle(this);
  }
}

class Rectangle implements Shape {
  public width: number;
  public height: number;
  constructor(width: number, height: number) {
    this.width = width;
    this.height = height;
  }
  accept(visitor: Visitor) {
    visitor.visitRectangle(this);
  }
}

// 구체 Visitor 1: 면적 계산
class AreaVisitor implements Visitor {
  visitCircle(circle: Circle): void {
    const area = Math.PI * circle.radius ** 2;
    console.log("원 면적:", area);
  }
  visitRectangle(rect: Rectangle): void {
    const area = rect.width * rect.height;
    console.log("사각형 면적:", area);
  }
}

// 구체 Visitor 2: 둘레 계산
class PerimeterVisitor implements Visitor {
  visitCircle(circle: Circle): void {
    const perimeter = 2 * Math.PI * circle.radius;
    console.log("원 둘레:", perimeter);
  }
  visitRectangle(rect: Rectangle): void {
    const perimeter = 2 * (rect.width + rect.height);
    console.log("사각형 둘레:", perimeter);
  }
}

// 사용 예
const shapes: Shape[] = [new Circle(5), new Rectangle(4, 6)];

const areaCalc = new AreaVisitor();
const perimeterCalc = new PerimeterVisitor();

shapes.forEach((shape) => {
  shape.accept(areaCalc);
  shape.accept(perimeterCalc);
});
```
