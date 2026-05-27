# Controlled vs Uncontrolled 컴포넌트

폼 입력 요소의 값을 **누가 관리하느냐**에 따라 둘로 나뉜다.

- **Controlled**: React state가 값을 관리한다.
- **Uncontrolled**: DOM이 직접 값을 관리하고, 필요할 때 `ref`로 꺼낸다.

## Controlled 컴포넌트

`value`와 `onChange`를 모두 달아서 state와 입력값을 **항상 동기화**한다.

```jsx
import { useState } from "react";

function LoginForm() {
  const [name, setName] = useState("");

  return (
    <input
      value={name}
      onChange={e => setName(e.target.value)}
      placeholder="이름 입력"
    />
  );
}
```

`value={name}`으로 묶었기 때문에 사용자가 타이핑할 때마다 `onChange`가 실행되고 state가 바뀌고 리렌더된다. React state가 **단일 진실 공급원(single source of truth)**.

### 언제 쓰나

- 입력값에 따라 UI를 즉시 반응시켜야 할 때 (실시간 유효성 검사, 검색 필터링 등)
- 폼 제출 시 현재 입력값을 state에서 바로 꺼낼 때
- 여러 입력 필드가 서로 연동될 때

```jsx
function SignupForm() {
  const [form, setForm] = useState({ id: "", pw: "" });

  const handleChange = e =>
    setForm(prev => ({ ...prev, [e.target.name]: e.target.value }));

  const isValid = form.id.length >= 4 && form.pw.length >= 8;

  return (
    <>
      <input name="id" value={form.id} onChange={handleChange} />
      <input name="pw" type="password" value={form.pw} onChange={handleChange} />
      <button disabled={!isValid}>가입</button>
    </>
  );
}
```

## Uncontrolled 컴포넌트

`value`를 달지 않고, `ref`로 DOM 노드를 직접 참조한다. 값은 **DOM이 들고 있다**.

```jsx
import { useRef } from "react";

function CommentForm() {
  const inputRef = useRef(null);

  function handleSubmit(e) {
    e.preventDefault();
    console.log(inputRef.current.value); // 제출 시에만 꺼냄
  }

  return (
    <form onSubmit={handleSubmit}>
      <input ref={inputRef} defaultValue="짱구" />
      <button type="submit">제출</button>
    </form>
  );
}
```

`defaultValue`로 초기값을 줄 수 있다. 이후 사용자가 변경한 값은 DOM이 관리하고, React는 모른다.

### 언제 쓰나

- 제출 시 딱 한 번만 값을 읽으면 되는 단순 폼
- 서드파티 라이브러리(Date Picker, Rich Text Editor 등)가 DOM을 직접 제어할 때
- 리렌더 비용을 최소화해야 할 때

## Controlled vs Uncontrolled 비교

| 항목 | Controlled | Uncontrolled |
| ---- | ---------- | ------------ |
| 값 저장 위치 | React state | DOM 노드 |
| 값 접근 방법 | state 변수 | `ref.current.value` |
| 실시간 검증 | 쉬움 | 어려움 |
| 리렌더 발생 | 타이핑마다 | 발생 안 함 |
| React 권장 | ✅ | 필요할 때만 |
| 주요 prop | `value` + `onChange` | `defaultValue` + `ref` |

## `defaultValue` vs `value`

| prop | 역할 |
| ---- | ---- |
| `defaultValue` | 초기값만 세팅, 이후 DOM이 관리 (Uncontrolled) |
| `value` | 매 렌더마다 강제로 주입, state와 동기화 (Controlled) |

`value`만 주고 `onChange`를 빠뜨리면 입력이 막힌다 → React가 경고를 띄운다.

## 헷갈렸던 포인트

**1. `value` 없이 `onChange`만 달면?**

Uncontrolled인데 변경 이벤트만 듣는 상태다. 동작은 하지만, 값이 state와 연결되지 않아 Controlled가 아니다.

**2. `ref`로 Controlled 흉내는 안 된다**

```jsx
// ❌ ref로 값을 바꿔봤자 리렌더도 없고 value prop과 충돌
inputRef.current.value = "둘리";
```

Controlled로 쓰려면 state를 써야 한다.

**3. 둘을 섞지 않는다**

같은 `<input>`에 `value`(Controlled)와 `ref`(Uncontrolled 방식)를 동시에 쓰면 혼란만 생긴다. 하나만 선택.

**4. 체크박스 / 셀렉트도 동일**

`<input type="checkbox">`는 `checked` + `onChange`, `<select>`는 `value` + `onChange`로 Controlled 처리한다.

```jsx
const [checked, setChecked] = useState(false);
<input type="checkbox" checked={checked} onChange={e => setChecked(e.target.checked)} />
```

## 요약

- Controlled: state가 주인, 실시간 제어 가능, React 공식 권장
- Uncontrolled: DOM이 주인, ref로 필요할 때만 꺼내 씀
- 실시간 검증이나 상호 연동이 필요하면 Controlled
- 단순 제출용 폼이거나 서드파티 위젯이면 Uncontrolled 검토

참고: https://ko.react.dev/learn/sharing-state-between-components#controlled-and-uncontrolled-components
