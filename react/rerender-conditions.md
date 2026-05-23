# 리렌더링 발생 조건

React에서 컴포넌트가 **다시 그려지는 시점**은 정해져 있다.
원인을 모르면 불필요한 렌더링이 쌓이고, 최적화도 엉뚱한 곳에 하게 된다.

## 리렌더링이 일어나는 4가지 조건

| 조건 | 설명 |
|---|---|
| **state 변경** | `useState`, `useReducer`의 setter 호출 |
| **부모 리렌더링** | 부모가 렌더링되면 자식도 기본적으로 따라 렌더링 |
| **props 변경** | 받은 props 값이 바뀌었을 때 (실제론 부모 리렌더의 결과) |
| **context 변경** | `useContext`로 구독 중인 Context 값이 바뀌었을 때 |

## 1. state 변경

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

`setCount`가 호출되면 Counter가 리렌더링된다.
단, **이전 값과 같은 값**을 넣으면 React가 렌더링을 건너뛴다(`Object.is` 비교).

```jsx
setCount(0); // 이미 0이면 리렌더링 없음
```

## 2. 부모 리렌더링 → 자식도 리렌더링

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>올려</button>
      <Child />
    </>
  );
}

function Child() {
  console.log("Child 렌더링");
  return <p>나는 짱구</p>;
}
```

`<Child />`는 아무 props도 받지 않는다. 그런데 부모 `count`가 바뀔 때마다 `"Child 렌더링"`이 찍힌다.
**부모가 렌더링되면 자식 컴포넌트 함수도 다시 호출**되기 때문이다.

### React.memo로 막기

```jsx
const Child = React.memo(function Child() {
  console.log("Child 렌더링");
  return <p>나는 짱구</p>;
});
```

`React.memo`로 감싸면 props가 실제로 바뀌었을 때만 리렌더링된다.
아무 props도 없으니 위 예시에서는 부모가 아무리 바뀌어도 Child는 조용하다.

## 3. props 변경 — 참조 동일성 주의

props가 객체나 함수면 **매 렌더마다 새 참조**가 만들어진다.

```jsx
function Parent() {
  const [count, setCount] = useState(0);

  const user = { name: "둘리" }; // 렌더마다 새 객체
  const handleClick = () => {};   // 렌더마다 새 함수

  return <Child user={user} onClick={handleClick} />;
}

const Child = React.memo(function Child({ user, onClick }) {
  console.log("Child 렌더링");
  return <p>{user.name}</p>;
});
```

`React.memo`를 써도 `user`와 `handleClick`이 매번 새 참조라 Child는 계속 리렌더링된다.
`useMemo` / `useCallback`으로 참조를 고정해야 한다.

```jsx
const user = useMemo(() => ({ name: "둘리" }), []);
const handleClick = useCallback(() => {}, []);
```

## 4. context 변경

```jsx
const ThemeContext = createContext("light");

function App() {
  const [theme, setTheme] = useState("light");
  return (
    <ThemeContext.Provider value={theme}>
      <Child />
    </ThemeContext.Provider>
  );
}

function Child() {
  const theme = useContext(ThemeContext);
  console.log("Child 렌더링:", theme);
  return <p>테마: {theme}</p>;
}
```

`theme`이 바뀌면 `useContext(ThemeContext)`를 쓰는 **모든** 컴포넌트가 리렌더링된다.
Context를 잘게 나누거나 구독 범위를 좁혀야 불필요한 렌더를 줄일 수 있다.

## 헷갈렸던 포인트

- **state를 같은 값으로 set해도 렌더링이 일어날 수 있다.** 첫 번째 같은-값 set에서는 자식까지 한 번 더 렌더링하고, 두 번째부터 완전히 bail-out한다. (React 공식 docs에도 적혀 있음)
- **props가 안 바뀌어도 부모가 렌더링되면 자식은 렌더링된다.** `React.memo` 없이는 props 비교 자체를 안 한다.
- **`React.memo`는 props 얕은 비교.** 객체/배열/함수 props는 `useMemo`/`useCallback`과 함께 써야 의미 있다.
- **Context는 구독만 해도 렌더링 대상.** 값의 일부만 쓰더라도 Context 객체 전체가 바뀌면 리렌더링된다. 분리하거나 `useMemo`로 값을 안정화한다.
- **리렌더링 자체는 나쁜 게 아니다.** React는 리렌더링 후 실제 DOM 변경이 없으면 커밋하지 않는다. 과도한 최적화보다 실제 성능 이슈를 먼저 확인한다.

## 요약

- state 변경, 부모 렌더링, context 변경 → 리렌더링 발생
- 부모 렌더링만으로도 자식은 함께 렌더링된다 → `React.memo`로 차단
- 객체·함수 props는 참조가 매번 바뀜 → `useMemo`/`useCallback` 병행
- 리렌더링 = 나쁜 것 아님, DOM 업데이트가 없으면 비용 없음

참고: https://react.dev/learn/render-and-commit
