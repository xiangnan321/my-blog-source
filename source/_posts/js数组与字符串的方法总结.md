---
title: js数组与字符串的方法总结
date: 2025-08-15 18:25:58
categories:
  - "前端"       # 一级分类
  - "javascript"
tags:
---

## 数组
### concat( ) 
给数组添加元素，会返回一个新的数组，不会改变原数组。

```js
let arr=[3,5,2,7,9,13,8];
let arr1=arr.concat(2);
console.log(arr);//[3,5,2,7,9,13,8]
console.log(arr1);//[3,5,2,7,9,13,8,2];
```

### join( )
join( separator ) 将数组中所有元素都转换为字符串，然后连接在一起。
separator 在返回的字符串中用于分隔数组元素的字符或字符串，它是可选的。如果省略了这个参数，用逗号作为分隔符。

```js
let arr=[3,5,2,7,9,13,8];
console.log(arr.join());//3,5,2,7,9,13,8
console.log(arr.join("/"));//3/5/2/7/9/13/8
```

### pop( ) 
从数组尾部删除一个项目。
```js
let arr=[3,5,2,7,9,13,8];
arr.pop();
console.log(arr);//[3,5,2,7,9,13]
```

### push( ) 
把一个项目添加到数组的尾部。
```js
let arr=[3,5,2,7,9,13,8];
arr.push(2);
console.log(arr);//[3,5,2,7,9,13,2]
```

### reverse( ) 
在原数组上颠倒数组中元素的顺序。
```js
let arr=[3,5,2,7,9,13,8];
arr.reverse();
console.log(arr);//[8,13,9,7,2,5,3]
```

### shift( ) 
将数组的头部元素移出数组头部。
```js
let arr=[3,5,2,7,9,13,8];
arr.shift();
console.log(arr);//[5,2,7,9,13,8]
```

### slice( ) 
返回一个截取后新数组。
第一个参数
数组片段开始处的数组下标。如果是负数，它声明从数组尾部开始算起的位置。 也就是说，-1指最后一个元素，-2指倒数第二个元素，以此类推。

第二个参数
数组片段结束处的后一个元素的数组下标。如果没有指定这个参数 包含从第一个参数开始到数组结束的所有元素。如果这个参数是负数， 从数组尾部开始算起的元素。

```js
let arr = [1,2,3,4,5,6,7,8,9,10];
// 一个参数
let i = arr.slice(3);
console.log(i); // [ 4, 5, 6, 7, 8, 9, 10 ]
console.log(arr); // [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]
// 二个参数
let j = arr.slice(2,6);
console.log(j); // [ 3, 4, 5, 6 ]
console.log(arr); // [ 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 ]
let i = arr.slice(-3);// 等于slice(7)
console.log(i); // [ 8, 9, 10 ]
let j = arr.slice(-6,-2); // 等于slice(4,8)
console.log(j); // [ 5, 6, 7, 8 ]
let k = arr.slice(-2,-6); // 等于slice(8,4)
console.log(k); // []
```

### sort( ) 
在原数组上对数组元素进行排序。按照ASCLL码的顺序进行升序排列

```js
let arr = [0,12,3,7,-12,23];
console.log(arr.sort());
// [ -12, 0, 12, 23, 3, 7 ]
比较大小可以这么写
arr.sort((a,b) => a - b)
```

### splice()
可以向数组中添加或删除元素，返回被修改的数组，会改变原数组
```js
let str=[1,4,2,5,3];
第一个值：起始位置，第二个值：删除元素的个数，第三个值添加的元素
str.splice(2,1,"hello");//[1,4,"hello",5,3]
```

### toLocaleString( ) 
把数组转换为一个局部字符串。

```js
let arr=[3,5,2,7,9,13,8];
console.log(arr.toLocaleString());//3,5,2,7,9,13,8
```

### toString( ) 
把数组转换为字符串。
```js
let arr=[3,5,2,7,9,13,8];
console.log(arr.toString());//3,5,2,7,9,13,8
```
### unshift( ) 
在数组的头部插入一个元素。

```js
let arr=[3,5,2,7,9,13,8];
arr.unshift(2)
console.log(arr);//[2,3,5,2,7,9,13]
```
## 字符串
### 1、查找字符串所在位置(空格也算一个字符)返回一个下标

```js
let str=" qwer easdfg er trew er ";
str.indexOf("w");//返回字符第一次出现的下标
str.lastIndexOf(w);//返回字符最后一次出现的下标
```

### 2、去除空格，返回一个新的字符串

```js
let str=" qwer easdfg er trew er ";
str.trimLeft();//去除左空格
str.trimRight();//去除右空格
str.trim();//去除左右空格,不会去除中间的
```

### 3、重复字符串，返回一个新的字符串
```js
let str=" qwer easdfg er trew er ";
str.repeat(重复次数);
```
### 4、查找某个字符是否匹配,返回ture,false

```js
let str=" qwer easdfg er trew er ";
str.startsWith("z");查询开始字符是否匹配
str.endsWith("a");查询结束字符是否匹配
let str = "Hello World";
console.log(str.includes("l")); // true
console.log(str.includes("M")); // false
```
### 5、访问特定字符
charAt()接受一个数字参数，找出该下标的元素

```js
let str = "Hello World";
console.log(str.charAt(1)); // e
console.log(str.charAt('a')); // H 因为 a 被转化为0
```
charCodeAt() 接受一个数字参数，找出该下标的字符编码是什么

```js
let str = "Hello World";
console.log(str.charCodeAt(1)); // 101
console.log(str.charCodeAt('a')); // 72
```
### 6、字符串操作
concat() 拼接字符串，使用很少一般用运算符 + 

slice() 和数组的slice()方法相似，接受一个或者两个参数，返回截取的字符串
如果是负值则与字符串长度相加
```js
let str = "Hello World";
let str2 = str.slice(2);
let str3 = str.slice(2,7); // 不包括 7
console.log(str); // Hello World
console.log(str2); // llo World
console.log(str3); // llo W
str1 = str.slice(2,-3); // 等于slice(2,8)
```
substr()与slice() 方法类似
负的第一个值与字符串长度相加，负的第二个值转换为0
```js
let str = "Hello World";
let str1 = str.slice(2);
let str2 = str.substr(2);
console.log(str1); // llo World
console.log(str2); // llo World
str1 = str.slice(2,7); // 不包括结束位置 7
str2 = str.substr(2,7); // 要返回的字符个数
console.log(str1); // llo W
console.log(str2); // llo Wor
str2 = str.substr(2,-3); // 等于substr(2,0)
```
substring()提取字符串
将所有负的值转换为0，从较小的数开始，到较大的数结束
```js
let str = "Hello World";
let str1 = str.slice(2);
let str2 = str.substr(2);
let str3 = str.substring(2);
console.log(str1); // llo World
console.log(str2); // llo World
console.log(str3); // llo World
str1 = str.slice(2,7); // 不包括结束位置 7
str2 = str.substr(2,7); // 要返回的字符个数
str3 = str.substring(2,7); // 不包括结束位置 7
console.log(str1); // llo W
console.log(str2); // llo Wor
console.log(str3); // llo W
str3 = str.substring(2,-3); //等于substring(2,0) 等于 substring(0,2)
```
### 7、转换字符串大小写
有4个方法：toLowerCase()，toLocaleLowerCase() ， toUpperCase() ，toLocaleUpperCase()
一般用的最多的是toLowerCase() 和toUpperCase() 。

```js
let str = "HELLO";
console.log(str.toLowerCase()); // hello
str = "hello";
console.log(str.toUpperCase()); // HELLO
```
