## version
현재 Vue의 버전을 노출한다.
### 타입
`string`
### 예시
```js
import { version } from 'vue'

console.log(version)
```

## `nextTick()`
다음 DOM 업데이트 플러시를 기다리기 위한 유틸리티다.

### 타입
```ts
function nextTick(callback?: () => void): Promise<void>
```
### 세부사항
Vue에서 반응형 상태를 변경하면, 그에 따른 DOM 업데이트는 동기적으로 적용되지 않는다.
대신 Vue는 "다음 틱"까지 이를 버퍼링하여, 여러 번 상태를 변경해도 각 컴포넌트가 한 번만 업데이트되도록 보장한다.

`nextTick()`은 상태 변경 직후에 DOM 업데이트가 완료될 때까지 기다릴 때 사용할 수 있다.
콜백을 인자로 전달하거나, 반환된 Promise를 await할 수 있다.
### 예시
```vue
<script setup>
import { ref, nextTick } from 'vue'

const count = ref(0)

async function increment() {
  count.value++

  // DOM이 아직 업데이트되지 않음
  console.log(document.getElementById('counter').textContent) // 0

  await nextTick()
  // DOM이 이제 업데이트됨
  console.log(document.getElementById('counter').textContent) // 1
}
</script>

<template>
  <button id="counter" @click="increment">{{ count }}</button>
</template>
```

## `defineComponent()`
타입 추론과 함께 Vue 컴포넌트를 정의하기 위한 타입 헬퍼이다.
### 타입
```ts
// 옵션 문법
function defineComponent(
  component: ComponentOptions
): ComponentConstructor

// 함수 문법 (3.3+ 필요)
function defineComponent(
  setup: ComponentOptions['setup'],
  extraOptions?: ComponentOptions
): () => any
```
> 가독성을 위해 타입이 단순화되었다.

### 세부사항
