# Advanced React Patterns

[1. Context Module Functions](#1-context-module-functions)

[2. Compound Components](#2-compound-components)

[3. Flexible Compound Components](#3-flexible-compound-components)

[4. Prop Collections and Getters](#4-prop-collections-and-getters)

[5. State Reducer](#5-state-reducer)

[6. Control Props](#6-control-props)

## 1. Context Module Functions

컨텍스트 모듈 함수 패턴을 사용하면 복잡한 상태 변경 세트를 트리 쉐이킹 및 지연 로드가 가능한 

유틸리티 함수로 캡슐화할 수 있습니다.

간단한 컨텍스트와 reducer의 예를 살펴보겠습니다.

```js
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({step = 1, initialCount = 0, ...props}) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return {...state, count: state.count + change}
        }
        case 'decrement': {
          return {...state, count: state.count - change}
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    {count: initialCount},
  )

  const value = [state, dispatch]
  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

export {CounterProvider, useCounter}
// src/screens/counter.js
import {useCounter} from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  const increment = () => dispatch({type: 'increment'})
  const decrement = () => dispatch({type: 'decrement'})
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={decrement}>-</button>
      <button onClick={increment}>+</button>
    </div>
  )
}
// src/index.js
import {CounterProvider} from 'context/counter'
import Counter from 'screens/counter'

function App() {
  return (
    <CounterProvider>
      <Counter />
    </CounterProvider>
  )
}
```

이것은 매우 훌륭한 api는 아니다.

호출해야 하는 일련의 디스패치 함수가 있는 경우 훨씬 더 골칫거리가 된다고 함.

첫 번째 경향은 helper 함수를 만들고 먼텍스트에 포함하는 것.

useCallback에 넣어주고 dependency array에 넣어준다.

```js
const increment = React.useCallback(
  () => dispatch({type: 'increment'}),
  [dispatch],
)
const decrement = React.useCallback(
  () => dispatch({type: 'decrement'}),
  [dispatch],
)
const value = {state, increment, decrement}
return <CounterContext.Provider value={value} {...props} />

// now users can consume it like this:

const {state, increment, decrement} = useCounter()
```

**헬퍼 메서드는 표면적으로 더 보기 좋은 구문 외에는 다른 목적으로 재생성하고 비교해야 하는 객체 정크입니다.**

Facebook이 권장하는 것은 패스 디스패치 이다.

```js
// src/context/counter.js
const CounterContext = React.createContext()

function CounterProvider({step = 1, initialCount = 0, ...props}) {
  const [state, dispatch] = React.useReducer(
    (state, action) => {
      const change = action.step ?? step
      switch (action.type) {
        case 'increment': {
          return {...state, count: state.count + change}
        }
        case 'decrement': {
          return {...state, count: state.count - change}
        }
        default: {
          throw new Error(`Unhandled action type: ${action.type}`)
        }
      }
    },
    {count: initialCount},
  )

  const value = [state, dispatch]

  return <CounterContext.Provider value={value} {...props} />
}

function useCounter() {
  const context = React.useContext(CounterContext)
  if (context === undefined) {
    throw new Error(`useCounter must be used within a CounterProvider`)
  }
  return context
}

const increment = dispatch => dispatch({type: 'increment'})
const decrement = dispatch => dispatch({type: 'decrement'})

export {CounterProvider, useCounter, increment, decrement}
// src/screens/counter.js
import {useCounter, increment, decrement} from 'context/counter'

function Counter() {
  const [state, dispatch] = useCounter()
  return (
    <div>
      <div>Current Count: {state.count}</div>
      <button onClick={() => decrement(dispatch)}>-</button>
      <button onClick={() => increment(dispatch)}>+</button>
    </div>
  )
}
```

이 패턴은 과잉처럼 보이고 실제로 그렇지만

경우에 따라 중복을 줄이는 데 도움이 될 수 있다.

성능을 개선하고 종속성 목록(dependency arrary)의 실수를 방지하는데도 도움이 된다. (때로는 !)

## 2. Compound Components

## 3. Flexible Compound Components

## 4. Prop Collections and Getters

## 5. State Reducer

## 6. Control Props
