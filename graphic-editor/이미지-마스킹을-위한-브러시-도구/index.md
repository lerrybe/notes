# 이미지 마스킹을 위한 브러시 도구 예제

## 💡 배경

- 이미지 처리 및 편집 작업에서 특정 영역을 선택하고 분리하는 기능입니다.
- 이러한 브러시 도구는 (Case Study)
  - 이미지 세그멘테이션 학습 데이터 생성
  - 이미지 합성을 위한 객체 분리
  - 의료 영상 분석의 관심 영역(ROI) 지정
  - UI/UX 디자인의 특정 요소 하이라이트
- 위와 같은 상황에서 활용됩니다.

## 💡 구현원리

- HTML5 Canvas API로 웹 환경에서 구현합니다.

### 1. 이중 캔버스 레이어 구조

- **sourceCanvas**: 원본 이미지 표시용 베이스 레이어
- **brushCanvas**: 브러시 스트로크 캡처용 상위 레이어
- 두 캔버스의 정확한 중첩을 위한 포지셔닝 처리

### 2. Canvas 해상도와 CSS Transform 분리

- 캔버스는 두 가지 크기 속성을 가집니다:

  ```jsx
  // 실제 해상도 설정 (픽셀 데이터 저장 공간)
  sourceCanvas.width = originalImageWidth;
  sourceCanvas.height = originalImageHeight;

  // CSS 표시 크기 (렌더링 크기)
  sourceCanvas.style.width = "100%";
  sourceCanvas.style.height = "100%";
  ```

- 이러한 분리의 장점:
  - 원본 해상도 보존으로 이미지 품질 유지
  - 실제 계산은 원본 해상도, 렌더링은 조정된 크기로 처리하여 성능 최적화
  - Retina 디스플레이 등에서도 선명한 출력

### 3. 브러시 스트로크 알고리즘

- 마우스 이벤트 캡처
- 캔버스 내 실제 좌표 계산

```jsx
const scaleX = canvas.width / canvas.getBoundingClientRect().width;
const realX = mouseX * scaleX;
```

- 이전 좌표와 현재 좌표 연결
- 특히 좌표 연결이 중요한 이유:
  - mousemove 이벤트는 빠른 이동 시 모든 픽셀을 캡처하지 못함
  - 이벤트 포인트 사이를 직선으로 보간하여 부드러운 스트로크 생성
  - 끊어짐 없는 연속적인 선 그리기 가능

### 4. 픽셀 데이터 구조와 마스크 생성

- ImageData 객체의 data 배열은 RGBA 값을 순차적으로 저장:
  ```jsx
  data[i]; // R (0-255) 빨간색 채널
  data[i + 1]; // G (0-255) 녹색 채널
  data[i + 2]; // B (0-255) 파란색 채널
  data[i + 3]; // A (0-255) 알파 채널
  ```
- 마스크 생성 프로세스:
  ```jsx
  // 브러시 스트로크가 있는 부분(알파값 > 0)을 흰색으로 변환
  if (brushData.data[i + 3] > 0) {
    extractData.data[i] = 255; // R = 255 (흰색)
    extractData.data[i + 1] = 255; // G = 255 (흰색)
    extractData.data[i + 2] = 255; // B = 255 (흰색)
    extractData.data[i + 3] = 255; // A = 255 (완전 불투명)
  }
  // 브러시 스트로크가 없는 부분은 기본값 0 유지 (검정색, 완전 불투명)
  ```
- 마스크 특성:
  - 브러시 영역: RGB(255, 255, 255) - 흰색
  - 비브러시 영역: RGB(0, 0, 0) - 검정색
  - 모든 픽셀의 알파값은 255로 설정 (완전 불투명)
  - 결과물은 이진 마스크 (픽셀 값이 0(검정) 또는 255(흰색)만 존재)로, 이미지 세그멘테이션에 바로 활용 가능
    - 후처리가 용이하고, 데이터 크기 최소화 가능

## 💡 코드 구현

### 1. 캔버스 크기 조정

```jsx
const maxWidth = window.innerWidth * 0.9;
const maxHeight = window.innerHeight * 0.7;
const imageRatio = width / height;

// 이미지 비율 유지하면서 최대 크기 계산
if (imageRatio > containerRatio) {
  newWidth = maxWidth;
  newHeight = newWidth / imageRatio;
} else {
  newHeight = maxHeight;
  newWidth = newHeight * imageRatio;
}
```

### 2. 브러시 스트로크 처리

```jsx
// 실제 캔버스 크기와 표시 크기의 비율 계산
const scaleX = brushCanvas.width / rect.width;
const scaleY = brushCanvas.height / rect.height;

// 마우스 좌표를 실제 캔버스 좌표로 변환
const x = (e.clientX - rect.left) * scaleX;
const y = (e.clientY - rect.top) * scaleY;
```

### 3. 마스크 데이터 처리

```jsx
// 브러시 캔버스의 픽셀 데이터 가져오기
const brushData = brushCtx.getImageData(0, 0, width, height);
// 추출 캔버스 생성 및 초기화 (검정 배경)
const extractData = extractCtx.getImageData(0, 0, width, height);

// 브러시 영역 마스킹
for (let i = 0; i < brushData.data.length; i += 4) {
  if (brushData.data[i + 3] > 0) {
    // 알파값 체크
    // RGBA 모두 255로 설정 (흰색)
    extractData.data[i] = 255;
    extractData.data[i + 1] = 255;
    extractData.data[i + 2] = 255;
    extractData.data[i + 3] = 255;
  }
}
```

## 💡 구현 시 주의사항

- 브라우저 리사이징 시 캔버스 크기와 내용 유지를 위한 이벤트 처리 필요
- 터치 이벤트 미구현으로 모바일 환경 지원이 제한적 (TBD)
- 브러시 크기 변경 시 실시간 UI 업데이트 구현 필요
- 대용량 이미지 처리 시 메모리 관리와 성능 최적화 고려 (TBD)
- 픽셀 데이터 조작 시 4바이트 단위(RGBA)로 처리해야 함
- 캔버스 좌표 계산 시 CSS 변환과 실제 해상도 차이 고려 필수

## 💡 Playground (연습예제)

- https://codepen.io/lerrybe/pen/gbYwXpG
