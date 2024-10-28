## Error boundary란?

- 컴포넌트 트리가 깨지는 대신 자식 컴포넌트 트리에서 에러를 잡아내고, fallback UI를 보여주는 React 컴포넌트
- 리액트 16버전에서 추가된 기능으로, 하위 컴포넌트에서 발생한 에러를 잡아주는 컴포넌트 역할을 수행
- 선언적인 오류처리가 가능
  - 선언적인 오류처리가 왜 좋은데?
    - 에러 흐름을 중앙화하거나 추상화하여, 처리 로직이 코드 전반에 분산되지 않음.
    - 에러 처리 로직을 더 간결하고 재사용 가능하게 만들어 코드 가독성이 높아짐.
    - 상태 관리나 비동기 로직이 좀 더 간단하게 작성될 수 있음.
  * 그렇지만, 개인의 취향 문제도 껴있다고 생각한다.
- 에러는 늘 존재하기 마련이지만, 이를 사용자에게 어떤 UI로 풀어가고 어떤 대처방안을 가이드하는지에 따라 사용자가 느낄 수 있는 경험의 차이 폭이 큰 것 같다.

### 목적

- 컴포넌트 일부에서 발생한 에러가 전체 어플리케이션을 중단시키면 안된다.
- 그렇기에 Fallback UI를 보여줌으로써 중단을 방지한다.

### 구현체 톺아보기

```typescript
# react-error-boundary

const initialState = {
  didCatch: false, // didCatch는 에러를 잡았는지 여부를 저장
  error: null // 에러 객체
};

class ErrorBoundary extends react.Component {
  constructor(props) {
    super(props);
    this.resetErrorBoundary = this.resetErrorBoundary.bind(this);
    this.state = initialState;
  }

  static getDerivedStateFromError(error) {  // 에러가 발생했을 때 상태를 업데이트할 수 있는 정적 생명주기 메서드
    return { didCatch: true, error };
  }

  resetErrorBoundary() {  // 에러 상태를 초기화하는 함수
    // ...
  }

  componentDidCatch(error, info) { // 컴포넌트가 에러를 잡았을 때 호출되는 메서드
    var _this$props$onError, _this$props2;
    // onError prop이 있을 경우 에러와 정보를 전달하며 호출
    (_this$props$onError = (_this$props2 = this.props).onError) === null || _this$props$onError === void 0 ? void 0 : _this$props$onError.call(_this$props2, error, info);
  }

  componentDidUpdate(prevProps, prevState) { // 컴포넌트가 업데이트될 때 호출되는 메서드
    const { didCatch } = this.state;
    const { resetKeys } = this.props;

    if (didCatch && prevState.error !== null && hasArrayChanged(prevProps.resetKeys, resetKeys)) {
      var _this$props$onReset2, _this$props3;
      (_this$props$onReset2 = (_this$props3 = this.props).onReset) === null || _this$props$onReset2 === void 0 ? void 0 : _this$props$onReset2.call(_this$props3, {
        next: resetKeys,
        prev: prevProps.resetKeys,
        reason: "keys"
      });
      this.setState(initialState); // 상태를 초기 상태로 리셋
    }
  }

  // 렌더링 함수로, 에러가 발생했을 때 보여줄 fallback UI를 결정
  render() {
    const { children, fallbackRender, FallbackComponent, fallback } = this.props;
    const { didCatch, error } = this.state; // 현재 상태에서 에러 발생 여부와 에러 객체를 가져옴
    let childToRender = children; // 기본적으로는 자식 컴포넌트를 렌더링
    if (didCatch) { // 에러가 발생한 경우
      const props = {
        error, // 에러 객체 전달
        resetErrorBoundary: this.resetErrorBoundary // 에러 리셋 함수 전달
      };
      if (typeof fallbackRender === "function") { // fallbackRender 함수가 있으면 호출
        childToRender = fallbackRender(props);
      } else if (FallbackComponent) { // FallbackComponent가 있으면 해당 컴포넌트를 렌더링
        childToRender = react.createElement(FallbackComponent, props);
      } else if (fallback === null || react.isValidElement(fallback)) { // fallback 엘리먼트가 유효하면 렌더링
        childToRender = fallback;
      } else {
        throw error; // fallback이 없으면 에러를 다시 던진다.
      }
    }
    // 에러 바운더리의 상태와 리셋 함수를 context로 제공하며 컴포넌트를 렌더링
    return react.createElement(ErrorBoundaryContext.Provider, {
      value: {
        didCatch,
        error,
        resetErrorBoundary: this.resetErrorBoundary
      }
    }, childToRender);
  }
}

```

### 에러 바운더리가 포착할 수 없는 이슈들

- 기본 기조
  - 실행 컨텍스트는 함수 실행에 필요한 환경을 관리하는 객체로, 에러가 발생하면 상위 컨텍스트로 전파. 비동기 코드나 특정 상황에서는 에러가 상위 컨텍스트에서 처리되지 않아 포착되지 않음. ErrorBoundary는 실행 컨텍스트 내에서 동작하며, 그 범위에 있는 에러만 포착 가능하다.

* 이벤트 핸들러
  - React의 이벤트 핸들링 방식(이벤트 위임) 때문에, ErrorBoundary가 이벤트 핸들러에서 발생한 에러를 잡을 수 없다. 이벤트는 root에서 처리되므로 ErrorBoundary의 컨텍스트를 벗어난다.
* 비동기적 코드 (setTimeout, requestAnimationFrame 콜백)
  - setTimeout, axios와 같은 비동기 코드 내 에러는 ErrorBoundary가 감지할 수 없다. 비동기 코드는 ErrorBoundary의 실행 컨텍스트가 아닌 곳에서 발생하기 때문
* 서버 사이드 렌더링
  - 서버에서 발생한 에러는 클라이언트 사이드에서 상태 변화를 처리하는 getDerivedStateFromError가 실행되지 않기 때문에 ErrorBoundary가 잡을 수 없다.
* 자식에서가 아닌 에러 경계 자체에서 발생하는 에러
  - ErrorBoundary 내부에서 발생한 에러는 상위 컴포넌트의 ErrorBoundary에서만 잡을 수 있다. 마치 try/catch 문에서 발생한 에러가 다시 던져지면 다른 try/catch에서 처리해야 하는 것과 같음

## Reference

- https://ko.legacy.reactjs.org/docs/error-boundaries.html
- https://happysisyphe.tistory.com/66
- https://velog.io/@apro_xo/React-Error-Boundary
- https://jikor1st.tistory.com/23
