---
title: 数据量很大时，JavaScript 数组操作的一些优化记录
date: 2020/1/17 12:05
comments: true
tags: Array
categories: JavaScript
---

持续更新。。。

## 创建已知数量的数组

使用赋值代替 push 操作

示例：创建 100w 长度值等于索引的数组

```js
// Bad 23.94091796875ms
var n = 1000000,
  arr = [];
for (let i = 0; i < n; i++) {
  arr.push(i);
}

// Good 7.080078125ms
var n = 1000000;
var arr = new Array(n);
for (let i = 0; i < n; i++) {
  arr[i] = i;
}
```

## 从数组中删除一部分数据

应尽量减少数组循环的次数

```js
/**
 * 迭代过滤数组
 * 当数据量很大时（超过1w）这个方法会比filter性能更好
 * @param {*} arr
 * @param {*} condition
 * @param {*} cb
 */
function reduceFilterArr(arr, condition, cb) {
  let w = 0,
    r = 0;
  for (; r < arr.length; ) {
    if (condition(arr[r], r)) {
      cb && cb(arr[r], r);
      r++;
      continue;
    }
    arr[w] = arr[r];
    w++;
    r++;
  }
  arr.length = w;
}
```

示例：从 10w 个数的数组中删除 4 的倍数的值

```js
// Bad 1170.112060546875ms
var arr = [0,1,2...]
var len = arr.length
for (let i=len; i>=0; i--) {
    if (i%4 == 0) arr.splice(i, 1)
}

// Good 5.9599609375ms
var arr = [0,1,2...]
var len = arr.length
for (let i=len; i>=0; i--) {
    if (i%4 == 0) delete arr[i]
}
arr = arr.filter(item => !!item)

// Batter 2.8291015625ms
reduceFilterArr(arr, item => item%4 == 0)
```

## 数组扁平化

切勿在循环中使用 `concat` 直接拼接数组

```js
function flatArr(arr = []) {
  let newArr = new Array(getLen(arr));
  let counter = 0;

  setArr(arr);

  return newArr;

  // 递归函数：获取长度
  function getLen(obj) {
    return Array.isArray(obj) ? obj.reduce((c, d) => c + getLen(d), 0) : 1;
  }

  // 递归函数：赋值
  function setArr(obj) {
    if (Array.isArray(obj)) {
      obj.forEach(item => setArr(item));
    } else {
      newArr[counter] = obj;
      counter++;
    }
  }
}
```

```js
// 获取所有弧上的顶点

// Bad
arr.reduce((a, b) => a.concat(b), []);

// Good
arr.flat(2);

// Batter
flatArr(arr);
```
