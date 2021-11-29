# 变量与字面量
Perl在做变量赋值时、在使用变量时，都会在变量前加上变量前缀，这些特殊的前缀，在Perl中称为Sigil。Sigil前缀隐含了多种含义，其中之二是：
* 根据变量所保存的内存地址去访问该地址所指向的内存空间
* 根据Sigil类型决定如何划分以及如何访问内存空间
## 标量 $
$前缀表示标量，只划分或只访问一个内存数据空间(chunk)。Perl的标量类型有三种基本类型：数值(整数、浮点数)、字符串、引用
## 数组 @
@前缀表示数组，划分或访问多个内存数据空间
## hash %
%前缀表示hash，划分或访问多个用于存放key-value的内存数据空间
## 引用（取地址） \
变量初始化之后就在它的栈中保存了一个指向堆内存中某块空间的地址，由于Perl允许原地修改内存，因此栈中的这个地址在perl程序运行期间将永不改变。
```

[huawei@n148 perl]$ cat 1.pl
#!/usr/bin/perl
use v5.12;
my $x;
say "addr x: ", \$x;
$x = "hello";
say "addr x: ", \$x;
$x = 33;
say "addr x: ", \$x;

[huawei@n148 perl]$ perl 1.pl
addr x: SCALAR(0x7bfab0)
addr x: SCALAR(0x7bfab0)
addr x: SCALAR(0x7bfab0)


--------------------------
cat 1.pl
#!/usr/bin/perl
use v5.12;
my $x = 33;
say "x: $x, ", "addr: ", \$x;
# 为变量重新赋值，变量指向的内存地址不变
$x = 44;
say "x: $x, ", "addr: ", \$x;
# 修改变量数据，变量指向的内存地址不变
++$x;
say "x: $x, ", "addr: ", \$x;
# 但重新声明同名变量，原有变量将被掩盖，新变量指向新内存地址
my $x = 55;
say "x: $x, ", "addr: ", \$x;

SCALAR(0x21c3ab0)[huawei@n148 perl]$ perl 1.pl
x: 33, addr: SCALAR(0x139fab0)
x: 44, addr: SCALAR(0x139fab0)
x: 45, addr: SCALAR(0x139fab0)
x: 55, addr: SCALAR(0x13ab748)
```

此处需要注意，虽然重新声明同名变量会掩盖已有变量使之不可用，但原有变量并未失效，它仍然指向某个堆内存地址，并将持续到程序退出。

