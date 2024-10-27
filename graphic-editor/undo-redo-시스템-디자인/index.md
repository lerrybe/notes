## 들어가며

- 이미지 에디터를 개발할 때 가장 중요한 기능 중 하나는 바로 Undo/Redo(실행 취소/다시 실행) 기능이다.
- 사용자가 실수로 작업을 삭제하거나 변경했을 때, 이전 상태로 돌아갈 수 있는 기능은 정말! 필수적이다. 😅

* 이런 Undo/Redo 기능을 구현하는 방법에는 여러 가지가 있는데, 대표적으로는 다음과 같은 방식이 있을 것이다.

> 1. 스냅샷(Snapshot) 방식
> 2. 행위에 대한 기록 및 추적 (Action Record) 방식

- 사실 1, 2번이 유사한 방식인데, 스냅샷을 찍는 것처럼 전체 상태를 저장해 복원하고 재실행하느냐 혹은 특정 델타 혹은 액션을 저장하고 복원 혹은 재실행 하느냐의 차이라고 생각한다. undo/redo가 필요한 모든 상태(행위)를 관리하기 위해서는, 특정 객체의 연결리스트 혹은 스택으로 관리하면 된다. (이하 history라 칭)

* 구현할 수 있는 방법은 무궁무진하고 디테일도 다 다르겠지만, 일단 이 두 가지 방법을 기준으로 살펴보자.

## 스냅샷 (Snapshot) 방식

### 구현 (수도코드)

- 스냅샷 방식은 객체의 상태를 특정 시점에 저장하는 방식
- 이렇게 저장된 상태 전체를 history에 넣어 Undo/Redo를 구현할 수 있다. 주요 함수는 다음과 같음

* 히스토리 관리

  ```typescript
  constructor() {
    this.undoHistory = [];
    this.redoHistory = [];
  }
  ```

  - 사용자가 어떤 작업을 수행할 때마다 해당 상태가 스택에 푸시(push)
  - 최근의 상태가 항상 스택의 맨 위에 위치

* `saveSnapshot` 함수

  ```typescript
  // example
  saveSnapshot() {
    const state = JSON.stringify(this.toJSON());
    this.undoHistory.push(snapshot);
  }
  ```

  1. **현재 상태를 직렬화해 기록 (스냅샷찍기)**

  - 예) `this.toJSON()`을 통해 객체의 현재 상태를 JSON 형태로 변환
  - 혹은 현재 상태를 식별할 수 있는 어떤 형태든 좋음

  2. **현재 상태를 저장 **

  - 직렬화된 상태를 undoHistory (stack)에 푸시
  - 이 스택은 사용자가 이전 상태로 돌아갈 수 있게 하는 역할을 함

* `undo` 함수

  ```typescript
  undo() {
    if (this.undoHistory.length > 1) {
      const currentState = this.undoHistory.pop();
      this.redoHistory.push(currentState);
      const prevState = this.undoHistory[this.undoHistory.length - 1];
      this.loadFromJSON(JSON.parse(prevState));
      this.renderAll();
    }
  }
  ```

  1. **상태 검사**
     - undoHistory에 두 개 이상의 항목이 있을 때만 undo가 가능
     - 즉, 현재 상태와 이전 상태가 모두 있어야 함
  2. **상태 이동**
     - 현재 상태를 redoHistory으로 이동 (pop 후 push).
     - 이로써 사용자가 redo를 원할 경우 다시 현재 상태로 돌아갈 수 있게 함
  3. **이전 상태 복원**
     - undoHistory에서 마지막 상태를 가져와 렌더함

* `redo` 함수
  ```typescript
  redo() {
    if (this.redoHistory.length > 0) {
      const state = this.redoHistory.pop();
      this.undoHistory.push(state);
      this.loadFromJSON(JSON.parse(state));
      this.renderAll();
    }
  }
  ```
  1. **상태 검사**
     - redoHistory에 항목이 있는지 확인하고, 항목이 있다면 redo를 진행
  2. **상태 이동**
     - redoHistory에서 최신 상태를 제거하고(pop), 이 상태를 undoHistory에 다시 푸시
     - 이는 redo 후에 다시 undo할 수 있게 만들기 위함
  3. **상태 복원**
     - 뽑은 상태를 기반으로 화면을 그림

### 장단점

- 장점
  - 구현이 비교적 간단하고 직관적
  - 상태 전체를 저장하기 때문에 복원이 비교적 쉬움
- 단점
  - 상태 전체를 저장하기 때문에 메모리 사용량이 많을 수 있음
  - 대규모 시스템에서는 상태 저장과 복원에 상당한 시간이 걸릴 수 있음

실제로 구현 중간에 snapshot 찍어서 얼리는 과정 (예를 들어 JSON.stringify)에서 object 30개만 되어도 성능상 이슈가 있었음

이러한 단점을 개선하기 위해, 상태 전체가 아닌 변경된 부분(Delta)만 저장하는 방법을 고려해볼 수도 있고, 또는 저장해야 할 상태를 최소화하기 위해 필요한 속성만 선별하여 저장하는 방법도 있을 것 같음. 하지만 완벽한 그래픽 복구를 위해서는 결국 많은 속성이 필요할 수 있을 것 같아 그 또한 아쉬울 수 있을 것 같음

## 행위에 대한 추적 (Action Record) 방식

- 행위 추적 방식은 사용자의 모든 행동을 별도의 객체(Action)로 저장하고, 이 객체들을 관리하는 방식.
- 각 Action 객체는 해당 행동을 다시 실행(Redo)하거나 취소(Undo)하는 방법을 알고 있음. (각각의 action 객체는 action에 대한 역(Inverse)을 알고 있다!) 아래는 구현과 관련된 수도코드이다.

