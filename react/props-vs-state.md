# props vs state

React 컴포넌트의 데이터를 다루는 두 가지 축. 헷갈리기 쉽지만 **누가 소유하고 바꿀 수 있느냐**를 기준으로 나뉜다.

- **props**: 부모가 자식에게 **내려주는** 값. 자식은 읽기만 한다.
- **state**: 컴포넌트가 **스스로 관리하는** 값. 변경 시 리렌더링이 일어난다.

## props — 부모가 주는 읽기 전용 데이터

```jsx
// 부모
function App() {
  return <Profile name="짱구" age={5} />;
}

// 자식
function Profile({ name, age }) {
  return <p>{name}은 {age}살이다.</p>;
}
```

자식 컴포넌트 안에서 `props.name = "둘리"` 같은 걸 해도 아무 의미 없다. React는 무시한다.

### 기본값 설정

```jsx
function Profile({ name, age = 0 }) {
  return <p>{name}은 {age}살이다.</p>;
}
```

부모가 `age`를 안 넘겨줬을 때 `0`으로 쓴다.

## state — 컴포넌트가 직접 들고 있는 값

```jsx
import { useState } from "react";

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <button onClick={() => setCount(count + 1)}>
      {count}번 클릭됨
    </button>
  );
}
```

`setCount`로 값을 바꾸면 React가 컴포넌트를 다시 그린다.
`count = count + 1` 같이 직접 바꾸면 리렌더링이 안 된다.

### state 올리기 (lifting state up)

두 자식이 같은 데이터를 공유해야 하면 state를 **공통 부모로 끌어올린다**.

```jsx
function App() {
  const [score, setScore] = useState(0);

  return (
    <>
      <ScoreBoard score={score} />
      <AddButton onAdd={() => setScore(score + 1)} />
    </>
  );
}

function ScoreBoard({ score }) {
  return <p>점수: {score}</p>;
}

function AddButton({ onAdd }) {
  return <button onClick={onAdd}>+1</button>;
}
```

`score`는 `App`의 state이고, 두 자식은 props로 받아 쓴다.

## props vs state 한눈 비교

| 구분       | props                       | state                         |
| ---------- | --------------------------- | ----------------------------- |
| 소유자     | 부모 컴포넌트               | 컴포넌트 자신                 |
| 변경 주체  | 부모만 바꿀 수 있음         | `useState`의 setter로만 바꿈  |
| 리렌더링   | 부모가 새 값을 내려주면 발생 | setter 호출 시 발생           |
| 읽기 전용? | 자식 입장에서 읽기 전용     | 직접 변경 가능 (setter 통해서) |

## 헷갈렸던 포인트

**1. props는 바꿀 수 없다 — 그런데 콜백은?**

```jsx
<Input value={text} onChange={(e) => setText(e.target.value)} />
```

`onChange`도 props다. 하지만 자식이 이걸 호출하면 **부모의 state가 바뀌고**, 바뀐 값이 다시 props로 내려온다. 자식이 props를 바꾼 게 아니라, 부모의 setter를 빌려 쓴 것.

**2. props로 받은 값을 state 초기값으로 쓰면?**

```jsx
function Profile({ initialName }) {
  const [name, setName] = useState(initialName);
  // ...
}
```

이렇게 하면 초기값은 딱 한 번만 적용된다. 이후 부모에서 `initialName`이 바뀌어도 `name` state는 그대로다. 부모 값과 동기를 맞추려면 props를 직접 쓰거나, `useEffect` + key prop으로 처리한다.

**3. state 변경은 비동기처럼 동작한다**

```jsx
const [count, setCount] = useState(0);

function handleClick() {
  setCount(count + 1);
  setCount(count + 1); // 두 번 호출해도 count는 1만 오름
  console.log(count);  // 아직 0 — 리렌더링 전이라 현재 클로저 값
}
```

같은 렌더 사이클에서 여러 번 올리려면 함수형 업데이트를 써야 한다.

```jsx
setCount(prev => prev + 1);
setCount(prev => prev + 1); // 이러면 2가 됨
```

## 요약

- props = 부모가 주는 읽기 전용 데이터
- state = 컴포넌트가 직접 관리하며 setter로만 바꿈
- 두 자식이 데이터를 공유해야 하면 → 공통 부모로 state 끌어올리기
- props로 state 초기화 시 → 이후 props 변경은 무시됨 주의

참고: https://ko.react.dev/learn/state-a-components-memory