```
可以使用$name获取堆中字符串数据，当然也可以使用$$name_ref获取这份堆中字符串数据。或者换个写法会更清晰：${name}和${$name_ref}是等价的，因为name和$name_ref是等价的。

[huawei@n148 perl]$ cat 1.pl
#!/usr/bin/perl
use v5.12;
my $name = "junmajinlong";
my $name_ref = \$name;
print "$name\n";        # ${name}
print "$$name_ref\n";   # ${$name_ref}

[huawei@n148 perl]$ perl 1.pl
junmajinlong
junmajinlong

```
## 赋值
### 按值拷贝
```
将$b赋值给$a时，根据Sigil的规则，出现在赋值操作符左边的$a表示写内存，出现在赋值操作符右边的$b表示读内存数据。也就是说，读取$b保存在堆内存中的数据，并将其写入$a对应的内存空间。因此，内存中将存在两份值相同但地址不同的数据。

[huawei@n148 perl]$ cat 1.pl
#!/usr/bin/perl
use v5.12;
my $b = "junmajinlong";
my $a = $b;
say \$a;
say \$b;

[huawei@n148 perl]$ perl 1.pl
SCALAR(0x1519b88)
SCALAR(0x1536ab0)
```
### 左值、右值
* 当Sigil出现在赋值操作符左边时，表示对变量进行赋值。找到变量的内存地址，然后将数据写入该内存。
* Perl并不仅仅只允许为变量赋值，还可以为数组的元素赋值，为hash的元素赋值，甚至还可以为某些函数调用后的返回结果进行赋值。
* 左值用来向内存中保存数据，右值用来从内存读取数据(即从内存返回数据)。
```
# 变量赋值
my $name = "junmajinlong";

# 为数组元素赋值
my @arr;
$arr[0] = "junmajinlong";

# 为hash元素赋值
my %person;
$person{name} = "junmajinlong";

# 对某些函数调用的返回结果赋值
my $name = 'jun';
substr($name, 3) = 'ma';
```
## 数值字面量
数值有三种保存方式：整数方式、双精度浮点数方式和十进制字符串方式。  
```
$n = 1234;              # 十进制整数
$n = 34_123_456;
$n = 33_22_56;
$n = 0b1110011;         # 二进制整数
$n = 01234;             # 八进制整数
$n = 0x1234;            # 十六进制整数
$n = 12.34e-56;         # 指数定义方式
$n = "-12.34e56";       # 字符串方式定义的数值
$n = "1234";            # 字符串方式定义的数值
$n = "   1234";         # 有前缀空白的字符串数值
$n = "  123a";          # 可以当数值123使用，但warnings时会警告
```
Perl可以将字符串格式的数值当作数值来使用
## 判断数值
```
#!/usr/bin/perl
use v5.12;
my $a;          # undef
my $b = "";     # 空字符串
my $c = 33;     # 数值
my $d = "33";   # 字符串数值
my $e = "3a";   # 字符串

if($a + 0 ne $a){ say "a not numeric"; }
if($b + 0 ne $b){ say "b not numeric"; }
if($c + 0 ne $c){ say "c not numeric"; }
if($d + 0 ne $d){ say "d not numeric"; }
if($e + 0 ne $e){ say "e not numeric"; }

undef和空字符串也可以当作数值0来使用
if($a + 0 ne $a && $a + 0 != 0){ say "a not numeric"; }
if($b + 0 ne $b && $b + 0 != 0){ say "b not numeric"; }
if($c + 0 ne $c && $c + 0 != 0){ say "c not numeric"; }
if($d + 0 ne $d && $d + 0 != 0){ say "d not numeric"; }
if($e + 0 ne $e && $e + 0 != 0){ say "e not numeric"; }


[huawei@n148 perl]$ perl 1.pl
a not numeric
b not numeric
e not numeric
e not numeric

```
## 字符串字面量
和Shell类似，使用双引号或单引号包围
* 双引号：双引号包围的是字符串字面量，但允许在双引号内使用变量内插(Interpolate)、表达式内插、反斜线转义、反斜线字符序列
* 单引号：单引号包围的字符串字面量不允许使用变量内插、表达式内插、反斜线序列，也禁止几乎所有的反斜线转义，只允许对单引号自身和反斜线自身进行反斜线转义

