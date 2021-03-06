# simple test with ReactDOM

## Background

> "The more your tests resemble the way your software is used, the more
> confidence they can give you." -
> [@kentcdodds](https://twitter.com/kentcdodds/status/977018512689455106)

This is a critical principle that you'll be learning about through this whole
workshop. Everything we do with testing our React components is walking the line
of trade-offs of getting our tests to resemble the way our software is actually
used and having something that's reasonably possible for testing.

When we think about how things are used, we need to consider who the users are:

1. The end user that's interacting with our code (clicking buttons/etc)
2. The developer user that's actually using our code (rendering it, calling our
   functions, etc.)

Often a _third_ user creeps into our tests and we want to avoid them as much as
possible: [The Test User](https://kentcdodds.com/blog/avoid-the-test-user).

When it comes to React components, our developer user will render our component
with `react-dom`'s `createRoot` API (similar concept for React Native) and in
some cases they'll pass props and/or wrap it in a context provider. The end user
will click buttons and assert on the output.

So that's what our test will do.

📜 You'll be using assertions from jest: https://jestjs.io/docs/en/expect

## Component

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

## Exercise

We have a simple counter component (if you have the app running locally, you can
interact with it at: http://localhost:3000/counter). Your job is to make sure
that it starts out saying "Current count: 0" and that when the user clicks
"Increment" it'll increase the count and when they click "Decrement" it'll
decrease the count.

To do this, you'll need to create a DOM node, add it to the body, and render the
component to that DOM node. You'll also need to clean up the DOM when your test
is finished so the next test has a clean DOM to interact with.

> NOTE: In React v18, you're required to wrap all your interactions in
> [`act`](https://reactjs.org/docs/test-utils.html#act). So when you render and
> click buttons make sure to do that. Luckily React Testing Library does this
> for you automatically so you'll be able to remove that when we get to that bit
> 🥳

```js
import * as React from 'react'
import {act} from 'react-dom/test-utils'
import {createRoot} from 'react-dom/client'
import Counter from '../../components/counter'

// NOTE: this is a new requirement in React 18
// https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html#configuring-your-testing-environment
// Luckily, it's handled for you by React Testing Library :)
global.IS_REACT_ACT_ENVIRONMENT = true

beforeEach(() => {
  document.body.innerHTML = ''
})

test('counter increments and decrements when the buttons are clicked', () => {
  const div = document.createElement('div')
  document.body.append(div)

  const root = createRoot(div)
  act(() => root.render(<Counter />))
  const [decrement, increment] = div.querySelectorAll('button')
  const message = div.firstChild.querySelector('div')

  expect(message.textContent).toBe('Current count: 0')
  act(() => increment.click())
  expect(message.textContent).toBe('Current count: 1')
  act(() => decrement.click())
  expect(message.textContent).toBe('Current count: 0')
})
```

## Extra Credit

### 1. 💯 use dispatchEvent

Using `.click` on a DOM node works fine, but what if you wanted to fire an event
that doesn't have a dedicated method (like mouseover). Rather than use
`button.click()`, try using `button.dispatchEvent`: 📜
https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent

> NOTE: Make sure that your event config sets `bubbles: true`

💰 Here's how you create a MouseEvent:

```javascript
new MouseEvent('click', {
  bubbles: true,
  cancelable: true,
  button: 0,
})
```

```js
import {act} from 'react-dom/test-utils'
import {createRoot} from 'react-dom/client'
import Counter from '../../components/counter'

// NOTE: this is a new requirement in React 18
// https://reactjs.org/blog/2022/03/08/react-18-upgrade-guide.html#configuring-your-testing-environment
// Luckily, it's handled for you by React Testing Library :)
global.IS_REACT_ACT_ENVIRONMENT = true

beforeEach(() => {
  document.body.innerHTML = ''
})

test('counter increments and decrements when the buttons are clicked', () => {
  const div = document.createElement('div')
  document.body.append(div)

  const root = createRoot(div)
  act(() => root.render(<Counter />))
  const [decrement, increment] = div.querySelectorAll('button')
  const message = div.firstChild.querySelector('div')

  expect(message.textContent).toBe('Current count: 0')
  const incrementClickEvent = new MouseEvent('click', {
    bubbles: true,
    cancelable: true,
    button: 0,
  })
  act(() => increment.dispatchEvent(incrementClickEvent))
  expect(message.textContent).toBe('Current count: 1')
  const decrementClickEvent = new MouseEvent('click', {
    bubbles: true,
    cancelable: true,
    button: 0,
  })
  act(() => decrement.dispatchEvent(decrementClickEvent))
  expect(message.textContent).toBe('Current count: 0')
})
```

