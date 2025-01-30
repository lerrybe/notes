# 회전 행렬을 활용한 그룹 해제 구현

## 💡 배경

- 캔버스 기반 그래픽 에디터를 개발하다 보면, 여러 객체를 그룹화하고 해제하는 기능이 필요합니다.
- 특히 그룹 해제 시에는 각 객체의 위치, 크기, 회전 등의 변환(transform) 정보를 올바르게 계산해야 합니다.
- 그룹 내 객체들은 부모 그룹을 기준으로 한 상대적인 위치와 변환 값을 가지고 있고, 그룹을 해제할 때는 이 상대적인 값들을 캔버스의 절대 좌표로 변환해야 하며, 이 과정에서 회전 행렬(rotation matrix)을 활용하여 올바르게 복구할 수 있습니다.
- 특히 n-depth의 그룹을 커버하기 위해 해당 작업이 필요하다고 생각했습니다.

### 1. 기존 코드의 문제점

- 단순히 부모의 위치에 상대 위치를 더하는 방식
- 회전이 있을 때 좌표계가 회전된다는 것을 고려하지 않음
- 결과적으로 회전된 그룹 내의 객체들이 잘못된 위치에 배치됨

### **2. 새 코드의 해결 방법**

- 회전 행렬을 사용하여 좌표계 변환을 정확하게 처리
- 부모 그룹의 회전을 고려하여 자식 객체의 위치를 계산

## 💡 구현 원리

### 1. 변환 계산의 기본 개념

- 그룹 해제 시 각 객체의 최종 위치와 변환값을 계산하기 위해서는 다음 세 가지 요소를 고려해야 합니다:
  - 위치 (x, y): 객체가 렌더되는 기준 좌표
  - 크기 (scaleX, scaleY): 객체의 가로/세로 배율
  - 회전 (rotation): 객체의 회전 각도

### 2. 회전 행렬의 활용

회전된 좌표를 계산할 때는 2D 회전 행렬을 사용합니다. 회전 행렬의 수식은 다음과 같습니다:

```
x' = x₀ + (x * cos θ - y * sin θ)
y' = y₀ + (x * sin θ + y * cos θ)
```

여기서:

- x₀, y₀: 회전 중심점 (부모 그룹의 기준 좌표)
- x, y: 원본 좌표
- θ: 회전 각도 (라디안)
- x', y': 회전 후의 새로운 좌표

### 3. 변환 순서

객체의 최종 위치를 계산하는 과정은 다음과 같습니다:

1. 부모 그룹 내에서의 상대 위치에 스케일 적용
2. 회전 행렬을 사용하여 회전 변환 적용
3. 부모 그룹의 위치를 더하여 절대 좌표 계산
4. 크기와 회전 각도의 누적 값 계산

## 💡 코드 구현

- _회전 변환과 관련된 코드만을 첨부합니다._

```tsx
/**
 📁 /ungroup-nodes-controller.ts
*/

private _addChildNodes = (
  frame: Frame,
  node: Node,
  parentTransform: {
    x: number
    y: number
    scaleX: number
    scaleY: number
    rotation: number
  }
): Node[] => {
  // 1. 회전 각도를 라디안으로 변환
  const angleInRadians = (parentTransform.rotation * Math.PI) / 180

  // 2. 상대 위치 계산
  const relativeX = node.x() * parentTransform.scaleX
  const relativeY = node.y() * parentTransform.scaleY

  // 3. 회전 행렬을 사용하여 절대 좌표 계산
  // 회전 행렬 공식:
  // x' = x₀ + (x * cos θ - y * sin θ)
  // y' = y₀ + (x * sin θ + y * cos θ)
  // x₀, y₀: 회전 중심점 (부모 그룹의 중심)
  // https://en.wikipedia.org/wiki/Rotation_matrix
  const absoluteX =
    parentTransform.x +
    (relativeX * Math.cos(angleInRadians) - relativeY * Math.sin(angleInRadians))
  const absoluteY =
    parentTransform.y +
    (relativeX * Math.sin(angleInRadians) + relativeY * Math.cos(angleInRadians))

  // 4. 최종 변환 값 계산
  const finalScaleX = parentTransform.scaleX * node.scaleX()
  const finalScaleY = parentTransform.scaleY * node.scaleY()
  const finalRotation = parentTransform.rotation + node.rotation()

  // 5. 새로운 노드 생성
  return nodeAdder({
    frame,
    type: node.attrs.type,
    config: {
      ...node.attrs,
      x: absoluteX,
      y: absoluteY,
      scaleX: finalScaleX,
      scaleY: finalScaleY,
      rotation: finalRotation,
    }
  })
}

```

## 💡 구현 시 주의사항

1. **각도 단위 변환**
   - Konva는 회전 각도를 도(degree) 단위로 사용
   - 회전 행렬 계산 시에는 라디안(radian) 단위 필요
   - `degree * (π/180)` 공식으로 변환
2. **변환 순서**
   - 스케일 → 회전 → 이동 순서로 적용
   - 순서가 바뀌면 원하는 결과가 나오지 않음
3. **누적 변환**
   - 스케일: 부모와 자식의 값을 곱하여 누적
   - 회전: 부모와 자식의 각도를 더하여 누적
   - 위치: 회전 행렬을 통해 계산된 절대 좌표 사용
4. **재귀적 처리**
   - 중첩된 그룹의 경우 재귀적으로 처리
   - 각 계층의 변환값을 올바르게 누적해야 함

## 💡 레퍼런스

- https://en.wikipedia.org/wiki/Rotation_matrix
- https://www.cs.helsinki.fi/group/goa/mallinnus/3dtransf/3drot.html
