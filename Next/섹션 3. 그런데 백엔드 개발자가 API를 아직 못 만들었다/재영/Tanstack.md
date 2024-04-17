<aside>
<img src="https://prod-files-secure.s3.us-west-2.amazonaws.com/99dcefe2-cfc3-4664-9694-26b02e6fe5e1/161418cc-261e-48fa-a855-1b09888f1698/React-Query-Logo.png" alt="https://prod-files-secure.s3.us-west-2.amazonaws.com/99dcefe2-cfc3-4664-9694-26b02e6fe5e1/161418cc-261e-48fa-a855-1b09888f1698/React-Query-Logo.png" width="40px" /> 기존의 다양한 State 관련 라이브러리들( Zustand Redux Recoil … 등 ) 은

비동기 및 서버 상태 관리에는 적합하지 않은 형태

</aside>

HTTP 캐싱에도 사용되는 stale-while-revalidate 방법을 사용함

> **서버 상태?**
>
> 클라이언트에서 제어하지 않거나 소유하지 않은 위치에서 관리되어지고,
>
> fetching 과 updating을 하기위해서는 비동기 API 가 필요함.
>
> 한 사용자에대한 상태가 아니기 때문에 다른 사용자가 모르는 사이 업데이트 될 수 있음.
>
> 조심하지않으면 `out of data` 상태가 될 수 있음

### 상태 라이프 사이클

**fresh**

데이터가 fetching이 된 직후 fresh 상태f가 되어지고 최신의 상태 라는 의미

**stale**

fresh 상태의 데이터가 `staleTime`의 값에 의해 stale 상태로 변경됨

오래된 데이터를 의미하게됨

**refresh**

stale 상태를 다시 fresh 상태로 변환하기위해 fetching을 다시 시도함

**inactive**

사용되지 않는 상태를 의미하게됨

해당 상태로 들어서게되면 설정된 gcTime( 기존의 cacheTime 으로 이름이 변경됨 / 가비지 컬렉터 타임 )

에 의해 해당하는 시간이 지나면 지워짐

### 사용 법

**QueryClient**

모든 State와 Cache를 지니게 될 하나의 클래스로 TanStackQuery 사용을 위해서는 반드시 선언해야함

```tsx
const queryClient = new QueryClient({
  defaultOptions: {
    // ...
  },
});

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  );
}
```

- QueryClient 호출 단계에 인수로 객체를 전달하고 객체 내부에 option을 전달할 수 있음

### **Queries**

서버에서 데이터를 받아올 때 사용하는 기능

useQuery를 이용해 사용함

```tsx
const {data , Error , isLoading , isFetching, isPending... 등등} = useQuery({
	queryKey : [ "배열로 queryKey를 전달해야함" ],
	queryFn : fetchFunction,

	{
			// option
	}

})

if(Error){
	return <></>
}

return <></>
```

- queryKey 은 캐시를 관리하기 위한 키값으로 배열을 통해 전달함
- queryFn 은 Promise를 반환하는 함수로 data를 resolve 하고 error를 던지는 일을 수행

- isPending 은 맨 처음 데이터를 불러오는 순간
- isFetching 은 isPending && isLoading과 동일

### refetch vs invalidate vs rest

- **refetch** : 요청시 데이터를 새롭게 받아옴
- **invalidate** : refetch 와 대부분 동일하지만, invalidate는 현재 데이터를 사용하는 시점에 새롭게 받아옴
- **reset** : initialData Option에 지정한 초기 데이터로 변경함, initialData가 없다면 새롭게 받아옴

### Options

- **enabled**
  default : `ture`
  자동으로 query를 실행할지에 대한 여부
- **retry**
  default : `3`
  query 동작 실패 시 전달한 n 번 만큼 retry를 실행함
- **select**

  `Responce data`에서 필요한 값만을 추출해 사용할 수 있는 데이터

  `{ select : data ⇒ ({ id: data.id  , data : data.name })}`

- **refechInterval**

  주기적으로 refetch 하는 간격을 설정하는 옵션

- **throwOnError**

  error boundary로 error를 전파할지에 대한 여부

- **staleTime**

  첫 fetching이 일어나 데이터를 받아온 후

  받아온 데이터가 fresh 상태에서 stale 상태로 변경되기까지의 delay time

  `staleTIme : 1000 * 60 * 3`

- **refetchOnWindowFocus**
  다른 탭을 방문후 다시 현재 탭으로 돌아오면 fetch를 새롭게 진행함
- **retryOnMout**

  컴포넌트가 다시 mout되어지는 경우 새롭게 fetch 진행

- **refetchOnReconnect**
  네트워크 연결이 정지되었다가 연결되는 경우
