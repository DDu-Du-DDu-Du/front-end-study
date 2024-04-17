<aside>
💡 특정 요청이 완료되기 이전 middleware가 동작함

요청의 결과에따라 redirect, rewriting, modifying을 수행할 수 있고 응답을 수정할 수 있음

</aside>

캐시된 컨텐츠와 경로가 일치하기 이전에 middleware가 실행됨

src / app / **middleware.ts** 최상위 경로에 추가할 수 있음

사용 예시

- 인증 및 권한 부여
- 서버 측 리다이렉션
- 경로 재작성
- Bot 탐지와 방어
- 로깅 및 분석

올바르지 못한 사용 예시

- 복잡한 데이터 가져오기 조작
- 무겁고 과도한 계산 작업
- 직접적인 데이터베이스 작업

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

// async await을 사용할 수 있음
export function middleware(request: NextRequest) {
  return NextResponse.redirect(new URL("/home", request.url));
}

// See "Matching Paths" below to learn more
export const config = {
  matcher: "/about/:path*",
};
```

## matcher

---

```tsx
export const config = {
  matcher: ["/about/:path*", "/dashboard/:path*"],

  // matcher : "/about/:path*"
};

export const config = {
  matcher: [
    /*
     * 아래 경로와 일치하는 경로들을 제외한 모든 경로들에대해 middleware 실행
     * - api (API routes)
     * - _next/static (static files)
     * - _next/image (image optimization files)
     * - favicon.ico (favicon file)
     */
    "/((?!api|_next/static|_next/image|favicon.ico).*)",
  ],
};
```

위 예시 코드와 같이 특정 경로에서 middleware가 실행되도록 필터링 할 수 있음

📢 **정규표현식을 이용해 특정 경로를 제외 등의 기능을 수행할 수 있습니다.**

- **has** **missing** 을 이용해 특정 요청을 우회할 수 있음

## 조건문

---

middleware 함수 내부에서 조건문을 통해 요청을 처리할 수 있음

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  if (request.nextUrl.pathname.startsWith("/about")) {
    return NextResponse.rewrite(new URL("/about-2", request.url));
  }

  if (request.nextUrl.pathname.startsWith("/dashboard")) {
    return NextResponse.rewrite(new URL("/dashboard/user", request.url));
  }
}
```

## NextResponse

---

NextResponse API를 이용해 여러가지 작업을 수행할 수 있음

- **redirect**

  해당 요청을 다른 URL로 리다이렉트 해줌

  ```tsx
  import { NextResponse } from "next/server";
  // 기본 사용
  return NextResponse.redirect(new URL("/new", request.url));
  ```

- rewrite
  사용자가 특정 URL에 접근하면 URL에는 변화가 없지만
  보여지는 페이지는 전달받은 다른 URL의 페이지가 보여짐

  ```tsx
  import { NextResponse } from "next/server";

  // request.url이 /about 이라 했을 때
  // 들어오는 요청: /about, 브라우저 /about 표시
  // rewrite된 요청: /proxy, 브라우저 /about 표시
  return NextResponse.rewrite(new URL("/proxy", request.url));
  ```

## 쿠키

---

쿠키는 일반 헤더로 **request**시 쿠키는 **header**에 저장되어짐,
**response**에서는 **Set-Cookie header**에 저장됨

**get** **getAll** **set** **delete** 등 쿠키를 조작할 수 있는 메서드가 제공됨

추가로 **has**를 통해 쿠키가 존재하는지 확인하고 **clear**를 통해 쿠키를 날려버릴 수도 있음

**예시**

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // Assume a "Cookie:nextjs=fast" header to be present on the incoming request
  // Getting cookies from the request using the `RequestCookies` API
  let cookie = request.cookies.get("nextjs");
  console.log(cookie); // => { name: 'nextjs', value: 'fast', Path: '/' }
  const allCookies = request.cookies.getAll();
  console.log(allCookies); // => [{ name: 'nextjs', value: 'fast' }]

  request.cookies.has("nextjs"); // => true
  request.cookies.delete("nextjs");
  request.cookies.has("nextjs"); // => false

  // Setting cookies on the response using the `ResponseCookies` API
  const response = NextResponse.next();
  response.cookies.set("vercel", "fast");
  response.cookies.set({
    name: "vercel",
    value: "fast",
    path: "/",
  });
  cookie = response.cookies.get("vercel");
  console.log(cookie); // => { name: 'vercel', value: 'fast', Path: '/' }
  // The outgoing response will have a `Set-Cookie:vercel=fast;path=/` header.

  return response;
}
```

### 응답 헤더

---

응답 헤더를 수정할수도 있음

```tsx
import { NextResponse } from "next/server";
import type { NextRequest } from "next/server";

export function middleware(request: NextRequest) {
  // 요청 헤더를 복제하고 특정 새 헤더를 설정함
  const requestHeaders = new Headers(request.headers);
  requestHeaders.set("x-hello-from-middleware1", "hello");

  // NextResponse.rewrite 에서도 헤더를 조작할 수 있음
  const response = NextResponse.next({
    request: {
      // 새로운 header
      headers: requestHeaders,
    },
  });

  // `x-hello-from-middleware2` 라는 새로운 헤더를 설정함
  response.headers.set("x-hello-from-middleware2", "hello");
  return response;
}
```

### CORS

---

```tsx
import { NextRequest, NextResponse } from "next/server";

const allowedOrigins = ["https://acme.com", "https://my-app.org"];

const corsOptions = {
  "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
  "Access-Control-Allow-Headers": "Content-Type, Authorization",
};

export function middleware(request: NextRequest) {
  // Check the origin from the request
  const origin = request.headers.get("origin") ?? "";
  const isAllowedOrigin = allowedOrigins.includes(origin);

  // Handle preflighted requests
  const isPreflight = request.method === "OPTIONS";

  if (isPreflight) {
    const preflightHeaders = {
      ...(isAllowedOrigin && { "Access-Control-Allow-Origin": origin }),
      ...corsOptions,
    };
    return NextResponse.json({}, { headers: preflightHeaders });
  }

  // Handle simple requests
  const response = NextResponse.next();

  if (isAllowedOrigin) {
    response.headers.set("Access-Control-Allow-Origin", origin);
  }

  Object.entries(corsOptions).forEach(([key, value]) => {
    response.headers.set(key, value);
  });

  return response;
}

export const config = {
  matcher: "/api/:path*",
};
```
