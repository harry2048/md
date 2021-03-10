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

