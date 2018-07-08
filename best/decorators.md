# How to (not) use decorators

Using ES.next decorators in MobX is optional. This section explains how to use them, or how to avoid them.

Advantages of using decorator syntax:

* Minimizes boilerplate, declarative.
* Easy to use and read. A majority of the MobX users use them.

Disadvantages of using decorator syntax:

* Stage-2 ES.next feature
* Requires a little setup and transpilation, only supported with Babel / Typescript transpilation so far

You can approach using decorators in two ways in MobX

1.  Enable the currently experimental decorator syntax in your compiler (read on)
2.  Don't enable decorator syntax, but leverage the MobX built-in utility `decorate` to apply decorators to your classes / objects.

Using decorator syntax:

```javascript
import { observable, computed, action } from "mobx";

class Timer {
  @observable start = Date.now();
  @observable current = Date.now();

  @computed
  get elapsedTime() {
    return this.current - this.start + "milliseconds";
  }

  @action
  tick() {
    this.current = Date.now();
  }
}
```

Using the `decorate` utility:

```javascript
import { observable, computed, action, decorate } from "mobx";

class Timer {
  start = Date.now();
  current = Date.now();

  get elapsedTime() {
    return this.current - this.start + "milliseconds";
  }

  tick() {
    this.current = Date.now();
  }
}
decorate(Timer, {
  start: observable,
  current: observable,
  elapsedTime: computed,
  tick: action
});
```

Note that the `observer` function from `mobx-react` is both a decorator and a function, that means that both all these syntax variants will work:

```javascript
@observer
class Timer extends React.Component {
	/* ... */
}

const Timer = observer(class Timer extends React.Component {
	/* ... */
})

const Timer = observer((props) => (
	/* rendering */
))
```

## Enabling decorator syntax

If you want to use decorators follow the following steps.

**TypeScript**

Enable the compiler option `"experimentalDecorators": true` in your `tsconfig.json`.

**Babel: using `babel-preset-mobx`**

A more convenient way to setup Babel for usage with mobx is to use the [`mobx`](https://github.com/zwhitchcox/babel-preset-mobx) preset, that incorporates decorators and several other plugins typically used with mobx:

```
npm install --save-dev babel-preset-mobx
```

.babelrc:

```json
{
  "presets": ["mobx"]
}
```

**Babel: manually enabling decorators**

To enable support for decorators without using the mobx preset, follow the following steps.
Install support for decorators: `npm i --save-dev babel-plugin-transform-decorators-legacy`. And enable it in your `.babelrc` file:

```json
{
  "presets": ["es2015", "stage-1"],
  "plugins": ["transform-decorators-legacy"]
}
```

Note that the order of plugins is important: `transform-decorators-legacy` should be listed _first_.
Having issues with the babel setup? Check this [issue](https://github.com/mobxjs/mobx/issues/105) first.

For babel 7, see [issue 1352](https://github.com/mobxjs/mobx/issues/1352) for an example setup.

## Decorator syntax and Create React App

* decorators are not supported out of the box in `create-react-app`. To fix this, you can either eject, or use [react-app-rewired](https://github.com/timarney/react-app-rewired/tree/master/packages/react-app-rewire-mobx).

---

## Disclamer: Limitations of decorator syntax:

_The current transpiler implementations of decorator syntax are quite limited and don't behave exactly the same.
Also, many compositional patterns are currently not possible with decorators, until the stage-2 proposal has been implemented by all transpilers.
For this reason the scope of decorator syntax support in MobX is currently scoped to make sure that the supported features
behave consistently accross all environments_

The following patterns are not officially supported by the MobX community:

* Redefining decorated class members in inheritance trees
* Decorating static class members
* Combining decorators provided by MobX with other decorators
* Hot module reloading (HMR) / React-hot-loader might not work as expected

Decorated properties might not be visible yet on class instances as _own_ property until the first read / write to that property occurred.

(N.B.: not supported doesn't mean it doesn't work, it means that if it doesn't work, reported issues will not be processed until the official spec has been moved forward)
