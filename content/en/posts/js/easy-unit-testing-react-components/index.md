---
date: "2015-10-03"
title: Easy unit testing React components (without a DOM)
tags:
  - js
  - react
  - tests
authors:
  - MoOx
header:
  credit: https://www.flickr.com/photos/dvids/7112879347
  linearGradient: 160deg, rgba(49, 142, 223, .9),rgba(115, 64, 196, .6)
template: Post
---

Recently, React team implemented a feature called
[Shallow rendering](http://facebook.github.io/react/docs/test-utils.html#shallow-rendering),
which

>lets you render a component "one level deep" and assert facts about
what its render method returns, without worrying about the behavior of child
components, which are not instantiated or rendered.
This does not require a DOM.

Sounds good, right? And guess what, Shallow rendering is currently the
[preferred way to test your React components](https://discuss.reactjs.org/t/whats-the-prefered-way-to-test-react-js-components/26).

As you can see in the post mentioned at the beginning of this one, the
actual code to test some components might seems a bit longer that what you could
expect.

Guess what? [Someone](https://github.com/vvo) created something pretty cool:
[react-element-to-jsx-string](https://github.com/algolia/react-element-to-jsx-string).

As the name of the package say, this library helps to render a react component
into a JSX string. Now things start to become interesting.

With those two things in mind (shallow render and react into jsx string), we can
easily add some basic unit tests to some components.

Let's do this with the following (dump) component:

```js
// web_modules/Picture/index.js

iimport React, { Component } from "react"
import { PropTypes } from "react"

const requiredComponent = PropTypes.oneOfType([
  PropTypes.element,
  PropTypes.func,
]).isRequired

export default class Picture extends Component {

  static propTypes = {
    img: PropTypes.object,
    title: PropTypes.string,
    Loader: requiredComponent,
    Title: requiredComponent,
  }

  render() {
    const {
      img,
      title,
      Loader,
      Title,
    } = this.props

    return (
      <div>
        {
          (!img || !img.src) && Loader &&
          <Loader />
        }
        {
          img && img.src &&
          <img src={ img.src } alt={ img.alt }/>
        }
        {
          title && Title &&
          <Title text={ title } />
        }
      </div>
    )
  }
}

```

This component display an image with a title (using a component) and if the
image data are not ready yet, it display a Loader component.

Now let's write some simple test for it. For the example we will use
[tape](https://medium.com/javascript-scene/why-i-use-tape-instead-of-mocha-so-should-you-6aa105d8eaf4)
with the help of [tape-jsx-equals](https://github.com/atabel/tape-jsx-equals).

```js
// web_modules/Picture/__tests__/index.js

import tape from "tape"
import addAssertions from "extend-tape"
import jsxEquals from "tape-jsx-equals"
const test = addAssertions(tape, { jsxEquals })

import React from "react"
import { createRenderer } from "react-addons-test-utils"

import Picture from ".."

// fixtures
const Loader = () => {}
const Title = () => {}

test("PageContainer is properly rendered", (t) => {
  const renderer = createRenderer()

  renderer.render(
    <Picture
      Loader={ Loader }
      Title={ Title }
    />
  )
  t.jsxEquals(
    renderer.getRenderOutput(),
    <div>
      <Loader />
    </div>,
    "can render a Loader component if no image data are passed"
  )

  renderer.render(
    <Picture
      Loader={ Loader }
      Title={ Title }
      img={ {
        src: "SRC",
        alt: "ALT",
      } }
    />
  )
  t.jsxEquals(
    renderer.getRenderOutput(),
    <div>
      <img src="SRC" alt="ALT" />
    </div>,
    "should render an image if data are passed"
  )

  renderer.render(
    <Picture
      Loader={ Loader }
      Title={ Title }
      img={ {
        src: "SRC",
        alt: "ALT",
      } }
      title={ "TITLE" }
    />
  )
  t.jsxEquals(
    renderer.getRenderOutput(),
    <div>
      <img src="SRC" alt="ALT" />
      <Title text="TITLE" />
    </div>,
    "can render a Title if data are passed"
  )

  t.end()
})
```

As you can see, tests here are pretty easy to write & straight forward.
These tests are minimal coverage to ensure you don't break things when you work
on a component.

You will probably need to write more complicated tests, so you should read the
[post written by Simon Smith](https://simonsmith.io/unit-testing-react-components-without-a-dom/)
that covers the same topic with more complicated use cases.

With all those examples, I hope you will not hesitate to add some testings to
all your React components now 😍.