```
#!/usr/bin/perl
use v5.12;
use warnings;

my $name = "junma";
my $age = 23;

say "hello";   # hello，普通字符串
say 'hello';   # hello，普通字符串

# 双引号内可以使用反斜线转义，可以使用单引号
# 单引号内只能转义单引号和反斜线，可以使用双引号
say "hello\"world, hello'world";  # hello"world, hello'world
say 'hello\'world, hello"world';  # hello'world, hello"world
say "hello\\world";  # hello\world
say 'hello\\world';  # hello\world

# 双引号内可以变量内插，单引号内不能变量内插
say "hello $name";    # hello junma
say 'hello $name';    # hello $name

# 双引号内可以表达式内插，单引号内不能表达式内插
say "age: @{[$age+2]}";  # age: 25
say 'age: @{[$age+2]}';  # age: @{[$age+2]}

# 双引号内转义变量内插、转义表达式内插
say "hello \$name";      # hello $name
say "age: \@{[$age+2]}"; # age: @{[23+2]}

# 双引号内可以使用反斜线字符序列，单引号内不允许使用
say "hello\tworld";  # hello  world
say 'hello\tworld';  # hello\tworld

my @arr = (11, 22, 33);
say "@arr";     # 输出：11 22 33
my %person = (name=>"junma", age=>23);
say "%person";  # 输出：%person，hash变量不内插



[huawei@n148 perl]$ perl 1.pl

[huawei@n148 perl]$ perl 1.pl
hello
hello
hello"world, hello'world
hello'world, hello"world
hello\world
hello\world
hello junma
hello $name
age: 25
age: @{[$age+2]}
hello $name
age: @{[23+2]}
hello   world
hello\tworld
11 22 33
%person
```
## 转义字符
* \u 修改下一个字符为大写
* \l 修改下一个字符小写 
* \U 修改后面所有字符大写 
* \L 修改后面所有字符小写 
* \Q 使后面的所有字符都成为字面符号
* \E 结束\U \L或\Q的效果
```
#!/usr/bin/perl
use v5.12;
say "\uabc"; # 输出Abc
say "\Uabc"; # 输出ABC
say "ab\Ucxyz";   # 输出abCXYZ
say "ab\Ucx\Eyz"; # 输出abCXyz
# 将匹配的字符换成大写
my $name = "junmajinlong";
$name =~ s/([j-n])/\U\1\E/g;
say $name;    # 输出：JuNMaJiNLoNg

[huawei@n148 perl]$ perl 1.pl
Abc
ABC
abCXYZ
abCXyz
JuNMaJiNLoNg
```
## q、qq
如果一个字符串比较复杂，使用单双引号包围字符串可能会非常麻烦。Perl中可以使用q实现单引号相同的引用功能，使用qq实现双引号相同的引用功能。
```
#!/usr/bin/perl
use v5.12;

say q!abc!;
say q<abc>;
say qq{abc};

say qq 1def1;   # 等价于qq(def)
say qq adefa;   # 等价于qq(def)

say qq{abc\}def};  # 转义终止符
say qq{abc\{def};  # 转义起始符
say qq ad\aefa;    # 转义起始符a，输出daef

say qq{ab{cd}e};   # ab{cd}e


[huawei@n148 perl]$ perl 1.pl
abc
abc
abc
def
def
abc}def
abc{def
daef
ab{cd}e

```
## bareword
不用任何引号包围的字符也被Perl当作字符串，这种字符串称为Bareword(裸字符串)。如果开启了warnings功能，使用Bareword时，perl会警告。
不建议使用bareword。
## v-str
## here doc
## 字符串连接和重复
Perl使用点.来串联字符串。Perl使用x(字母xyz的x)来重复字符串指定次数，如果x重复次数是小数，则截断为整数，如果x是0，则清空字符串。
```
#!/usr/bin/perl
use v5.12;
my $name = "junmajinlong";
say "www.".$name.".com";  # 输出：www.junmajinlong.com
say '-' x 5;   # 重复5次，输出：-----
say '-' x 5.6;  # 输出-----
say '-' x 0;    # 输出空字符串


[huawei@n148 perl]$ perl 1.pl
www.junmajinlong.com
-----
-----

[huawei@n148 perl]$

```

