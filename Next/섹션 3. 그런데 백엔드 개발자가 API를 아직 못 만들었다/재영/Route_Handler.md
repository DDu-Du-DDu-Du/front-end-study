<aside>
💡 App Router 방식에서는 **Route Handlers** /  Page Router 방삭에서는 API Router 라고함

</aside>

### 생성

원하는 경로내부 page.tsx와 같이 **route.ts 파일을 생성**함으로 Route Handler를 사용할 수 있음

( `src`**/**`app`**/**`api`/`auth`**/`route.ts`** )

**🔥 route.ts가 존재하는 폴더 내부에는 page.tsx layout.tsx 등이 존재할 수 없음**

- 기본적으로 지원되지 않는 Method는 405 Error가 발생함
- GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS 를 사용할 수 있음

- GET 메서드는 기본적으로 **캐시**되어짐
  캐싱을 원하지 않을 경우 아래와 같이 Segment Config Options를 통해 제어가능함

  ```tsx
  export const dynamic = "auto";
  export const dynamicParams = true;
  export const revalidate = false;
  export const fetchCache = "auto";
  export const runtime = "nodejs";
  export const preferredRegion = "auto";

  export async function GET() {
    // ...
  }
  ```

  - 캐시 데이터 재검증

  ```tsx
  export async function GET() {
    const res = await fetch("주소", {
      next: { revalidate: 60 }, // 여기서 재검증 시간 지정
    });

    const data = await res.json();

    return Response.json(data);
  }
  ```

  혹은

  ```tsx
  export const revalidate = 60;
  ```

### 리다이렉트

특정 메서드 동작 후 리다이렉트 요청

```tsx
import { redirect } from "next/navigation";

export async function GET(request: Request) {
  redirect("https://nextjs.org/");
}
```

### 동적 Segment

`src`/`app`/`items`/`[slug]`/`route.ts`

위 경로와 동일하게 [ ] 를 통해서 **Method Parameter에서도 동적 라우트를 이용**할 수 있음

```tsx
interface ParamsType {
	params: { slug: string }
}

export async function GET( request: Request, { params }: ParamsType }) {
  const slug = params.slug
}
```

### Query String

존재하는 query string의 값을 request내부 searchParams를 통해 꺼내 사용할 수 있음

```tsx
import { type NextRequest } from "next/server";

export function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams;
  const query = searchParams.get("query");
}
```
