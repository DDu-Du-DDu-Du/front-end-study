<aside>
💡 Mock Service Worker

</aside>

공식 문서 : [🔗링크](https://v1.mswjs.io/docs/getting-started/install)

🔥 Next.js 기준

### install

```markdown
// public folder에 설치됨

npx msw init pulbic/ --save
```

```markdown
npm install msw --save-dev

# or

yarn add msw --dev
```

### Setting

- `src` 내부 `mocks` 폴더 생성
- `mocks` 폴더 내부 `browser.ts` 생성 후 아래 코드 작성

  browser 즉 client 환경에서 동작

  ```tsx
  // browser.ts

  import { setupWorker } from "wms/browser";
  // 밑에서 만들어질 핸들러 파일
  import { handlers } from "./handlers";

  const worker = setupWorker(...handlers);

  export default worker;
  ```

- `mocks` 폴더 내부 `http.ts` 생성 후 아래 코드 작성
  server에서 동작

  ```markdown
  npm i -D @mswjs/http-middleware express cors @types/cors @types/express
  ```

  ```tsx
  // http.ts

  import { createMiddleware } from "@mswjs/http-middleware";
  import express from "express";
  import cors from "cors";
  // 밑에서 만들어질 핸들러 파일
  import { handlers } from "./handlers";

  const app = express();
  const port = 9090;

  // origin은 현재 로컬 작업 환경 주소
  app.use(
    cors({
      origin: "http://localhost:3000",
      optionsSuccessStatus: 200,
      credentials: true,
    })
  );
  app.use(express.json());
  app.use(createMiddleware(...handlers));

  app.listen(port, () =>
    console.log(`Mock server is running on port: ${port}`)
  );
  ```

- mock 폴더 내부 handlers.ts 생성
  실제 Mock 들을 해당 파일 내부에서 생성

  ```tsx

  import { http, HttpResponse } from "msw"

  export const handlers = [
  	http.post("경로" , () => {

  		return HttpResponse.json({ 내보낼 데이터들 })
  	}),

  	// Cookie 설정시
  	http.post("경로" , () => {

  		// HttpResponse.json( body , init )
  		return new HttpResponse.json(null,{
  			header : {
  				"set-cookie" : "쿠키 내용"
  			}
  		})
  	}),


  ]
  ```

- 이후 package.json에 해당 내용 추가후 실행
  ```tsx
  {
  	"scripts" : {
  		"mock" : "npx tsx watch ./src/mocks/http.ts",
  	}
  }
  ```

이후 사용은 터미널 추가 생성 후npm run mock

- 추가로 src / app 내부 \_component 생성
- 해당 폴더 내부에 MSWComponent.tsx 생성후 아래와 같이 생서
  해당 컴포넌트는 layout상단에 존재하며 원하는 API 요청을 가로체 Mock을 실행시키는 역할

  ```tsx
  "use client";
  import { useEffect } from "react";

  export const MSWComponent = () => {
    useEffect(() => {
      if (process.env.NEXT_PUBLIC_API_MOCKING === "enable") {
        require("@/mocks/browser");
      }
    }, []);

    return null;
  };
  ```

  🔥 Mock 사용을 원할 경우 .env 파일 내부 해당 값을 생성 후 값을 enable로 지정하면됨
