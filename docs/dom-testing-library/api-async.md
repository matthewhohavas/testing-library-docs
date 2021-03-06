---
id: api-async
title: Async Utilities
---

Several utilities are provided for dealing with asynchronous code. These can be
useful to wait for an element to appear or disappear in response to an action.
(See the [guide to testing disappearance](guide-disappearance.md).)

All the async utils are built on top of `waitFor`.

## `waitFor`

```typescript
function waitFor<T>(
  callback: () => void,
  options?: {
    container?: HTMLElement
    timeout?: number
    interval?: number
    mutationObserverOptions?: MutationObserverInit
  }
): Promise<T>
```

When in need to wait for any period of time you can use `waitFor`, to wait for your
expectations to pass. Here's a simple example:

```javascript
// ...
// Wait until the callback does not throw an error. In this case, that means
// it'll wait until the mock function has been called once.
await waitFor(() => expect(mockAPI).toHaveBeenCalledTimes(1))
// ...
```

This can be useful if you have a unit test that mocks API calls and you need to
wait for your mock promises to all resolve.

The default `container` is the global `document`. Make sure the elements you
wait for are descendants of `container`.

The default `interval` is `50ms`. However it will run your callback immediately
before starting the intervals.

The default `timeout` is `1000ms` which will keep you under
[Jest's default timeout of `5000ms`](https://jestjs.io/docs/en/jest-object.html#jestsettimeouttimeout).

<a name="mutationobserveroptions"></a>The default `mutationObserverOptions` is
`{subtree: true, childList: true, attributes: true, characterData: true}` which
will detect additions and removals of child elements (including text nodes) in
the `container` and any of its descendants. It will also detect attribute
changes. When any of those changes occur, it will re-run the callback.

## `waitForElementToBeRemoved`

```typescript
function waitForElementToBeRemoved<T>(
  callback: (() => T) | T,
  options?: {
    container?: HTMLElement
    timeout?: number
    interval?: number
    mutationObserverOptions?: MutationObserverInit
  }
): Promise<T>
```

To wait for the removal of element(s) from the DOM you can use
`waitForElementToBeRemoved`. The `waitForElementToBeRemoved` function is a small
wrapper around the `wait` utility.

The first argument must be an element, array of elements, or a callback which returns
an element or array of elements.

Here is an example where the promise resolves with `true` because the element is
removed:

```javascript
const el = document.querySelector('div.getOuttaHere')

waitForElementToBeRemoved(document.querySelector('div.getOuttaHere'))
  .then(() => console.log('Element no longer in DOM'))

el.setAttribute('data-neat', true)
// other mutations are ignored...

el.parentElement.removeChild(el)
// logs 'Element no longer in DOM'
```

`waitForElementToBeRemoved` will throw an error if the first argument is `null`
or an empty array:

```javascript
waitForElementToBeRemoved(null).catch(err => console.log(err))
waitForElementToBeRemoved(queryByText(/not here/i)).catch(err => console.log(err))
waitForElementToBeRemoved(queryAllByText(/not here/i)).catch(err => console.log(err))
waitForElementToBeRemoved(() => getByText(/not here/i)).catch(err => console.log(err))

// Error: The element(s) given to waitForElementToBeRemoved are already removed. waitForElementToBeRemoved requires that the element(s) exist(s) before waiting for removal.
```

The options object is forwarded to `waitFor`.

## `wait` (DEPRECATED, use waitFor instead)

```typescript
function wait<T>(
  callback: () => void,
  options?: {
    container?: HTMLElement
    timeout?: number
    interval?: number
    mutationObserverOptions?: MutationObserverInit
  }
): Promise<T>
```
Previously, wait was a wrapper around wait-for-expect and used polling instead of a MutationObserver to look for changes.  It is now an alias to waitFor and will be removed in a future release.

Unlike wait, the callback parameter is mandatory in waitFor. Although you can migrate an existing `wait()` call to `waitFor( () => {} )`, it is considered bad practice to use an empty callback because it will make the tests more fragile. 

## `waitForDomChange` (DEPRECATED, use waitFor instead)

```typescript
function waitForDomChange<T>(options?: {
  container?: HTMLElement
  timeout?: number
  mutationObserverOptions?: MutationObserverInit
}): Promise<T>
```

When in need to wait for the DOM to change you can use `waitForDomChange`. The
`waitForDomChange` function is a small wrapper around the
[`MutationObserver`](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver).

Here is an example where the promise will be resolved because the container is
changed:

```javascript
const container = document.createElement('div')
waitForDomChange({ container })
  .then(() => console.log('DOM changed!'))
  .catch(err => console.log(`Error you need to deal with: ${err}`))
container.append(document.createElement('p'))
// if 👆 was the only code affecting the container and it was not run,
// waitForDomChange would throw an error
```

The promise will resolve with a
[`mutationsList`](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver/MutationObserver)
which you can use to determine what kind of a change (or changes) affected the
container

```javascript
const container = document.createElement('div')
container.setAttribute('data-cool', 'true')
waitForDomChange({ container }).then(mutationsList => {
  const mutation = mutationsList[0]
  console.log(
    `was cool: ${mutation.oldValue}\ncurrently cool: ${
      mutation.target.dataset.cool
    }`
  )
})
container.setAttribute('data-cool', 'false')
/*
  logs:
    was cool: true
    currently cool: false
*/
```

The default `container` is the global `document`. Make sure the elements you
wait for are descendants of `container`.

The default `timeout` is `1000ms` which will keep you under
[Jest's default timeout of `5000ms`](https://jestjs.io/docs/en/jest-object.html#jestsettimeouttimeout).

<a name="mutationobserveroptions"></a>The default `mutationObserverOptions` is
`{subtree: true, childList: true, attributes: true, characterData: true}` which
will detect additions and removals of child elements (including text nodes) in
the `container` and any of its descendants. It will also detect attribute
changes.


## `waitForElement` (DEPRECATED, use `find*` queries or `waitFor`)

```typescript
function waitForElement<T>(
  callback: () => T,
  options?: {
    container?: HTMLElement
    timeout?: number
    mutationObserverOptions?: MutationObserverInit
  }
): Promise<T>
```

When in need to wait for DOM elements to appear, disappear, or change you can
use `waitForElement`. The `waitForElement` function is a small wrapper around
the
[`MutationObserver`](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver).

Here's a simple example:

```javascript
// ...
// Wait until the callback does not throw an error and returns a truthy value. In this case, that means
// it'll wait until we can get a form control with a label that matches "username".
// Previously, the difference from `wait` is that rather than running your callback on
// an interval, it's run as soon as there are DOM changes in the container
// and returns the value returned by the callback.
const usernameElement = await waitForElement(
  () => getByLabelText(container, 'username'),
  { container }
)
usernameElement.value = 'chucknorris'
// ...
```

You can also wait for multiple elements at once:

```javascript
const [usernameElement, passwordElement] = await waitForElement(
  () => [
    getByLabelText(container, 'username'),
    getByLabelText(container, 'password'),
  ],
  { container }
)
```

The default `container` is the global `document`. Make sure the elements you
wait for will be attached to it, or set a different `container`.

The default `timeout` is `4500ms` which will keep you under
[Jest's default timeout of `5000ms`](https://facebook.github.io/jest/docs/en/jest-object.html#jestsettimeouttimeout).

<a name="mutationobserveroptions"></a>The default `mutationObserverOptions` is
`{subtree: true, childList: true, attributes: true, characterData: true}` which
will detect additions and removals of child elements (including text nodes) in
the `container` and any of its descendants. It will also detect attribute
changes.
