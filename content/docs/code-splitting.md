---
id: code-splitting
title: Code-Splitting
permalink: docs/code-splitting.html
---

## 打包

大多数 React 应用都会通过类似 [Webpack](https://webpack.js.org/) 或 [Browserify](http://browserify.org/) 构建自己的文件 “包（bundled）”。构建是一个将文件引入并合并到一个单独文件：“包（bundle）” 的环节。该包包含在一个 web  页面上用以立刻加载整个应用。

#### 例子

**App:**

```js
// app.js
import { add } from './math.js';

console.log(add(16, 26)); // 42
```

```js
// math.js
export function add(a, b) {
  return a + b;
}
```

**Bundle:**

```js
function add(a, b) {
  return a + b;
}

console.log(add(16, 26)); // 42
```

> 注意：
>
> 你的包最终看起来会和这个有很大的不同。

若你正在使用 [Create React App](https://github.com/facebookincubator/create-react-app), [Next.js](https://github.com/zeit/next.js/)、[Gatsby](https://www.gatsbyjs.org/) 或类似工具，你将有一个能够立即对你的应用进行打包的 webpack 配置。

若你未使用，你职责需要自己来进行配置。例如，查看 webpack 文档上的[安装](https://webpack.js.org/guides/installation/)和
[入门](https://webpack.js.org/guides/getting-started/)指南。

## 代码分隔

打包非常棒，但随着你的应用增长，你的代码包也将随之增长。尤其是如果你包含了体积大的第三方库。你需要关注你代码包中所包含的代码以避免体积过大而使得加载时间过长。

为了避免清理大体积的代码包，在一开始就解决该问题并开始对代码包进行分割则十分不错。[代码分割](https://webpack.js.org/guides/code-splitting/)是由如 webpack 和 Browserify（通过 [factor-bundle](https://github.com/browserify/factor-bundle)）等打包器支持的一项能够创建多个包并在运行时动态加载的特性。

代码分割你的应用能够帮助你“懒加载”当前用户所需要的内容，能够显著地提高你的应用性能。尽管你不用减少你的应用中过多的代码体积，你仍然能够避免加载用户永远不需要的代码，并在初始化时候减少所需加载的代码量。

## `import()`

在你的应用中引入代码分割的最佳方式是通过动态 `import()` 语法。

**之前:**

```js
import { add } from './math';

console.log(add(16, 26));
```

**之后:**

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

> Note:
>
> The dynamic `import()` syntax is a ECMAScript (JavaScript)
> [proposal](https://github.com/tc39/proposal-dynamic-import) not currently
> part of the language standard. It is expected to be accepted in the
> near future.

When Webpack comes across this syntax, it automatically starts code-splitting
your app. If you're using Create React App, this is already configured for you
and you can [start using it](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md#code-splitting) immediately. It's also supported
out of the box in [Next.js](https://github.com/zeit/next.js/#dynamic-import).

If you're setting up Webpack yourself, you'll probably want to read Webpack's
[guide on code splitting](https://webpack.js.org/guides/code-splitting/). Your Webpack config should look vaguely [like this](https://gist.github.com/gaearon/ca6e803f5c604d37468b0091d9959269).

When using [Babel](http://babeljs.io/), you'll need to make sure that Babel can
parse the dynamic import syntax but is not transforming it. For that you will need [babel-plugin-syntax-dynamic-import](https://yarnpkg.com/en/package/babel-plugin-syntax-dynamic-import).

## `React.lazy`

> Note:
>
> `React.lazy` and Suspense is not yet available for server-side rendering. If you want to do code-splitting in a server rendered app, we recommend [Loadable Components](https://github.com/smooth-code/loadable-components). It has a nice [guide for bundle splitting with server-side rendering](https://github.com/smooth-code/loadable-components/blob/master/packages/server/README.md).

The `React.lazy` function lets you render a dynamic import as a regular component.

**Before:**

```js
import OtherComponent from './OtherComponent';

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```

**After:**

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <OtherComponent />
    </div>
  );
}
```

This will automatically load the bundle containing the `OtherComponent` when this component gets rendered.

`React.lazy` takes a function that must call a dynamic `import()`. This must return a `Promise` which resolves to a module with a `default` export containing a React component.

### Suspense

If the module containing the `OtherComponent` is not yet loaded by the time `MyComponent` renders, we must show some fallback content while we're waiting for it to load - such as a loading indicator. This is done using the `Suspense` component.

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

The `fallback` prop accepts any React elements that you want to render while waiting for the component to load. You can place the `Suspense` component anywhere above the lazy component. You can even wrap multiple lazy components with a single `Suspense` component.

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </div>
  );
}
```

### Error boundaries

If the other module fails to load (for example, due to network failure), it will trigger an error. You can handle these errors to show a nice user experience and manage recovery with [Error Boundaries](/docs/error-boundaries.html). Once you've created your Error Boundary, you can use it anywhere above your lazy components to display an error state when there's a network error.

```js
import MyErrorBoundary from './MyErrorBoundary';
const OtherComponent = React.lazy(() => import('./OtherComponent'));
const AnotherComponent = React.lazy(() => import('./AnotherComponent'));

const MyComponent = () => (
  <div>
    <MyErrorBoundary>
      <Suspense fallback={<div>Loading...</div>}>
        <section>
          <OtherComponent />
          <AnotherComponent />
        </section>
      </Suspense>
    </MyErrorBoundary>
  </div>
);
```

## Route-based code splitting

Deciding where in your app to introduce code splitting can be a bit tricky. You
want to make sure you choose places that will split bundles evenly, but won't
disrupt the user experience.

A good place to start is with routes. Most people on the web are used to
page transitions taking some amount of time to load. You also tend to be
re-rendering the entire page at once so your users are unlikely to be
interacting with other elements on the page at the same time.

Here's an example of how to setup route-based code splitting into your app using
libraries like [React Router](https://reacttraining.com/react-router/) with `React.lazy`.

```js
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import React, { Suspense, lazy } from 'react';

const Home = lazy(() => import('./routes/Home'));
const About = lazy(() => import('./routes/About'));

const App = () => (
  <Router>
    <Suspense fallback={<div>Loading...</div>}>
      <Switch>
        <Route exact path="/" component={Home}/>
        <Route path="/about" component={About}/>
      </Switch>
    </Suspense>
  </Router>
);
```

## Named Exports

`React.lazy` currently only supports default exports. If the module you want to import uses named exports, you can create an intermediate module that reexports it as the default. This ensures that treeshaking keeps working and that you don't pull in unused components.

```js
// ManyComponents.js
export const MyComponent = /* ... */;
export const MyUnusedComponent = /* ... */;
```

```js
// MyComponent.js
export { MyComponent as default } from "./ManyComponents.js";
```

```js
// MyApp.js
import React, { lazy } from 'react';
const MyComponent = lazy(() => import("./MyComponent.js"));
```
