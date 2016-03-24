### AWK教程
#### 1. 什么是awk
awk 以逐行方式扫描文件(或输入)，从第一行到最后一行，以查找匹配某个特定模式的文本行，并对这些文本行执行(括在花括号中的)指定动作。如果只给出模式而未指定动作，则所有匹配该模式的行都显示在屏幕上；如果只指定动作而未定义模式，会对所有输入行执行指定动作。
#### 2. awk的格式
awk 指令由模式、操作、或模式与操作的组合组成。模式是由某种类型的表达式组成的语句。如果某个表达式中没有出现关键字if，但实际计算时却暗含if这个词，那么，这个表达式就是模式。操作由括在大括号内的一条或多条语句组成，语句之间用分号或换行符隔开，模式则不能被括在大括号中，模式由括在两个正斜杠之间的正则表达式、一个或多个awk 操作符组成的表达式组成。

以下三种模式，仅操作、仅匹配模式、匹配模式和操作：
```
$awk '{action}' fi1ename 
$awk 'pattern' filename
$awk 'pattern {action}' fi1ename
```
范例：
```
$awk '/Mary/' employees
Mary Adams 5346 11/4/63 28765
```
```
$awk '{print $1}' employees
Tom
Mary
Sally
Billy
```
```
$awk '/Sally/{print $1,$2}' employees
Sally Chang
```
可以将一条或多条Linux命令的输出通过管道发给awk处理。格式如下：
```
$command | awk 'pattern'
$command | awk '{action}'
$command | awk 'pattern {action}'
```
范例：
```
$cat employees | awk '/Sally/{print $1}'
Sally Chang
```
#### 3. awk工作原理
1.awk使用一行作为输入(通过文件或者管道)，并将这一行赋给内部变量$0 ，默认时每一行也可以称为一个记录，以换行符结束。
![](http://www.linuxawk.com/wp-content/uploads/2015/03/001.png)
2.然后，行被空格分解成字段(单词)，每一个字段存储在已经编号的变量中，从$1 开始，可以多达100 个字段。
![](http://www.linuxawk.com/wp-content/uploads/2015/03/002.png)
3.awk如何知道空格是用来分隔字段的呢？因为有另一个内部变量FS用来确定字段的分隔符。初始时，**FS**被赋为空格（包含制表符和空格符）。如果需要使用其他的字符分隔字段，如冒号或破折号，则需要将FS 变量的值设为新的字段分隔符。

4.awk打印字段时，将以下面方式使用print函数。

```
{print $1,$3}
Tom 100
Molly 200
John 300
```
awk在Tom和100之间加入了空格，因为在\$1和\$3之间存在一个逗号。逗号比较特殊，它映射为另一个内部变量，称为输出字段分隔符(OFS), **OFS**默认为空格。逗号被OFS变量中存储的字符替换。

5.awk输出之后，将从文件中获取另一行，并将其存储到$0中，覆盖原来的内容，然后将新的字符串分隔成字段并进行处理。这个过程将持续到整个文件的所有行都处理完毕。
#### 4. print函数
awk命令的操作部分被括在大括号内。如果未指定操作，则匹配到模式时， awk 会采取默认操作，即在屏幕上打印包含模式的行。print函数用于简单的输出。更为复杂的输出则要使用printf和sprintf 函数。如果熟悉C语言，那么一定懂得如何使用printf和sprintf。

也可以用{print}形式在awk命令的动作部分显式地调用print函数。print函数的参数可以是变量、数值或字符串常量。字符串必须用双引号括起来。参数之间用逗号分隔，如果没有逗号，所有的参数就会被串在一起。逗号等价于OFS中的值，默认情况下是空格。

print函数的输出可以被重定向，也可以通过管道传给另一个程序。其他程序的输出也可以通过管道交给awk打印。
范例：

```
$ date
Thu Mar 12 12:23:23 CST 2015
$ date | awk '{print "Month: "$2 "\nYear: "$6}'
Month: Mar
Year: 2015
```
说明：Linux中， date命令的输出经管道发送给awk。打印显示为字符串Month:，后面跟date输出结果中的第2个字段，然后是另一个字符串，该串中包含换行符\n和Year:，最后是date输出结果中的第6个字段($6)。

转义序列：转义序列用一个反斜杠后跟一个字母或数字来表示。它们可以用在字符申中，代表制表符、换行符、换页符等(参见下表) 。
|转义序列|含义|
| :-------- | --------:| :--: |
|\b	|退格|
|\f	|换页|
|\n	|换行|
|\r	|回车|
|\t	|制表符|
|\047|八进制值47. 即单引号
|\c	|C 代表任一其他字符，例如"\"
范例：
```
$ cat employees
Tome  Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500
$ awk '/Sally/{print "\t\tHave a nice day," $1,$2 "\!"}' employees
        Have a nice day,Sa11y Chang!
```
说明：如果某一行包含模式Sally，print函数就会打印出两个制表符，字符串"Have a nice day"，该行的第一个字段Sally和第二个字段Chang，后面跟一个带感叹号的字符串。
#### 5. OFMT变量
打印数字时，可能需要控制数字的格式。这可以通过printf函数来实现，但是，通过设置一个特殊的awk变量OFMT，使用print函数时也可以控制数字的打印格式。OFMT的默认值是"%.6gd"，表示只打印小数部分的前6 位(之后我们会介绍如何修改OFMT的值)。
```
$ awk 'BEGIN{OFMT="%.2f";print 1.2567,12E-2}'
1.25 0.12
```
说明：如果设置了变量OFMT，在打印浮点数时，就只打印小数部分的前两位。百分号(%)表示接下来要定义格式。

#### 6. printf函数
打印输出时，可能需要指定字段间的空格数，从而把列排整齐。在print函数中使用制表符并不能保证得到想要的输出，因此，可以用printf函数来格式化特别的输出。

printf函数返回一个带格式的字符串给标准输出，如同C语言中的printf语句一样。printf语句包括一个加引号的控制串，控制串中可能嵌有若干格式说明和修饰符。控制串后面跟一个逗号，之后是一列由逗号分隔的表达式。printf函数根据控制串中的说明编排这些表达式的格式。与print函数不同的是， printf不会在行尾自动换行。因此，如果要换行，就必须在控制串中提供转义字符\n。

每一个百分号和格式说明都必须有一个对应的变量。要打印百分号就必须在控制串中给出两个百分号。请参考print转义字符和printf修饰符。格式说明由百分号引出，另外还列出了printf所用的格式说明符。

**printf使用的转义字符**
|**转义字符**	|**定义**
| :-------- | --------:| :--: |
|c|	字符
|s|	字符串
|d	|十进制整数
|ld|	十进制长整数
|u|	十进制无符号整数
|lu|	十进制无符号长整数
|x|	十六进制整数
|lx|	十六进制长整数
|o|	八进制整数
|lo|	八进制长整数
|e	|用科学记数法(e 记数法)表示的浮点数
|f|	浮点数
|g|	选用e或f中较短的一种形式



























**printf的修饰符**
|字符|	定义|
| :-------- | --------:| :--: |
|-	|左对齐修饰符
|#	|显示8 进制整数时在前面加个0,显示16 进制整数时在前面加0x
|+	|显示使用d 、e 、f 和g 转换的整数时，加上正负号+或-
|0	|用0而不是空白符来填充所显示的值
**printf的格式说明符**
|格式说明符|	功能|示例|输出|
| :-------- | --------:| :--: |
|%c	|打印单个ASCII字符 |printf("The character is %c\n",x)|The character is A
|%d	|打印一个十进制数 |printf("The boy is %d years old\n",y)|The boy is 15 years old
|%e	|打印数字的e 记数法形式 |printf("z is %e\n",z) |z is 2.3e+0 1
|%f	|打印一个浮点数 |printf("z is %f\n", 2.3 * 2)|z is 4.600000
|%o	|打印数字的八进制 |printf("y is %o\n",y) |z is 17
|%s	|打印一个字符串 |print("The name of the culprit is %s\n",$1) |The name of the culprit is Bob Smith
|%x	|打印数字的十六进制值 |printf("y is %x\n",y) |x is f

打印变量时，输出所在的位置称为"域"(field)，域的宽度(width)是指该域中所包含的字符个数。下面这些例子中， printf控制串里的管道符(竖杠)是文本的一部分， 用于指示格式的起始与结束。
**范例**
```
$ echo "Linux" | awk '{printf "|%-15s|\n",$1}'
|Linux          |
```
说明：对于echo命令的输出，Linux是经管道发给awk。printf函数包含一个控制串。百分号让printf做好准备，它要打印一个占15个格、向左对齐的字符串，这个字符串夹在两个竖杠之间，并且以换行符结尾。百分号后的短划线表示左对齐。控制串后面跟了一个逗号和$1。printf将根据控制串中的格式说明来格式化字符串Linux。
**范例**
```
$ echo "Linux" | awk '{printf "|%15s|\n",$1}'
|          Linux|
```
说明：字符串Linux被打印成一个占15 格、向右对齐的字符串，夹在两个竖杠之间，以
换行符结尾。
**范例**

```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ awk '{printf "The name is: %-15s ID is %8d\n",$1,$3}' employees
The name is Tom             ID is 4424
The name is Mary            ID is 5346
The name is Sally           ID is 1654
The name is Billy           ID is 1683
```
说明：要打印的字符串放置在两个双引号之间。第一个格式说明符是%-15s，它对应的参数是$1，紧挨着控制串的右半边引号后面的那个逗号。百分号引出格式说明：短划线表示左对齐，15s表示占15格的字符串。这条命令用来打印一个左对齐、占15格的字符串，后面跟着字符串的ID和一个整数。

格式：%8d表示在字符串的这个位置打印$2 的十进制(整数)值。这个整数占8格，向右对齐。您也可以选择将加引号的字符串和表达式放在圆括号里。


#### 7. 文件中的awk命令
如果awk命令被写在文件里，就要用-f选项指定awk的文件名，后面再加上所要处理的输入文件的文件名。awk从缓冲区读入一条记录，接着测试awk文件中的每一条命令，然后对读入的记录执行命令。处理完第一条记录后，awk将其丢弃，接着将下一条记录读入缓冲区，依次处理所有记录。如果没有模式限制，默认的操作就是打印全部记录。而模式如果没有相应的操作，则默认行为是打印匹配它的记录。
**范例**
```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ cat awkfile
/^Mary/{print "Hello Mary!"}
{print $1, $2, $3}

$ awk -f awkfile employees
Tom Jones 4424
Hello Mary!
Mäty Adams 5346
Sally Chang 1654
Billy Black 1683
```
说明：
1.如果记录以正则表达式Mary开头，则打印字符串"Hello Mary!"。操作取决于它前面的模式是否匹配。且打印的字段之间以空白符分隔。
2.打印每条记录的第1 、第2、第3字段。awk对每行都执行该操作，因为没有限制该操作的模式。

#### 8. awk记录
在awk命令看来输入数据具有格式和结构， 而不是无休止的字符串。默认情况下， 每一行称为一条记录，以换行符结束。

记录分隔符默认情况下，输入和输出记录的分隔符(行分隔符)都是回车符(换行符)，分别保存在awk的内置变量**ORS**和**RS**中。ORS和RS的值可以修改，但是只能以特定方式进行修改。

变量\$0：awk用\$0指代整条记录(当$0因替换或赋值而改变时，NF的值，即字段的数目值也可能改变)。换行符的值保存在awk的内置变量RS中，其默认值为回车。
#### 9. awk字段
每条awk记录都是由字段(field)组成，默认情况下，字段间用空白符(即空格或制表符)分隔。每个这样的词称为一个字段，awk在内置变量NF中保存记录的字段数。NF的值因行而异，其上限与具体版本的实现相关，通常是每行最多100个字段。可以创建新的字段。下面这个例子中有4条记录(行)和5个字段(列)。每条记录都是从第一个字段(用\$1表示)开始，然后是第二个字段($2) ，以此类推。

#### 10. 字段分隔符
输入字段分隔符：awk的内置变量FS中保存了输入字段分隔符的值。使用FS的默认值时，awk用空格或制表符来分隔字段，并且删除各字段前多余的空格或制表符。可以通过在BEGIN语句中或命令行上赋值来改变FS的值。接下来我们就要在命令行上给FS指定一个新的值。在命令行上改变FS的值需要使用-F选项，后面指定代表新分隔符的字符。

从命令行改变字段分隔符：范例中演示了如何使用-F选项在命令行中改变输入字段分隔符。

```
$ cat employees
Tom Jones:4424:5/12/66:543354
Mary Adams:5346:11/4/63:28765
Sally Chang:1654:7/22/54:650000
Billy Black:1683:9/23/44:336500

$ awk -F: '/Tom Jones/{print $1,$2}' employees
Tom Jones 4424
```

说明：-F选项用来在命令行重新设置输入字段分隔符的值。当冒号紧跟在-F选项的后面时，awk 就会在文件中查找冒号，用以分隔字段。

**使用多个字段分隔符**：你可以指定多个输入字段分隔符。如果有多个字符被用于字段分隔符FS，则FS对应是一个正则表达式字符串，并且被括在方括号中。下面的范例中，字段分隔符是空格、冒号或制表符。

```
$ awk -F'[ :\t]' '{print $1,$2,$3}' employees
Tom Jones 4424
Mary Adams 5346
Sally Chang 1654
Billy Black 1683
```
说明：-F选项后面跟了一个位于方括号中的正则表达式，当遇到空格、冒号或制表符时，awk会把它当成字段分隔符。这个表达式两头加了引号，这样就不会被shell当成自己的元字符来解释(注意， shell使用方括号来进行文件名扩展)。

**输出字段分隔符**：默认的输出字段分隔符是单个空格，被保存于awk的内置变量OFS中。此前的所有例子中，我们都是用print语句把输出打印到屏幕上。因此，无论OFS如何设置，print语句中用于分隔字段的逗号，在输出时都被转换成OFS的值。如果用OFS的默认值，则\$1和$2之间的逗号会被转换为单个空格，print函数打印这两个字段时会在它们之间加一个空格。

如果没有用逗号来分隔字段，则输出结果中的字段将堆在一起。另外，OFS的值可以改变。

范例
```
$ cat employees
Tom Jones:4424:5/12/66:543354
Mary Adams:5346:11/4/63:28765
Sally Chang:1654:7/22/54:650000
Billy Black:1683:9/23/44:336500

$ awk -F: '/Tom Jones/{print $1,$2,$3,$4}' employees
Tom Jones 4424 5/12/66 543354
```

说明：输出字段分隔符，即空格，保存在awk的变量OFS中。字段间的逗号在输出时被转换成OFS中的值。字段被打印到标准输出上，字段之间用空格分隔。

范例

```
$ awk -F: '/Tom Jones/{print $0}' employees
Tom Jones:4424:5/12/66:543354
```

说明：变量$0按输入文件中的原样保存当前记录。记录被原封不动地显示到屏幕上。

```
$ awk -F: '/Tom Jones/{OFS="|";print $1,$2,$3,$4}' employees
Tom Jones|4424|5/12/66|543354
```
说明：OFS的值可以修改，输出时，“,”替换为修改后的OFS的值。

#### 11. awk模
awk模式用来控制awk命令对输入的文本行执行什么操作。**模式由正则表达式、判别条件真伪的表达式或二者的组合构成**。awk的默认操作是打印所有使表达式结果为真的文本行。模式表达式中暗含着if语句。如果模式表达式含有if(如果)的意思，就不必用花括号把它括起来。当if是显式地给出时，这个表达式就成了操作语句，语法将不一样(参见条件语句 )。

范例
```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ awk '/Tom/' employees
Tom   Jones 4424 5/12/66 543354
```
说明：如果在输入文件中匹配到模式Tom，则打印Tom所在的记录。如果没有显式地指定操作，默认操作是打印文本行，等价于命令：
```
$ awk '$0~/Tom/{print $0}' employees
```
范例
```
$ awk '$3 < 4000' employees
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500
```
说明：如果第3个字段的值小于4000，则打印该记录。
#### 12. awk操作
awk操作(action)是花括号中以分号分隔的语句。如果操作前面有个模式，则该模式控制执行操作的时间。操作可以是简单的语句或复杂的语句。**同一行内的多条语句由分号分隔，独占一行的语句则以换行符分隔**。
格式：

```
{action}
```

范例：

```
{print $1,$2}
```

说明：该操作用来打印前两个字段。
模式可以与操作结合使用。记住，操作是括在花括号中的语句，模式控制它后面第一个左花括号到第一个右花括号之间的操作。
格式：

```
模式 { 操作语句;操作语句; ...;}
```

或

```
模式{
  操作语句
  操作语句
}
```

范例

```
$ awk '/Tom/{print "Hello there, "$1}' employees
Hello there, Tom
```
说明：如果记录中包含模式Tom，就会打印出字符串"Hel1o there, Tom"。
如果没有为模式指定操作，就会打印所有匹配该模式的文本行。用于匹配字符串的模式包括了夹在两个正斜杠之间的正则表达式。

#### 13. awk正则表达式
对awk命令而言，正则表达式是置于两个正斜杠之间、由字符组成的模式。
awk支持使用(与egrep相同的)正则表达式元字符对正则表达式进行某种方式的修改。如果输入行中的某个字符串与正则表达式相匹配，则最终条件为真，于是执行与该表达式关联的所有操作。如果没有指定操作，则打印与正则表达式匹配的记录。

范例
```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ awk '/Mary/' employees
Mary  Adams 5346 11/4/63 28765
```

说明：显示文件employees中所有包含正则表达式Mary的行。

范例

```
$ awk '/Mary/{print $1,$2}' employees
Mary Adams
```
说明：显示文件employees中所有包含正则表达式Mary的行的头两个字段。

**awk正则表达式元字符**
|元字符	|说明
| :-------- | --------:| :--: |
|^	|匹配串首
|$|	匹配串尾
|.|	匹配单个任意字符
|*	|匹配零个或多个前导字符
|+	|匹配一个或多个前导字符
|?|	匹配零个或一个前导字符
|[ABC]	|匹配指定字符组(即A、B和C)中任一字符
|[^ABC]|	匹配任何一个不在指定字符组(即A、B和C)中的字符
|[A-Z]|	匹配A至Z之间的任一字符
|A|B|	匹配A或B
|(AB)+|	匹配一个或多个AB 的组合，例如: AB 、ABAB 、ABABAB
|\*	|匹配星号本身
|&	|用在替代串中，代表查找串中匹配到的内容

以下是grep与sed支持，而awk不支持的元字符。
**awk不支持的元字符**
|元字符	|说明
| :-------- | --------:| :--: |
|\\< \\>|	单词定位
|\\( \\)|	向前引用
|\\{ \\}	|重复

#### 14. 匹配整行
如果没有指定操作，则单个正则表达式将对整行进行模式匹配，并打印出所匹配的行。可以使用元字符^来表示需要进行行首匹配的正则表达式。

范例
```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ awk '/^Mary/' employees
Mary  Adams 5346 11/4/63 28765
```
说明：显示文件employees中所有以正则表达式Mary开头的行。

范例

```
$ awk '/^[A-Z][a-z]+ /' employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500
```
说明：显示文件employees中所有以大写字母开头、后跟一个或多个小写字母、再跟一个空格的行。
#### 15. awk匹配操作符
匹配操作符(~)用于对记录或字段的表达式进行匹配。

范例

```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ awk '$1 ~ /[Bb]ill/' employees
Billy Black 1683 9/23/44 336500
```
说明：显示所有在第一个字段里匹配到Bill或bill的行。

范例

```
$ awk '$1 !~ /ly$/' employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
```
说明：显示所有第一个字段不是以ly结尾的行。

POSIX字符类 POSIX(the Portable Operating System Interface，可移植操作系统接口)是一种工业标准，确保程序可以跨操作系统移植。为了保证可移植， POSIX可以识别字符、阿拉伯数字和符号在不同国家或不同场合的编码方法，以及时间和时期的不同表示。为了处理不同类型的字符，POSIX增加了基本的和扩展的正则表达式，下表对括号字符类进行了说明。对于UNIX，gawk支持这些新的元字符类，而awk不支持；而对于Linux，则应当明白awk是链接到gawk上的，也就是说awk和gawk的命令同样有效。

**POSIX增加的括号字符类**
|括号类|	含义
| :-------- | --------:| :--: |
|[:alnum:]	|字母数字字符
|[:alpha:]	|字母字符
|[:cntrl:]	|控制字符
|[:digit:]	|数字字符
|[:graph:]	|非空白字符(非空格、控制字符等)
|[:lower:]	|小写字母
|[:print:]|	与[:graph:]相似，但是包含空格字符
|[:punct:]	|标点字符
|[:space:]|	所有的空白字符(换行符、空格、制表符)
|[:upper:]	|大写字母
|[:xdigit:]	|允许十六进制的数字(0-9a-fA-F)

在类中， [:alnum:]是另一种表示A-Z、a-z和0-9的形式，使用这种类时，必须要用另外一个方括号扩起来，例如"A-Za-z0-9" 自己本身不是正则表达式，而[A-Za-z0-9]则是正则表达式。类似地，[:alnum:] 应该写为[[:alnum:]] 。第一种形式[A-Za-z0-9]与方括号形式[[:alnum:]] 的不同之处在于，前者依赖于ASCII字符编码的形式，而第二种形式允许其他语言的字符在类中表示，例如瑞典语和德语。

```
$ gawk '/[[:lower:]]+g[[:space:]]+[[:digit:]]/' employees
Sally Chang 1654 7/22/54 650000
```
说明：gawk搜索一个或多个小写字母，后面跟着一个字母"g"，再后面为一个或多个空格，然后是一个数字的模式。
#### 16. awk关系运算符
下表列出了所有关系运算符。关系表达式的计算结果为真时，表达式的值为1；反之，则为0。
**关系运算符**
|运算符	|含义	|示例
| :-------- | --------:| :--: |
|<	|小于	|x < y
|<=	|小于等于	|x <= y
|==	|等于	|x == y
|!=	|不等于	|x != y
|>=	|大于等于	|x >= y
|>	|大于	|x > y
|~	|与正则表达式匹配|	x ~ /y/
|!~	|与正则表达式不匹配|	x !~ /y/
范例

```
$ cat employees
Tom   Jones 4424 5/12/66 543354
Mary  Adams 5346 11/4/63 28765
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500

$ awk '$3==5346' employees
Mary  Adams 5346 11/4/63 28765
```
说明：如果某行的第3个字段的值等于5346，则条件为真，awk将执行默认操作一一打印该行。如果表达式中隐含着if条件，它就是一个条件模式测试。

```
$ awk '$3 > 5000{print $1}' employees
Mary
```

说明：如果某条记录的第3个字段的值大于5000，awk就打印该记录的第1个字段。

```
$ awk '$2 ~ /Adam/' employees
Mary  Adams 5346 11/4/63 28765
```

说明：如果记录的第2个字段匹配正则表达式Adam，则打印该记录。

```
$ awk '$2 !~ /Adam/' employees
Tom   Jones 4424 5/12/66 543354
Sally Chang 1654 7/22/54 650000
Billy Black 1683 9/23/44 336500
```
说明：如果记录的第2个字段不匹配正则表达式Adam，则打印该记录。假设表达式的值是一个数，与其比较的却是个字符串值，二者之间的运算符如果是用于数值比较的运算符，则字符串值将被转换为一个数值，如果是用于字符串比较的运算符，则数值被转换为字符串值。

#### 17. 条件表达式
数值常量可以表示为整数(如243)、浮点数(如3.14)或用科学记数法表示的数(如.723E-1或3.4e7)。字符串则括在双引号中，例如"Hello world"。

初始化与强制类型转换：只要在awk程序中被提到，变量就开始存在。变量可以是一个字符串或一个数字，也可以既是字符串又是数字。变量被设置后，就变成与等号右边那个表达式相同的类型。未经初始化的变量的值是0就""，究竟是哪个则取决于它们被使用时的上下文。

```
name = "Nancy" name是字符串
x++            x是数字，它被初始化为0，然后加1
number = 35    number是数字
```
如果要将一个字符串强制转换为数字，方法为:
```
name + 0
```
将数字转换成字符串的方法则是:
```
number""
```
所有由split函数创建的字段或数组元素都被视为字符串，除非它们只包含数字值。如果某个字段或数组元素为空，它的值就是空串。空行也可以被视为空串。
#### 18. 算术运算
可以在模式中执行计算。awk命令都将按浮点方式执行算术运算。下表列出了所有的算术运算符。
|运算符|	含义|	例子
| :-------- | --------:| :--: |
|+|	加|	x+y
|-|	减|	x-y
|\*|	乘|	x*y
|/	除	|x/y
|%	|模	|x%y
|^|	幂|	x^y
**范例**

```
$ awk '$3*$4==500' filename
```

说明：awk将记录的第3个字段(\$3)与第4个字段(\$4)的值相乘，如果乘积等于500，则显示该行(假定filename是含有输入数据的文件)。
#### 19. 逻辑操作符和复合模式
逻辑操作符用来测试表达式或者模式的真假。符号&&，表示逻辑与，当所有表达式均为真时，整个表达式为真，只要有一个表达式为假，整个表达式就为假。符号||表示逻辑或，只要有一个表达式或者模式为真，整个表达式就为真。如果所有表达式为假，则整个表达式为假。

复合模式是用逻辑运算符(参见下表)将模式组合起来形成的表达式。表达式的计算是从左往右的。
**逻辑运算符**
|运算符|	含义|	例子
| :-------- | --------:| :--: |
|&&	|逻辑与|	a && b
|\|\||	逻辑或|	a \|\| b
|!|	逻辑非|	!a
**范例**

```
$ awk '$2 > 5 && $2 <= 15' filename
```

说明：awk将显示同时符合这两个条件的行，即该行的第2个字段($2)的值大于5，且小于或等于15。运算符&&要求两个条件都必须为真(假定filename 是包含输入数据的文件)。
**范例**

```
$ awk '$3 == 100 || $4 > 50' filename
```

说明：awk将显示符合两个条件之一的行，即第3个字段等于100或第4个字段大于50的行。运算符||只要求有一个条件必须为真(假定filename是包含输入数据的文件)。
范例
```
$ awk '!($2 < 100 && $3 < 20)' filename
```
说明：如果两个条件都为真，awk将否定该表达式并取消对该行的打印。所以显示出来的行都有一个条件为假，甚至两个条件都为假。一元运算符!对条件的结果求反，所以，如果表达式本来产生一个为真的条件，“非”操作会将其变为假，反之亦然(假定filename 是包含输入数据的文件)。
#### 20. awk范围模式
awk范围模式先匹配从第一个模式的首次出现到第二个模式的首次出现之间的内容，然后匹配从第一个模式的下一次出现到第二个的下一次出现，以此类推。如果匹配到第一个模式而没有发现第二个模式， awk 就将显示从第一个模式首次出现的行到文件末尾之间的所有行。
**范例**

```
$ awk '/Tom/,/Suzanne/' filename
```

说明：awk将显示从Tom首次出现的行到Suzanne首次出现的行这个范围内的所有行，包括两个边界在内。如果没有找到Suzanne，awk将继续打印各行直至文件末尾。如果打印完Tom到Suzanne的内容之后，又出现了Tom， awk就又一次开始显示行，直至找到下一个Suzanne 或文件末尾。
#### 21. awk变量
数值常量可以表示为整数(如243)、浮点数(如3.14)或用科学记数法表示的数(如.723E-1或3.4e7)。字符串则括在双引号中，例如"Hello world"。

初始化与强制类型转换：只要在awk程序中被提到，变量就开始存在。变量可以是一个字符串或一个数字，也可以既是字符串又是数字。变量被设置后，就变成与等号右边那个表达式相同的类型。未经初始化的变量的值是0就""，究竟是哪个则取决于它们被使用时的上下文。

```
name = "Nancy" name是字符串
x++            x是数字，它被初始化为0，然后加1
number = 35    number是数字
```

如果要将一个字符串强制转换为数字，方法为:

```
name + 0
```

将数字转换成字符串的方法则是:

```
number""
```

所有由split函数创建的字段或数组元素都被视为字符串，除非它们只包含数字值。如果某个字段或数组元素为空，它的值就是空串。空行也可以被视为空串。
#### 22. 用户自定义变量
用户自定义变量的变量名可以由字母、数字和下划线组成，但是不能以数字开头。awk的变量不用声明其类型，awk可以从变量在表达式中的上下文推导出它的数据类型。如果变量未被初始化，awk会将字符串变量初始化为空串，将数值变量初始化为0。必要时，awk会将字符型变量转换为数值型变量，或者反向转换。对变量赋值要使用awk的赋值运算符（参见下表）。
**赋值运算符**
|运算符|	含义|	等效表达式|
| :-------- | --------:| :--: |
|=	|a=5|	a=5|
|+=	|a=a+5	|a+=5|
|-=	|a=a-5	|a-=5|
|\*=	|a=a\*5	|a*=5|
|/=|	a=a/5|	a/=5|
|%=|	a=a%5|	a%=5|
|^=	|a=a^5|	a^=5|
最简单的赋值方式是求出表达式的结果，然后将其赋给变量。

**范例**

```
$ awk '$1 ~ /Tom/{wage = $2 * $3;print wage}' filename
```

说明：awk将在第1个字段中扫描Tom。如果发现某一行符合条件，就将其第2个字段的值与第3个字段的值相乘，乘积赋值给用户定义的变量wage。由于乘法是算术运算，所以awk把wage的初始值设为0。

**递增和递减运算符**：如果要将操作数加1，可以使用递增运算符。表达式x++等价于x = x + 1。 类似地，递减运算符则使操作数减少1 。表达式x--等价于x=x-1 。当进行循环操作时，如果只需要递增或递减一个计数器，这种运算符就很有用。递增和递减运算符可以放在操作数的前面，如++x; 也可以置于操作数之后，如x++。用于赋值语句时，这两个运算符的位置不同可能会造成运算结果的差异。

```
{x = 1;y = x++;print x,y}
```
上面这个例子中的++称为后递增运算符: **y先被赋值为1**，然后x才加1。这样，当所有运算做完后，y等于1 ，而x等于2。
```
{x = 1;y = ++x;print x,y}
```
上面这个例子中的++称为先递增运算符:先将x加1，然后才将值2赋给y。这样，在所有运算完成后，y等于2，x也等于2。

**命令行上的用户自定义变量**：可以在命令行上对变量赋值，然后将其传递给awk脚本。如果想对参数处理和ARGV有更多了解，请青参见“处理命令行参数”
```
awk -F: -f awkscript month=4 year=2015 filename
```
说明：用户自定义的变量month和year分别被赋值为4和2015。在awk脚本中可以使用这些变量，就好像它们是在脚本中生成的一样。注意，如果命令行中filename的位置在变量之前，**这些变量将不能在BEGIN语句中使用(参见“awk的BEGIN与END模式”)**。

**字段变量**：字段变量可以像用户自定义的变量一样使用，唯一的区别是它们引用了字段。新的字段可以通过赋值来创建。字段变量引用的字段如果没有值，则被赋值为空串。字段的值发生变化时， awk会以OFS的值作为字段分隔符重新计算$0变量的值。字段数目通常被限制在100 以内。

**范例**

```
$ awk '{$5 = 1000 * $3 / $2; print}' filename
```

说明：如果不存在第5个字段(\$5)，awk将创建它并将表达式1000\*\$3/$2的结果赋给它。如果存在第5个字段，就直接将表达式的结果赋给它，覆盖该字段原来的内容。

范例：
```
$ awk '$4 == "CA" {$4 = "California";print}' filename
```
说明：如果第4个字段($4)匹配字符串CA，awk就将其重新赋值为California。双引号是必需的，如果没有这对双引号，字符串CA就会被当成一个初始值为空的用户自定义变量。

**内量变量**：内置变量的名字都是大写的。它们可以被用于表达式，也可以被重置。请参见下表中所列的内置变量。

**内置变量**
|变量名|	含义
| :-------- | --------:| :------: |
|ARGC	|命令行参数的数目
|ARGIND	|命令行中当前文件在ARGV 内的索引
|ARGV	|命令行参数构成的数组
|CONVFMT	|数字转换格式，默认为%.6g
|ENVIRON	|包含当前shell 环境变量值的数组
|ERRNO	|当使用getline 函数进行读操作或者使用close 函数时，因重定向操作而产生的系统错误描述
|FIELDWIDTHS	|在分隔固定宽度的列表时，使用空白而不是FS 进行分隔的字段宽度列表
|FILENAME	|当前输入文件的文件名
|FNR	|当前文件的记录数
|FS	|输入字段分隔符，默认为空格
|IGNORECASE	|在正则表达式和字符串匹配中不区分大小写
|NF	|当前记录中的字段数
|NR	|目前的记录数
|OFMT|	数字的输出格式
|OFS|	输出字段分隔符
|ORS	|输出记录分隔符
|RLENGTH	|match 函数匹配到的字符串的长度
|RS	|输入记录分隔符
|RSTART	|match 函数匹配到的字符串的偏移量
|RT	|记录终结符，对于匹配字符或者用RS 指定的regex，awk将RT设置到输入文本
|SUBSEP	|数组下标分隔符

**范例**

```
$ cat employees
Tom Jones:4423:5/12/66:543354
Mary Adams:5346:11/4/63:28765
Sally Chang:1654:7/22/54:650000
Mary Black:1683:9/23/44:336500
$ awk -F: '$1 == "Mary Adams"{print NR,$1,$2,$NF}' employees
2 Mary Adams 5346 28765
```

说明：-F选项把字段分隔符设置为冒号。print函数依次打印出记录号、第1个字段、第2个字段和**最后一个字段（$NF）**。

**范例**

```
$ awk -F: '{IGNORECASE=1};\
$1 == "mary adams"{print NR,$1,$2,$NF}' employees
2 Mary Adams 5346 28765
```

说明：-F选项把字段分隔符设置为冒号。若awk的内置变量IGNORECASE为非0值，则在正则表达式和字符串匹配中不区分大小写。接着，字符串mary adams匹配(写成Mary Adams也没关系)。最后，print函数依次打印出记录号、第1个字段、第2个字段和最后一个字段($NF) 。
#### 23. awk的BEGIN与END模式
BEGIN模式后面跟了一个操作块。awk命令必须在对输入文件进行任何处理之前先执行该操作块。实际上，不需要任何输入文件，也能对BEGIN块进行测试，因为awk要在执行完BEGIN操作块后才开始读取输入。BEGIN操作常常被用于修改内置变量(OFS、RS、FS等)的值、为用户自定义变量赋初值和打印输出的页眉或标题。

**范例**

```
$ awk 'BEGIN{FS=":";OFS="\t";ORS="\n\n"}{print $1,$2,$3}' file
```

说明：在处理输入文件之前，awk先把字段分隔符(FS)设为冒号，把输出分隔符(OFS)设为制表符，还把输出记录分隔符(ORS)设为两个换行符。如果BEGIN的操作块中有两条或两条以上语句，必须用分号分隔它们或每行只写一条语句(在shell的命令提示符下输入时，必须用反斜杠来转义换行符)。

**范例**

```
$ awk 'BEGIN{print "MAKE YEAR"}'
MAKE YEAR
```

说明：awk将显示MAKE YEAR。awk打开输入文件之前先执行该print函数，即使没有指定输入文件，awk也照样打印MAKE和YEAR。调试awk脚本时，可以先测试好BEGIN块的操作，再编写程序的其余部分。

**END模式不匹配任何输入行，而是执行任何与之关联的操作。awk处理完所有输入行之后才处理END模式。**

**范例**

```
$ awk 'END{print "The number of record is " NR}' filename
The number of record is 5
```

说明：awk处理完整个文件后才开始执行END块。此时NR的值是最后这条记录的记录号。

**范例**
```
$ awk '/Mary/{count++}END{print "Mary was found " count " times."}' employees
Mary was found 2 times.
```
说明：每遇到一个包含模式Mary的行，用户自定义的变量counter的值就加1。awk处理完整个文件后，END块打印字符串Mary was found，再跟上变量count的值和字符串times。

#### 24. 输出重定向
将awk命令的输出重定向到UNIX/Linux 文件时，会用到shell的重定向操作符。重定向的目标文件名必须用双引号括起来。**如果使用的重定向操作符为>，则文件被打开并请空。**文件一旦打开，就会保持打开状态直至显示关闭或awk程序终止。此后print语句的输出都将追加到文件尾部。

**符号>>也用于打开文件，但是不消除文件内容，它只向文件追加内容。**

**范例**

```
$ awk '$4 >= 70 {print $1,$2 > "passing_file"}' filename
```

说明：如果记录的第4个字段的值大于或等于70，它的头两个字段就被打印到文件
passing_ file 中。
#### 25. 输入重定向(getline)
getline函数： getline函数用于从标准输入、管道或文件(非当前处理的文件)读取输入。getline函数用于读取下一输入行，并且设置内置变量NF、NR和FNR。如果读到一条记录，函数就返回1. 如果读到EOF(end of fiJe，文件末尾)则返回0。如果发生错误，比如打开文件失败，则getline函数返回-1。

范例

$ awk 'BEGIN{"date" | getline d;print d}' filename
Thu Jan 14 11:24:24 PST 2015
说明：先执行UNIX/Linux的date命令，将输出通过管道发给getline，再通过getline将传来的内容赋值给用户自定义的变量d，然后打印d。

范例

$ awk 'BEGIN{"date" | getline d;split(d,mon);print mon[2]}' filename
Jan
说明：先执行date命令，将输出通过管道发给getline，接着，getline从管道读取输入，然后保存在用户自定义变量d中。split函数从d中生成一个名为mon的数组.最后，程序打印出数组mon的第2个元素。

范例

$ awk 'BEGIN{while("ls" | getline) print}'
a.out
db
dbook
file
说明：ls命令的输出将传递给getline: 每循环一次，getline就从ls的输出中读取一行，并将其显示到屏幕上，不需要输入文件， 因为awk会在文件打开之前先处理完BEGIN块。

范例

$ awk 'BEGIN{printf "What is your name?";\
getline name < "/dev/tty"}\
$1 ~ name {print "Found " name " on line ", NR "."}\
END{print "See ya, " name "."}' filename
What is your name? Ellie  < Waits for input from user >
Found Ellie on line 5.
See ya, Ellie.
说明：1.在屏幕上显示What is your name ? 然后等待用户响应. get1ine函数将从终端(/dev/tty)接收输入，直到用户换行，然后，将输入保存在用户自定义的变量name中。
2. 如果第一个字段匹配之前赋给name的值，则执行print函数。
3. END语句打印出"See ya,"，然后显示Ellie(保存在变量name中的值)，再跟上一个句点。

范例

$ awk 'BEGIN{while (getline < "/etc/passwd" >0) lc++;print lc}' file
16
说明：awk将逐行读取文件/etc/passwd， lc随之递增直至到达EOF，然后打印lc的值，即文件passwd的行数。
注意，如果文件不存在，get1ine的值将是-1。如果读到文件尾，返回值是0，而读到一行时，返回值则是1。因此，命令

while (getline < "/etc/junk")

26. awk管道

27. awk if语句

28. awk循环

29. awk控制语句

30. awk数组介绍

31. awk字符串函数

32. 内置算术函数

33. 用户自定义函数

34. 固定字段

35. 多行记录

36. awk内置函数复习



























