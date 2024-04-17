<aside>
💡 page.tsx 가 존재하는 폴더 내부 **error.tsx**를 생성하여 사용할 수 있음

page.tsx내부에서 Error가 발생할 경우 **error.tsx**를 통해 처리할 수 있음

</aside>

- Error에 대한 UI를 제공할 수 있음
- 이를 통해 어플리케이션 자체에 문제가 발생하지않고 에러가 발생한 특정 부위에 대해 처리가 가능
- Error에 대한 복구 시도가 가능함

```tsx
"use clinet";

const ErrorPage = ({ error, reset }) => {
  return (
    <main>
      <h6> 현재 에러가 발생했습니다. </h6>
      <button onClick={reset}> 다시 시도하기 </button>
    </main>
  );
};

export default ErrorPage;
```

**reset 메서드**

메서드를 통해 오류가 발생한 위치에대해 다시 한번 렌더링을 시도할 수 있음

**error props 내부**

- **error.message**
  에러에 대한 메세지가 존재함
- **error.digest**
  서버의 로그와 해당 오류가 일치하는지 확인할때 사용되어짐

### app/global-error.tsx

---

**app / error.tsx** 는 동작하지 않음

따라서 app 에서는 **global-error.tsx** 파일을 통해 에러를 처리할 수 있음

🔥**global-error.tsx**는 **error.tsx가 존재하지 않는 별도의 구조에서 포괄적인 error를 처리**함

```tsx
"use clinet";

const GlobalErrorPage = ({ error, reset }) => {
  return (
    <html>
      <body>
        <h6> 현재 에러가 발생했습니다. </h6>
        <button onClick={reset}> 다시 시도하기 </button>
      </body>
    </html>
  );
};

export default ErrorPage;
```

**주의 할점**

- page.tsx를 대체하는것이 아닌 layout.tsx를 대체하기 때문에 내부 html body 태그등 새롭게 정의해야함
