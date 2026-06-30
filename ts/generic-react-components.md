# React + TS 제네릭 컴포넌트 (Generic Components)

리스트, 셀렉트, 테이블처럼 **어떤 타입의 데이터든 받아서 렌더링하는** 컴포넌트를 만들다 보면
props 타입을 `any`로 두고 싶어진다. 하지만 그러면 자식으로 넘긴 콜백에서 타입이 다 풀려버린다.
이럴 때 컴포넌트 자체를 제네릭(generic)으로 만들면 호출하는 쪽 데이터에 맞춰 타입이 자동으로 따라온다.

## any로 두면 생기는 문제

```tsx
interface BadListProps {
  items: any[];
  renderItem: (item: any) => React.ReactNode;
}

function BadList({ items, renderItem }: BadListProps) {
  return <ul>{items.map(renderItem)}</ul>;
}

// 사용할 때 item이 any라 자동완성도 안 되고 오타도 못 잡는다
<BadList items={users} renderItem={(u) => <li>{u.naem}</li>} />; // 오타인데 통과됨
```

`items`에 무슨 타입을 넣든 `renderItem`의 `item`은 `any`다. 타입스크립트를 쓰는 의미가 없어진다.

## 제네릭 컴포넌트로 만들기

props 인터페이스에 타입 파라미터 `<T>`를 붙이고, 컴포넌트 함수에도 `<T,>`를 선언한다.

```tsx
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyOf: (item: T) => string | number;
}

function List<T>({ items, renderItem, keyOf }: ListProps<T>) {
  return (
    <ul>
      {items.map((item) => (
        <li key={keyOf(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

이제 호출하는 쪽에서 넘긴 `items` 타입이 `T`로 추론된다.

```tsx
interface Character {
  id: number;
  name: string;
}

const characters: Character[] = [
  { id: 1, name: "짱구" },
  { id: 2, name: "둘리" },
];

// item이 Character로 추론됨 → 자동완성 O, 오타 잡힘 O
<List
  items={characters}
  keyOf={(c) => c.id}
  renderItem={(c) => <span>{c.name}</span>}
/>;
```

`renderItem`의 `c`에 점을 찍으면 `id`, `name`이 뜬다. `c.naem`을 쓰면 바로 에러가 난다.

## 화살표 함수로 만들 때: `<T,>` 트레일링 콤마

`.tsx` 파일에서 화살표 함수에 그냥 `<T>`를 쓰면 컴파일러가 **JSX 태그로 오해한다**.
그래서 트레일링 콤마를 붙여 `<T,>`로 쓰거나 `extends`를 달아준다.

```tsx
// ❌ JSX로 착각해서 에러
const List = <T>(props: ListProps<T>) => { /* ... */ };

// ✅ 트레일링 콤마
const List = <T,>(props: ListProps<T>) => { /* ... */ };

// ✅ extends로 명시 (이쪽이 더 명확)
const List = <T extends unknown>(props: ListProps<T>) => { /* ... */ };
```

`function` 선언으로 쓰면 이 문제가 없어서, 제네릭 컴포넌트는 `function` 키워드가 편하다.

## 제약 걸기 (extends)

"id를 가진 데이터만 받겠다" 같은 제약은 `extends`로 건다.

```tsx
interface HasId {
  id: string | number;
}

function Table<T extends HasId>({ rows }: { rows: T[] }) {
  return (
    <table>
      <tbody>
        {rows.map((row) => (
          <tr key={row.id}>{/* row.id 접근 보장됨 */}</tr>
        ))}
      </tbody>
    </table>
  );
}
```

`T extends HasId` 덕분에 `row.id`를 안전하게 쓸 수 있고, `id` 없는 데이터를 넘기면 호출 쪽에서 에러가 난다.

## any vs 제네릭 비교

| 구분 | `any[]` props | 제네릭 `<T>` props |
| --- | --- | --- |
| 콜백 인자 타입 | `any` (다 풀림) | `T`로 추론 |
| 자동완성 | 안 됨 | 됨 |
| 오타/필드 검증 | 못 잡음 | 잡음 |
| 호출부와 타입 연결 | 끊김 | 이어짐 |

## 헷갈렸던 포인트

- **`<T,>`의 콤마는 오타가 아니다.** `.tsx`에서 화살표 함수 제네릭이 JSX와 충돌하는 걸 피하려는 문법이다. `.ts` 파일에선 필요 없다.

- **`forwardRef`와 제네릭은 같이 쓰기 까다롭다.** `forwardRef`로 감싸면 타입 추론이 깨지는 경우가 많다. 보통 함수 컴포넌트로 두거나, 별도 타입 단언으로 우회한다.

- **제네릭 기본값도 줄 수 있다.** `function List<T = string>(...)`처럼 기본 타입을 정해두면 타입을 명시하지 않아도 동작한다.

- **props 구조 분해와 추론은 별개다.** `<T,>`만 선언해두면 props를 어떻게 구조 분해하든 추론은 호출부 `items`에서 시작된다.

## 요약

- 어떤 데이터든 받는 재사용 컴포넌트는 `any` 대신 제네릭 `<T>`로 만든다
- props 인터페이스 `ListProps<T>` + 컴포넌트 `function List<T>(...)` 형태가 기본
- 화살표 함수로 쓸 땐 `.tsx`에서 `<T,>` 트레일링 콤마 필요
- 특정 필드가 필요하면 `T extends HasId`로 제약을 건다
- 호출부 데이터 타입이 그대로 콜백까지 따라와서 자동완성·오타 검증이 살아난다

참고: https://react-typescript-cheatsheet.netlify.app/docs/advanced/patterns_by_usecase/#generic-components
