# 조건부 렌더링 패턴 (Conditional Rendering)

React에서 조건에 따라 다른 UI를 그리는 방법은 여러 가지다. 상황마다 어울리는 패턴이 다르다.

## 1. if 문

가장 기본적인 방법. JSX 밖에서 분기한다.

```jsx
function Greeting({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>어서 오세요, 짱구!</h1>;
  }
  return <h1>로그인이 필요해요.</h1>;
}
```

조건에 따라 렌더링 결과 자체가 완전히 달라질 때 쓴다. 가장 읽기 쉽다.

## 2. 삼항 연산자 (`? :`)

JSX **안**에서 인라인으로 분기할 때 자주 쓴다.

```jsx
function Status({ isActive }) {
  return (
    <span className={isActive ? "active" : "inactive"}>
      {isActive ? "활성" : "비활성"}
    </span>
  );
}
```

두 경우 모두 렌더링할 게 있을 때 적합하다. 중첩하면 읽기 어려워지니 2단계 이상은 피한다.

```jsx
// ❌ 중첩 삼항 — 읽기 어려움
{a ? (b ? <A /> : <B />) : <C />}

// ✅ if 문이나 컴포넌트 분리로
```

## 3. `&&` 단락 평가 (Short-circuit)

조건이 참일 때만 렌더링하고, 거짓이면 아무것도 그리지 않을 때 쓴다.

```jsx
function TodoItem({ todo }) {
  return (
    <li>
      {todo.text}
      {todo.isImportant && <span>⭐ 중요</span>}
    </li>
  );
}
```

`조건 && <JSX />` — 조건이 falsy면 `&&` 오른쪽을 평가하지 않아 JSX가 렌더되지 않는다.

### `0` 함정

```jsx
// ❌ count가 0이면 "0"이 화면에 찍힘
{count && <span>항목 {count}개</span>}

// ✅ 명시적으로 boolean으로
{count > 0 && <span>항목 {count}개</span>}
// 또는
{!!count && <span>항목 {count}개</span>}
```

`0`은 falsy지만 React가 `false`와 달리 `0`은 그대로 출력한다. 숫자 변수를 `&&` 왼쪽에 쓸 때 주의.

## 4. `??` (Nullish Coalescing)

값이 `null` / `undefined`일 때만 대체값을 쓰고, `0`이나 `""` 같은 falsy 값은 그대로 쓸 때.

```jsx
function Profile({ nickname }) {
  return <p>{nickname ?? "이름 없음"}</p>;
}
```

`nickname`이 `""` 빈 문자열이면 `&&`는 대체값을 보여주지 않고 `||`는 대체값을 보여주지만, `??`는 빈 문자열을 그대로 보여준다.

## 5. 조기 리턴 (Early Return)

렌더링 전에 빠져나오는 패턴. 가드 절(guard clause)이라고도 한다.

```jsx
function UserCard({ user }) {
  if (!user) return null; // 데이터 없으면 아무것도 렌더 안 함
  if (user.isBanned) return <p>제한된 계정입니다.</p>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}
```

중첩 없이 위에서 아래로 읽히니 복잡한 조건 분기에 적합하다. `null`을 반환하면 해당 위치에 빈 공간도 생기지 않는다.

## 6. 객체 맵 (Object Map)

switch 대신 객체를 쓰면 JSX 안에서도 쓸 수 있다.

```jsx
const STATUS_LABEL = {
  loading: <span>로딩 중...</span>,
  error: <span className="error">오류 발생</span>,
  success: <span className="success">완료</span>,
};

function StatusBadge({ status }) {
  return STATUS_LABEL[status] ?? <span>알 수 없음</span>;
}
```

상태 값이 여러 개일 때 삼항 중첩보다 훨씬 읽기 쉽다.

## 패턴 비교

| 패턴             | 적합한 상황                               | 주의점                     |
| ---------------- | ----------------------------------------- | -------------------------- |
| `if` 문          | JSX 밖, 결과가 완전히 달라질 때           | 인라인 불가                |
| 삼항 `? :`       | 둘 중 하나를 JSX 안에서                   | 2단계 이상 중첩 지양        |
| `&&`             | 조건 참일 때만 렌더                       | 숫자 0 함정                |
| `??`             | null/undefined일 때만 대체값             | falsy 전체 처리는 `\|\|`   |
| 조기 리턴        | 여러 조건을 순서대로 처리, guard 패턴     | —                          |
| 객체 맵          | 상태 값이 3개 이상, switch처럼 분기       | 동적 키 타입에 주의         |

## 컴포넌트 분리로 정리하기

조건부 렌더링이 복잡해지면 **컴포넌트로 분리**하는 게 최선이다.

```jsx
// ❌ 한 컴포넌트 안에 모든 분기
function Dashboard({ user, isLoading, error }) {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return null;
  return (
    <div>
      {user.isAdmin && <AdminPanel />}
      {user.notifications.length > 0 && <NotificationBadge count={user.notifications.length} />}
      <UserProfile user={user} />
    </div>
  );
}

// ✅ 각 역할을 컴포넌트로 분리해 읽기 쉽게
function Dashboard({ user, isLoading, error }) {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return null;
  return <UserDashboard user={user} />;
}
```

## 헷갈렸던 포인트

**1. `null` vs `undefined` vs `false` 반환**

셋 다 화면에 아무것도 그리지 않는다. 명시적으로 숨길 때 `null`이 관례.

```jsx
if (!show) return null; // 빈 DOM 노드도 없음
```

**2. `&&`와 삼항 중 뭘 써야 하나**

한쪽만 렌더링하면 `&&`, 양쪽 모두 렌더링하면 삼항. 기준은 간단하다.

**3. `display: none`과의 차이**

조건부 렌더링으로 컴포넌트를 제거하면 **DOM에서 완전히 사라지고 state도 초기화**된다. CSS `display: none`은 DOM은 유지하면서 숨기는 것이라 state가 살아있다. 둘리 폼에 입력 중인 값이 날아가면 안 되는 경우엔 CSS로 숨겨야 한다.

**4. 조건부 렌더링 안에서 Hook 호출 금지**

```jsx
// ❌ Hook을 조건문 안에서 호출
if (isLoggedIn) {
  const [count, setCount] = useState(0); // 규칙 위반
}
```

Hook은 항상 컴포넌트 최상위에서만 호출한다. 조건에 따라 다른 Hook이 필요하면 컴포넌트를 분리한다.

## 요약

- 단순 분기: `if` 문 또는 조기 리턴
- 인라인 둘 중 하나: 삼항 `? :`
- 한쪽만 렌더링: `&&` (숫자 변수는 `> 0`으로 변환 먼저)
- null/undefined 대체: `??`
- 상태가 여러 개: 객체 맵
- 복잡해지면: 컴포넌트 분리

참고: https://ko.react.dev/learn/conditional-rendering
