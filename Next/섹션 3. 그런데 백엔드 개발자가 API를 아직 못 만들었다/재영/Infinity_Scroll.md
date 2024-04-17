### 초기 설정 Provider

```tsx
"use client"

import React, { useState } from "react"
import { QueryClientProvider, QueryClient } from "@tanstack/react-query"
// dev tool 설치한 경우
import { ReactQueryDevtools } from "@tanstack/react-query-devtools"

interface ReactQueryProviderProps {
	children : React.ReactNode
}

const ReactQueryProvider = () => {
	const [ client ] = useState(
		new QueryClient({
			defaultOption : {
			// 전역 설정 하는 곳
				queries : {
					refetchOnWindowFocus : false,
					retry : false,
				},
			}
		})
	)

	return (
		<QueryClientProvider client={client}>
			{ children }
			<ReactQueryDevtools initialIsOpen={process.env.NEXT_PUBLIC_MODE === "local"} />
		</QueryClientProvider/>
	)
}

export default ReactQueryProvider
```

### Query 사용

```tsx
const Component = await () => {

	const queryClient = new QueryClient();

	await queryClient.prefetchQuery({ queryKey : [] , queryFn : 패치함수 })

	const dehydratedState = dehydrate(queryClient)

	// 쿼리키와 일치하는 원하는 데이터를 받아올 수 있음
	queryClient.getQueryData([ "쿼리 키" ])

	return (
		<section>
			<HydrationBoundary state={dehydratedState} >


				// ... 내부 내용들

			</HydrationBoundary>

		</section>

	)
}

```