## 数值和字符串的转换
* 当使用运算符+ - * / % -(负号) ++ -- abs以及< <= > >= == !=时，都会将操作数强制转换为数值。
* 对于字符串来说，当使用.串联字符串或使用x来重复字符串时，会将操作对象强制转换为字符串
* 当字符串操作符和数值操作符混用时，注意它们的优先级。如果不确定优先级，使用括号强制改变优先级：
```
#!/usr/bin/perl
use v5.12;
say "0333" + 22;  # 返回355
say "12abc" * 3;  # 36
say "abc12" * 4;  # 0
say " 12abc" * 3; # 36
say "033".22;    # 返回03322
say 033.22;      # 返回2722，033表示8进制，转换为十进制为27(3*8+3)
say "abc".5*3; # 返回abc15，乘法先运算
say "abc".5 + 3; # 返回3，"."先运算
say "abc".(5+3) # 返回abc8

[huawei@n148 perl]$ perl 1.pl
355
36
0
36
03322
2722
abc15
3
abc8

```
# 数值和字符串常见操作函数
## 取整 int
截断为整数，如  
say int(3.78);  # 3
## 随机数 rand
返回指定范围内的随机数，不指定范围，则返回[0,1)之间的随机数
```
say rand();     # 返回0到1之间的随机数：0.25324216212994
say rand(1.5);  # 返回0到1.5之间的随机数：1.21238530987939
say rand(10);   # 返回0到10之间的随机数：9.61505077404098
say int(rand(10)); # 要获取随机整数，加上int()。返回0到10之间的随机整数
```
## 大小写转换
* lc：(lower case)将后面的字母转换为小写，是\L的实现
* uc：(uppercase)将后面的字母转换为大写，是\U的实现
* fc：(foldcase)和lc基本等价，但fc可以处理UTF-8类的字母
* lcfirst：将后面第一个字母转换为小写，是\l的实现
* ucfirst：将后面第一个字母转换为大写，是\u的实现
```
say lc("HELLO");      # hello
say ucfirst("hello"); # Hello
```
## 进制、编码转换
* chr：ASCII码(或unicode码点)转换为对应字符
* ord：字符转换为ASCII码(或unicode码点)
```
say chr(65);    # A
say ord('A');   # 65
say ord('AB');  # 65
```
* hex：十六进制字符串转换为十进制数值  
注意，如果给定的不是字符串，而是数值本身，则数值转换为十进制后被当作十六进制字符串进行处理。
```
hex 将十六进制字符串转换为十进制数值。
say hex '0x32'; # 50

# 没有前缀0x，32也当作十六进制字符串
say hex '32';   # 50

# 给定数值作为参数
# 0x32对应数值50，hex将50当作待处理字符串
# 等价于hex '0x50'
say hex 0x32;   # 80
```
* oct：进制字符串转换为十进制数值的通用函数。可以处理八进制、十六进制、二进制：  
1 当给定字符串以0x或0X开头，等价于hex函数  
2 当给定字符串以0b或0B开头，将字符串当作二进制字符串转换为十进制数值  
3 当给定字符串以0开头或没有特殊前缀开头，将字符串当作八进制字符串转换为十进制数值  
4 当给定的是数值参数而非字符串，则数值转换为十进制后被当作八进制字符串处理  
``` 
oct：
say oct '0x32';   # 50，等价于hex '0x32'
say oct '0b11';   # 3
say oct '050';    # 40
say oct '50';     # 40
say oct 0x32;     # 40，0x32转换为十进制50，等价于oct '50'

如果想要将十进制数值转换为指定的进制字符串，使用printf或sprintf：

printf "%#o\n", 420;  # 0644
printf "%#b\n", 3;    # 0b11
printf "%#x\n", 50;   # 0x32

printf "%#B\n", 3;    # 0B11
printf "%#X\n", 50;   # 0X32

printf "%o\n", 420;  # 644
printf "%b\n", 3;    # 11
printf "%x\n", 50;   # 32
```
## 字符串处理
### 移除尾部换行符 chomp
移除尾部换行符，如果尾部没有换行符，则不做任何事。实际上，chomp移除的是$/变量值对应的字符，该变量表示输入记录分隔符，默认为换行符，因此默认移除字符串尾部换行符。注意：
* chomp不能对字符串字面量进行操作
* chomp可以对左值进行操作
* chomp可以操作列表，表示移除列表每项元素的尾部换行符
* chomp也可以操作hash，表示移除每项value的为尾部换行符
* chomp返回成功移除的总次数   
    1 对于字符串，如果有换行符，则返回1，否则返回0，因此可通过该返回值判断字符串是否有结尾换行符  
    2 对于列表，可以通过返回值来判断总共操作了多少行数据
```
my $name = "junmajinlong\n";
chomp $name;  # name变为"junmajinlong"
chomp $name;  # name仍然是"junmajinlong"
chomp(my $tmp = "junmajinlong\n"); # 对左值变量进行操作

chomp操作数组或列表
my @arr = ("junma\n","gaoxiao\n");
chomp @arr;    # @arr=("junma","gaoxiao")

chomp操作hash
my %lines = (
  line1 => "abc\n",
  line2 => "def\n",
);
chomp %lines;
say "$lines{line1}, $lines{line2}";  # abc def
```

### 去除行尾字符 chop
是chomp的通用版，移除尾部单个字符，等价于s/.$//s，但效率更高。
```
my $name1 = "junmajinlong\n";
my $name2 = "junmajinlong";
chop $name1;   # $name1 = "junmajinlong"
chop $name2;   # $name2 = "junmajinlon"
```



### 取子串 substr
语法
```
substr STRING,OFFSET,LENGTH,REPLACEMENT
substr STRING,OFFSET,LENGTH
substr STRING,OFFSET
```
* offset从0开始计算
* offset为负数时，表示从尾部位移(从-1开始计算)往前位移
* length如果忽略，则从offset处开始取到最尾部
* length为正数时，从offset处开始取length个字符
* length为负数时，length则表示从后往前的位移位置，所以将从offset开始提取，直到尾部保* 留-length个的字符
* replacement替换string中提取出来的字串。需注意：加了replacement，将返回提取的子串，* 同时会修改源字符串STRING

