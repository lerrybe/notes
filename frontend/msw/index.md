## msw 도입

### Why

- 백엔드 / AI 엔지니어 분들과 스키마 협의 후 실제 데이터가 완성되기까지 딜레이
- 이는 실제 API 연결 시 정합성 감소, 개발 속도 감소를 불러옴
  - 기존 mock data 제거 시간
  - 에러 및 로딩에 대한 플로우 처리에 대한 시간
- 사용시 기대효과
  - AI 데이터 연결할 때 정합성을 높일 수 있음
  - 관련 플로우를 더 가다듬을 수 있음
  - 개발 속도 up으로 AI 엔지니어에게 이전보다 빠르게 테스트 환경을 제공할 수 있음
  - 불필요한 AI API 콜을 줄일 수 있음

### What

- 이를 해결하기 위해 [msw](https://mswjs.io/docs/getting-started) 를 장착하고, 현재 evaluate 과정을 mocking 한다.

### How

- **테스트 환경 분리**
  - `npm run dev`로 실행하면 실제 api 응답을 연결한다.
  - `npm run dev:msw`로 실행할 시 mocking된 응답을 연결한다.
  - 현재 컨트롤 가능 요소
    - `{pollCount}` (몇 번 poll 후 응답을 받을 것인지, 스토리지에 저장)
    - `isAIError`: 에러응답/정상응답 중 선택
    - AI 종류 별로 다른 헬퍼함수 주입
    - cf.
      ```tsx
      const resolverMap: {
        [key: string]: (options: Options) => StrictResponse<JsonBodyType>;
      } = {
        "banner-toolkit": bannerToolkitHelper,
        "image-gen": imageGenHelper,
      };
      ```
