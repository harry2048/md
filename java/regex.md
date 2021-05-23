http://caibaojian.com/zhongwen-regexp.html

## 长度

> 长度为1-3
>
> ```js
> ^.{1,3}$
> ```

## 数字+字母

> 长度限制为1-5
>
> ```js
> ^[0-9a-zA-Z]{1,5}$
> ```

## 中文

```js
/^[\u4E00-\u9FFF]{1,2}$/.test("耿威")
^[\u4e00-\u9fa5]{1,2}$

// 数字或中文
^[a-zA-Z0-9]{4,8}$|^[\u4e00-\u9fa5]{2,4}$
```



## 邮箱

以 @cebbvendor.com或 @cebbank.com结尾的

```js
^[\w]{1,10}(@cebbvendor.com|@cebbank.com)$
// 以cebbank或cebvendor结尾
/^[\w]{1,8}@cebvendor.com$|^[\w]{1,10}@cebbank.com$/.test('gengwei@cebbank.com')
```

## 密码

要求：由数字和字母组成，并且要同时含有数字和字母，且长度要在8-16位之间。 

```js
^(?![0-9]+$)(?![a-zA-Z]+$)[0-9A-Za-z]{8,16}$
```

分开来注释一下：
^ 匹配一行的开头位置
(?![0-9]+$) 预测该位置后面不全是数字
(?![a-zA-Z]+$) 预测该位置后面不全是字母
[0-9A-Za-z] {8,16} 由8-16位数字或这字母组成
$ 匹配行结尾位置

注：(?!xxxx) 是正则表达式的负向零宽断言一种形式，标识预该位置后不是xxxx字符。