```
$str="love your everything";

say substr $str,5;     # 输出：your everything
say substr $str,-10;   # 从后往前取：everything
say substr $str,5,4;   # 从前往后取4个字符：your
say substr $str,5,-3;  # 从位移5取到位移-3(从后往前算)：your everyth

say substr $str,5,4,"fairy's";  # 替换源字符串，但返回提取子串：your
say $str;              # 源字符串已被替换：love fairy's everything

$str="love your everything";
substr($str,5,4) = "fairy's";
say $str;       # 源字符串已被替换：love fairy's everything

```

### 分段 split

## 字符索引
用来找出给定字符串中某个子串或某个字符的索引位置。
* index：获取字符所在索引位置
* rindex：从后向前搜索，获取字符所在索引位置

语法
```
(r)index STRING,SUBSTR,POSITION
(r)index STRDING,SUBSTR
```  
* index搜索STRING中第一次出现SUBSTR的位置，rindex则从后向前搜索第一次出现的位置，也* 就是从前向后最后一次出现的位置
* 如果省略position，则从起始位置(从0开始计算)开始搜索第一次出现的子串
* 给定position，则从position处开始搜索，如果是rindex，则是找position左边的
* 如果STRING中找不到SUBSTR，则返回-1
```
$str="love you and your everything";

say index $str,"you";     # 输出：5
say index $str,"yours";   # 输出：-1
say index $str,"you",136; # 输出：-1
say index $str,"you",6;   # 从offset=6之后搜索，输出：13
say rindex $str,"you";    # 输出：13
say rindex $str,"you",10; # 找出offset=10左边的最后一个you，输出：5
```

## 字符串字符数 length
返回字符串字符数。不能用于数组和hash。注意，如果包含多字节字符，要加上use utf8才能计算出字符数量。
```
此处结果待修正。。。

say length("abc");     # 3
say length("我爱你");  # 9

# 加上use utf8
use utf8;
say length("我爱你啊a");  # 5
```
## 加密字符串 crypt
实际上是计算MD5摘要信息。第一个参数是待计算摘要的字符串，第二个参数是salt字符串，salt至少两个字符且只取前两个字符(salt允许使用的字符为[0-9a-zA-Z./]共64个字符)。
如果待计算的字符串相同，且salt相同，那么计算出来的摘要信息一定相同。计算的结果中，前两位是salt的前两位字符，后面11位是计算结果。
```
say crypt("hello", "world");  # woglQSsVNh3SM
say crypt("hello", "wo");     # woglQSsVNh3SM
say crypt("hello", "wx");     # wxNrzGG7p9cyw
```

