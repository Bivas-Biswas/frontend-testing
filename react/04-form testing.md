# form testing

## Background

Our users spend a lot of time interacting with forms and many of our forms are
among the most important parts of our application (like the "checkout" form of
an e-commerce app or the "login" form of most apps). Because of this, it's
pretty critical to have confidence that those continue to work over time.

You need to ensure that the user can find inputs in the form, fill in their
information, and validate that when they submit the form the submitted data is
correct.

## Components

```js
import * as React from 'react'

function Login({onSubmit}) {
  function handleSubmit(event) {
    event.preventDefault()
    const {username, password} = event.target.elements

    onSubmit({
      username: username.value,
      password: password.value,
    })
  }
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label htmlFor="username-field">Username</label>
        <input id="username-field" name="username" type="text" />
      </div>
      <div>
        <label htmlFor="password-field">Password</label>
        <input id="password-field" name="password" type="password" />
      </div>
      <div>
        <button type="submit">Submit</button>
      </div>
    </form>
  )
}

export default Login
```

## Exercise

In this exercise, we'll be testing a Login form that has a username and
password. The Login form accepts an `onSubmit` handler which will be called with
the form data when the user submits the form. Your job is to write a test for
this form.

Make sure to keep your test implementation detail free and refactor friendly!

```js
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Login from '../../components/login'

test('submitting the form calls onSubmit with username and password', async () => {
  let submittedData
  const handleSubmit = data => (submittedData = data)
  render(<Login onSubmit={handleSubmit} />)
  const username = 'chucknorris'
  const password = 'i need no password'

  await userEvent.type(screen.getByLabelText(/username/i), username)
  await userEvent.type(screen.getByLabelText(/password/i), password)
  await userEvent.click(screen.getByRole('button', {name: /submit/i}))

  expect(submittedData).toEqual({
    username,
    password,
  })
})
```

## Extra Credit

### 1. 💯 use a jest mock function

Jest has built-in "mock" function APIs. Rather than creating the `submittedData`
variable, try to use a mock function and assert it was called correctly:

- 📜 `jest.fn()`: https://jestjs.io/docs/en/mock-function-api
- 📜 `toHaveBeenCalledWith`:
  https://jestjs.io/docs/en/expect#tohavebeencalledwitharg1-arg2-

```js
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import Login from '../../components/login'

test('submitting the form calls onSubmit with username and password', async () => {
  const handleSubmit = jest.fn()
  render(<Login onSubmit={handleSubmit} />)
  const username = 'chucknorris'
  const password = 'i need no password'

  await userEvent.type(screen.getByLabelText(/username/i), username)
  await userEvent.type(screen.getByLabelText(/password/i), password)
  await userEvent.click(screen.getByRole('button', {name: /submit/i}))

  expect(handleSubmit).toHaveBeenCalledWith({
    username,
    password,
  })
  expect(handleSubmit).toHaveBeenCalledTimes(1)
})
```

### 2. 💯 generate test data

An important thing to keep in mind when testing is simplifying the maintenance
of the tests by reducing the amount of unrelated cruft in the test. You want to
make it so the code for the test communicates what's important and what is not
important.

Specifically, in my solution I have these values:

```javascript
const username = 'chucknorris'
const password = 'i need no password'
```

Does my code behave differently when the username is `chucknorris`? Do I have
special logic around that? Without looking at the implementation I cannot be
completely sure. What would be better is if the code communicated that the
actual value is irrelevant. But how do you communicate that? A code comment?
Nah, let's generate the value!

```javascript
const username = getRandomUsername()
const password = getRandomPassword()
```

That communicates the intent really well. As a reader of the test I can think:
"Oh, ok, great, so it doesn't matter what the username _is_, just that it's a
typical username."

Luckily, there's a package we can use for this called
[faker](https://www.npmjs.com/package/@faker-js/faker). You can get a random
username and password from `faker.internet.userName()` (note the capital `N`)
and `faker.internet.password()`. We've already got it installed in this project,
so go ahead and import that and generate the username and password.

Even better, create a `buildLoginForm` function which allows me to call it like
this:

```javascript
const {username, password} = buildLoginForm()
```

```js
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import faker from 'faker'
import Login from '../../components/login'

function buildLoginForm() {
  return {
    username: faker.internet.userName(),
    password: faker.internet.password(),
  }
}

test('submitting the form calls onSubmit with username and password', async () => {
  const handleSubmit = jest.fn()
  render(<Login onSubmit={handleSubmit} />)
  const {username, password} = buildLoginForm()

  await userEvent.type(screen.getByLabelText(/username/i), username)
  await userEvent.type(screen.getByLabelText(/password/i), password)
  await userEvent.click(screen.getByRole('button', {name: /submit/i}))

  expect(handleSubmit).toHaveBeenCalledWith({
    username,
    password,
  })
  expect(handleSubmit).toHaveBeenCalledTimes(1)
})
```

### 3. 💯 allow for overrides

Sometimes you actually _do_ have some specific data that's important for the
test. For example, if our form performed validation on the password being a
certain strength, then we might not want a randomly generated password and we'd
instead want a specific password.

Try to make your `buildLoginForm` function accept overrides as well:

```javascript
const {username, password} = buildLoginForm({password: 'abc'})
// password === 'abc'
```

That communicates the reader of the test: "We just need a normal login form,
except the password needs to be something specific for this test."

```js
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import faker from 'faker'
import Login from '../../components/login'

function buildLoginForm(overrides) {
  return {
    username: faker.internet.userName(),
    password: faker.internet.password(),
    ...overrides,
  }
}

test('submitting the form calls onSubmit with username and password', async () => {
  const handleSubmit = jest.fn()
  render(<Login onSubmit={handleSubmit} />)
  const {username, password} = buildLoginForm()

  await userEvent.type(screen.getByLabelText(/username/i), username)
  await userEvent.type(screen.getByLabelText(/password/i), password)
  await userEvent.click(screen.getByRole('button', {name: /submit/i}))

  expect(handleSubmit).toHaveBeenCalledWith({
    username,
    password,
  })
  expect(handleSubmit).toHaveBeenCalledTimes(1)
})
```

### 4. 💯 use Test Data Bot

There's a library I like to use for generating test data:
[`@jackfranklin/test-data-bot`](https://www.npmjs.com/package/@jackfranklin/test-data-bot).
It provides a few nice utilities. Check out the docs there and swap your custom
`buildLoginForm` with one you create using the Test Data Bot.

```js
import * as React from 'react'
import {render, screen} from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import {build, fake} from '@jackfranklin/test-data-bot'
import Login from '../../components/login'

const buildLoginForm = build({
  fields: {
    username: fake(f => f.internet.userName()),
    password: fake(f => f.internet.password()),
  },
})

test('submitting the form calls onSubmit with username and password', async () => {
  const handleSubmit = jest.fn()
  render(<Login onSubmit={handleSubmit} />)
  const {username, password} = buildLoginForm()

  await userEvent.type(screen.getByLabelText(/username/i), username)
  await userEvent.type(screen.getByLabelText(/password/i), password)
  await userEvent.click(screen.getByRole('button', {name: /submit/i}))

  expect(handleSubmit).toHaveBeenCalledWith({
    username,
    password,
  })
  expect(handleSubmit).toHaveBeenCalledTimes(1)
})
```
