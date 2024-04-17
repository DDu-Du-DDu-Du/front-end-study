<aside>
💡 <Suspense> 태그로 감싼 자식 요소들의 준비 과정( fetch 등.. )이 완료되기전까지
전달 받은 요소를 대체로 보여줌

</aside>

```tsx
<Suspense fallback={<Loading />}>
  <SomeComponent />
</Suspense>
```

SomeComponent 내부 Data fetching로직이 있다면 fetching 중에는 **fallback에 전달받은 <loading /> 컴포넌트를 보여주고 fetcing이 완료된 후 SomeComponent가 보여짐**

### Suspense 동작 조건

---

1. 서버( Next )에서 데이터를 fetch 받아오고 있을때
2. lazy loading을 하는 경우
3. use 라는 hook으로 값을 불러오는 경우

### useSuspenseQuery

---

Tanstack Query에서 `useSuspenseQuery` `useSuspenseInfinityQuery`등을 사용하면 Suspense에 감지됨
