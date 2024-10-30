## SVG

- SVG는 웹 환경의 그래픽 에셋을 위한 형식
- 벡터 그래픽은 다양한 웹 환경에서 동일한 경험을 제공하기 위해 사용
- SVG는 벡터 기반 그래픽을 정의할 수 있으며, 이를 이용하여 다양한 그래픽 효과를 만들 수 있음
- SVG는 **Scalable Vector Graphics**의 약자이며, XML 기반

## SVG 필터

- SVG 필터는 그래픽 요소에 특별한 효과를 적용하기 위한 선언적인 필터 효과이다.
- 필터는 Source Graphic에 적용되어 수정된 그래픽을 만들어내는 일련의 작업이다.
- 각 필터는 여러 **필터 프리미티브(filter primitive)**의 조합으로 이루어져 있다.
- SVG 필터는 `<filter>` 요소를 사용하여 정의하며, `<feGaussianBlur>`, `<feColorMatrix>`, `<feOffset>` 같은 필터 프리미티브로 구성된다.
- 필터를 특정 대상에 적용하려면, 해당 그래픽 요소의 `filter` 속성에 필터의 ID를 참조한다. 이는 해당 요소에 필터에서 정의한 명령어 집합을 적용하라는 지시이다.

```html
<svg>
  <filter id="filter">
    <!-- 효과 생성 -->
  </filter>
</svg>
```

```css
.circle {
  background: red;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  filter: url("#filter");
}
```

- 필터는 그 자체로 렌더링되지 않는 요소이므로, `<defs>` 내부에 선언하는 것을 권장한다.
- 필터 안에는 다음과 같은 요소들이 들어갈 수 있다:
  - **설명 요소(descriptive elements)**: 필터에 대한 설명이나 메타데이터를 포함한다.
  - **필터 프리미티브 요소(filter primitive elements)**: 실제로 효과를 만들어내는 요소이다. 주요 종류로는 `feBlend`, `feColorMatrix`, `feComponentTransfer`, `feComposite`, `feGaussianBlur` 등이 있다.
  - **애니메이션 요소(animate, set)**: 필터의 속성을 시간에 따라 변화시켜 애니메이션 효과를 준다.

### `result`와 `in` 속성

- SVG 필터의 하위 요소는 위에서부터 순차적으로 적용되며, 필터를 다양하게 조합하기 위해 `result`와 `in` 속성을 사용한다.
- `result`는 해당 필터 프리미티브의 결과에 이름을 지정하는 역할을 한다.
- `in`은 해당 필터 프리미티브가 적용될 입력 소스를 지정하는 특성으로, 기존 `result`로 선언된 이름을 사용하거나 `SourceGraphic` 값을 사용해 원본 이미지를 참조한다. `in`이 생략되면 자동으로 상위 요소에서 필터 결과를 참조한다.

```html
<filter id="exampleFilter">
  <feGaussianBlur in="SourceGraphic" stdDeviation="5" result="blurResult" />
  <feOffset in="blurResult" dx="10" dy="10" />
</filter>
```

- 위 코드는 원본 이미지를 흐리게 만든 후, 그 흐린 이미지를 10px만큼 이동시키는 효과이다.

### `filterUnits`와 `primitiveUnits`

- `filterUnits`는 `<filter>` 요소의 `x`, `y`, `width`, `height` 속성의 좌표 기준을 결정한다.
- `primitiveUnits`는 필터 내부의 필터 프리미티브 요소들의 `x`, `y`, `width`, `height` 속성의 좌표 기준을 결정한다.
- 두 속성 모두 `objectBoundingBox`와 `userSpaceOnUse` 값을 가질 수 있다.

#### `filterUnits`

1. **`objectBoundingBox`** (기본값)

   - 대상 객체의 경계 박스(bounding box)를 기준으로 한다.
   - 좌표와 길이 값을 0~1 사이의 백분율 값으로 해석한다.
   - **예시**:

     ```svg
     <filter id="myFilter" filterUnits="objectBoundingBox" x="0" y="0" width="1" height="1">
       <!-- 필터 내용 -->
     </filter>
     ```

     - 이 경우, 필터 영역은 대상 객체 전체에 해당한다.

2. **`userSpaceOnUse`**

   - 상위 SVG 좌표 공간을 기준으로 한다.
   - 좌표와 길이 값을 실제 단위(px 등)로 해석한다.
   - **예시**:

     ```svg
     <filter id="myFilter" filterUnits="userSpaceOnUse" x="0" y="0" width="200" height="200">
       <!-- 필터 내용 -->
     </filter>
     ```

     - 이 경우, 필터 영역은 SVG 좌표 공간에서 (0,0) 위치부터 너비와 높이 200px 영역이다.

#### `primitiveUnits`

1. **`userSpaceOnUse`** (기본값)

   - 필터 프리미티브의 좌표가 상위 SVG 좌표 공간을 기준으로 한다.
   - 좌표와 길이 값을 실제 단위(px 등)로 해석한다.
   - **예시**:

     ```svg
     <filter id="myFilter" primitiveUnits="userSpaceOnUse">
       <feOffset dx="10" dy="10" />
     </filter>
     ```

     - `<feOffset>`의 `dx`, `dy` 값은 각각 10px을 의미한다.

