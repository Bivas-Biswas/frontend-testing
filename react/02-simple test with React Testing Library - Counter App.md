# simple test with React Testing Library - Counter App

## Background

As much as I enjoy creating DOM nodes and appending them to the `body`, that
seems like boilerplate that could live in an abstraction. And it is! Among other
things, that's what React Testing Library does.

[React Testing Library](https://testing-library.com/react) is the React
implementation of the [DOM Testing Library](https://testing-library.com)
(there's also a
[React Native Testing Library](https://testing-library.com/react-native) and
many others). Testing Library comes with a ton of really useful features which
we'll be using throughout this workshop, but for now, we'll just start out with
cleaning up some of this boilerplate.

Here's a simple example of how to use this:

```javascript
import {render, fireEvent, screen} from '@testing-library/react'

test('it works', () => {
  const {container} = render(<Example />)
  // container is the div that your component has been mounted onto.

  const input = container.querySelector('input')
  fireEvent.mouseEnter(input) // fires a mouseEnter event on the input

  screen.debug() // logs the current state of the DOM (with syntax highlighting!)
})
```

Notice the lack of `cleanup` functionality. That's thanks to
`@testing-library/react`'s
[auto-cleanup feature](https://testing-library.com/docs/react-testing-library/api#cleanup)

Another automatic feature of React Testing Library is its handling of
[React's `act` function](https://reactjs.org/docs/test-utils.html#act). If
you've ever seen a warning about something not being wrapped in `act`, that's
what we're talking about. As mentioned in the React docs, React Testing Library
is recommended for avoiding the issues `act` is warning you about. You can learn
more about this from my blog post
[Fix the "not wrapped in act(...)" warning](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning).

## Component

```js
import * as React from 'react'

function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(c => c + 1)
  const decrement = () => setCount(c => c - 1)
  return (
    <div>
      <div>Current count: {count}</div>
      <button onClick={decrement}>Decrement</button>
      <button onClick={increment}>Increment</button>
    </div>
  )
}

export default Counter
```

## Exercise

In this exercise, we're going to remove some of our boilerplate that React
Testing Library does for us. The emoji should guide you pretty well on this one
so I'll let you have at it!

```js
import * as React from 'react'
import {render, fireEvent} from '@testing-library/react'
import Counter from '../../components/counter'

test('counter increments and decrements when the buttons are clicked', () => {
  const {container} = render(<Counter />)
  const [decrement, increment] = container.querySelectorAll('button')
  const message = container.firstChild.querySelector('div')

  expect(message.textContent).toBe('Current count: 0')
  fireEvent.click(increment)
  expect(message.textContent).toBe('Current count: 1')
  fireEvent.click(decrement)
  expect(message.textContent).toBe('Current count: 0')
})
```

## Extra Credit

### 1. 💯 use @testing-library/jest-dom

Testing Library also has a suite of assertions that can be installed with Jest.
They're already added to this project, so you can switch from Jest's built-in
assertions to more specific assertions which will give you better error
messages. So go ahead and swap the `expect(message.textContent).toBe(...)`
assertions with `toHaveTextContent` from
[`@testing-library/jest-dom`](http://testing-library.com/jest-dom).

```js
import * as React from 'react'
import {render, fireEvent} from '@testing-library/react'
import Counter from '../../components/counter'

test('counter increments and decrements when the buttons are clicked', () => {
  const {container} = render(<Counter />)
  const [decrement, increment] = container.querySelectorAll('button')
  const message = container.firstChild.querySelector('div')

  expect(message).toHaveTextContent('Current count: 0')
  fireEvent.click(increment)
  expect(message).toHaveTextContent('Current count: 1')
  fireEvent.click(decrement)
  expect(message).toHaveTextContent('Current count: 0')
})
```
