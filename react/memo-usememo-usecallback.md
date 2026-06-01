# memo / useMemo / useCallback

세 가지 모두 **불필요한 연산·렌더링을 건너뛰기** 위한 메모이제이션 도구다.
목적은 같지만 대상이 다르다.

- `React.memo` — **컴포넌트** 자체를 메모이제이션 (props가 안 바뀌면 리렌더 생략)
- `useMemo` — **계산된 값**을 메모이제이션
- `useCallback` — **함수 참조**를 메모이제이션

## React.memo

컴포넌트를 감싸면, 부모가 리렌더되더라도 **props가 이전과 동일하면** 자식 리렌더를 건너뛴다.

```jsx
const Profile = React.memo(function Profile({ name, age }) {
  console.log("Profile 렌더");
  return <p>{name} ({age})</p>;
});

function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+1</button>
      {/* count가 바뀌어도 name/age가 같으면 Profile은 안 그려짐 */}
      <Profile name="짱구" age={5} />
    </>
  );
}
```

비교는 **얕은 비교(shallow equal)**다. 객체/배열/함수를 props로 넘기면 렌더마다 새 참조가 생겨 매번 다르다고 판단한다. 이때 `useMemo`·`useCallback`과 같이 쓴다.

## useMemo

```jsx
const value = useMemo(() => 계산(), [deps]);
```

의존성 배열 `deps`가 변할 때만 `계산()`을 다시 실행하고, 그 결과를 캐싱한다.

```jsx
function ShoppingCart({ items, discount }) {
  const total = useMemo(() => {
    console.log("합계 계산");
    return items.reduce((sum, item) => sum + item.price, 0) * (1 - discount);
  }, [items, discount]);

  return <p>총액: {total}원</p>;
}
```

`items`와 `discount`가 안 바뀌면 `total`은 이전 값을 그대로 쓴다.

### 언제 쓰나

- 리스트 필터링, 정렬 같은 **비싼 계산**
- `React.memo`로 감싼 자식에게 **객체/배열을 props로 넘길 때** (참조 안정화)

```jsx
const filteredList = useMemo(
  () => todos.filter(t => t.done === showDone),
  [todos, showDone]
);

// filteredList가 안 바뀌면 TodoList도 리렌더 안 함
return <TodoList items={filteredList} />;
```

## useCallback

```jsx
const fn = useCallback(() => { ... }, [deps]);
```

함수를 **deps가 바뀔 때만 새로 만든다**. 그 외에는 같은 참조를 유지한다.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState("짱구");

  // count가 바뀌어도 name이 그대로면 같은 함수 참조
  const handleNameChange = useCallback((newName) => {
    setName(newName);
  }, []); // name을 쓰지 않으니 deps 비어도 됨

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      {/* handleNameChange 참조가 안정적 → NameInput 리렌더 생략 */}
      <NameInput onChange={handleNameChange} />
    </>
  );
}

const NameInput = React.memo(function NameInput({ onChange }) {
  console.log("NameInput 렌더");
  return <input onChange={e => onChange(e.target.value)} />;
});
```

`useCallback`이 없으면 `Parent`가 리렌더될 때마다 `handleNameChange`가 새 함수로 만들어지고, `React.memo`가 있어도 `NameInput`이 함께 리렌더된다.

## useMemo vs useCallback

`useCallback(fn, deps)`는 사실 `useMemo(() => fn, deps)`와 같다.
함수를 값으로 캐싱하느냐, 계산 결과를 캐싱하느냐 차이.

| 항목 | useMemo | useCallback |
|------|---------|-------------|
| 캐싱 대상 | 계산 결과 (값) | 함수 참조 |
| 반환 | 콜백의 반환값 | 콜백 자체 |
| 주 용도 | 비싼 계산, 참조 안정 객체 | 자식에 넘길 이벤트 핸들러 |

## 세 가지 비교

| 항목 | React.memo | useMemo | useCallback |
|------|-----------|---------|-------------|
| 적용 대상 | 컴포넌트 | 값 | 함수 |
| 사용 위치 | 컴포넌트 선언부 | 컴포넌트 내부 (훅) | 컴포넌트 내부 (훅) |
| 재계산 조건 | props 변경 | deps 변경 | deps 변경 |

## 헷갈렸던 포인트

**1. 남발하면 오히려 느려진다**

메모이제이션도 비용이다. 이전 값과 현재 값을 비교하는 연산이 추가된다. 연산이 단순하거나 컴포넌트가 어차피 자주 리렌더되어야 한다면 없는 게 낫다.

**2. React.memo는 부모 리렌더를 막지 않는다**

자식 컴포넌트의 리렌더를 줄이는 도구지, 부모를 최적화하지 않는다. state가 부모에 있으면 부모는 항상 리렌더된다.

**3. deps에 함수나 객체를 넣으면 매번 새 참조**

```jsx
// ❌ 매 렌더마다 options가 새 객체 → useMemo 의미 없음
const result = useMemo(() => compute(options), [options]);

// ✅ options를 useMemo로 안정화하거나 원시값만 deps에 넣기
const result = useMemo(() => compute({ a, b }), [a, b]);
```

**4. useCallback의 deps는 함수 안에서 쓰는 외부 값 전부**

```jsx
const handleClick = useCallback(() => {
  console.log(name); // name을 참조하니 deps에 포함
}, [name]);
```

deps에 빠뜨리면 함수 내부에서 낡은 값을 보는 클로저 문제가 생긴다.

**5. 자식이 React.memo로 감싸져 있지 않으면 useCallback은 무의미**

자식이 어차피 부모 리렌더마다 같이 렌더되기 때문이다. `React.memo`와 세트로 써야 효과가 있다.

## 요약

- `React.memo` — props 얕은 비교, 같으면 리렌더 스킵
- `useMemo` — 비싼 계산 결과 캐싱, deps 바뀔 때만 재계산
- `useCallback` — 함수 참조 안정화, React.memo 자식에 넘길 때 세트
- 세 가지 모두 **필요한 곳에만** 쓰는 것이 원칙 — 과도한 메모이제이션은 독

참고: https://ko.react.dev/reference/react/memo
