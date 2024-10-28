## Summary

> 접근 권한이 없는 페이지는 방문하지 않고, 서버 사이드에서 바로 로그인 페이지로 리다이렉트하고 싶다.

> 현재 UserRoles (MEMBER, ADMIN 등)에서 각각 역할별 접근 가능 여부를 외부에서 주입하고 싶다.

- Next.js를 사용하는 김에, 위 두 가지 니즈를 충족할 수 있는 `authServerSideProps`를 구현한다. `authServerSideProps` 함수는 Next.js에서 서버 사이드 렌더링(SSR) 시 인증 및 권한 검사를 수행하고, 접근 권한이 없는 페이지에 대해서는 로그인 페이지로 리다이렉트하는 기능을 제공한다. 또한 사용자의 역할(role)에 따라 페이지 접근 권한을 제어할 수 있도록 설계하였다.

## Definition

### 용어 정리

- `allowedRoles`: 접근이 허용되지 않는 사용자 역할(role)의 배열이다. 이 배열에 포함된 역할을 가진 사용자는 해당 페이지에 접근할 수 없다.
- `getServerSidePropsFunc`: 인증 및 권한 검사 후 추가적으로 실행할 `getServerSideProps` 함수이다. 이 함수는 인증된 사용자의 정보를 포함한 `context` 객체를 받는다.

  - `UserRoles`

    ```typescript
    export enum UserRole {
      ADMIN = "ADMIN",
      MEMBER = "MEMBER",
      // ...
    }
    ```

## Current

- 기존 리다이렉트 로직은 다음과 같다.
  - 목표 페이지에 진입 후 "사용자 정보가 있음 & 사용자 역할이 NON_MEMBER가 아닌 경우"를 확인한다.
  - 인증 및 권한 검사를 통과하지 못했을 시 리다이렉트 주소를 들고 클라이언트 사이드에서 로그인 페이지로 히스토리 푸시한다.
- 그에 따른 문제점은 다음과 같다.
  - 기존의 타겟 페이지가 잠깐 등장하는데, 사용자 정보나 권한이 없어서 깨진 상태로 보인다.
  - 권한별 제어가 불가능하다.

## Goals

- 인증 및 권한 검사 로직을 재사용 가능한 함수로 분리하여 코드 중복을 줄이고 유지보수성을 향상시킨다.
- 접근 권한이 없는 페이지에 대해서는 서버 사이드에서 로그인 페이지로 리다이렉트한다.
- 사용자의 역할(role)에 따라 페이지 접근 권한을 동적으로 제어할 수 있도록 한다.

## Non-Goals

- 클라이언트 사이드에서의 인증 및 권한 검사는 다루지 않는다.
- 이번 구현에서는 사용자 역할(role) 이외의 다른 조건에 따른 접근 제어는 고려하지 않는다.

## Plans

1. **`getServerSideProps`에 대하여**

   [Data Fetching: getServerSideProps](https://nextjs.org/docs/pages/building-your-application/data-fetching/get-server-side-props)

2. **구현체**

   - `allowedRoles` 매개변수를 통해 접근이 허용되는 사용자 역할을 받는다.
   - `getServerSidePropsFunc` 매개변수를 통해 인증 및 권한 검사 후 실행할 추가 로직을 받는다.
   - 사용자 정보를 확인한다.
   - 사용자의 역할이 `allowedRoles`에 포함되어 있는지 확인하고, 접근 권한을 판단한다.
   - 접근 권한이 없는 경우 로그인 페이지로 리다이렉트, 아닌 경우 `getServerSidePropsFunc`를 실행해, 그 결과를 반환한다.

3. **사용 예시**

   - 인증이 필요한 페이지의 경우, 각 페이지의 `getServerSideProps` 함수에서 `authServerSideProps` 함수를 호출하여 인증 및 권한 검사를 수행한다.
   - 구체적인 사용 예시는 아래와 같다.

     ```typescript
     export default function SomePage() {
       // ... 로직
     }

     export const getServerSideProps: GetServerSideProps = authServerSideProps({
       allowedRoles: [UserRole.ADMIN],
       getServerSidePropsFunc: // ...
     });
     ```

4. **구현 상 이슈**
   - **Issue**
     - Next.js의 `getServerSideProps`에서 리다이렉션이 발생할 경우, 데이터 프리페칭 메커니즘으로 인해 실제 의도한 대상 URL이 아닌 `/_next/data/...` 형태의 캐싱된 경로로 리다이렉트되는 문제가 있다. 예를 들어 `/ai`로 리다이렉트하려고 했는데 `/_next/data/development/ai.json`와 같은 URL로 리다이렉트되는 상황이 발생한다.
     - Next.js의 데이터 프리페칭 메커니즘 때문에 발생하는 것으로 보이며, 페이지 전환 시 다음 페이지의 데이터를 미리 가져오기 위해 내부적으로 `/_next/data/...` 형태의 URL을 생성하고, `getServerSideProps`를 호출한다고 한다. 이때 `getServerSideProps`에서 리다이렉션이 발생하면, Next.js는 해당 캐싱된 경로로 리다이렉트하게 된다.
     - `context.req.headers.referer`를 사용하여 직전 페이지 주소를 알 수 있지만, `getServerSideProps`에서 리다이렉션이 발생하면 Next.js 라우터가 아닌 내부 메커니즘에 의해 처리되므로, 이 기작이 인터셉팅처럼 동작해 라우트 히스토리에 실제 대상 경로가 기록되지 않게 되는 것으로 보인다.
   - **Resolution**
     - `context` 객체 내부의 `resolvedUrl`로 원래 의도했던 주소를 가져오면 된다.
