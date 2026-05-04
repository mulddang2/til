# map / filter / forEach 언제 뭐 쓰나

세 메서드 모두 배열을 순회하지만, **반환값과 목적**이 다르다.

| 메서드 | 반환값 | 쓰는 상황 |
|---|---|---|
| `forEach` | `undefined` | 그냥 순회만 (side effect) |
| `map` | **새 배열** (원본 길이 유지) | 각 요소를 변환 |
| `filter` | **새 배열** (조건 통과만) | 일부만 골라내기 |

```js
const nums = [1, 2, 3, 4];

// forEach: 반환값 없음 → 변수에 담으면 undefined
nums.forEach((n) => console.log(n));

// map: 변환된 새 배열
const doubled = nums.map((n) => n * 2);     // [2, 4, 6, 8]

// filter: 조건 통과한 것만
const evens = nums.filter((n) => n % 2 === 0); // [2, 4]
```

## 헷갈렸던 포인트

`forEach`로 새 배열을 만들려고 했는데 안 됐다.

```js
// ❌ 안 됨 — forEach는 항상 undefined 반환
const doubled = nums.forEach((n) => n * 2);

// ✅ map을 써야 함
const doubled = nums.map((n) => n * 2);
```

## React에서 왜 `map`을 매일 쓰나

리스트 렌더링은 "데이터 배열 → JSX 배열" 변환이라 `map`이 정확히 맞다.

```jsx
{users.map((user) => <li key={user.id}>{user.name}</li>)}
```

`forEach`로는 JSX를 모아서 반환할 수 없다 (반환값이 undefined이므로).

참고: https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array
