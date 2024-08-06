---
title: 猿人学web逆向-第二题
tags: 
- 逆向
categories:
- 爬虫
---

目标链接：https://match.yuanrenxue.cn/match/2

### 0x01 初步分析

首先进入网页，打开开发者工具，碰到无限debugger,右键选择 never pause here，然后下一步断点过掉

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806143217.png)

然后打开Network，查看目标接口2，确认为返回目标数据的接口，

通过下一页对比，每次请求会携带不同的m值的cookie，可见这个m值就是这次逆向的目标。

### 0x02 逆向解析

使用fiddler对cookies中的m值进行hook，使用的是编程猫的插件，hook脚本为：

```javascript
(function() {
    'use strict';
    var cookieTemp = '';
    Object.defineProperty(document, 'cookie', {
        set: function(val) {
            if (val.indexOf('m') != -1) {
                debugger;
            }
            console.log('Hook捕获到cookie设置->', val);
            cookieTemp = val;
            return val;
        },
        get: function() {
            return cookieTemp;
        },
    });
})();

```

点击下一页，成功hook到m值生成的位置，由于cookie中还有带m的参数，如果没hook到多刷新试试，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806144425.png)

向上跟栈，调到下图所示位置

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806144716.png)

分析这条语句，并在控制台打印下，发现就是m值生成的位置，将后面m值的位置截取下来，进行手动还原

```javascript
_0xca873c[$dbsm_0x1720('\x30\x78\x32\x65\x61', '\x5e\x77\x4b\x67') + '\x4f\x41'](_0xca873c[$dbsm_0x1720('\x30\x78\x33\x37\x66', '\x66\x61\x34\x7a') + '\x4f\x41'](_0xca873c['\x4a\x59\x5a' + '\x4f\x41'](_0xca873c['\x4a\x59\x5a' + '\x4f\x41'](_0xca873c['\x49\x75\x76' + '\x6e\x68'](_0xca873c[$dbsm_0x1720('\x30\x78\x63\x32', '\x56\x59\x6a\x2a') + '\x78\x51']('\x6d', _0xca873c[$dbsm_0x1720('\x30\x78\x33\x65\x36', '\x76\x68\x29\x38') + '\x6d\x55'](_0x30e201)), '\x3d'), _0xca873c[$dbsm_0x1720('\x30\x78\x32\x33\x32', '\x36\x62\x33\x50') + '\x48\x4c'](_0x1eff21, _0x35104f)), '\x7c'), _0x35104f), _0xca873c[$dbsm_0x1720('\x30\x78\x32\x33\x31', '\x33\x77\x42\x52') + '\x41\x57'])

```

在notepad++或者vscode中，先将16进制还原成字符串，调整格式，

```javascript
_0xca873c['JYZOA'](
_0xca873c['JYZOA'](
_0xca873c['JYZOA'](
_0xca873c['JYZOA'](
_0xca873c['Iuvnh'](
_0xca873c['FRVxQ'](
'm', _0xca873c['tEYmU'](
_0x30e201)), '='), _0xca873c['eFLHL'](
_0x1eff21, _0x35104f)), '|'), _0x35104f), _0xca873c['qimAW'])
```

再通过打印台，对函数进行打印和手动还原，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240723155714.png)

最后这句m值生成的语句，最终还原成下列：

```
'm='+ _0x1eff21(_0x35104f)+ '|'+ _0x35104f+ '; path=/'
```

可以看出m值的组成为两部分，后面部分是时间戳，前面部分是传入时间戳后的函数返回字符串，

可以写成下列代码在node环境中运行，

```
var _0x35104f = Date.parse(new Date());
var m = _0x1eff21(_0x35104f) + '|' + _0x35104f
console.log(m);
```

后面就开始补环境，为保证不遗漏和方便，将原来hook住的js全部复制到本地notepad记事本，

这样每次缺什么，从本地文件ctrl+x就行，

需要注意的是，每次记得需要补的函数行数，这样能避免补不全，然后补到开头的数组后，记得要将后面的自执行位移函数也补上，

当补完开头大数组后，就发现在node中运行不动了，出现内存不足或者length超出的情况，

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806152841.png)

打开一个新的无痕浏览器窗口，新建一个Snippets代码，将补的当前代码全部复制到浏览器中调试，这样的好处是能看到报错的位置和信息，

需要注意的点 :

##### 1.有两处格式化校验的地方，一直是无限for循环，

第一处

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806152944.png)

第二处

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806153157.png)

断点到第二处方框代码位置，单步调试发现一直在当前这行来回执行，

```
for (var _0x3a1ec6 = 0x0, _0x1ea78f = this['XMGFMr']['length']; _0x3a1ec6 < _0x1ea78f; _0x3a1ec6++) {
                    this['XMGFMr']['push'](Math['round'](Math['random']()));
                    _0x1ea78f = this['XMGFMr']['length'];
                }
```

向上跟踪堆栈，

```
_0x2441a6['prototype']['ymQNnY'] = function(_0x171f9c) {
                if (!Boolean(~_0x171f9c)) {
                    return _0x171f9c;
                }
                return this['JWYcDT'](this['Vxshgd']);
            }
```

这里进行了一个 if 判断，~ 为按位取反，意思是如果 !Boolean(~_0x4859ef) 的值为 false，则执行 ymQNnY的无限循环行为，直至程序崩溃，接着跟进到 bvlPOU部分，查看 _0x4859ef 是啥，对什么进行了判断

```
_0x2441a6['prototype']['bvlPOU'] = function() {
                var _0x4fcb77 = new RegExp(this['resfBu'] + this['sUoDsl']);
                var _0x1bcff2 = _0x4fcb77['test'](this['qFYmCX']['toString']()) ? --this['XMGFMr'][0x1] : --this['XMGFMr'][0x0];
                return this['ymQNnY'](_0x1bcff2);
            }
```

这是一个三目表达式，

```
_0x4fcb77['test'](this['qFYmCX']['toString']()) ? --this['XMGFMr'][0x1] : --this['XMGFMr'][0x0];


--this['XMGFMr'][0x1]的值固定为 -1，而每运行一次,--this['XMGFMr'][0x0]的值即减一,
所以只有当 _0x4fcb77['test'](this['qFYmCX']['toString']()) 的值为 true 时才不会进入无限循环， 的值为 true 时才不会进入无限循环，
```

![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/undefined20240806154510.png)

```
_0x4fcb77 = /\w+ *\(\) *{\w+ *['|"].+['|"];? *}/
```

所以匹配样式大致如下：XXX( ){ XXX ' XXX ' ;}才能够通过正则化校验。

##### 2.有两个自执行函数，记得去掉后面的()，在扣代码的时候注意，分别是_0xf7afd3和_0xf7afd3

##### 3.navigator 未定义

```
var window = global; 
window.navigator = {};
```

##### 4.history 未被定义 console.log 被重写

```
result = console.log;
```

加到前面，后续使用result,

或者用console.warn,console.debug

##### 5.注意在过格式化，要修改本地的文件，在浏览器中修改格式会被自动恢复



### 0x03 最终代码

https://github.com/bqzzd1/m2w/code/yrx2

// yrx2-0 原始的hook浏览器js，便于后续练习

// yrx2.js  扣的js，生成m值，用于python调用

// yrx2.py  python代码，编写调用请求


