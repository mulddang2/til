# key prop

React가 리스트를 렌더링할 때 **어떤 항목이 바뀌었는지, 추가됐는지, 삭제됐는지** 판단하기 위해 쓰는 식별자다. `key`가 없으면 React는 순서만 보고 비교하는데, 이게 예상치 못한 버그로 이어진다.

## 기본 사용법

```jsx
const friends = ["짱구", "둘리", "또치"];

function FriendList() {
  return (
    <ul>
      {friends.map((name) => (
        <li key={name}>{name}</li>
      ))}
    </ul>
  );
}
```

`key`는 반드시 **형제 노드 사이에서 고유**하면 된다. 전체 앱에서 고유할 필요 없다.

## 왜 필요한가 — 재조정(Reconciliation)

React는 렌더링마다 새 VDOM 트리를 만들고, 이전 트리와 비교해서 실제 DOM에 최소한의 변경만 가한다. 이 과정을 **재조정**이라 한다.

리스트에서 key가 있으면 같은 key끼리 매칭해서 변경 여부를 판단한다. key가 없으면 위치(인덱스)만 보고 비교한다.

### key 없이 맨 앞에 항목을 추가하면

```jsx
// 이전
<ul>
  <li>둘리</li>
  <li>또치</li>
</ul>

// 이후 (짱구를 맨 앞에 추가)
<ul>
  <li>짱구</li>
  <li>둘리</li>
  <li>또치</li>
</ul>
```

React는 위치로만 비교하므로 "1번째 항목이 둘리 → 짱구로 바뀌었다", "2번째 항목이 또치 → 둘리로 바뀌었다"로 인식한다. 세 항목 모두 DOM을 갱신한다. key가 있었다면 기존 둘리, 또치 DOM은 그대로 두고 짱구 DOM만 새로 만들었을 것이다.

## index를 key로 쓰면 안 되는 이유

```jsx
// 위험한 패턴
{items.map((item, index) => (
  <li key={index}>{item.name}</li>
))}
```

항목을 추가·삭제·정렬하면 인덱스가 뒤섞이고, React가 잘못된 컴포넌트를 재사용한다. 특히 input처럼 **자체 상태를 가진 컴포넌트**에서 문제가 심하다.

```jsx
// 문제 재현 예시
function TodoList() {
  const [items, setItems] = useState([
    { id: 1, name: "짱구" },
    { id: 2, name: "둘리" },
  ]);

  function removeFirst() {
    setItems((prev) => prev.slice(1));
  }

  return (
    <>
      <button onClick={removeFirst}>첫 번째 삭제</button>
      <ul>
        {items.map((item, index) => (
          // index를 key로 쓰면 input 내부 상태가 꼬임
          <li key={index}>
            {item.name}: <input placeholder="메모" />
          </li>
        ))}
      </ul>
    </>
  );
}
```

"짱구" 행의 input에 뭔가 입력한 상태에서 첫 번째 항목을 삭제하면, 입력값이 "둘리" 행에 그대로 남는다. React가 key(=0)가 같다고 판단해 같은 컴포넌트를 재사용했기 때문이다.

### 올바른 key — 고유한 ID 사용

```jsx
{items.map((item) => (
  <li key={item.id}>
    {item.name}: <input placeholder="메모" />
  </li>
))}
```

`item.id`처럼 데이터에 실제로 존재하는 식별자를 쓴다.

## index를 key로 써도 괜찮은 경우

아래 세 조건을 모두 만족하면 index도 무방하다.

1. 리스트가 **재정렬되거나 필터링되지 않는다**
2. 항목이 **추가·삭제되지 않는다**
3. 항목 컴포넌트에 **자체 state가 없다**

정적인 설명 텍스트 목록 같은 경우가 해당한다.

## key로 컴포넌트를 강제로 리셋하기

key가 바뀌면 React는 기존 컴포넌트를 완전히 날리고 새로 만든다. 이 특성을 이용해서 컴포넌트를 리셋할 수 있다.

```jsx
function App() {
  const [userId, setUserId] = useState(1);

  return (
    <>
      <button onClick={() => setUserId(2)}>둘리로 전환</button>
      {/* key가 바뀌면 Profile이 언마운트됐다가 새로 마운트됨 */}
      <Profile key={userId} userId={userId} />
    </>
  );
}
```

`useEffect`로 초기화 로직을 짜는 것보다 key를 바꾸는 게 훨씬 간단한 경우가 많다.

## key 사용 규칙 요약 표

| 상황                          | 권장 key          |
| ----------------------------- | ----------------- |
| DB에서 가져온 데이터          | `item.id`         |
| 로컬에서 생성한 데이터        | `crypto.randomUUID()` (생성 시점에 한 번만) |
| 정적 목록, 변경 없음          | `index` 가능      |
| 컴포넌트 강제 리셋            | 의미 있는 식별자로 key 변경 |

## 헷갈렸던 포인트

**1. key는 props가 아니다**

```jsx
function Card({ key, name }) {
  console.log(key); // undefined — 절대 전달 안 됨
}

<Card key="짱구" name="짱구" />
```

key는 React 내부에서만 쓰고 컴포넌트 안으로 전달되지 않는다. 컴포넌트에서 key 값이 필요하면 별도 prop으로 따로 넘겨야 한다.

**2. 렌더링 중에 key를 동적으로 생성하면 안 된다**

```jsx
// 매 렌더마다 새 key가 생겨서 항상 리마운트됨 — 성능 폭탄
{items.map((item) => (
  <li key={Math.random()}>{item.name}</li>
))}
```

key는 안정적이어야 한다. 렌더링마다 달라지면 React가 매번 새 컴포넌트로 인식해서 리마운트한다.

**3. Fragment에도 key가 필요할 수 있다**

```jsx
{items.map((item) => (
  <React.Fragment key={item.id}>
    <dt>{item.name}</dt>
    <dd>{item.desc}</dd>
  </React.Fragment>
))}
```

`<>...</>` 단축 문법은 key를 붙일 수 없어서, 이 경우엔 `React.Fragment`를 명시해야 한다.

## 요약

- key는 React가 리스트 항목을 추적하는 식별자
- 데이터에 있는 고유 ID를 쓰는 게 원칙
- index를 key로 쓰면 재정렬·추가·삭제 시 버그 발생 가능
- key가 바뀌면 컴포넌트가 리마운트된다 → 의도적 리셋에 활용 가능
- key는 컴포넌트 내부로 전달되지 않는다

참고: https://ko.react.dev/learn/rendering-lists#keeping-list-items-in-order-with-key