### Action 인터페이스

```typescript
// ex. Action class
abstract class Action {
  constructor() {}
  undo() {} // inverse action
  redo() {} // 원래 action
}
```

- 모든 행위(Action)들은 이 인터페이스를 상속받아 Undo/Redo 동작을 각각 구현한다.
- Undo는 반드시 Redo가 한 행위를 반대로 되돌려야 한다.
- 예를 들어 `ActionMove`의 경우, 어떤 대상을 dx, dy만큼 이동하는 행위이고, Redo에서 dx, dy만큼을 더했다면, Undo에서는 dx, dy만큼을 빼도록 구현한다.

### Action 인터페이스를 상속받은 실제 액션들 (예: ActionMove, ActionCopy 등)

```typescript
// ex. ActionMove
class ActionMove extends Action {
  constructor(target, dx, dy) {
    super();
    this.target = target;
    this.dx = dx;
    this.dy = dy;
  }
  undo() {
    this.target.move(-this.dx, -this.dy);
  }
  redo() {
    this.target.move(this.dx, this.dy);
  }
}
```

- `Action` 인터페이스를 상속받아 Undo/Redo를 구현
- 사용자가 메뉴에서 복사, 붙여넣기 행위를 한다면 각각 `ActionCopy`, `ActionPaste` 객체를 생성하여 `ActionControl`의 `execute`에 전달

* `ActionControl.execute()`의 내부에서는 전달받은 `Action` 인스턴스의 `redo` 함수를 호출해주는 역할을 함

### Action들을 저장/관리해주는 컨트롤러 (ActionControl)

```typescript
// ex. ActionControl
class ActionControl {
  constructor() {
    this._undoHistory = []; // Undo 스택
    this._redoHistory = []; // Redo 스택
  }

  // 액션을 실행하고 undo 스택에 추가
  execute(action: Action) {
    action.redo();
    this._undoHistory.push(action);
  }

  // 가장 최근에 실행된 액션을 undo하고 redo 스택에 추가
  undo() {
    if (this._undoHistory.length > 0) {
      const action = this._undoHistory.pop();
      action.undo();
      this._redoHistory.push(action);
    }
  }

  // Redo 스택에서 액션을 꺼내 다시 redo
  redo() {
    if (this._redoHistory.length > 0) {
      const action = this._redoHistory.pop();
      action.redo();
      this._undoHistory.push(action); // Redo 후 해당 액션을 다시 undo 스택에 추가
    }
  }
}
```

- ActionControl은 생성된 모든 Action 인스턴스들을 자료구조(연결 리스트, 스택 등)에 지속적으로 저장

* execute() 호출 시 Action의 redo를 실행하고, 해당 Action 인스턴스를 \_undoHistory 컨테이너에 저장
* 외부에서 되돌리기(Undo) 기능이 필요하다면 \_undoHistory 컨테이너에서 가장 최근의 Action 인스턴스를 꺼내와 undo를 호출하면 됨
* 간단히 말해, 행위를 되돌리기 위해 Inverse Action을 다시 실행한다고 생각하면 됨!

### 사용 예시

```typescript
// 사용 예시
const target = new Shape();
const actionControl = new ActionControl();

actionControl.execute(new ActionMove(target, 10, 20));
// 실행 취소
actionControl.undo();
// 다시 실행
actionControl.redo();
```

### 장단점

- 단점
  - 모든 행위에 대해 Undo/Redo 로직을 일일이 구현해야 하므로 코드량이 늘어날 수 있.
  - 복잡한 행위의 경우 Undo/Redo 로직을 정의하기 어려울 수 있다.
    - -> **이런 걸 극복하기 위해 만약 json으로 관리되고 있는 객체면 json diff를 통해 액션을 추적 가능!** (현재 서비스에서 사용하고 있는 방식이다.)
- 장점
  1. **메모리 효율성**: Action 기반 방식은 전체 상태의 스냅샷을 저장하는 대신에, 사용자의 구체적인 행위만을 기록한다. 예를 들어, 그림 편집 프로그램에서 펜으로 선을 그은 행위는 단지 그 선의 경로와 속성만을 저장하면 된다. 이는 전체 스냅샷을 저장하는 것보다 훨씬 적은 메모리를 사용하게될 수 있음
  2. **성능 최적화**: 스냅샷 방식은 상태를 저장할 때마다 전체 상태를 복사해야 하기 때문에 시간이 많이 소요될 수 있다. 반면, action 기반 방식은 특정 행위를 빠르게 기록하고, 이를 또한 신속하게 실행 취소하거나 다시 실행할 수 있다. 이는 특히 사용자의 행동이 많고 복잡할 때 성능 이점을 제공함.
  3. **확장성**: Action을 객체로 관리함으로써, 새로운 종류의 행동을 시스템에 쉽게 추가할 수 있다. 각 Action 객체는 자신의 undo와 redo 로직을 캡슐화하기 때문에, 시스템 전체를 변경하지 않고도 새로운 기능을 통합할 수 있다.
  4. **세밀한 제어**: 사용자의 각각의 행위를 개별적으로 추적함으로써, 더 세밀한 수준에서 실행 취소 및 다시 실행을 제어할 수 있다. 예를 들어, 사용자가 여러 행위를 그룹화하거나 특정 행위만을 선택적으로 실행 취소하고 싶을 때 유용한 듯 하다. → 또한 사용자의 행위를 추적할 수 있다는 것은 이에 대한 데이터를 모을 수도 있다는 것이기도 함. (오예!)
