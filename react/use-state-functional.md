# useState 함수형 업데이트 (Functional Update)

`useState`의 setter에 **값** 대신 **함수**를 넘기는 방식이다. 함수는 이전 state를 인자로 받아 새 state를 반환한다.

```jsx
setCount(count + 1);        // 직접 업데이트 — 현재 렌더의 count에 의존
setCount(prev => prev + 1); // 함수형 업데이트 — React가 최신 값을 넘겨줌
```

언뜻 같아 보이지만, **여러 번 연속 호출할 때** 결과가 달라진다.

## 왜 필요한가 — 클로저 문제

React는 같은 렌더 사이클 내 여러 `setState` 호출을 **배칭(batching)** 한다. 직접 업데이트는 클로저에 잡힌 현재 렌더의 값만 본다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleTripleClick() {
    setCount(count + 1); // count = 0 → 1
    setCount(count + 1); // count = 0 → 1  (여전히 0에서 시작)
    setCount(count + 1); // count = 0 → 1  (여전히 0에서 시작)
  }

  return <button onClick={handleTripleClick}>{count}</button>;
}
```

세 번 호출해도 버튼을 클릭하면 `count`는 `1`이 된다. 세 호출 모두 현재 렌더의 `0`을 보기 때문이다.

함수형 업데이트를 쓰면 React가 각 호출에 최신 값을 직접 건네준다.

```jsx
function handleTripleClick() {
  setCount(prev => prev + 1); // 0 → 1
  setCount(prev => prev + 1); // 1 → 2
  setCount(prev => prev + 1); // 2 → 3
}
```

버튼 한 번 클릭에 `count`가 `3`이 된다.

## 비동기 상황에서도 안전하다

`setTimeout`, 이벤트 리스너, 비동기 콜백 안에서는 state가 콜백 정의 시점에 캡처된다. 함수형 업데이트는 이 문제를 우회한다.

```jsx
function Counter() {
  const [count, setCount] = useState(0);

  function handleDelayedIncrement() {
    setTimeout(() => {
      // count는 클릭 시점의 값에 묶여 있음
      setCount(count + 1); // 1초 후에도 클릭 당시 count 사용

      // 함수형 업데이트는 실행 시점의 최신 state를 받음
      setCount(prev => prev + 1); // 항상 최신 값 기준
    }, 1000);
  }

  return (
    <>
      <p>{count}</p>
      <button onClick={handleDelayedIncrement}>1초 뒤 +1</button>
    </>
  );
}
```

빠르게 여러 번 클릭 후 1초를 기다리면, 직접 업데이트는 마지막 한 번만 반영되지만 함수형 업데이트는 클릭 횟수만큼 쌓인다.

## 객체 state에서도 동일하게

```jsx
const [user, setUser] = useState({ name: "짱구", age: 5 });

// 직접 업데이트 — 다른 필드 날릴 수 있음
setUser({ age: 6 }); // ❌ name 사라짐

// spread + 함수형 업데이트
setUser(prev => ({ ...prev, age: 6 })); // ✅ name 유지
```

`prev`를 받아 spread하면 나머지 필드를 안전하게 유지할 수 있다.

### 배열 state 예시

```jsx
const [todos, setTodos] = useState(["밥 먹기", "공부하기"]);

function addTodo(item) {
  setTodos(prev => [...prev, item]);
}

function removeTodo(index) {
  setTodos(prev => prev.filter((_, i) => i !== index));
}
```

## 직접 업데이트 vs 함수형 업데이트

| 상황                          | 직접 업데이트              | 함수형 업데이트              |
| ----------------------------- | -------------------------- | ---------------------------- |
| 단순 단발 클릭                | 문제 없음                  | 문제 없음                    |
| 같은 사이클에서 여러 번 호출  | 마지막 값만 반영될 수 있음 | 큐에 쌓여 순서대로 반영됨    |
| 비동기 콜백 내부              | 클로저에 묶인 값 사용      | 실행 시점의 최신 값 사용     |
| 이전 값 기반으로 계산해야 할 때 | 의도치 않은 버그 가능성  | 안전                         |

## 헷갈렸던 포인트

**1. 함수형 업데이트가 무조건 좋은 건 아니다**

새 값이 이전 값과 **전혀 무관하다면** 직접 업데이트가 더 읽기 쉽다.

```jsx
setIsOpen(true);          // 이전 값과 관계없이 true로 고정
setIsOpen(prev => !prev); // 이전 값을 반전 → 함수형이 적합
```

**2. `prev`는 React가 관리하는 최신 state다**

렌더링된 값과 같을 수도 있고, 아직 반영 안 된 대기 중인 값일 수도 있다. React가 큐를 처리할 때 알아서 넘겨준다.

**3. 초기값도 함수형으로 줄 수 있다**

```jsx
const [items, setItems] = useState(() => parseLocalStorage()); // 초기화 함수
```

이건 함수형 업데이트와 다른 개념이지만 자주 같이 나온다. 초기값 계산 비용이 클 때 최초 렌더에서만 실행되게 한다.

## 요약

- `setState(prev => ...)` 형태가 함수형 업데이트
- 이전 state를 기반으로 새 state를 만들 때 쓴다
- 같은 렌더 사이클 내 여러 번 호출하거나, 비동기 콜백에서 안전하게 state를 올릴 때 필수
- 직접 업데이트는 이전 값과 무관할 때 쓰면 충분

참고: https://ko.react.dev/reference/react/useState#updating-state-based-on-the-previous-state
