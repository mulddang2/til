# useEffect 의존성 배열 (Dependency Array)

`useEffect`의 두 번째 인자인 의존성 배열은 **"어떤 값이 바뀔 때 effect를 다시 실행할지"** 를 결정한다.

## 기본 형태

```js
useEffect(() => {
  // effect 실행
}, [의존성]);
```

세 가지 케이스:

| 두 번째 인자 | 실행 시점 |
| --- | --- |
| 없음 | 매 렌더링마다 실행 |
| `[]` (빈 배열) | 마운트 시 1회만 실행 |
| `[a, b]` | 마운트 + `a` 또는 `b`가 바뀔 때마다 실행 |

## 코드 예시

### 매 렌더링마다

```js
useEffect(() => {
  console.log("렌더링될 때마다 실행");
});
```

두 번째 인자 자체가 없으면 렌더링할 때마다 effect가 돈다. 성능 문제가 생기기 쉬워서 거의 쓰지 않는다.

### 마운트 시 1회

```js
useEffect(() => {
  fetchUserData(); // API 최초 호출
}, []);
```

빈 배열이면 "의존하는 값 없음 = 변화 없음"이라 마운트 때만 실행된다.

### 특정 값이 바뀔 때

```js
const [짱구Id, set짱구Id] = useState(1);

useEffect(() => {
  fetch(`/api/user/${짱구Id}`)
    .then((res) => res.json())
    .then((data) => setUser(data));
}, [짱구Id]);
```

`짱구Id`가 바뀔 때마다 API를 다시 호출한다.

## 클린업 (cleanup)

effect에서 반환하는 함수는 **다음 effect 실행 전 또는 언마운트 시** 호출된다. 구독, 타이머, 이벤트 리스너 해제에 쓴다.

```js
useEffect(() => {
  const id = setInterval(() => {
    console.log("또치 시간 확인 중...");
  }, 1000);

  return () => clearInterval(id); // 클린업
}, []);
```

의존성 배열에 값이 있으면, 값이 바뀔 때마다 **이전 클린업 → 새 effect** 순서로 실행된다.

```js
useEffect(() => {
  console.log("구독 시작:", roomId);
  return () => console.log("구독 해제:", roomId); // 다음 실행 전에 호출
}, [roomId]);
```

## 의존성 빠뜨리면?

```js
const [count, setCount] = useState(0);

useEffect(() => {
  const id = setInterval(() => {
    setCount(count + 1); // count를 의존성에 넣어야 함
  }, 1000);
  return () => clearInterval(id);
}, []); // ❌ count가 빠짐 → count는 항상 0으로 클로저에 고정
```

`count`를 의존성에서 빠뜨리면 effect 안의 `count`는 마운트 시점의 `0`에 갇힌다. 값이 아무리 바뀌어도 `0 + 1 = 1`만 나온다.

**해결 방법 1**: 의존성 배열에 추가.

```js
}, [count]);
```

**해결 방법 2**: 함수형 업데이트로 현재 값에 의존하지 않기.

```js
setCount((prev) => prev + 1); // count를 직접 읽지 않음
```

## 객체·함수는 매번 새로 만들어진다

```js
useEffect(() => {
  // 매 렌더링마다 실행됨 — options가 매번 새 객체이므로
}, [{ page: 1 }]); // ❌
```

객체·배열·함수는 참조값 비교(`Object.is`)를 하기 때문에, 렌더링마다 새로 생성된 값은 항상 "바뀐 것"으로 인식된다.

```js
// ✅ 상태나 useMemo/useCallback으로 안정화
const options = useMemo(() => ({ page: 1 }), []);
useEffect(() => { ... }, [options]);
```

## 헷갈렸던 포인트

- **`[]`는 "최초 1회"가 아니라 "의존성 없음"이 정확한 의미**. 결과적으로 1회만 실행되는 것.
- **ESLint `exhaustive-deps` 규칙을 끄지 말자**. 빠진 의존성을 자동으로 경고해줘서 스테일 클로저(stale closure) 버그를 예방한다.
- **클린업은 언마운트 때만 도는 게 아니다**. 의존성이 바뀔 때도 이전 effect의 클린업이 먼저 실행된다.
- **effect 안에서 `async` 함수 직접 사용 불가**. effect 콜백 자체는 Promise를 반환하면 안 된다 (클린업 함수 자리에 Promise가 들어가는 문제).

```js
// ❌
useEffect(async () => { ... }, []);

// ✅
useEffect(() => {
  async function load() { ... }
  load();
}, []);
```

## 요약

- 두 번째 인자 없음 → 매 렌더링, `[]` → 마운트 1회, `[값]` → 값 변경 시마다
- effect 안에서 사용하는 변수는 **모두 의존성 배열에 넣어야** 스테일 클로저 방지
- 클린업 함수는 다음 effect 실행 직전에도 호출된다
- 객체·함수는 `useMemo` / `useCallback`으로 안정화하거나 배열 밖으로 꺼내야 무한 루프 방지

참고: https://react.dev/reference/react/useEffect
