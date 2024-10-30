## 프론트엔드에서의 오류

### 작업 목표

- 오류 상황에서의 빠른 정상화

### FE에서 발생할 수 있는 오류

- JavaScript 런타임 에러: ReferenceError, SyntaxError 등
- 비동기 에러: Promise나 Async/Await 호출 중 오류
- 네트워크 에러: API 요청 실패, HTTP 4xx/5xx 응답
- UI 렌더링 에러: 잘못된 props나 상태 변경으로 인한 렌더링 실패
- 사용자 입력 에러: 입력값 유효성 검사 실패
- 브라우저 호환성 이슈: 특정 브라우저에서 발생하는 문제
- 리소스 로드 실패: 이미지, 스크립트, CSS 등의 로드 문제

* 데이터 영역
* 화면 영역
* 그리고 예상할 수 없는 네트워크 이슈
* 특정 브라우저 버전, 단말기 OS 업데이트 같은 외부 요인에 의한 오류
  - 크롬의 쿠키 정책 변경
  - Safari 특정 버전의 Indexed DB API 버그
* 예상치 못한 런타임 오류

* 자바스크립트 빌트인 에러 객체

  - ReferenceError
  - SyntaxError
  - TypeError
  - RangeError
  - ...
  - 빌트인 에러 객체를 상속한 새로운 에러 객체를 생성해 오류를 핸들링하고 Sentry에 전송하자! (의도가 담기게)

* 다른 관점에서
  - 문법 오류
  - API 관련 오류
  - third party library에서 발생하는 오류
  - 비즈니스 로직 내에서 발생하는 오류
  - unexpected 오류 (예상 범위 밖)

## 모니터링

### 중요 요소

- 이슈 파악 (이슈 식별) - 타이블, 그룹화, 중요도

### Sentry

- 실시간 로그 취합 및 분석 도구이자 모니터링 플랫폼

* 얻을 수 있는 정보
  - Exception & Message: 이벤트 로그 메시지 및 코드 라인 정보 (source map 설정을 해야 정확한 코드라인을 파악 가능)
  - Device: 이벤트 발생 장비 정보 (name, family, model, memory 등)
  - Browser: 이벤트 발생 브라우저 정보 (name, version 등)
  - OS: 이벤트 발생 OS 정보 (name, version, build, kernelVersion 등)
  - Breadcrumbs: 이벤트 발생 과정
  - Context 기능으로 기본적으로 제공되는 정보 외에 특정 이벤트에 대한 추가 정보를 수집할 수도 있음

### Stack Trace

- stack trace는 코드에서 에러가 발생했을 때, 해당 에러가 발생한 위치와 그 에러에 이르게 된 함수 호출의 순서를 추적하여 제공하는 정보.
- 이를 통해 개발자는 어떤 함수에서 에러가 발생했는지, 그리고 어떤 경로를 통해 그 함수가 호출되었는지 알 수 있음

### 어떤 에러를 찍을 것인가?

- 에러 추적에 필요한 것 (예시)

  - 에러 메시지 (Error Message): 에러의 원인을 설명하는 간단한 메시지
  - 스택 트레이스 (Stack Trace): 에러가 발생한 함수 호출 경로와 위치(파일, 라인 번호)
  - 발생 위치 (Source Location): 에러가 발생한 파일 이름과 코드 라인 번호
  - 발생 환경 (Environment): 에러가 발생한 환경(production, development, device, browser, os 등)

- 에러 레벨

  - info
  - warn
  - error

### 에러 캡쳐하기

- captureException
  - error 객체나 문자열 전송 가능
- captureMessage
  - 문자열 전송 가능

### 데이터 추적하기

- scope 단위로 이벤트 데이터를 얻어낼 수 있음
- configureScope
- withScope
- context
- tag

### 에러 노티하기

- 이벤트마다 level을 설정 가능
- fingerprint -> fingerprint가 동일한 이벤트들은 자동으로 하나의 이슈로 그룹화
- 담으면 좋은 정보
  - 응급도, 유저 정보, method 및 url, headers, data (request, response)

### 언제 오류를 쌓는가?

- 어느 상황부터를 장애 상황으로 보아야 하는가?

### reference

- https://tech.kakaopay.com/post/frontend-sentry-monitoring/
