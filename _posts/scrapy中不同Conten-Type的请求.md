---
title: scrapy中不同Conten-Type的请求
tags: 
- 踩坑记录
categories:
- python
- 爬虫
---

比较常见的Content-Type：

- 表单：application/x-www-form-urlencoded
- json格式：application/json
- 表单/上传文件：multipart/form-data
- xml文件标记：application/xml;
- HTML文档标记：text/xml

Scrapy.Request GET请求和requests GET大体没什么区别，主要注意下需要用Scrapy.Request 做POST请求且Content-Type为application/x-www-form-urlencoded时，

需要用到Scrapy.FormRequest，

这个类是继承自Request ，在其基础上增加了对POST请求的支持，它会将原来的请求字典转为字符，直接放进请求主体里。



下面放一段FormRequest的源码：

```
class FormRequest(Request):
    valid_form_methods = ['GET', 'POST']

    def __init__(self, *args, **kwargs):
        formdata = kwargs.pop('formdata', None)
        if formdata and kwargs.get('method') is None:
            kwargs['method'] = 'POST'

        super().__init__(*args, **kwargs)

        if formdata:
            items = formdata.items() if isinstance(formdata, dict) else formdata
            querystr = _urlencode(items, self.encoding)
            if self.method == 'POST':
                self.headers.setdefault(b'Content-Type', b'application/x-www-form-urlencoded')
                self._set_body(querystr)
            else:
                self._set_url(self.url + ('&' if '?' in self.url else '?') + querystr)
```



补充：

今天做爬虫，又被payload的参数坑了一次，还是要记录下，否则下次还是容易忘。
如下图所示，在这个请求中，payload中有个参数在F12中有个‘str’=null的参数，在大家需要写的时候，如果真的在dict中也这么写，很容易会出现问题的。要不返回400，要不没有数据。
那么在这里详细的说下在scrapy中payload参数的构成方式。
首先要明确下：
1、在F12中虽然提示的是post请求，但因为不是参数为formdata，所以还是要用scrapy.Request,并不需要FormdataRequeat.
2、在构建headers的过程中，一定要添加‘Conten-Type’:‘application/json’,其他的根据自己的情况自行拼接
3、在request中，要使用method=“POST”,来明确这个请求类型为POST
4、将payload参数构建完成后，需要用json.dumps，把该参数变成json格式的数据
5、如果在F12中，看到某个参数的值为null，那么就需要大家自己测试下，这个需要的参数，到底值是null字符串，还是None.这个坑如果不在意，真的是每次都要掉里面一次，
![](https://wordpress-1308610994.cos.ap-nanjing.myqcloud.com/img/20220809182508.png)



​	