2. **`objectBoundingBox`**

   - 대상 객체의 경계 박스를 기준으로 한다.
   - 좌표와 길이 값을 0~1 사이의 백분율 값으로 해석한다.
   - **예시**:

     ```svg
     <filter id="myFilter" primitiveUnits="objectBoundingBox">
       <feOffset dx="0.1" dy="0.1" />
     </filter>
     ```

     - `<feOffset>`의 `dx`, `dy` 값은 대상 객체의 너비와 높이의 10%를 의미한다.

#### 종합 예시

```svg
<svg width="200" height="200">
  <defs>
    <filter id="myFilter"
            x="0" y="0" width="1" height="1"
            filterUnits="objectBoundingBox"
            primitiveUnits="objectBoundingBox">
      <feGaussianBlur in="SourceGraphic" stdDeviation="0.05" />
    </filter>
  </defs>

  <rect x="50" y="50" width="100" height="100" fill="blue" filter="url(#myFilter)" />
</svg>
```

- **설명**:
  - `filterUnits="objectBoundingBox"`:
    - 필터의 `x`, `y`, `width`, `height`가 대상 `<rect>`의 경계 박스를 기준으로 해석된다.
    - 즉, 필터는 대상 객체 전체 영역에 적용된다.
  - `primitiveUnits="objectBoundingBox"`:
    - `<feGaussianBlur>`의 `stdDeviation` 값이 대상 객체의 크기에 비례하여 적용된다.
    - `stdDeviation="0.05"`는 대상 객체의 너비와 높이의 5%에 해당하는 블러 효과를 의미한다.

### 요약

- `filterUnits`는 필터 전체 영역의 좌표 기준을 설정하며, 기본값은 `objectBoundingBox`이다.
  - `objectBoundingBox`: 대상 객체 기준 백분율 좌표.
  - `userSpaceOnUse`: SVG 좌표계 기준 절대 좌표.
- `primitiveUnits`는 필터 프리미티브 요소들의 좌표 기준을 설정하며, 기본값은 `userSpaceOnUse`이다.
  - `userSpaceOnUse`: SVG 좌표계 기준 절대 좌표.
  - `objectBoundingBox`: 대상 객체 기준 백분율 좌표.
- **사용 목적**:
  - `objectBoundingBox`를 사용하면 대상 객체의 크기에 비례하여 효과를 적용할 수 있어, 다양한 크기의 객체에 일관된 효과를 줄 수 있다.
  - `userSpaceOnUse`를 사용하면 절대적인 좌표와 크기로 효과를 적용할 수 있어, 정밀한 제어가 가능하다.

## 추가 고려사항

### 렌더링 과정에서의 래스터화

- SVG는 벡터 데이터를 기반으로 하지만, 화면에 표시될 때는 렌더링 엔진이 벡터 데이터를 픽셀 단위의 래스터 이미지로 변환합니다. 이 과정에서 필터 효과도 함께 적용된다. 중요한 점은, 이 래스터화 과정은 렌더링 시점에만 발생하며, 원본 SVG 파일 내의 벡터 데이터는 그대로 유지된다는 것임.

- 자 이제 동적 처리 시의 재계산을 생각해보면, 그래픽 요소가 동적으로 변경되거나(예: 애니메이션, 크기 조정 등) 화면 해상도가 바뀌면 브라우저는 원본 벡터 데이터를 기반으로 다시 렌더링한다. 이때 필터 효과도 벡터 데이터를 바탕으로 새로운 픽셀 값으로 다시 계산되어 적용된다. 따라서 확대나 축소를 해도 필터 효과는 그에 맞게 재계산되어 품질이 유지된다.

- 이 때 필터 효과가 복잡하거나 동적 변화가 빈번하면, 브라우저가 매번 재계산해야 하므로 성능에 영향을 줄 수 있다. 이 때는 캐시를 사용하거나 추가 최적화 작업을 거쳐야 성능에 문제가 없다.

## 실습 및 참고 자료

- **Playground**: [SVG Filters Playground](https://svgfilters.client.io/)

## 참고 문헌

- [디자이너는 이해하지 못하는, 개발자는 무심한](https://velog.io/@whrudtjr/series/%EB%94%94%EC%9E%90%EC%9D%B4%EB%84%88%EB%8A%94-%EC%9D%B4%ED%95%B4%ED%95%98%EC%A7%80-%EB%AA%BB%ED%95%98%EB%8A%94-%EA%B0%9C%EB%B0%9C%EC%9E%90%EB%8A%94-%EB%AC%B4%EC%8B%AC%ED%95%9C)
- [[MDN] filter](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/filter)
- [[MDN] animate](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/animate)
- [[MDN] Filter Effects](https://developer.mozilla.org/en-US/docs/Web/SVG/Tutorial/Filter_effects)
