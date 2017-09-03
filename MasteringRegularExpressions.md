# Regular Expression
正则表达式常用于处理文本信息，找到文本信息特定的内容，或替换，或split。如果是知道我们需要找出来的文本是精确匹配的，那肯定用不上正则表达式，在编程中直接调用contains或equals方法就能判断。但现实中往往有这样的需求：例如找出爬出来的网页中含pdf的下载链接；或者交换每行文件中第一个和第二个单词的顺序；或者判断用户输入的邮箱地址是否正确等这类问题就不是简单的equals或contains方法或函数所能完成的，我们需要准确的告诉一个文本模式，而不是一段精确的语句。正则表达式就能完美的解决这一类问题，当然你也许会反驳我说，不用正则表达式也能解决这类问题，当然没错，但是正则表达式有其通用的规则，很很多语言里面都有完整的支持，例如java，.net，python，JavaScript，perl，php等，在某类问题上，使用正则来解决或许更方便更简单。
举个例子，比如说我有一个文本里面有大量重叠的字母，比如：aaaaaaaabbbbbbbcccd，现在我需要对这个文件通过某种方式压缩，采取的办法是将字母出现的次数修饰前一个字母，所以压缩后的文本变为a8b7c3d，假设输入文本中没有数字。那么如果通过编程的方式解决这个问题呢，在这里为了方便，使用Python，或许你会写这样的代码：
```Python
def lineEncoding(s):
    i = 0
    result = ''
    prev = ''
    pc = 0
    while i < len(s):
        if s[i] != prev:
            result = appendStr(result, pc, prev)
            prev = s[i]
            pc = 1
        else:
            pc += 1
            i += 1
            result = appendStr(result, pc, prev)
	return result

def appendStr(str, count, ch):
    if ch == '':
    	return str
    elif count == 1:
    	return '%s%s' % (str, ch)
    else:
    	return '%s%d%s' % (str, count, ch)
```
但是通过使用正则表达式，就简单了：
```Python
def lineEncoding(s):
	return re.sub(r"(.)\1+", lambda m: str(len(m.group(0))) + m.group(1), s)

```
# Metacharacters
元字符是指在正则表达式中的特殊意义的字符，如我们编码中常用到的'\n'表示换行符之类的。
## Basic
这里列一些常见：

1. `\d`	表示0-9的数字
2. `\D`	表示非数字
3. `\w`	表示a-z或A-Z或0-9
4. `\W`	表示非\w
5. `\t` `\r` `\n` 这三个相信大家都见过，分别表示tab符，Carriage-return 和换行符
6. `\s`	表示空格或\t或\r或\n，总之就是表示空格相关
7. `.`	匹配任意字符，默认不包括换行符

## Anchor
还有一些并不是匹配字符用的，而是匹配位置，列举如下：

