---
id: javascriptUtils
title: JS常用代码工具
author: Amos Xu
author_title: Front End Engineer|前端开发
author_url: https://github.com/AmossXu
author_image_url: https://avatars.githubusercontent.com/AmossXu
tags: [javascript]
---

## 罗列常用的代码工具  如js小数的加减法精度问题  字符串转义问题

<!--truncate-->
### 加减法精度问题 
```
/**
 * 加法获取精确值
 * @param arg1 加数1
 * @param arg2 加数2
 */
const accAdd = (arg1, arg2) => {
  // tslint:disable-next-line: one-variable-per-declaration
  let r1: number, r2: number, m: number;
  try {
    r1 = arg1.toString().split('.')[1].length;
  } catch (e) {
    r1 = 0;
  }
  try {
    r2 = arg2.toString().split('.')[1].length;
  } catch (e) {
    r2 = 0;
  }
  /**
   * 原理： 最后所返回的值为乘上一个10的n次幂的数再进行除法  m就是10的n次幂
   */
  m = Math.pow(10, Math.max(r1, r2));
  return (arg1 * m + arg2 * m) / m;
}
/**
 * 减法获取准确值
 * @param arg1 被减数
 * @param arg2 减数
 */
const subtr = (arg1, arg2) => {
  // tslint:disable-next-line: one-variable-per-declaration
  let r1: number, r2: number, m: number, n: number;
  try {
    r1 = arg1.toString().split('.')[1].length;
  } catch (e) {
    r1 = 0;
  }
  try {
    r2 = arg2.toString().split('.')[1].length;
  } catch (e) {
    r2 = 0;
  }
  m = Math.pow(10, Math.max(r1, r2));
  // last modify by deeka
  // 动态控制精度长度
  n = (r1 >= r2) ? r1 : r2;
  return ((arg1 * m - arg2 * m) / m).toFixed(n);
}
/**
 * 乘法获取准确值
 * @param arg1 乘数1
 * @param arg2 乘数2
 */
const accMul = (arg1, arg2) => {
  // tslint:disable-next-line: one-variable-per-declaration
  let m: number = 0
  // tslint:disable-next-line: one-variable-per-declaration
  const s1: string = arg1.toString(),
    s2: string = arg2.toString();
  // tslint:disable-next-line: no-empty
  try { m += s1.split('.')[1].length } catch (e) { }
  // tslint:disable-next-line: no-empty
  try { m += s2.split('.')[1].length } catch (e) { }
  return Number(s1.replace('.', '')) * Number(s2.replace('.', '')) / Math.pow(10, m)
}

/**
 * 除法获取精确值
 * @param arg1 被除数
 * @param arg2 除数
 */
const accDiv = (arg1, arg2) => {
  // tslint:disable-next-line: one-variable-per-declaration
  let t1: number = 0, t2: number = 0, r1: number, r2: number;
  // tslint:disable-next-line: no-empty
  try { t1 = arg1.toString().split('.')[1].length } catch (e) { }
  // tslint:disable-next-line: no-empty
  try { t2 = arg2.toString().split('.')[1].length } catch (e) { }
  r1 = Number(arg1.toString().replace('.', ''))
  r2 = Number(arg2.toString().replace('.', ''))
  return (r1 / r2) * Math.pow(10, t2 - t1);
}

export { accAdd, subtr, accMul, accDiv }

```


### 字符串转义
```
//去掉html标签

function removeHtmlTab(tab) {
 return tab.replace(/<[^<>]+?>/g,'');//删除所有HTML标签
}
//普通字符转换成转意符

function html2Escape(sHtml) {
 return sHtml.replace(/[<>&"]/g,function(c){return {'<':'&lt;','>':'&gt;','&':'&amp;','"':'&quot;'}[c];});
}
//转意符换成普通字符

function escape2Html(str) {
 var arrEntities={'lt':'<','gt':'>','nbsp':' ','amp':'&','quot':'"'};
 return str.replace(/&(lt|gt|nbsp|amp|quot);/ig,function(all,t){return arrEntities[t];});
}
// &nbsp;转成空格

function nbsp2Space(str) {
 var arrEntities = {'nbsp' : ' '};
 return str.replace(/&(nbsp);/ig, function(all, t){return arrEntities[t]})
}
//回车转为br标签

function return2Br(str) {
 return str.replace(/\r?\n/g,"<br />");
}
//去除开头结尾换行,并将连续3次以上换行转换成2次换行

function trimBr(str) {
 str=str.replace(/((\s|&nbsp;)*\r?\n){3,}/g,"\r\n\r\n");//限制最多2次换行
 str=str.replace(/^((\s|&nbsp;)*\r?\n)+/g,'');//清除开头换行
 str=str.replace(/((\s|&nbsp;)*\r?\n)+$/g,'');//清除结尾换行
 return str;
}
// 将多个连续空格合并成一个空格

function mergeSpace(str) {
 str=str.replace(/(\s|&nbsp;)+/g,' ');
 return str;
}
```