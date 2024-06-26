<aside>
💡 page.tsx 가 존재하는 폴더 내부에 loading.tsx파일을 생성하여 사용할 수 있음

React Suspense와 같이 page.tsx내부 준비 단계가 존재한다면 준비가 끝나기 전까지 loading.tsx파일이 보여짐

</aside>

**src / app / loading.tsx**

```tsx
const loadingPage = () => {

	return (
		// ... 로딩시 보여질 JSX
	)

}

export default loadingPage
```

### Suspense vs Loading.ts

---

- loading.tsx를 이용해 로딩에대한 처리를 해줄 경우
  준비 단계가 모두 끝나야 모든 페이지가 보여지는 단점이 존재함
- Suspense를 사용할 경우 전체적인 페이지는 보여지지만 원하는 특정 부분에서만
- 로딩에대한 처리를 해줄 수 있음

Suspense 내부 하위 컴포넌트는 async await이 필요

### 스트리밍

---

사용자가 페이지에 접근시 SSR 동작 순서

1. 특정 페이지에 해당하는 데이터를 서버가 가져옴
2. 서버는 가져온 데이터 페이지의 HTML을 렌더링함
3. 서버는 페이지의 HTML CSS JS를 클라이언트에게 제공함
4. HTML CSS를 가지고 사용자의 화면에 그림
5. 이후 하이드레이션을 통해 인터렉티브하게 만듬

하지만 이런 과정에서 문제는 위와같이 모든 순차 과정이 진행되어야 사용자에게 화면에 보여지고

만약 **서버가 데이터를 불러오는 과정이 길어지면 페이지 전체가 보여지지 않음**

이런 문제점을 개선하고자
**모든 HTML을 더 작은 청크로 나누어 로딩이 필요하지 않은 부분을 먼저 사용자에게 제공하고 로딩이 완료되는대로 점진적으로 클라이언트에게 요소를 전달하는 방법**
