# shell基础教程

## shell变量

```bash
your_name = "runoob"
```

* 只包含字母、数字和下划线： 变量名可以包含字母（大小写敏感）、数字和下划线 _，不能包含其他特殊字符。
* 不能以数字开头： 变量名不能以数字开头，但可以包含数字。
* 避免使用 Shell 关键字： 不要使用Shell的关键字（例如 if、then、else、fi、for、while 等）作为变量名，以免引起混淆。
* 使用大写字母表示常量： 习惯上，常量的变量名通常使用大写字母，例如 PI=3.14。
* 避免使用特殊符号： 尽量避免在变量名中使用特殊符号，因为它们可能与 Shell 的语法产生冲突。
* 避免使用空格： 变量名中不应该包含空格，因为空格通常用于分隔命令和参数。

## 使用变量

```bash
your_name="qinjx"
echo $your_name
echo ${your_name}
```
