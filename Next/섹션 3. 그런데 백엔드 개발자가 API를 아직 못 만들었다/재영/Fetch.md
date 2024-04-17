[공식 문서 링크](https://nextjs.org/docs/app/building-your-application/data-fetching/fetching-caching-and-revalidating)

### React에서의 fetch?

---

기존 React 에서와 같이 data를 fetch하는 과정에서는 useState , useEffect 등이 필요하고

hook 을 사용하게되면 “use client” 즉 클라이언트 컴포넌트를 사용해야함

```tsx
"use client";

import { useEffect, useState } from "react";

const FetchComponent = () => {
  const [data, setData] = useState([]);
  const [isLoading, setIsLoading] = useState(true);

  const getData = async () => {
    const response = await fetch("경로");
    const newData = await response.json();
    setData(data);
    setIsLoading(false);
  };

  useEffect(() => {
    getData();
  }, []);
  return <h1></h1>;
};

export default FetchComponent;
```

위와 같이 사용하면 metadata 또한 해당 컴포넌트 내부에서 정의할 수 없게되는 단점이 존재함

## Next.js Fetch

---

```tsx
const URL = "path";

const getData = async () => {
  const response = await fetch(URL);

  if(!response.ok){
	  return throw new Error("error message")
  }

  return response.json();
};

const FetchComponent = async () => {

  const fetchedData = await getData();

  return <h1>{fetchedData}</h1>;
};

export default FetchComponent;
```

위와 같이 실행하면

서버에서 fetch가 이루어지기 때문에 api 주소 내부에 민감한 정보가 들어있어도 문제없음

- **Next 에서는 fetch 데이터를 자동으로 캐싱해줌 ( POST에 대해서도 자동으로 캐싱 )**

  ```tsx
  // 'force-cache' is the default, and can be omitted
  fetch("https://...", { cache: "force-cache" });
  ```

- **캐싱에 대해 다시 한번 fetch를 원하는 경우**

  - 지정한 시간을 기반으로 캐싱을 다시 받아올 수 있음
    ```tsx
    fetch("https://...", { next: { revalidate: 3600 } });
    ```
    중복으로 설정 했을 경우 가장 낮은 시간을 기준으로 동작함
  - 직접 캐싱된 데이터를 다시 한번 요철 할 수 있음

    ```tsx
    export default async function Page() {
      const res = await fetch("https://...", {
        next: { tags: ["collection"] },
      });
      const data = await res.json();
      // ...
    }

    // 사용처
    ("use server");

    import { revalidateTag } from "next/cache";

    export default async function action() {
      revalidateTag("collection");
    }
    ```

    요청하려면 캐싱에 대해 태그를 설정해줘야함
    다음 다시 한번 fetch를 원하는 곳에서 **revalidateTag( tagName )** 을 통해 사용 가능

- **재 요청시 오류가 발생하면 바로 이전 데이터를 사용함**

- **캐싱이 발생하지 않는 경우**
  - cache : “no-store” 를 옵션에 추가한 경우
  - revalidate 을 0으로 설정한 경우
  - 등등

새로운 값으로 캐싱을 해야하는 경우 revalidation 을 이용함

## 데이터 캐싱

---

Next.js에서는 fetch에 대해 자동으로 데이터를 캐싱함

따라서 한번 캐시된 데이터는 새롭게 refetch받아오지 않고 캐시된 데이터를 사용

```tsx
// 'force-cache'는 기본값

fetch("https://...", { cache: "force-cache" });
```

**캐싱되지 않는 예외 상황**

- Server Action 내부에서 사용한 경우
- Route Handler 내부 POST의 경우

### Options

- **next.cache : “force-cache” | “no-store”**

  **🔥 캐싱을 원하지 않는다면?**

  fetch options cache에 대해 **“no-store”**를 전달함

  ```tsx
  fetch("https://...", { cache: "no-store" });
  ```

- **next.revalidate : false | 0 | number**

  캐시의 수명을 지정함 ( 아래 참고 )

- **next.tags : string[]**
  캐시에 태그를 지정해줌 이후 재검증에 사용됨

## 재검증

---

데이터 캐시를 제거하고 새롭게 최신 데이터를 불러올때 사용

- **시간을 통한 재검증**

  ```tsx
  fetch("https://...", { next: { revalidate: 3600 } });
  ```

  위와 같이 fetch에 대해 일정 시간을 지정하여 **캐시의 수명을 지정**하는 방법
    <aside>
    💡 여러개의 fetch가 존재하고 모든 fetch에 공통적인 캐시 수명을 지정하고 싶다면 
    **Segment Config Options**를 사용할 수 있음
    
    **layout.tsx**
    
    ```tsx
    export const revalidate = 3600
    ```
    
    </aside>

- **요청을 통한 재검증**
  **fetch option에 tag를 지정**하고 해당 tag를 호출하여 재검증을 실행함

  ```tsx
  export default async function Page() {
    const res = await fetch("https://...", { next: { tags: ["collection"] } });
    const data = await res.json();
    // ...
  }
  ```

  재검증 요청을 할때는 **revbalidateTag** 메서드를 통해 진행

  ```tsx
  "use server";

  import { revalidateTag } from "next/cache";

  export default async function action() {
    revalidateTag("collection");
  }
  ```

두가지의 재검증 방법이 존재함

## **revalidatePath**

---

해당 메서드에 경로를 전달해 경로 내부의 모든 fetch의 캐시를 제거함

```tsx
import { revalidatePath } from "next/cache";
revalidatePath("/blog/post-1");
```

- 만약 특정 경로의 layout.tsx 내부의 페이지에 대해 재검증이 필요하다면?

```tsx
import { revalidatePath } from "next/cache";
revalidatePath("/blog/[slug]", "layout");
// or with route groups
revalidatePath("/(main)/post/[slug]", "layout");
```

- 특정 경로의 페이지만 재검증이 필요할때 아래와 같이 사용가능함

**🔥 하위 페이지 모두를 수행하지않고 경로에 해당하는 page.tsx 내부 fetch 에대해서만 캐시를 제거**

```tsx
import { revalidatePath } from "next/cache";
revalidatePath("/blog/[slug]", "page");
// or with route groups
revalidatePath("/(main)/post/[slug]", "page");
```

- 모든 데이터를 재검증해야한다면?

```tsx
import { revalidatePath } from "next/cache";

revalidatePath("/", "layout");
```

## 병렬 fetching

---

하나의 SSR 컴포넌트 내부에서 여러개의 fetch를 통시에 받아올때 사용할 수 있음

- Promise.all을 통해 병렬적으로 처리해줌

```tsx
const [fetch1Value, fetch2Value] = Promise.all([fetch1(), fetch2()]);
```
