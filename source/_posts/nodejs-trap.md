title: 坑003:也谈node.js开发中的坑
date: 2015-12-09 21:12:00
tags:
- node
- javascript
categories: node
---
# 缘起
node.js因为javascript的缘故,存在各种大大小小的坑,尽管各种注意,仍然免不了中招.

# 继承于javascript的坑
## 弱类型
  1. 判断对象是否不存在使用if(!obj), 此时如果obj为`''` 或 `0` 或 `false`,也会误认为不存在;
  2. 字符串连接,一般会自动转成string, 但不巧如果头两个正好可以转数字,那么它会按数字相加.如:
    ```
    var a = 1,b = 'b',c = 10;
    var s1 = a + c + b;
    var s2 = '' + a + c + b;
    console.log(s1); //"11b"
    console.log(s2); //"110b"
    ```

<!-- more -->

# node专属坑
## 异步处理
  1. error处理忘记return.
  2. 循环中嵌套异步方法,如果循环次数不多,可使用递归的方式解决.
  3. 一个异步方法中处理不当有可能会多次触发callback,又或者是一个callback都没触发,需要仔细处理逻辑流程.
## http
  1. request结束的event
  2. 注意请求处理类型,有时候get和post方式不对,则请求看上去没有处理
  3. 做proxy转发的时候注意http的head部分,请把host填对了再转
  4.
## 错误堆栈
  1. node的错误堆栈有时根本看不出问题在哪,最好是自己来包装和抛出有用的错误信息,为此我写了vlog模块.
## 其他
  1. 随时添加吧...

