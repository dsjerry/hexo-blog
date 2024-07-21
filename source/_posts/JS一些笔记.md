---
title: JS一些笔记
date: 2020-07-12 10:58:32
tags: js
categories: 前端学习
toc: true
---

js会用到集合，每次遇到但可能又忘记了什么原理怎么用的一些乱七八糟的东西。

持续更新 ~<!-- more -->

<!-- toc -->

# Array 对象

## forEach()

用来调用数组的每个元素，并将元素传递给回调函数。

用在空数组中是不会执行回调函数的。

```javascript
array.forEach( function (item, index, arr), thisValue)
```

- item：当前元素，即是每次循环对应的那个值。必选
- index：当前元素的索引值。可选
- arr：当前元素所属的数组对象
- thisValue：算了，用到再说，现在不知道怎么用 

```javascript
var array = [1, 3, 6, 9, 12];
array.forEach(function (item, index, arr) {
	console.log(item);
});
// 分别输出 1, 3, 6, 9, 12
```

## some()

用于检测数组中的元素是否满足指定条件

会依次执行数组的每个元素

- 如果有一个满足条件。表达式返回 *true* ，剩余的元素不会执行
- 如果没有满足条件的元素，返回 *false*

不会执行空数组，不会改变原来的数组

```javascript
var array = [1, 3, 6, 9, 12];
array.some(function (item, index, arr) {
	if (item > 3) {
		console.log(item);
	}
});
// 分别输出 1，9，12
```



# String 对象

## split()

将一个字符串分割为字符串数组

- 如果传入空字符串作为第一个参数，那每个字符之间都会分割
- 不改变原始的字符串

```javascript
string.split(jerry, limit)
```

- jerry：可选，字符串或正则表达式
- limit：可选。指定返回数组的最大长度

```javascript
var str = "Are you OK ?";
console.log(str.split());
// 输出结果：["Are you OK ?"]
console.log(str.split(""));
// 输出结果：["A", "r", "e", " ", "y", "o", "u", " ", "O", "K", " ", "?"]
console.log(str.split(" "));
// 输出结果：["Are", "you", "OK", "?"]
```









