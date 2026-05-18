# 이벤트 버블링 / 캡처링 (Event Bubbling / Capturing)

DOM 이벤트는 발생 즉시 딱 그 요소에서만 실행되지 않는다. 부모 방향으로 쭉 타고 올라가거나, 반대로 위에서부터 내려오기도 한다.

## 이벤트 전파 3단계

클릭 같은 이벤트는 DOM을 따라 세 단계로 이동한다.

```
1. 캡처링(Capturing) — window → document → html → ... → 타겟
2. 타겟(Target)      — 실제 이벤트 발생 요소
3. 버블링(Bubbling)  — 타겟 → ... → html → document → window
```

기본적으로 `addEventListener`는 **버블링 단계**에서 실행된다.

## 버블링 — 아래에서 위로

```html
<div id="outer">
  <button id="inner">클릭</button>
</div>
```

```js
document.getElementById("outer").addEventListener("click", () => {
  console.log("outer 클릭됨");
});

document.getElementById("inner").addEventListener("click", () => {
  console.log("inner 클릭됨");
});
```

버튼을 클릭하면:

```
inner 클릭됨
outer 클릭됨
```

`inner` → `outer` 순서로 올라간다. 이게 버블링.

## 버블링 막기 — stopPropagation

부모까지 전파되는 게 싫으면 `stopPropagation()`으로 막는다.

```js
document.getElementById("inner").addEventListener("click", (e) => {
  e.stopPropagation();
  console.log("inner 클릭됨 (여기서 멈춤)");
});
```

이제 `outer` 핸들러는 실행되지 않는다.

> **주의**: `stopPropagation`을 남발하면 부모에 달린 다른 핸들러가 조용히 죽어버린다. 디버깅이 어려워지니 꼭 필요한 경우에만 쓴다.

## 캡처링 — 위에서 아래로

`addEventListener` 세 번째 인자로 `{ capture: true }` (또는 `true`) 를 넘기면 캡처링 단계에서 실행된다.

```js
document.getElementById("outer").addEventListener(
  "click",
  () => { console.log("outer 캡처링"); },
  { capture: true }
);

document.getElementById("inner").addEventListener("click", () => {
  console.log("inner 버블링");
});
```

버튼 클릭 시:

```
outer 캡처링   ← 캡처링 단계(위에서 내려옴)
inner 버블링   ← 버블링 단계
```

실무에서 캡처링을 명시적으로 쓰는 경우는 드물다.

## 이벤트 위임 (Event Delegation)

버블링 덕분에 쓸 수 있는 패턴이다. 자식 요소 하나하나에 핸들러를 붙이는 대신, 공통 부모 하나에만 달아서 처리한다.

```html
<ul id="menu">
  <li data-item="짱구">짱구</li>
  <li data-item="둘리">둘리</li>
  <li data-item="또치">또치</li>
</ul>
```

```js
document.getElementById("menu").addEventListener("click", (e) => {
  const item = e.target.closest("li");
  if (!item) return;
  console.log(item.dataset.item, "선택됨");
});
```

`li`가 나중에 동적으로 추가돼도 핸들러를 새로 달 필요가 없다. 메모리도 아낀다.

## event.target vs event.currentTarget

| 속성 | 의미 |
|---|---|
| `event.target` | 실제로 클릭된 요소 (이벤트 발생 지점) |
| `event.currentTarget` | 현재 핸들러가 붙어있는 요소 |

```js
document.getElementById("outer").addEventListener("click", (e) => {
  console.log(e.target);        // 실제 클릭된 inner 버튼
  console.log(e.currentTarget); // 핸들러가 붙은 outer div
});
```

위임 패턴에서 `e.target`으로 실제 클릭 요소를 구분하는 게 핵심.

## 버블링이 안 되는 이벤트

모든 이벤트가 버블링되는 건 아니다. 대표적인 예외:

| 이벤트 | 버블링 |
|---|---|
| `focus` / `blur` | X (대신 `focusin` / `focusout` 사용) |
| `mouseenter` / `mouseleave` | X (대신 `mouseover` / `mouseout` 사용) |
| `scroll` | 대부분 X |
| `load` | X |

위임 패턴이 안 먹히는 이유가 여기서 나오는 경우가 많다.

## 헷갈렸던 포인트

- **버블링은 기본, 캡처링은 옵션**. 평소엔 버블링만 신경 쓰면 된다.
- **`stopPropagation` ≠ `preventDefault`**. `stopPropagation`은 이벤트 전파를 막고, `preventDefault`는 브라우저 기본 동작(링크 이동, 폼 제출 등)을 막는다.
- **이벤트 위임할 때 `e.target`이 예상 요소의 자식일 수 있다**. `<li>` 안에 `<span>`이 있으면 span이 `target`이 된다. `e.target.closest("li")`로 올라가면서 찾는 게 안전하다.

## 요약

- 이벤트는 **캡처링(위→아래)** → **타겟** → **버블링(아래→위)** 순서로 전파된다.
- 기본은 버블링 단계에서 핸들러 실행.
- 전파를 막으려면 `e.stopPropagation()`, 기본 동작을 막으려면 `e.preventDefault()`.
- **이벤트 위임**: 부모 하나에 핸들러를 달아 자식 이벤트를 버블링으로 처리 → 동적 자식에도 유효, 메모리 절약.
- `event.target` = 클릭된 요소, `event.currentTarget` = 핸들러가 붙은 요소.

참고: https://developer.mozilla.org/ko/docs/Learn_web_development/Core/Scripting/Event_bubbling
