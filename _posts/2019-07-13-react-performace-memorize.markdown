---
layout:     post
title:      "react性能优化之memorize"
subtitle:   ""
date:       2019-07-13
author:     "keyood"
header-img: "img/post-bg-react.png"
tags:
    - react
---

众所周知，react 中每当有 state 或 props 改变时，就会触发 render 方法，当 render 方法中包含一些大计算量的的函数时，尽管其入参和返回值没后变化也会运行，这大大的浪费了计算资源。

**有没有方法可以缓存这些计算值、跳过无用的计算?**

### 使用闭包

推荐库 [memoize-one](https://github.com/alexreardon/memoize-one)


memoize-one 接受两个参数，一个是需要缓存返回值的方法, 自定义比对方法。根据示例可知道，自带的对比方法只是浅对比。

```js
import memoizeOne from 'memoize-one';
import isDeepEqual from 'lodash.isequal';

const identity = x => x;
const shallowMemoized = memoizeOne(identity);
const deepMemoized = memoizeOne(identity, isDeepEqual);

const result1 = shallowMemoized({ foo: 'bar' });
const result2 = shallowMemoized({ foo: 'bar' });

result1 === result2; // false - difference reference

const result3 = deepMemoized({ foo: 'bar' });
const result4 = deepMemoized({ foo: 'bar' });

result3 === result4; // true - arguments are deep equal
```

让我们来看下源码，很简单

```js
// are-inputs-equal.ts
export default function areInputsEqual(
  newInputs: readonly unknown[],
  lastInputs: readonly unknown[],
): boolean {
  // 如果参数的长度改变，直接返回false
  if (newInputs.length !== lastInputs.length) {
    return false;
  }
  // for循环遍历每个值
  for (let i = 0; i < newInputs.length; i++) {
    // 浅对比
    if (newInputs[i] !== lastInputs[i]) {
      return false;
    }
  }
  return true;
}
```

```js
// memoize-one.ts
import areInputsEqual from './are-inputs-equal';

export type EqualityFn = (newArgs: any[], lastArgs: any[]) => boolean;

export default function memoizeOne<
  ResultFn extends (this: any, ...newArgs: any[]) => ReturnType<ResultFn>
>(resultFn: ResultFn, isEqual: EqualityFn = areInputsEqual): ResultFn {
  let lastThis: unknown;
  let lastArgs: unknown[] = [];
  let lastResult: ReturnType<ResultFn>;
  let calledOnce: boolean = false;

  function memoized(this: unknown, ...newArgs: unknown[]): ReturnType<ResultFn> {
    // 计算过 && 作用域相同 && 本次的参数和上次的参数一样，则返回lastResult
    if (calledOnce && lastThis === this && isEqual(newArgs, lastArgs)) {
      return lastResult;
    }
    // 重新计算
    lastResult = resultFn.apply(this, newArgs);
    calledOnce = true;
    lastThis = this;
    lastArgs = newArgs;
    return lastResult;
  }

  return memoized as ResultFn;
}

```