## sprintf
# 数组
数组名指向堆中的连续内存，堆中的连续内存里的每个地址再分别指向具体元素的地址，如下图。数组元素可以异质
![](https://perl-book.junmajinlong.com/imgs/1610678693613.png)
## 创建 qw
qw的全称是quote words，即自动引用单词。perl会在编译期间将qw所引用的单词自动替换为单引号。qw中使用空格分隔各元素，如果某个元素包含空格，则无法使用qw字面量语法
```
my @arr = (11, 22, 33, 44);  # 只包含数值的数组

my @arr = ();   # 声明空数组
push @arr, 11;
push @arr, 22;
say "@arr";  # 别忘了，双引号允许内插数组
# 输出：11 22

my @arr1 = ('a', 'b', 'c', 'd');
my @arr2 = qw(a b c d);
my @arr = qw(11,22,33);
my $a = (11,22,33)[2]; # $a=33

my @a = (1,,,,3,2,,);  # 等价于(1,3,2)，括号里(或列表中)使用连续逗号，不会产生任何效果。
```
另外，上面示例使用的是qw()，其中括号可以替换为其他成对的符号，这一点和q或qq的用法完全一致。qw()、qw[]、qw!!、qw%%
## 索引
* 下标从0开始。注意，索引数组元素时使用前缀$ 加数组标量名加索引下标值，使用$前缀的原因是要访问数组中这个元素的单个内存chunk
```
my @arr = (11,22,33,44);
读取index=0位置处也就是第一个元素
say $arr[0];
```  
* 支持负数索引，表示从数组尾部开始计算元素的位置。index=-1表示倒数第一个也就是最后一个元素的位置，index=-2表示倒数第二个元素的位置。
```
my @arr = (11,22,33,44);
say $arr[-1];     # 44
say $arr[-2];     # 33
```
* 对数组进行索引时，它也可以作为左值被赋值，这表示将所赋值数据写入该元素对应的内存空间。
```
my @arr = (11,22,33,44);
作为左值被赋值
$arr[1] = 222;
$arr[-1] = 444;
say "@arr";   # 输出：11 222 33 444
```
* 数组越界访问时，perl不会报错，而是返回undef。数组通过正数索引越界赋值时，将使用undef填充中间缺失的元素。
```
my @arr = (11,22,33,44);
say $arr[10];     # 输出空字符串，数组不变

数组长度为11，最后一个索引为index=10, 中间index=4到index=9的元素都被填充为undef
$arr[10] = 9999;
```
## 连接
要扩展数组，Perl中的逻辑是将数组放进小括号中，这会将数组转变成列表格式，然后在列表上扩展更多元素。
```
my @arr = (11,22,33);
@arr = (@arr, 44, 55);  # 扩展数组
say "@arr";  # 11 22 33 44 55

my @arr = (11,22,33);
# 数组先展开，得到(11,22,33)，然后赋值给@arr1
my @arr1 = (@arr);

有时候会使用数组长度作为索引，来向数组尾部追加一个元素：
my @arr = (11,22,33);
$arr[scalar @arr] = 44;
say "@arr";  # 输出：11 22 33 44
```
## 长度与最大索引
长度即数组有多少个元素，有些场景直接@数组返回的也是长度，如下演示
```
my @arr = (11,22,33,44);

scalar使其参数强制进入标量上下文
say scalar @arr;    # 输出：4

下面的加法操作会将操作数@arr转换为标量得到arr数组的长度4
say 1+@arr;    # 5，等价于1+4
```
#arr表示获取数组最大索引值，因为索引是一个标量数值(是可以手动修改的值，下面演示了使用)，所以加上$前缀来访问该标量值。
```
my @arr = (11, 22, 33, 44);
say $#arr;   # 3
```
可以通过$arr[$#arr]表示数组最后一个元素数据。通过下面这种方式，可以向数组尾部追加一个元素：
```
my @arr = (11,22,33);
$arr[++$#arr] = 44;
say "@arr";  # 输出：11 22 33 44
```

最大索引值和长度会相互影响，修改最大索引值会同步修改数组长度，这会导致数组的扩展或截断。
```
my @arr = (11,22,33,44);
$#arr = 10; # 将修改长度为11，缺失元素使用undef填充
$#arr = 2;  # 将长度修改为3，截断元素44和后面的空元素
```
## 转字符串
双引号内插数组时，是使用内置变量$"的值作为数组元素分隔符的，该变量默认值为空格
```
my @arr = (11, 22, 33);
内插后，得到字符串："arr: 11 22 33"
my $str = "arr: @arr";

因此修改$"的值，可以改变数组元素的分隔符。
my @arr = (11, 22, 33, 44);
$" = "-";
say "@arr";   # 11-22-33-44
```
## 打印
```
当未将数组内插进双引号字符串时，print/say默认会将数组各元素紧密相连
my @arr = (11, 22, 33);
say @arr;  # 输出：112233
say @arr,'a','b';  # 输出：112233ab

print/say输出时，默认使用内置变量$,来分隔列表各元素，该内置变量默认值为undef，因此默认情况下各元素被输出时紧密相连。
$, = "_";   # 将其修改为下划线
say @arr,'a','b';  # 11_22_33_a_b
```
## 数组转标量
数组转换为标量得到的是数组的长度
```
my @arr = (11,22,33,44);
下面的加法操作会将操作数@arr转换为标量，得到arr数组的长度4
say 1+@arr;    # 5，等价于1+4

使用加0操作得到标量环境且不影响标量结果
say @arr+0;

使用双波浪号 ~~ 转换为标量上下文，
单个~是位取反操作，要求标量数据，两次位取反得到原数据的标量形式
say ~~@arr;

# 使用scalar强制转换为标量数据
say scalar @arr;
```
## 列表
* Perl中的列表不是数据类型，而是Perl在内部用来临时存放数据的一种方式，只能由Perl自行维护。
* 列表临时保存在栈中，当使用了列表数据后，这些列表数据就会出栈
* 可以将Perl列表看作是一种特殊的底层可迭代对象，它看起来像数组，但不是数组。
```
my @arr = (11,22,33);  # 数组arr的元素来自于列表
```
* push与pop这类仅用于数组，元素排序、元素筛选、迭代遍历等操作，才用于列表

## 列表转标量
列表会返回最后一个列表元素作为标量数据，其他元素被丢弃。
```
my $a = (11,22,33);
my $b = qw(11 22 33);
say $a, "-", $b;   # 输出：33-33


say "h".(1,2,3,4);  # 输出：h4
say ~~(11,22,33);   # 输出：33


my @arr = (11,22,33);
@arr = @arr + (55,66); # 数组转标量得到元素数量，等价于@arr=3+66
say "@arr";         # 输出69

```
## 切片 Slice
* 切片时中括号里提供的索引是一个列表，索引可以重复，可以用负数索引，可以用范围表达式(如1..3表示1 2 3)。但要注意，索引越界将取得undef值。

```
列表切片，取列表中index=0、1和3的元素，列表上下文中返回所取得元素的列表，赋值给数组
my @arr = (11, 22, 33, 44)[0,1,3];  # 11 22 44

数组切片：
my @arr = (11, 22, 33, 44, 55);
my @arr = @arr[1,2,3]  # 22 33 44

(11, 22, 33, 44)[0,1,1,-1];  # 11 22 22 44
(11, 22, 33, 44)[0..2];      # 11 22 33
(11, 22, 33, 44)[0, 2..3];   # 11 33 44
(11, 22, 33, 44)[0, 1, 30];  # 11 22 undef

```
* 可以将rand()的结果作为切片的索引值
```
my @arr = ('a'..'z', 'A'..'Z');
say "@arr[rand 52, rand 52]";  # 取两个随机元素
```

* 取最后四个元素
```
my @arr = (11,22,33,44,55,66);
取最后一个元素
my @arr[$#arr];   # 或：@arr[-1]
my @arr[$#arr-3..$#arr];
```

数组切片和列表切片在使用上有一些区别：
* 数组切片可以内插到双引号中，而列表切片不能内插到双引号
* 数组切片可以作为左值，列表切片则不行
```
内插数组切片
my @arr = (11,22,33,44);
say "@arr[0,1,1]"; # 11 22 22

数组切片作为左值
my @langs = qw(perl python shell php);
@langs[1,2]=qw(ruby bash); # 将python改为ruby，shell改为bash
say "@langs";   # perl ruby bash php
```
## 遍历

```
#!/usr/bin/perl
use v5.12;
my @arr = qw(a b c d e f);

#while
my $i = 0;
while($i <= $#arr){
  say "index: $i, value: $arr[$i]";
  $i++;
}

#for
for(my $i=0;$i<=$#arr;$i++){
  say "index: $i, value: $arr[$i]";
}

[huawei@n148 perl]$ perl 1.pl
index: 0, value: a
index: 1, value: b
index: 2, value: c
index: 3, value: d
index: 4, value: e
index: 5, value: f
index: 0, value: a
index: 1, value: b
index: 2, value: c
index: 3, value: d
index: 4, value: e
index: 5, value: f



#!/usr/bin/perl
use v5.12;
my @arr = qw(a b c d e f);

#for
for my $v (@arr) {
  say "value: $v";
}

#foreach
foreach my $v (@arr) {
  say "value: $v";
}
[huawei@n148 perl]$ perl 1.pl
value: a
value: b
value: c
value: d
value: e
value: f
value: a
value: b
value: c
value: d
value: e
value: f

```
注意，迭代每个列表元素时，元素是按引用传递给控制变量的，控制变量在栈中保存了元素的内存地址，每次迭代时控制变量的地址都发生改变。因此可以推断，每次迭代时，Perl都会重新声明控制变量，每次声明的控制变量仅在本次迭代过程中有效。

如果在循环体内修改控制变量，也将直接修改列表中该元素的值。
```
my @arr = (11, 22, 33, 44);
for my $v (@arr) {
  $v++;
}

say "@arr";  # 输出：12 23 34 45
```
## 默认迭代变量 $_
控制变量是可以省略的，此时将使用Perl的默认标量变量$_。Perl的很多操作都允许省略操作目标，此时将使用默认变量$_作为这些操作的操作目标
```
for(11,22,33){ say $_; }
for $_ (11,22,33) { say $_; }
```
## 小心使用each