1. `^`	匹配文档的开始，当没有指定\m的修饰符的时候（见[Regex Flag](##Regex Flag)）,如果制定了\m的修饰符，那么^表示行的开始这一位置
2. `$`	匹配文档末尾，或行的末尾，取决于\m的指定
3. `\b`	匹配单词的首或尾的位置
4. `\B`	匹配非单词的首或尾的位置
5. `\G`	匹配前一次匹配的位置（支持这个的语言比较少）
6. `\A`	匹配文档的开始
7. `\Z`	匹配文档结束

# Regex Flag
一般在编译正则表达式的时候需要指定一些flag来描述编译后的pattern的结果，常见的flag列举如下：

1. Case-Insensitive，例如Python中的`RE.I`表示大小写敏感，也就说可以使用`abc`匹配`abc`，`Abc`， `ABc`， `ABC`等
2. MultipleLine，多行模式，这个flag会影响`^`和`$`匹配的位置
3. DOTALL，`.`匹配所有字符包括换行符

# Character Class
Character Class表示某一类字符集合，通过使用`[...]`来表示字符集合。常用的用法如下：

1. `[1-5A-E]`	匹配1-5或A-E中的一个，在这里hyphen`-`在`[...]`具有特殊的意义，表示从哪里到哪里的闭区间，要想使hyphen不具有特殊意义可以使用`\-`表示hyphen符号
2. `[-245ABC]`	匹配字符集合`-245ABC`中的一个，在这里dash`-`在`[...]`没有特殊意义，就是一般的dash字符，当dash出现在`[...]`的开头，那么不需要使用转义，当然使用转义也不会错
3. `[^12345]`	匹配非集合，非`12345`中的一个，注意，这里的caret`^`在`[...]`表示非的意思，具有特殊意义，同样当caret在外面使用的时候表示行或文档的开始位置。同样这里可以使用`\^`转义表示caret符号

# Quantifiers
还有一下用来匹配前一匹配的个数相关，列举如下：

1. `?`	匹配0个或一个，例如: `ab?`匹配`ab`或者`a`
2. `+`	匹配1个或多个，例如： `ab+`匹配`ab`或者`abbb`
3. `*`	匹配0个或多个，例如： `ab*`匹配`a`或者`abbbb`
4. `{n}`	匹配n次，例如： `ab{2}`匹配`abb`
4. `{n,m}`	匹配n到m个，其中n<=m,例如`ab{2,3}`匹配`abb`或`abbb`
5. `{n,}`	匹配n次或n次以上

# Grouping

## Subexpression
使用`(...)`可以对匹配的字符进行分组，例如`(ab){2}`匹配`abab`,在编程的时候,对于parentheses`()`里面匹配出来的内容，会缓存起来，例如Python中可以通过group方法调用得到group中的值，
```Python
m = re.match(r"(\w+) (\w+)", "Isaac Newton, physicist")
print m.group(1)	# 'Isaac'
print m.group(2)	# 'Newton'
```
在这里可以看到传入group的index值到Group方法就能得到对应parentheses中的值，例子中的第一个parentheses就是group1，第二个为group2，这种情况比较容易得到group对应的index值，就是根据group对应的顺序。假设表达式中有嵌套的parentheses，怎么认定group index的值呢？举个例子，对于正则表达式：
`1(23(45)67(89))`
group index的值就是根据左括号出现的顺序，所以在这个例子中，group1匹配的是最外面的group，group2匹配45，group3匹配89，Python代码如下：
```python
import re
m = re.match(r"1(23(45)67(89))", "123456789")
print m.group(1)  # '23456789'
print m.group(2)  # '45'
print m.group(3)  # '89'
```
当然有的语言还支持使用name来定义和索引group。
## Noncapturing Group
因为将匹配出来的subexpression缓存起来方便接下来的步骤调用，这一操作是有损耗性能的，所以当不需要把parentheses中的内容capture到变量的时候，我们可以通过使用表达式`(?：...)`，例如：
```Python
import re
m = re.match(r"1(?:23(45)67(89))", "123456789")
print m.group(1)  # '45'
print m.group(2)  # '89'
```

## Backreferencing
在正则表达式中，也可以通过`\groupindex`来引用Group中的值，例如匹配连续两个重复的单词：
`(\b\w+\b) \1`就能匹配`abc abc`或`def def`
# Lookahead and Lookbehind
Lookahead和Lookbehind也是匹配位置，不消费字符，列举如下：

1. `(?=...)`	Lookahead			例如： `abc(?=123)`表示匹配`abc`而且要求`abc`之后的位置紧跟着`123`，例如匹配`abc111 abc123`中的第二个`abc`
2. `(?<=...)`	Lookafter			例如： `(?<=123)abc`表示匹配`abc`，而且是在含有`123`这个匹配的位置之后，例如匹配`123abc 111abc`中的第一个`abc`
3. `(?!...)`	Negative Lookahead	例如： `abc(?!123)`表示匹配`abc`而且要求`abc`之后的位置没有紧跟着`123`，例如匹配`abc111 abc123`中的第一个`abc`
4. `(?<!...)`	Negative Lookafter	例如： `(?<!123)abc`表示匹配`abc`，而且匹配`abc`之前的位置是不能匹配`123`，例如匹配`123abc 111abc`中的第二个`abc`

# Greeding and Laziness
# Performance
## NFA Engine
## Backtracking
## Atomatic Group and Possessive