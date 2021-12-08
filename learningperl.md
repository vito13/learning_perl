# 基础知识
## 内存布局
当声明变量时，Perl会为该变量初始化(默认初始化为undef)，它在栈中保存了一个内存地址，以后向该变量赋值时，数据都将存放在栈中这个地址所指向的内存处。
```
my $n;
$n = 33;
$n = 'junma';
```
![](https://perl-book.junmajinlong.com/imgs/2021-01-25_19-49-17.png)
Perl中的数组、hash结构，它们保存的元素也都是各数据的引用，而不是直接将数据保存在数组或hash容器中。
```
my @arr = qw(a b c);
my %hash = (one=>1, two=>2, three=>3,);
```
![](https://perl-book.junmajinlong.com/imgs/2021-01-25_23-46-43.png)

## Sigil
Perl在做变量赋值时、在使用变量时，都会在变量前加上变量前缀，这些特殊的前缀，在Perl中称为Sigil。Sigil前缀隐含了多种含义，其中之二是：
* 根据变量所保存的内存地址去访问该地址所指向的内存空间
* 根据Sigil类型决定如何划分以及如何访问内存空间
## 标量 $
标量是一个单一的数据单元。 数据可以是整数，浮点数，字符，字符串，段落等。简单的说它可以是任何东西
```
$age = 25;             # 整型
$name = "runoob";      # 字符串
$salary = 1445.50;     # 浮点数
 
print "Age = $age\n";
print "Name = $name\n";
print "Salary = $salary\n";
```
## 数组 @
数组是用于存储一个有序的标量值的变量，索引从 0 开始。
```
@ages = (25, 30, 40);             
@names = ("google", "runoob", "taobao");
 
print "\$ages[0] = $ages[0]\n";
print "\$ages[1] = $ages[1]\n";
print "\$ages[2] = $ages[2]\n";
print "\$names[0] = $names[0]\n";
print "\$names[1] = $names[1]\n";
print "\$names[2] = $names[2]\n";
```
## 哈希 %
哈希是一个无序的 key/value 对集合。可以使用键作为下标获取值。哈希变量以字符 % 开头。
```
%data = ('google', 45, 'runoob', 30, 'taobao', 40);
 
print "\$data{'google'} = $data{'google'}\n";
print "\$data{'runoob'} = $data{'runoob'}\n";
print "\$data{'taobao'} = $data{'taobao'}\n";
```
## 何谓上下文
所谓上下文：指的是表达式所在的位置。上下文是由等号左边的变量类型决定的，等号左边是标量，则是标量上下文，等号左边是列表，则是列表上下文。Perl 解释器会根据上下文来决定变量的类型
```
@names = ('google', 'runoob', 'taobao');
 
@copy = @names;   # 复制数组
$size = @names;   # 数组赋值给标量，返回数组元素个数
 
print "名字为 : @copy\n";
print "名字数为 : $size\n";

名字为 : google runoob taobao
名字数为 : 3

代码中 @names 是一个数组，它应用在了两个不同的上下文中。第一个将其复制给另外一个数组，所以它输出了数组的所有元素。第二个我们将数组赋值给一个标量，它返回了数组的元素个数。
```
## 打印文件名、行号、包名
```
print "文件名 ". __FILE__ . "\n";
print "行号 " . __LINE__ ."\n";
print "包名 " . __PACKAGE__ ."\n";
```
## 按值拷贝
```
将$b赋值给$a时，根据Sigil的规则，出现在赋值操作符左边的$a表示写内存，出现在赋值操作符右边的$b表示读内存数据。也就是说，读取$b保存在堆内存中的数据，并将其写入$a对应的内存空间。因此，内存中将存在两份值相同但地址不同的数据。

[huawei@n148 perl]$ cat 1.pl
#!/usr/bin/perl
use 5.12;
my $b = "junmajinlong";
my $a = $b;
say \$a;
say \$b;

[huawei@n148 perl]$ perl 1.pl
SCALAR(0x1519b88)
SCALAR(0x1536ab0)
```
## 赋值
变量初始化之后就在它的栈中保存了一个指向堆内存中某块空间的地址，由于Perl允许原地修改内存，因此栈中的这个地址在perl程序运行期间将永不改变。


```

[huawei@n148 perl]$ cat 1.pl
#!/usr/bin/perl
use 5.12;
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
use 5.12;
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
use 5.12;
my $name = "junmajinlong";
my $name_ref = \$name;
print "$name\n";        # ${name}
print "$$name_ref\n";   # ${$name_ref}

[huawei@n148 perl]$ perl 1.pl
junmajinlong
junmajinlong

```


## 左值、右值
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
## 判断数据类型
```
#!/usr/bin/perl
use 5.12;
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
## 比较运算符
https://www.runoob.com/perl/perl-operators.html
```
数值     字符串      意义
-----------------------------
==       eq        相等
!=       ne        不等
<        lt        小于
>        gt        大于
<=       le        小于或等于
>=       ge        大于或等于
<=>      cmp       返回值-1/0/1。
				   a小于b时，返回-1
				   a等于b时，返回0
				   a大于b时，返回1
				   对于<=>，如果比较的双方有一方不是数值，该操作符将返回undef

示例
35 != 30 + 5       # false
35 == 35.0         # true
'35'   eq '35.0'   # false(str compare)
'fly'  lt 'bly'    # false
'fly'  lt 'free'   # true
'red'  eq 'red'    # true
'red'  eq 'Red'    # false
' '    gt ''       # true，空格大于空串
10<=>20            # -1
20<=>20            # 0
30<=>20            # 1				   
```
## &&、||、!
逻辑运算操作符
```
not expr          # 逻辑取反
expr1 and expr2   # 逻辑与
expr1  or expr2   # 逻辑或

! expr            # 逻辑取反
expr1 && expr2    # 逻辑与
expr1 || expr2    # 逻辑或
```
not、and、or基本等价于对应的!、&&、||，但文字格式的逻辑运算符优先级非常低，而符号格式的逻辑运算符优先级则较高。因为符号格式的逻辑运算符优先级很高，所以往往左边和右边都会加上括号，而文字格式的优先级很低，左右两边不需加括号
```
if (($n >=60) && ($n <80)){ print "..."; }
if ($n >=60 and $n <80){ print "..."; }

or运算符往往会用于连接两个【成功执行，否则就】的子句。例如，打开文件，如果打开失败，就报错退出perl程序：
open LOG '<' "/tmp/a.log" or die "Can't open file!";


and运算符也常用于连接两个行为：左边为真，就执行右边的操作(例如赋值)。
$m < $n and $m = $n;   # 将$m和$n之间较大值保存到变量m


!!在布尔判断效果上不会变化，但会将值转换为undef或1。
say !!"abc";   # 1
say !!"";      # 空
say ((!!"") + 2);  # 2
```
## 逻辑定义或 //
||会短路计算，且有返回值。结合这两点，可以为变量做默认赋值。
```
my $name = $myname || "junmajinlong"
```
但是，这样的方式不严谨，因为有两种情况都会将junmajinlong作为默认值赋值给变量name：
* 变量myname处于未定义状态
* 变量myname已定义，但其值为undef、空字符串、数值0等代表布尔假的值

因此Perl v5.10提供了另一种逻辑运算符//【逻辑定义或】(logical defined-or)。如果左边的值不是undef(包括未定义变量)，则短路运算且返回左边的值，如果左边的值为undef，则计算并返回右边表达式的值。
```
use 5.010;
my $name = $myname // "junmajinloing";

当开启了warnings时，//可以免除使用undef值的警告，但仍然无法避开use strict模式下使用未定义变量的编译错误。
use warnings;
#无警告，尽管使用了undef
my $name = undef // "junmajinloing";
use strict;
#报错，使用了未定义变量myname
my $name = $myname // "junmajinloing";
```
## 引号运算符 q、qq、qx
- q{ } 	为字符串添加单引号 	q{abcd} 结果为 'abcd'
- qq{ } 	为字符串添加双引号 	qq{abcd} 结果为 "abcd"
- qx{ } 	为字符串添加反引号 	qx{abcd} 结果为 `abcd`

```
$a = 10;
$b = q{a = $a};
print "q{a = \$a} = $b\n";
$b = qq{a = $a};
print "qq{a = \$a} = $b\n";
# 使用 unix 的 date 命令执行
$t = qx{date};
print "qx{date} = $t\n";

以上程序执行输出结果为：
q{a = $a} = a = $a
qq{a = $a} = a = 10
qx{date} = 2016年 6月10日 星期五 16时22分33秒 CST



#!/usr/bin/perl
use 5.12;

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
## 范围运算符 .. 与 ...
可以使用两点运算符..或三点运算符...表示一个范围。在列表上下文中两者等价，在标量上下文中两者不等价。
* 用于列表  
在列表上下文中，对于范围A..B来说，它返回从A到B中间所有的值，且包含边界的A和B，每一个值都是前一个值自增(即++运算符)之后的结果。如果左边的A值大于右边的B值，将表示空范围。
```
my @arr1 = 1..3;   # 1 2 3
my @arr2 = ('A'..'Z'); # 所有大写字母
my @arr3 = ('a'..'z'); # 所有小写字母
my @arr4 = ('a'..'z','A'..'Z'); # 所有大小写字母
my @arr5 = (0,3..5,7,10..20); # 离散数据：0 3 4 5 7 10到20
my @arr6 = ('01'..'31');  # 两位数日期
my @arr7 = ('01'..'12');  # 两位数月份


@c = (2..5);
print "(2..5) = @c\n";
以上程序执行输出结果为：
(2..5) = 2 3 4 5


#循环10次
for(1..10){
  say $_;
}
```
* 用于标量  
在标量上下文中，范围表示一个布尔值。

    A..B：从表达式A返回布尔真开始，到表达式B返回布尔真结束，评估表达式A之后会立即评估表达式B
    A...B：从表达式A返回布尔真开始，到表达式B返回布尔真结束，评估表达式A之后不会立即评估表达式B，而是下一次再评估B

```
my $i = 0;
#从0开始，左表达式为真，开始进入范围
#到5结束，此时右表达式为真，结束范围
#范围不要放在for、foreach里，它们是列表上下文
while(($i==0)..($i==5)){
  say $i;   # 输出：0 1 2 3 4 5
  $i++;
}


my $i = 0;
#i为0时，左右表达式都为真，立即终止范围
#因此只输出0
while(($i==0)..($i>=0)){
  say $i;
  $i++;
}

$i = 0;
# 变量i为0时，左表达式为真，开始进入范围，
# 此时不评估右表达式，第二轮循环才评估右表达式
# 因此输出0和1
while(($i==0)...($i>=0)){
  say $i;
  $i++;
}

```
## bareword
不用任何引号包围的字符也被Perl当作字符串，这种字符串称为Bareword(裸字符串)。如果开启了warnings功能，使用Bareword时，perl会警告。
不建议使用bareword。
## v-str
## here doc


## 数值和字符串的转换
* 当使用运算符+ - * / % -(负号) ++ -- abs以及< <= > >= == !=时，都会将操作数强制转换为数值。
* 对于字符串来说，当使用.串联字符串或使用x来重复字符串时，会将操作对象强制转换为字符串
* 当字符串操作符和数值操作符混用时，注意它们的优先级。如果不确定优先级，使用括号强制改变优先级：
```
#!/usr/bin/perl
use 5.12;
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
## 读取标准输入
```
使用一对尖括号格式的<STDIN>来读取来自非文件的标准输入，例如来自管道的数据，来自输入重定向的数据或者来自键盘的输入，<STDIN>读取的输入会自带换行符，所以print输出的时候不要加上额外的换行符。如需去除行尾的换行，使用chomp

脚本内容：
my $data=<STDIN>;
say "$data";
下面是循环的样式
foreach (<STDIN>){
    say "$_";
}

[huawei@n148 perl]$ echo "aaa\nbbb"|perl 1.pl 
aaa\nbbb

[huawei@n148 perl]$ 
```
# 引用
引用就是指针，Perl 引用是一个标量类型可以指向变量、数组、哈希表甚至子程序，可以应用在程序的任何地方。
## 创建引用
```
在一个变量前加一个'\'号，你就得到了这个变量的'引用'。通过引用查看变量所保存数据的内存地址


my $name = "junmajinlong";
say \$name;    # SCALAR(0x55f80476d588)
```
除了下面几种，还有其他类型的引用，比如子程序的引用，文件句柄的引用等
```
$name    ->   \$name   # 标量变量的引用
@array   ->   \@array  # 数组变量的引用
%hash    ->   \%hash   # hash变量的引用
"abc"    ->   \"abc"   # 字面数据值的引用
```
无论是何种类型的引用，它们都是一种指针，指针中保存了其指向数据的内存地址。打印输出引用时，不同类型的引用，输出结果有所不同：
```
my $name = "junma";
my @arr = qw(a b c);
my %h = ( name => "junma", age  => 23 );

say \$name;  # SCALAR(0x55ad7f974dc8)
say \@arr;   # ARRAY(0x55ad7f974738)
say \%h;     # HASH(0x55ad7fa99150)
```
引用是一种指针，因此引用是一种标量数据（数组的引用、hash的引用，也都是标量数据）。
```
my $n = 'junma';
my $n_ref = \$n;
```
n_ref是一个标量变量，它保存变量n的引用，或者说，n_ref保存了变量n所指向字符串数据"junma"的内存地址。
![](https://perl-book.junmajinlong.com/imgs/2021-01-25_19-55-59.png)

```
# 数组变量名和数组引用：名称arr可被替换为$arr_ref
my @arr = qw(a b c);
my $arr_ref = \@arr;

say $arr[0];        # 或${arr}[0]
say $$arr_ref[0];   # 或${$arr}[0]

# hash变量名和hash引用：名称hash可被替换为$h_ref
my %hash = (
  name => "junma",
  age => 23,
);
my $h_ref = \%hash;

say $hash{name};      # 或${hash}{name};
say $$h_ref{name};    # 或${$h_ref}{name}

打印出
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
a
a
junma
junma
```
## 赋值
```
my $name = "junma";
my $name1 = \$name;
$name = "junmajinlong";
say $$name1;   # junmajinlong
创建了指针name1执行了name，然后name的内容变量，打印$$肯定也变了
```
```
my $name = "junma";
my $name1 = \$name;
#name1不再是引用变量，而是字符串变量，此句相当于将name1在堆上内容由原来的地址直接改为了字符串
$name1 = "junmajinlong";  
say $name1;   # junmajinlong
say $name;    # junma
```
和一些语法糖
```
# 将字面量的引用赋值给变量
my $name = \"junmajinlong";  
say $name;

# 声明变量的同时，将其引用赋值给变量
my $age_ref = \my $age;
say $age_ref;

# 声明变量的同时赋值，且将其引用赋值给变量
my $age1_ref = \(my $age1 = 33);
say $$age1_ref;    # 33
```
## 查询引用类型 ref
返回值列表如下，如果没有以下的值返回 false
* SCALAR
* ARRAY
* HASH
* CODE
* GLOB
* REF
```
$var = 10;
$r = \$var;
print "r 的引用类型 : ", ref($r), "\n";
 
@var = (1, 2, 3);
$r = \@var;
print "r 的引用类型 : ", ref($r), "\n";
 
%var = ('key1' => 10, 'key2' => 20);
$r = \%var;
print "r 的引用类型 : ", ref($r), "\n";

执行以上实例执行结果为：

r 的引用类型 : SCALAR
r 的引用类型 : ARRAY
r 的引用类型 : HASH
```
## 解引用
根据引用获取其指向的原始数据。
* 引用的是一个标量，解引用时加上sigil前缀$
* 引用的是一个数组，解引用时加上sigil前缀@
* 引用的是一个哈希，解引用时加上sigil前缀%

```
$var = 10;
# $r 引用 $var 标量
$r = \$var;
# 输出本地存储的 $r 的变量值
print "$var 为 : ", $$r, "\n";
@var = (1, 2, 3);
# $r 引用  @var 数组
$r = \@var;
# 输出本地存储的 $r 的变量值
print "@var 为: ",  @$r, "\n";
%var = ('key1' => 10, 'key2' => 20);
# $r 引用  %var 哈希
$r = \%var;
# 输出本地存储的 $r 的变量值
print "\%var 为 : ", %$r, "\n";

执行以上实例执行结果为：
10 为 : 10
1 2 3 为: 123
\%var 为 : key110key220

-----------------------
普通方式
$name  ->  $$name_ref
@arr   ->  @$arr_ref
%hash  ->  %$hash_ref


完全限定语法
${name}  ->  ${$name_ref}
@{arr}   ->  @{$arr_ref}
%{hash}  ->  %{$hash_ref}
```
对于数组或hash
```
取数组或hash的单个元素：
$arr[0]        ->   $$arr_ref[0]
${arr}[0]      ->   ${$arr_ref}[0]
$hash{name}    ->   $$hash_ref{name}
${hash}{name}  ->   ${$hash_ref}{name}


my @name=qw(junma jinlong);
my $ref_name=\@name;
say "@{ $ref_name }";
say "@$ref_name";
say "$$ref_name[0]";
say "${$ref_name}[0]";
```
也可使用->方式。
```
my @names = qw(junma jinlong);
my $ref_names = \@names;
say $ref_names->[0];   # 等价于${$ref_names}[0]

my %hash=(
    name => "junmajinlong",
    age  => 23,
);
my $ref_hash =\%hash;
say $ref_hash->{name};  # 等价于${$ref_hash}{name}
```
当数组中嵌套数组或hash，hash中嵌套数组或hash时，优先使用瘦箭头解引用的方式来取元素，这样整个取值过程更清晰。
```
# $ref_Config是一个hash引用，取得其中的urllist值
# urllist值是一个数组，取得第二个元素，该元素仍为数组，
# 再取得第四个元素，依然是数组，最后取得第二个元素
say ${$ref_Config}{urllist}[1][3][1];
say ${${${$ref_Config}{urllist}[1]}[3]}[1];
say $ref_Config->{urllist}->[1]->[3]->[1];

并且，连续多个瘦箭头时，从第二个瘦箭头开始可以省略瘦箭头。例如上面最后一种写法可简写为：
say $ref_Config->{urllist}[1][3][1];
```
## 函数的引用
函数引用格式: \&  
调用引用函数格式: & + 创建的引用名。
```
# 函数定义
sub PrintHash{
   my (%hash) = @_;
   
   foreach $item (%hash){
      print "元素 : $item\n";
   }
}
%hash = ('name' => 'runoob', 'age' => 3);
 
# 创建函数的引用
$cref = \&PrintHash;
 
# 使用引用调用函数
&$cref(%hash);

执行以上实例执行结果为：

元素 : age
元素 : 3
元素 : name
元素 : runoob
```
# 流程控制
## 关于true与false
Perl没有直接代表布尔值的false值和true值。代表布尔假的值有如下：

    数值0、0.0
    空字符串''、字符串"0"
    undef
    空列表，包括() ((())) ((),())
    空数组
    空hash
除以上代表布尔假的值之外，其余都是布尔真。

## if、unless、？：
```
COND可以是任意一个表示布尔值的值或表达式，它是一个标量上下文。Perl中任何一个需要进行条件判断的地方都是标量上下文。

if(COND){  # 或者 unless(COND)
  command
}

if(COND1){  # 或者 unless(COND1)
  command1
}elsif(COND2){
  command2
}elsif(COND3){
  command3
} else {
  commandN
}

if(COND){  # 或者 unless(COND)
  command1
}else{
  command2
}


#如果数组元素数量大于等于5，则输出前5个元素
if(@arr >= 5){
  say "@arr[0..4]";
}
#除非数组元素数量小于5，否则就输出前5个元素
unless(@arr < 5){
  say "@arr[0..4]";
}


#如果$score小于60分，则mark为c
#如果大于等于60小于80，则mark为b
#如果大于等于80小于90，则mark为c
#其余情况，mark为d

$mark = ($score < 60) ? "c" : 
        ($score < 80) ? "b" : 
        ($score < 90) ? "a" : 
        "a++";          # 默认值
say $mark;

```

## switch
安装 Switch.pm 模块。打开命令窗口，输入 cpan 命令，然后输入 install Switch 命令，完毕后exit退出cpan即可
```
use Switch;
 
$var = 10;
@array = (10, 20, 30);
%hash = ('key1' => 10, 'key2' => 20);
 
switch($var){
   case 10           { print "数字 10\n"; next; }  # 匹配后继续执行
   case "a"          { print "string a" }
   case [1..10,42]   { print "数字在列表中" }
   case (\@array)    { print "数字在数组中" }
   case (\%hash)     { print "在哈希中" }
   else              { print "没有匹配的条件" }
}
```
## while、until
* 对于while循环，只要条件判断为布尔真，就执行循环体，直到条件判断为假
* 对于until循环，只要条件判断为布尔假，就执行循环体，直到条件判断为真
* 注意条件判断处于标量上下文
```
while(CONDITION){
    commands;
}

until(CONDITION){
    commands;
}


my $i = 0;
while($i<10){
  say $i;
  $i++;
}

while(my($k,$v)=each %p){
  say "k: $k, v: $v";
}


$a = 5;
# 执行 until 循环
until( $a > 10 ){
   printf "a 的值为 : $a\n";
   $a = $a + 1;
}
```
## for、foreach

```
for (my $i = 1;$i<=10;$i++ ){
    print $i,"\n"; # 输出1到10
}

#每次删除字符串开头一个字符，直到删除完所有字符
for(my $str="junmajinlong";$str =~ s/(.)//;){
  say $str;
}

#无限循环
for(;;){
  say "never stop";
}


#循环5次
for (1..5){
  say $_;
}

#遍历数组
my @arr1 = qw(a b c d e f);
for my $i (@arr1){
  say $i;
}

#带索引的数组遍历
my @arr2 = qw(a b c d e f);
for(0..$#arr2){
  say "index: $_, value: $arr2[$_]";
}

修改$i也会影响原始数据
my @arr = qw(1 2 3 4 5);
for (@arr){
  say $_;
  $_++;
}
say "@arr";   # 2 3 4 5 6

改变列表长度，也会影响迭代过程
my @arr = qw(1 2 3 4 5);
for (@arr){
  say $_;        # 输出1 3 5
  shift @arr;
}
say "@arr";   # 4 5
```
```
@list = (2, 12, 36, 42, 51);
 
# 执行foreach 循环
foreach $a (@list){
    print "a 的值为: $a\n";
}
```
## do while
```
$a = 10;
 
# 执行 do...while 循环
do{
   printf "a 的值为: $a\n";
   $a = $a + 1;
}while( $a < 15 );
```
## last、next、redo、continue

* last 语句用于退出循环语句块，从而结束循环，last语句之后的语句不再执行，continue语句块也不再执行。
* next 语句用于停止执行从next语句的下一语句开始到循环体结束标识符之间的语句，转去执行continue语句块，然后再返回到循环体的起始处开始执行下一次循环。
* redo 语句直接转到循环体的第一行开始重复执行本次循环，redo语句之后的语句不再执行，continue语句块也不再执行。continue 语句可用在 while 和 foreach 循环中。
* continue 块通常在条件语句再次判断前执行。continue 语句可用在 while 和 foreach 循环中。
```
while(){
    # <- redo会跳到这里
    CODE
} continue {
    # <- next总是跳到这里
    CODE
}
# <- last跳到这里


---last----------------------------------------------

$a = 10;
while( $a < 20 ){
   if( $a == 15)
   {
       # 退出循环
       $a = $a + 1;
       last;
   }
   print "a 的值为: $a\n";
   $a = $a + 1;
}

#当变量i等于5时退出循环
#该循环将输出1 2 3 4
for my $i (1..10){
  if($i==5){
    last;
  }
  say $i;
}

---next----------------------------------------------
 
$a = 10;
while( $a < 20 ){
   if( $a == 15)
   {
       # 跳出迭代
       $a = $a + 1;
       next;
   }
   print "a 的值为: $a\n";
   $a = $a + 1;
}

#当变量i等于5时进入下一轮循环
#该循环将输出1 2 3 4 6 7 8 9 10
for my $i (1..10){
  $i == 5 and next;
  say $i;
}

---redo----------------------------------------------

$a = 0;
while($a < 10){
   if( $a == 5 ){
      $a = $a + 1;
      redo;
   }
   print "a = $a\n";
}continue{
   $a = $a + 1;
}

#当变量i等于5时，再次执行第5轮循环
#该循环将输出2 3 4 6 6 7 8 9 10 11
for my $i (1..10){
  $i++;
  $i == 5 and redo;
  say $i;
}


---continue----------------------------------------------
$a = 0;
while($a < 3){
   print "a = $a\n";
}continue{
   $a = $a + 1;
}

执行以上程序，输出结果为：
a = 0
a = 1
a = 2



@list = (1, 2, 3, 4, 5);
foreach $a (@list){
   print "a = $a\n";
}continue{
   last if $a == 4;
}
执行以上程序，输出结果为：
a = 1
a = 2
a = 3
a = 4


my $a=3;
while($a<8){
  if($a<5){
    say '$a in main if block: ',$a;
    next;
  }
} continue {
  say '$a in continue block: ',$a;
  $a++;
}

输出结果：

$a in main if block: 3
$a in continue block: 3
$a in main if block: 4
$a in continue block: 4
$a in continue block: 5
$a in continue block: 6
$a in continue block: 7
```
## 标签 label
Perl允许为循环结构打标签，为循环结构打标签后，last、next等可以控制循环流程的关键字就可以指定要控制哪个层次的循环结构。
```
LABEL while (EXPR) BLOCK
LABEL until (EXPR) BLOCK
LABEL for (EXPR; EXPR; EXPR) BLOCK
LABEL for VAR (LIST) BLOCK
LABEL foreach (EXPR; EXPR; EXPR) BLOCK
LABEL foreach VAR (LIST) BLOCK
#纯语句块也可以打标签，它是执行一次的循环结构
LABEL BLOCK  
```
默认情况下，last、next和redo控制的都是当前层次的循环结构，可以为它们指定标签来决定要控制哪个外层循环。下面是使用标签控制外层循环的一个示例：
```
OUTER:
while (1) {
    print "start outer\n";
    while (1) {
        print "start inner\n";
        last OUTER;
        print "end inner\n";
    }
    print "end outer\n";
}
print "done\n";


输出：
start outer
start inner
done
```
## goto
Perl 有三种 goto 形式：goto LABLE，goto EXPR，和 goto &NAME：
```
--------goto LABLE----------------
$a = 10;
LOOP:do
{
    if( $a == 15){
       # 跳过迭代
       $a = $a + 1;
       # 使用 goto LABEL 形式
       print "跳出输出 \n";
       goto LOOP;
       print "这一句不会被执行 \n";
    }
    print "a = $a\n";
    $a = $a + 1;
}while( $a < 20 );

执行以上程序，输出结果为：
a = 10
a = 11
a = 12
a = 13
a = 14
跳出输出 
a = 16
a = 17
a = 18
a = 19



--------goto EXPR----------------
$a = 10;
$str1 = "LO";
$str2 = "OP";
 
LOOP:do
{
    if( $a == 15){
       # 跳过迭代
       $a = $a + 1;
       # 使用 goto EXPR 形式
       goto $str1.$str2;    # 类似 goto LOOP
    }
    print "a = $a\n";
    $a = $a + 1;
}while( $a < 20 );
执行以上程序，输出结果为：
a = 10
a = 11
a = 12
a = 13
a = 14
a = 16
a = 17
a = 18
a = 19
```
## 流程控制语句修饰符
Perl支持单条语句后面加流程控制符。觉得就是语法糖。。。  
```
command Operator Cond;
```
这样的结构称为流程控制表达式。其中：
* command部分是要执行的语句或表达式
* Operator部分支持的操作符有if、unless、while、until和foreach，它们称为流程控制语句修饰符
* Cond部分是条件判断

整个结果的逻辑是：如果条件判断通过，则根据操作符决定是否要执行command部分。

```
print "true.\n" if     $m > $n;
print "true.\n" unless $m > $n;
print "true.\n" while  $m > $n;
print "true.\n" until  $m > $n;
print "$_"      foreach @arr;
```
## do语句块
do语句块像是匿名函数一样，给定一个语句块，直接执行。且和函数一样，do语句块有返回值，它的返回值是最后一个被执行语句的返回值。
```
这段与下面的那段作用一样
my $name=do{
  if($gender eq "male"){"Junmajinlong"}
  elsif($gender eq "female") {"Gaoxiaofang"}
  else {"RenYao"}
};     # 注意结尾的分号

my $name;
if($gender eq "male"){
  $name="Junmajinlong";
} elsif ($gender eq "female"){
  $name="Gaoxiaofang";
} else {
  $name="RenYao";
}

--------------------------
my $a=3;
do {
  say "statement1";
  say "statement2";
} if $a > 2;
```

# 常用函数
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



# 字符串相关
## 单引号和双引号



```
$a = 10;
print "a = $a\n";
print 'a = $a\n';
print "Hello, world\n";    # 双引号
print 'Hello, world\n';    # 单引号

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
a = 10
a = $a\nHello, world
Hello, world\n[huawei@n148 perl]$ 

双引号 \n 输出了换行，而单引号没有。
Perl双引号和单引号的区别: 双引号可以正常解析一些转义字符与变量，而单引号无法解析会原样输出。
```
## 多行字符串
可以使用单引号来输出多行字符串
```
$string = '
菜鸟教程
    —— 学的不仅是技术，更是梦想！
';
 
print "$string\n";
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
use 5.12;
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

## 字符串连接和重复
Perl使用点.来串联字符串。Perl使用x(字母x)来重复字符串指定次数，如果x重复次数是小数，则截断为整数，如果x是0，则清空字符串。
```

$a = "run";
$b = "oob";
print "\$a  = $a ， \$b = $b\n";
$c = $a . $b;
print "\$a . \$b = $c\n";
$c = "-" x 3;
print "\"-\" x 3 = $c\n";

以上程序执行输出结果为：
$a  = run ， $b = oob
$a . $b = runoob
"-" x 3 = ---
(2..5) = 2 3 4 5


#!/usr/bin/perl
use 5.12;
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
## 移除尾部换行符 chomp
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

## 去除行尾字符 chop
是chomp的通用版，移除尾部单个字符，等价于s/.$//s，但效率更高。
```
my $name1 = "junmajinlong\n";
my $name2 = "junmajinlong";
chop $name1;   # $name1 = "junmajinlong"
chop $name2;   # $name2 = "junmajinlon"
```



## 取子串 substr
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

## 将字符串分割为数组 split
使用给定分隔符将字符串划分为列表，分隔符支持使用正则表达式。
在列表上下文，返回划分后得到的列表，在标量上下文，返回划分后列表的元素数量。
```
split /PATTERN/,EXPR,LIMIT
split /PATTERN/,EXPR
split /PATTERN/
split

参数说明：
* PATTERN：分隔符，默认为空格。
* EXPR：指定字符串数。
* LIMIT：如果指定该参数，则返回该数组的元素个数。
```

```
# 定义字符串
$var_test = "runoob";
$var_string = "www-runoob-com";
$var_names = "google,taobao,runoob,weibo";
 
# 字符串转为数组
@test = split('', $var_test);	分隔符为空就是每个字符转为一个数组元素
@string = split('-', $var_string);
@names  = split(',', $var_names);
 
print "$test[3]\n";  # 输出 o
print "$string[2]\n";  # 输出 com
print "$names[3]\n";   # 输出 weibo


my $str="abc:def::123:xyz";
my @list = split /:/,$str;
say join ',', @list;   # abc,def,,123,xyz

my $str="abc:def::12:xyz";
my @list = split /::/,$str;     # 返回："abc:def","12:xyz"
my @list = split /[:]+/,$str;   # 返回："abc","def","12","xyz"
my @list = split /[:0-9]/,$str; # 返回："abc","def","","","","","xyz"


可以加上一个limit参数，限制最多分隔为多少个元素。例如，指定limit=2，表示只分隔一次
my $str="abc:def::123:xyz";
my @list = split /:/,$str,2;   # 返回"abc","def::123:xyz"两个元素

省略limit时，默认limit=0，表示尽可能多地划分元素，且忽略后缀空元素，但会保留前缀空元素。limit为负数时，几乎等价于limit=0，但不忽略后缀空元素。
my $str=":::abc:def:123:xyz::::";
my @new_list1=join(".",split /:/,$str);
my @new_list2=join(".",split /:/,$str, -1);

say "@new_list1";   # ...abc.def.123.xyz
say "@new_list2";   # ...abc.def.1234.xyz....

省略字符串参数时(意味着也必须省略limit)，split默认对$_进行划分：
split /:/;   # 等价于 split /:/, $_;


将pattern指定为空格" "时(注意，不是正则里的空格/ /)，和awk的行为一样：忽略前缀空白，且将一个或多个空白作为分隔符
my $str = "  a  b    c   ";
my @arr = split " ", $str;
say join ",", @arr;   # a,b,c


省略pattern时(意味着后面其他参数也被省略)，即不带任何参数的split，默认pattern为空格" "，对$_变量进行划分

将pattern指定为//时(空正则表达式)，字符串的各字符都被划分
my $str = "abc";
my @arr = split //, $str;
say join ",", @arr;   # a,b,c
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



# 数组
* Perl 数组一个是存储标量值的列表变量，变量可以是不同类型。数组变量以 @ 开头。访问数组元素使用 $ + 变量名称 + [索引值] 格式来读取
* 数组名指向堆中的连续内存，堆中的连续内存里的每个地址再分别指向具体元素的地址，如下图。数组元素可以异质。  
![](https://perl-book.junmajinlong.com/imgs/1610678693613.png)  
* 数组的常见操作还有：each、pop、push、shift、unshift、keys、values、splice。  
## 创建
数组变量以 @ 符号开始，元素放在括号内，也可以以 qw 开始定义数组。qw中使用空格分隔各元素，如果某个元素包含空格，则无法使用qw字面量语法
```
@hits = (25, 30, 40);             
@names = ("google", "runoob", "taobao");
print "\$hits[0] = $hits[0]\n";
print "\$hits[1] = $hits[1]\n";
print "\$hits[2] = $hits[2]\n";
print "\$names[0] = $names[0]\n";
print "\$names[1] = $names[1]\n";
print "\$names[2] = $names[2]\n";

@array = (1, 2, 'Hello');
@array = qw/这是 一个 数组/;
第二个数组使用 qw// 运算符，它返回字符串列表，数组元素以空格分隔。当然也可以使用多行来定义数组


也可以使用多行来定义数组
@days = qw/google
taobao
...
runoob/;

```
qw//可以替换为其他成对的符号，这一点和q或qq的用法完全一致。qw()、qw[]、qw!!、qw%%

Perl 提供了可以按序列输出的数组形式，格式为 起始值 + .. + 结束值
```

@var_10 = (1..10);
@var_20 = (10..20);
@var_abc = ('a'..'z');
 
print "@var_10\n";   # 输出 1 到 10
print "@var_20\n";   # 输出 10 到 20
print "@var_abc\n";  # 输出 a 到 z
```
## 访问
访问数组元素使用 $ + 变量名称 + [索引值] 格式来读取。数组索引值从 0 开始，即 0 为第一个元素，1 为第二个元素，以此类推。负数从反向开始读取，-1 为第一个元素， -2 为第二个元素 
```
@sites = qw/google taobao runoob/;
print "$sites[0]\n";
print "$sites[1]\n";
print "$sites[2]\n";
print "$sites[-1]\n";    # 负数，反向读取

```  
数组越界访问时，perl不会报错，而是返回undef。数组通过正数索引越界赋值时，将使用undef填充中间缺失的元素。
```
my @arr = (11,22,33,44);
say $arr[10];     # 输出空字符串，数组不变

数组长度为11，最后一个索引为index=10, 中间index=4到index=9的元素都被填充为undef
$arr[10] = 9999;
```
## 选择范围 ..
可以在数组中使用 .. 来读取指定范围的元素
```
@list = (5,4,3,2,1)[1..3];
print "list 的值 = @list\n";
list 的值 = 4 3 2
```

## 长度与最大索引
长度即数组有多少个元素，有些场景直接@数组返回的也是长度，如下演示
```
my @arr = (11,22,33,44);
scalar使其参数强制进入标量上下文
say scalar @arr;
my $size = @arr;
print "数组大小:  $size\n";

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
* 双引号内插数组时，是使用内置变量$"的值作为数组元素分隔符的，该变量默认值为空格
```
my @arr = (11, 22, 33);
内插后，得到字符串："arr: 11 22 33"
my $str = "arr: @arr";

因此修改$"的值，可以改变数组元素的分隔符。
my @arr = (11, 22, 33, 44);
$" = "-";
say "@arr";   # 11-22-33-44
```
* 使用join() 函数，见列表的join

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
## 转标量（得length）
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

## 切片
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

my @sites = qw/google taobao runoob weibo qq facebook 网易/;
my @sites2 = @sites[3..5];
print "@sites2\n";
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
use 5.12;
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
use 5.12;
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
## each

## keys、values
* keys函数在列表上下文返回数组的所有索引或hash的所有key，在标量上下文返回数组或hash的元素数量
* values函数在列表上下文返回数组的所有元素或hash的所有value，在标量上下文返回数组或hash的元素数量
```
my @arr = (11, 22, 33, 44);
列表上下文
my @arr_keys = keys @arr;
my @arr_values = values @arr;

say "@arr_keys";    # 0 1 2 3
say "@arr_values";  # 11 22 33 44

for(keys @arr){say $_;}   # 0 1 2 3
for(values @arr){ say $_; } # 11 22 33 44

values获取的值是对元素的引用，因此修改values获取的值，也将修改源数据。
my @arr_values = values @arr;
$arr_values[0] = 111;
say "@arr_values";   # 111 22 33 44
```
## 添加和删除数组元素 pop、push、shift、unshift
* pop从数组中移除并返回最后一个元素，数组为空则返回undef
* push向数组尾部追加一个元素或一个列表，返回追加完成后数组长度
* shift弹出数组第一个值，并返回它。数组的索引值也依次减一。数组为空则返回undef
* unshift向数组头部添加一个元素或一个列表，返回追加完成后数组长度
```
#!/usr/bin/perl
use 5.12;
use warnings;
# pop
my @arr1 = (11,22,33);
say pop @arr1;  # 33
say pop @arr1;  # 22
say pop @arr1;  # 11
say pop @arr1;  # undef，警告模式下会给出警告，类似这样“ Use of uninitialized value in say at 1.pl line 9.” ，否则就不打印警告，只是个空行

# shift
my @arr2 = (11,22,33);
say shift @arr2;  # 11
say shift @arr2;  # 22
say shift @arr2;  # 33
say shift @arr2;  # undef，警告模式下会给出警告


# push
my @arr1 = (11,22,33);
push @arr1, 44;      # 追加单个元素
push @arr1, 55, 66;  # 追加列表，列表的小括号被省略
say push @arr1, (77,88); # 输出8，push返回数组长度
say "@arr1";  # 11 22 33 44 55 66 77 88

# unshift
my @arr2 = (11,22,33);
unshift @arr2, 'a', 'b';
unshift @arr2, qw(aa bb);
say "@arr2"; # aa bb a b 11 22 33
```
## 替换数组元素 splice
语法
```
splice ARRAY
splice ARRAY,OFFSET
splice ARRAY,OFFSET,LENGTH
splice ARRAY,OFFSET,LENGTH,LIST
```
* 一个参数时，即splice ARRAY，表示清空ARRAY。
```
use 5.12;
@arr=qw(perl py php shell);
@new_arr=splice @arr;
say "original arr: @arr";   # 输出：空
say "new arr: @new_arr";    # 输出原列表内容


如果splice在标量上下文，则返回最后一个被移除的元素
my @arr=qw(perl py php shell);
my $new_arr=splice @arr;
say "$new_arr";    # 输出：shell
```
* 两个参数时，即splice ARRAY,OFFSET，表示从OFFSET处开始删除元素直到结尾。
```
@arr=qw(perl py php shell);
@new_arr=splice @arr,2;
say "original arr: @arr";   # 输出：perl py
say "new arr: @new_arr";    # 输出：php shell

如果offset为负数，则表示从后向前数第几个元素，-1表示最后一个元素。
use 5.010;
@arr=qw(perl py php shell);
@new_arr=splice @arr,-3;
say "original arr: @arr";    # 输出：perl
say "new arr: @new_arr";     # 输出：py php shell
```
* 三个参数时，即splice ARRAY,OFFSET,LENGTH，表示从OFFSET处开始向后删除LENGTH个元素。
```
use 5.010;
@arr=qw(perl py php shell ruby);
@new_arr=splice @arr,2,2;
say "original arr: @arr";   # 输出：perl py ruby
say "new arr: @new_arr";    # 输出：php shell

如果length为负数，则表示从offset处开始删除，直到尾部还保留-length个元素(例如length为-3时，表示尾部保留3个元素)。
use 5.010;
@arr=qw(perl py php shell ruby java c c++ js);
@new_arr=splice @arr,2,-2;   # 从php开始删除，最后只保留c++和js两个元素
say "original arr: @arr";    # 输出：perl py c++ js
say "new arr: @new_arr";     # 输出：php shell ruby java c
```
* 四个参数时，即splice ARRAY,OFFSET,LENGTH,LIST，表示将LIST插入到删除的位置，也就是替换数组的部分位置连续的元素。
```
use 5.010;
@arr=qw(perl py php shell ruby);
@list=qw(java c);
@new_arr=splice @arr,2,2,@list;
say "original arr: @arr";   # 输出：perl py java c ruby
say "new arr: @new_arr";    # 输出：php shell

如果想原地插入新元素，而不删除任何元素，可以将length设置为0，它会将新列表插入到offset的位置。
use 5.010;
@arr=qw(perl py php shell ruby);
@list=qw(java c);
@new_arr=splice @arr,2,0,@list;
say "original arr: @arr";   # 输出：perl py java c php shell ruby
say "new arr: @new_arr";    # 输出：空
```
## 排序
见列表sort
## 合并
要扩展数组，Perl中的逻辑是将数组放进小括号中，这会将数组转变成列表格式，然后在列表上扩展更多元素。也可以在数组中嵌入多个数组
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



@numbers = (1,3,(4,5,6));
print "numbers = @numbers\n";
@odd = (1,3,5);
@even = (2, 4, 6);
@numbers = (@odd, @even);
print "numbers = @numbers\n";
```
# 列表
* Perl中的列表不是数据类型，而是Perl在内部用来临时存放数据的一种方式，只能由Perl自行维护。
* 列表临时保存在栈中，当使用了列表数据后，这些列表数据就会出栈
* 可以将Perl列表看作是一种特殊的底层可迭代对象，它看起来像数组，但不是数组。
```
my @arr = (11,22,33);  # 数组arr的元素来自于列表
```
* push与pop这类仅用于数组，元素排序、元素筛选、迭代遍历等操作，才用于列表
* 列表常见操作包括：grep、join、map、reverse、sort、unpack、x操作符执行列表重复，等等
* 标准库List::Utils中也提供了很多常见的列表操作，如reduce、first、any、sum、uniq、shuffle等。

## 转标量（空得undef，非空得尾元素）
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
## 列表重复 x
列表重复通常用于初始化构建一个特定大小的数组，也常用于生成测试数据
```
my @arr = (1, 2) x 3;	#@arr = (1,2,1,2,1,2)

#创建包含100个undef元素的数组
#等价于$arr[99] = undef;
my @arr = (undef) x 100;

#生成一个大数组(20W个元素)，用于某些测试
my @test_data = (11,22) x 100000;

一定要注意，不能将操作符x用于数组，因为x会被解析成字符串重复操作，使得数组处于标量上下文，然后进行字符串重复
my @arr = (11,22);
say @arr x 3;  # 输出：222

如果需要对数组进行重复，将它放进小括号转换为列表即可：
my @arr = (11,22);
@arr = (@arr) x 3;
```
## 从列表中选择元素
在列表后指定索引值可以读取指定的元素
```
$var = (5,4,3,2,1)[4];
print "var 的值为 = $var\n"
var 的值为 = 1
```
## join
用给定字符将列表中各元素连接起来，返回连接后的字符串。
```
say join "-",qw(a b c d e);   # 输出："a-b-c-d-e"


# 定义字符串
$var_string = "www-runoob-com";
$var_names = "google,taobao,runoob,weibo";
 
# 字符串转为数组
@string = split('-', $var_string);
@names  = split(',', $var_names);
 
 
# 数组转为字符串
$string1 = join( '-', @string );
$string2 = join( ',', @names );
 
print "$string1\n";
print "$string2\n";
```

## grep
从列表中筛选符合条件的元素，在列表上下文返回符合条件的元素列表，在标量上下文中返回符合条件的元素数量。grep会迭代列表中的每一个元素，并将这些元素逐次【赋值】给默认变量$_，在给定的语句块{BLOCK}中可以使用该默认变量，
```
my @nums = (11,22,33,44,55,66);
my @odds = grep {$_ % 2} @nums;   # 取奇数
my @evens = grep {$_ % 2 == 0} @nums;  # 取偶数
say "@odds";
say "@evens";

当BLOCK中的代码评估结果为布尔真，则将本次迭代的元素放进返回值列表中等待被返回。当BLOCK中只有一条语句或一个表达式时可以去除{}
grep $_ % 2, @nums;
grep $_ % 2 == 0, @nums;


grep在迭代列表各元素时，$_是各元素的别名引用，在代码块中修改$_，也将影响到源列表，也因此会影响返回值列表。
my @nums = (11,22,33,44,55,66);
my @arr = grep {$_++; $_ % 2} @nums;
say "@arr";     # 23 45 67
say "@nums";    # 12 23 34 45 56 67
```
## map
map迭代列表的每个元素，并将表达式或语句块中返回的值放进一个列表中，最后返回这个列表。
```
my @chars = map(chr, (65..70));
say "@chars";  # A B C D E F

my @arr = map { $_ * 2 } (1..5);
say "@arr";  # 2 4 6 8 10

当语句块中只有一条语句时，可使用表达式写法。
my @arr = map $_*2, (1..5);
say "@arr";  # 2 4 6 8 10


如果语句块中返回空列表()，相当于没有向返回列表中追加元素。例如：
my @arr = (11,22,33,44,55);
#@evens = (undef,22,undef,44,undef)
my @evens = map {$_ if $_%2==0} @arr;

#@evens = (22,44)
my @evens = map {$_%2==0 ? $_ : ()} @arr;
#等价于 map {$_} grep {$_%2==0} @arr;


map允许在一个迭代过程中保存多个元素到返回列表中。
my @name=qw(ma long shuai);
my @new_names=map {$_,$_ x 2} @name;
say "@new_names";  # ma mama long longlong shuai shuaishuai
```
## sort
sort用于对列表元素进行排序，返回排序后的列表。
```
my @str=qw(abc Abc ABc 123);
my @sorted=sort @str;
say "@sorted";     # 123 ABc Abc abc


my @nums = (11,33,4,55,7,12);

#升序排序
my @sorted_nums_asc = sort {$a<=>$b} @nums;
#降序排序
my @sorted_nums_desc = sort {$b<=>$a} @nums;
say "@sorted_nums_asc";  # 4 7 11 12 33 55
say "@sorted_nums_desc"; # 55 33 12 11 7 4
```
## reverse
reverse用于反转列表：在列表上下文中返回元素被反转后的列表，在标量上下文中，返回原始列表各元素组成的字符串的反转字符串
```
my @arr1 = qw(aa bb cc dd);
say "@{[reverse @arr1]}";  # dd cc bb aa
say ~~(reverse @arr1);     # ddccbbaa，返回aabbccdd的反转

say ~~reverse "hello";  # olleh
```
# 哈希
* hash结构中的key是唯一的
* hash结构不保证键值对的顺序
* hash结构的内存空间利用率不高
* hash结构的搜索速度和增删键值对的速度很快，且不会随着所存储键值对元素数量的增长而变慢，它由hash桶的大小决定
* 当某次向hash中存储键值对时因空间不够而触发了扩容，速度会很慢
```
%data = ('google', 'google.com', 'runoob', 'runoob.com', 'taobao', 'taobao.com');
 
print "\$data{'google'} = $data{'google'}\n";
print "\$data{'runoob'} = $data{'runoob'}\n";
print "\$data{'taobao'} = $data{'taobao'}\n";
```
## 创建、赋值、访问
为每个 key 设置 value
```
$data{'google'} = 'google.com';
$data{'runoob'} = 'runoob.com';
$data{'taobao'} = 'taobao.com';
```
通过列表设置
```
列表中第一个元素为 key，第二个为 value。

%data = ('google', 'google.com', 'runoob', 'runoob.com', 'taobao', 'taobao.com');

也可以使用 => 符号来设置 key/value:

%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');

以下实例是上面实例的变种，使用 - 来代替引号：

%data = (-google=>'google.com', -runoob=>'runoob.com', -taobao=>'taobao.com');

使用这种方式 key 不能出现空格，读取元素方式为：

$val = $data{-google}
$val = $data{-runoob}
-----------------------------------------
my %person = (
  "name"  , "junmajinlong",
  "age"   , 23,
  "gender", "male"
);

my %person = (
  "name"  => "junmajinlong",
  "age"   => 23,
  "gender"=> "male"
);

my %person = (
  name   => "junmajinlong",
  age    => 23,
  gender => "male"
);
```
访问哈希元素格式：${key}
```
%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
 
print "\$data{'google'} = $data{'google'}\n";
print "\$data{'runoob'} = $data{'runoob'}\n";
print "\$data{'taobao'} = $data{'taobao'}\n";

-------------------------------
say "$person{name}";
say $person{"age"};

如果访问hash中不存在的key，则返回undef，而不会报错：
say $person{class};  # undef

hash变量不能内插到双引号。
say "%person";   # 直接输出：%person

可以这样添加数据与赋值
$h{k1} = "v1";

使用2个变量组合作为k
my %h;
my ($x, $y) = qw(x, y);
$h{$x.$y} = "junmajinlong.com";  # 等价于$h{"$x$y"}
say $h{"$x$y"};


上面的方式不安全，当使用逗号分隔多份数据组合为key时，Perl会自动将每份数据使用下标连接符(默认值为\034)连接起来，最终得到的字符串作为key。\034通常可以认为是安全的连接符，它是一个ASCII中的控制字符，几乎不会出现在文本数据中。
my %h;
my ($x, $y) = qw(x y);
$h{$x, $y, "name"} = "junmajinlong.com";
say $h{$x, $y, "name"};
say $h{"$x\034$y\034name"};  # 等价形式

Perl使用的下标连接符由内置变量$;控制，该内置变量的默认值为\034。因此，下面这种写法也和上面的写法等价。
say $h{join($;, $x, $y, "name")};
```
## 添加、删除、清空
添加 key/value 对可以通过简单的赋值来完成。但是删除哈希元素你需要使用 delete 函数

```
%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
@keys = keys %data;
$size = @keys;
print "1 - 哈希大小: $size\n";
 
# 添加元素
$data{'facebook'} = 'facebook.com';
@keys = keys %data;
$size = @keys;
print "2 - 哈希大小: $size\n";
 
# 删除哈希中的元素
delete $data{'taobao'};
@keys = keys %data;
$size = @keys;
print "3 - 哈希大小: $size\n";
```


```
my %hash = (
  a => "aa",  b => "bb",
  c => "cc",  d => "dd"
);

delete $hash{a};         # 删除单个键值对
delete @hash{qw(b c)};   # 根据hash切片删除
$, = "-";
say %hash;   # d-dd

# 清空
%hash = ();     # 直接清空hash
undef %hash;    # 注销hash变量
say %hash;
```
## 转列表（所有kv相连）
在列表上下文，hash变量会自动隐式转换为(k1,v1,k2,v2,k3,v3)格式的列表。
```
my %person = (
  name   => "junmajinlong",
  age    => 23,
  gender => "male"
);

#hash展开成列表，默认所有的key和value紧密相连输出
#修改内置变量`$,`可设置say/print输出时的列表分隔符
say %person;  # namejunmajinlonggendermaleage23
$, = "-";
say %person;  # name-junmajinlong-gender-male-age-23
```
## 转标量（空得0，非空得M/N）
在标量上下文，如果hash为空，则转换为数值0，如果hash结构非空，则hash变量会转换为m/n格式的标量，m表示当前的键值对数量，n表示hash结构当前的容量。因此，可以直接将hash变量作为布尔值判断：非空hash为true、空hash为false。
```
my %h;     # 空hash
say ~~%h;  # 输出：0
if(%h){say "empty hash"}  # 不输出

$h{k1} = "v1";
say ~~%h;   # 输出：1/8
if(%h){say "not empty hash"}  # 输出



将hash变量赋值给另一个hash变量时，由于赋值hash时在列表上下文，因此会先将hash展开为列表，再赋值。
my %person = (
  name   => "junmajinlong",
  age    => 23,
  gender => "male"
);

my %p = %person;  # %person展开为列表，然后构建%p
```
## 切片（批量取值与赋值）
```
%data = (-taobao => 45, -google => 30, -runoob => 40);
@array = @data{-taobao, -runoob};
print "Array : @array\n";
执行以上程序，输出结果为：
Array : 45 40


my %phone_num = (
  longshuai =>"18012345678",
  xiaofang =>"17012345678",
  tun_er =>"16012345678",
  fairy =>"15012345678"
);
#双引号中内插hash切片
say "@phone_num{qw(fairy longshuai)}";
#hash切片作为左值
@phone_num{qw(fairy longshuai)} = qw(155555555 188888888);
say "@phone_num{qw(fairy longshuai)}"; # 输出：155555555 188888888
my ($a,$b,$c) = @phone_num{qw(xiaofang fairy xiaofang)};
say $a;
say $b;
say $c;
```
## keys和values
keys和values分别用来获取hash的key列表和value列表。注意，hash各元素的出现顺序是不可预测的。
```
my %h = qw(k1 v1 k2 v2 k3 v3);
my @keys = keys %h;
my @values = values %h;
say "keys: @keys";     # 输出：keys: k3 k2 k1
say "values: @values"; # 输出：values: v3 v2 v1
```
## 获取哈希大小
先获取 key 或 value 的所有元素数组，再计算数组元素多少来获取哈希的大小
```
%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
@keys = keys %data;
$size = @keys;
print "1 - 哈希大小: $size\n";
```
## 遍历
```
my %h = qw(k1 v1 k2 v2 k3 v3);
while(my ($k, $v) = each %h){
  say "key: $k, v: $v";
}

%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
while(($key, $value) = each(%data)){
    print "$data{$key}\n";
}

#foreach迭代遍历key
foreach my $k (sort keys %h){
 say $k, $h{$k};
}


%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
foreach $key (keys %data){
    print "$data{$key}\n";
}


#迭代遍历value
foreach my $v (values %h){
  say $v;
}
```
## 检测元素是否存在 exists
exists用来判断hash结构中是否存在某个key
```
%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
 
if( exists($data{'facebook'} ) ){
   print "facebook 的网址为 $data{'facebook'} \n";
}
else
{
   print "facebook 键不存在\n";
}
```

# 复杂数据结构
## 创建
在需要嵌套数组或hash的时候，应当使用它们的引用
```
hash的v是数组

my @arr = qw(a b c);
my %h = ( aa => \@arr, bb => 1);
my $aa_value = $h{aa};

下面3种结果一样
say ${$aa_value}[0];
say $h{aa}->[0];
say $h{aa}[0];  # 可省略瘦箭头


-----------------------
在将上面的hash放到数组中
my @arr = qw(a b c);
my %h = ( aa => \@arr, bb => 1);
my @outer = ( 'x', 'y', \%h, 'z' );

say $outer[2]->{aa}->[0];
say $outer[2]->{aa}[0];
say $outer[2]{aa}[0];

-----------------------

复杂的结构，通过$ref_Config取得@more_urllist中的第二个元素

my @more_urllist=qw(http://mirrors.shu.edu.cn/CPAN/
  http://mirror.lzu.edu.cn/CPAN/
);
my @my_urllist=('http://mirrors.aliyun.com/CPAN/',
  'https://mirrors.tuna.tsinghua.edu.cn/CPAN/',
  'https://mirrors.163.com/cpan/',
  \@more_urllist   # 将数组more_urllist引用作为元素
);
my %Config = (
  'auto_commit' => '0',
  'build_dir' => '/home/fairy/.cpan/build',
  'bzip2' => '/bin/bzip2',
  'urllist' => [
    'http://cpan.metacpan.org/',
    \@my_urllist  # 将数组my_urllist作为元素
  ],
  'wget' => '/usr/bin/wget',
);

my $ref_Config=\%Config;
say ${$ref_Config}{'urllist'}[1][3][1];
say ${${${$ref_Config}{'urllist'}[1]}[3]}[1];
say $ref_Config->{urllist}->[1]->[3]->[1];
say $ref_Config->{urllist}[1][3][1];
```
## 打印
Data::Printer、Data::Dump未安装
## 匿名数组[ ]、匿名hash{ }
* 使用中括号[]构建匿名数组，使用大括号{}构建匿名hash
* 它们都是引用，因此可以解除引用，还原得到原始数据对象。
* 不包含任何元素的[]和{}算是匿名空数组、匿名空hash


1. 匿名数组 [ ]
```
#匿名数组、匿名hash都是引用，因此赋值给标量变量
my $anonymou_array = []; # 空数组
my $anonymou_hash = {};  # 空hash

#将匿名数组嵌套在其他数组或hash中
my @name=('fairy', ['longshuai','wugui','xiaofang']);
my %hash=('longshuai' => ['male',18,'jiangxi'],
          'wugui'     => ['male',20,'zhejiang'],
          'xiaofang'  => ['female',19,'fujian'],
         );

say "$name[1][2]";
say "$hash{wugui}[1]";
```
如果不想在匿名数组中输入引号，可以使用qw()。以下等价
```
my @name=('fairy',['longshuai','wugui','xiaofang']);
my @name=('fairy',[qw(longshuai wugui xiaofang)]);
```
2. 匿名hash { }
```
my @name=(         # 匿名hash作为数组的元素
  {    # 第一个匿名hash
   'name'=>'longshuai', 'age'=>18,
  },
  {    # 第二个匿名hash
   'name'=>'wugui', 'age'=>20,
  },
  {    # 第三个匿名hash
   'name'=>'xiaofang',  'age'=>19,
  },
);

my %hash=(         # 匿名hash作为hash的value
  'longshuai'=>{   # 第一个匿名hash
                'gender'=>'male', 'age'=>18,
               },
  'wugui'=>{       # 第二个匿名hash
            'gender'=>'male', 'age'=>20,
           },
  'xiaofang'=>{    # 第三个匿名hash
               'gender'=>'female', 'age'=>19,
              },
);
```
## 解除匿名对象的引用
```
my $arr_ref = [qw(a b c)];
say "@$arr_ref";  # 解除引用得到匿名数组
```
除了可以通过解除引用变量的方式来得到匿名数据结构，还可以直接解除匿名数据结构：
* 解除匿名数组的引用@{[]}
* 解除匿名hash的引用%{ {} }

之所以可以直接解除匿名数组和匿名hash，是因为构建匿名数组的[]和构建匿名hash的{}本身就返回引用。
```
#解除匿名数组的引用，得到数组，再将其内插到双引号
say "@{ ['longshuai','xiaofang'] }";
say "@{ [qw(longshuai xiaofang)] }"; 
#获取匿名对象中的第二个元素
say "@{ [qw(longshuai xiaofang)] }[1]";

下面的有语法错误，暂未解决
say %{   # 解除匿名hash
  { # 构造匿名hash
    longshuai=> ['male',18,'jiangxi'],
    wugui    => ['male',20,'zhejiang'],
    xiaofang => ['female',19,'fujian'],
  }
}{longshuai}->[1];
```
## 检查引用的类型
ref函数可用来检查引用的类型，并返回类型。perl中内置了如下几种引用类型，如果检查的不是引用，则返回undef。除了ref函数，Perl模块Scalar::Util提供的reftype函数也用来检测类型，它还适用于对象相关的检测。
- SCALAR
- ARRAY
- HASH
- CODE
- REF
- GLOB
- LVALUE
- FORMAT
- IO
- VSTRING
- Regexp
```
my @name=qw(longshuai wugui);
my $ref_name=\@name;

my %myhash=(
    longshuai => "18012345678",
    xiaofang  => "17012345678",
    wugui     => "16012345678",
    tuner     => "15012345678"
);
my $ref_myhash =\%myhash;

say ref $ref_name;     # ARRAY
say ref $ref_myhash;   # HASH

可以让ref对空hash、空数组等进行检测，然后对比
my $ref_type = ref $ref_myhash;
say "expect HASH reference" unless $ref_type eq 'HASH';
say "expect HASH reference" unless $ref_type eq ref {};

将HASH、ARRAY这样的引用类型名定义为常量：
use constant HASH => ref {};
use constant ARRAY => ref [];

say "expect HASH reference" unless $ref_type eq HASH;
say "expect Array reference" unless $ref_type eq ARRAY;
```

# 异常处理
## die、warn
* Perl自带了warn 函数用于触发一个警告信息，不会有其他操作，输出到 STDERR(标准输出文件)，die 函数类似于 warn, 但它会执行退出。
* 并非所有的错误都会收集到$ !变量中，只有涉及到系统调用且出错时，才会设置$!
* die和warn默认会输出程序名称和行号，但如果在错误消息后面加上\n换行符，则不会报告程序名称和行号。

```
程序中变量 $! 返回了错误信息

if(chdir("/etc1")){
}else{
   die "Error: 无法打开文件 - $!";
}


chdir("/etc1") || die "Error: 无法打开文件 - $!";



unless(chdir("/etc1")){
   die "Error: 无法打开目录 - $!";
}



die "Error: 无法打开目录!: $!" unless(chdir("/etc1"));



chdir('/etc1') or warn "无法切换目录";
```

## croak、carp
Perl自带的die和warn有时候并不友好，它们只会报告代码出错的位置，即哪里使用了die或warn，就报告这个地方有问题。Carp模块提供的croak和carp函数提供了更细致的错误追踪功能，用法分别对应die和warn，区别仅在于它们会展示更具体的错误位置。
```
案例待添加
```
## eval错误处理
使用die或Carp的croak，都将报错退出程序，如果不想退出程序，可以使用eval来处理错误：当eval指定的代码出错时，它将返回undef而不是退出。eval有两种用法: 
* 字符串作为参数
* 语句块作为参数
```
eval 'say "hello world"';
eval {say "hello world"};
```

无论是字符串参数还是语句块参数，参数部分都会被当作代码执行一次。
* 如果eval参数部分执行时没有产生错误，则eval的返回值是最后一条被执行语句的计算结果，并且特殊变量$@为空。
* 如果eval参数部分执行时产生了错误，则eval的返回值为undef或空列表，同时将错误信息设置到$@中。
* 之所以将 $ @ 保存起来，是因为handle_error可能也有eval，这时handle_error的eval将覆盖之前的$@。
```
my $res;
my $ok = eval { $res = some_func(); 1};
if ($ok){
  ...代码正确...
} else {
  ...eval代码出错...
  my $err = $@;
  ...
}
```
# 正则
Perl的正则表达式的三种形式，分别是匹配，替换和转化:
* 匹配：m//
* 替换：s///
* 转化：tr///
  
这三种形式一般都和 =~ 或 !~ 搭配使用， =~ 表示相匹配，!~ 表示不匹配。

## 匹配操作
匹配操作符 m// 用于匹配一个字符串语句或者一个正则表达式。换掉斜线也行 m()，m{}，m!!，m%%，只有使用斜线才可省略m

* 使用data =~ m/reg/，可以明确指定要对data对应的内容进行正则匹配
* 直接/reg/，因为省略了参数，所以使用默认参数变量，它等价于 $_ =~ m/reg/，也就是对 $_保存的内容进行正则匹配
* 返回真假

### 匹配给定字符串内容
```
$bar = "I am runoob site. welcome to runoob site.";
if ($bar =~ /run/){
   print "第一次匹配\n";
}else{
   print "第一次不匹配\n";
}
 
$bar = "run";
if ($bar =~ /run/){
   print "第二次匹配\n";
}else{
   print "第二次不匹配\n";
}

执行以上程序，输出结果为：

第一次匹配
第二次匹配
```
### 匹配来自管道的内容
```
foreach (<STDIN>){
        chomp;
        if (/gao/){
            print "$_ was matched 'gao'\n";
        }
    }

[huawei@n148 perl]$ echo -e "malongshuai gaoxiaofang" | perl 1.pl 
malongshuai gaoxiaofang was matched 'gao'
```
### 匹配来自文件的内容
```
foreach (<>){
    chomp;
    if (/gao/){
        print "$_ was matched 'gao'\n";
    }
}
[huawei@n148 perl]$ perl 1.pl err.txt
malongshuai gaoxiaofang was matched 'gao'
```
模式匹配有一些常用的修饰符，如下表所示：
```
修饰符	描述
i	忽略模式中的大小写
m 	多行模式
o	仅赋值一次
s	单行模式，"."匹配"\n"（默认不匹配）
x	忽略模式中的空白
g	全局匹配
cg	全局匹配失败后，允许再次查找匹配串
```

## 正则特殊变量变量
下面是正则表达式相关的特殊变量总结
```
$1 $2 $3...
保存了各个分组捕获的内容

$`: 匹配部分的前一部分字符串
$&: 匹配的字符串
$': 还没有匹配的剩余字符串

演示：
my $str = "abAB12cdCD34";
say "$`： $& ： $'" if($str =~ /\d+/);		abAB： 12 ： cdCD34



$string = "welcome to runoob site.";
$string =~ m/run/;
print "匹配前的字符串: $`\n";
print "匹配的字符串: $&\n";
print "匹配后的字符串: $'\n";

执行以上程序输出结果为：
匹配前的字符串: welcome to 
匹配的字符串: run
匹配后的字符串: oob site.
```

这些特殊变量是只读的，每次正则匹配时Perl会自动重置这些变量。这些变量只在当前作用域内有效，并且只在本次正则匹配后、下次正则匹配前有效。如下面的第二次say就失败了
```
my $str = "abbc123";
$str=~/a(.*)c/;
say $1;
$str=~/a.*c/;
say $1;   # undef
```
还有几个比较常用的特殊变量是：
```
$+
最后一个匹配成功的分组括号所匹配的内容(PAREN是括号parentheses的缩写)

%+
保存本次匹配过程中所有命名分组捕获的内容，hash的key是分组名称，value是分组捕获的内容

@-
保存了各个分组匹配的起始位置

@+
保存了各个分组匹配的结束位置


@- @+这两个变量结合substr用起来可以非常强大，通过它们可以构造出和$` $& $'等价的值，且能构造出更多匹配结果。例如：
$` == substr($var, 0, $-[0])
$& == substr($var, $-[0], $+[0] - $-[0])
$' == substr($var, $+[0])
$1 == substr($var, $-[1], $+[1] - $-[1])
$2 == substr($var, $-[2], $+[2] - $-[2])
$3 == substr($var, $-[3], $+[3] - $-[3])
```

## 替换操作
替换操作符 s/// 是匹配操作符的扩展，使用新的字符串替换指定的字符串
```
 
$string = "welcome to google site.";
$string =~ s/google/runoob/;
 
print "$string\n";

执行以上程序输出结果为：

welcome to runoob site.
```
替换操作修饰符如下表所示：
```
修饰符	描述
i	如果在修饰符中加上"i"，则正则将会取消大小写敏感性，即"a"和"A" 是一样的。
m	默认的正则开始"^"和结束"$"只是对于正则字符串如果在修饰符中加上"m"，那么开始和结束将会指字符串的每一行：每一行的开头就是"^"，结尾就是"$"。
o	表达式只执行一次。
s	如果在修饰符中加入"s"，那么默认的"."代表除了换行符以外的任何字符将会变成任意字符，也就是包括换行符！
x	如果加上该修饰符，表达式中的空白字符将会被忽略，除非它已经被转义。
g	替换所有匹配的字符串。
e	替换字符串作为表达式
```

## 转化操作
```
使用 /s 将变量 $string 重复的字符删除：
$string = 'runoob';
$string =~ tr/a-z/a-z/s;
print "$string\n";

执行以上程序输出结果为：
runob
```
转化操作符相关的修饰符：
```
修饰符	描述
c	转化所有未指定字符
d	删除所有指定字符
s	把多个相同的输出字符缩成一个
```
## 模式匹配修饰符
perl总共支持以下几种修饰符，这些修饰符可以连用，连用时顺序可随意
* i：匹配时忽略大小写
* g：全局匹配，默认情况下，正则表达式"abc"匹配"abcdabc"字符串的时候，将之匹* 配左边的abc，使用g将匹配两个"abc"
* c：在开启g的情况下，如果匹配失败，将不重置搜索位置
* m：多行匹配模式
* s：让.可以匹配换行符"\n"，也就是说该修饰符让.真的可以匹配任意字符
* x：允许正则表达式使用空白符号，免得让整个表达式难读难懂，但这样会让原本的空* 白符号失去意义，这是可以使用\s来表示空白
* o：只编译一次正则表达式
* n：非捕获模式
* p：保存匹配的字符串到 $ {^PREMATCH}、$ {^MATCH}、$ {^POSTMATCH}中，它们在结果* 上对应$`、$& 和 $'，但性能上要更好
* a和u和l：分别表示用ASCII、Unicode和Locale的方式来解释正则表达式，一般不用* 考虑这几个修饰符
* d：使用unicode或原生字符集，就像5.12和之前那样，也不用考虑这个修饰符
### 忽略大小写 i
```
my $name="aAbBcC";
if($name =~ m/ab/i){
    print "pre match: $` \n";     # 输出a
    print "match: $& \n";         # 输出Ab
    print "post match: $' \n";    # 输出BcC
}
```
### 全局匹配 g
全局匹配情况下，匹配时是可以跳过匹配失败的字符继续匹配的，当某个字符匹配失败，它会后移一位继续去匹配，直到匹配成功或匹配结束。
```
my @arr = "abcabc" =~ /ab/g;  # 匹配返回qw(ab ab)
say "@arr";  # ab ab


使用循环匹配的方式来验证全局匹配中的每次匹配结果：
my $name="aAbBcCaBc";
while($name =~ m/ab/gi){
  say "pre match : $`";
  say "match     : $&";
  say "post match: $'";
}

执行它，将输出如下内容：

pre match : a
match     : Ab
post match: BcCaBc
pre match : aAbBcC
match     : aB
post match: c

-------------------------------------


my $txt="1234ab56";
$txt =~ /\d\d/g;
say "matched $&: ",pos $txt;
$txt =~ /\d\d/g;
say "matched $&: ",pos $txt;
$txt =~ /\d/g;   # 字母a匹配失败，后移一位，字母b匹配失败，后移一位，数值5匹配成功
say "matched $&: ",pos $txt;
$txt =~ /\d/g;   # 数值6匹配成功
say "matched $&: ",pos $txt;

执行上述程序，将输出：

matched 12: 2
matched 34: 4
matched 5: 7
matched 6: 8
```
### 匹配成功位移值 pos
在开启了g全局匹配后，perl会在每次匹配成功后记下匹配的偏移位置，以便下次匹配时可以从该位移处继续向后匹配。每次匹配成功后的位移值，都可以通过pos()函数获取。偏移值从0开始算，0位移代表的是第一个字符左边的位置。如果本次匹配导致位移指针重置，pos将返回undef。

```
my $txt="1234a56";
$txt =~ /\d\d/g;      # 匹配成功：12，位移向后移两位
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d/g;      # 匹配成功：34，位移向后移两位
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d\d/g;      # 匹配失败，位移指针回到0处，pos()返回undef
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/g;          # 匹配成功：1，位移向后移1位
print "matched $&: ",pos $txt,"\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched 12: 2
matched 34: 4
matched 34: 
matched 1: 1
```
匹配失败的时候，正则匹配操作会返回假，所以可以作为if或while等的条件语句来终止全局匹配。例如：
```
my $name="123ab456";
while($name =~ m/\d\d/g){
    print "matched string: $&, position: ",pos $name,"\n";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched string: 12, position: 2
matched string: 45, position: 7

```

### 全局匹配失败不重置位移 gc
默认全局匹配情况下，当本次匹配失败，指针将重置到起始位置0处，也就是说，下次匹配将从头开始匹配
```
my $txt="1234a56";
$txt =~ /\d\d/g;      # 匹配成功：12，位移向后移两位
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d/g;      # 匹配成功：34，位移向后移两位
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d\d/g;      # 匹配失败，位移指针回到0处，pos()返回undef
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/g;          # 匹配成功：1，位移向后移1位
print "matched $&: ",pos $txt,"\n";

执行上述程序，将输出：

matched 12: 2
matched 34: 4
matched 34:   #<-- warning: use undef value
matched 1: 1
```
如果"g"修饰符下同时使用"c"修饰符，也就是"gc"，它表示全局匹配失败的时候不重置位移指针。也就是说，本次匹配失败后，位移指针会向后移一位，下次匹配将从后移的这个位置处开始匹配。当位移移到了结尾，将无法再移动，此时位移指针将一直指向最后一个位置。
```
观察第三次与最后一次匹配的参数gc

my $txt="1234a56";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d\d/gc;   # 匹配失败，位移向后移1位，$&和pos()保留上一次匹配成功的内容
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/g;        # 匹配成功：5，位移向后移1位
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/g;        # 匹配成功：6，位移向后移1位
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/gc;        # 匹配失败：位移无法再后移，将一直指向最后一个位置
print "matched $&: ",pos $txt,"\n";

执行上述程序，将输出：

matched 12: 2
matched 34: 4
matched 34: 4
matched 5: 6
matched 6: 7
matched 6: 7
```
### 与gc配合的 \G
先回顾一下全局匹配情况下，匹配时是可以跳过匹配失败的字符继续匹配的：当某个字符匹配失败，它会后移一位继续去匹配，直到匹配成功或匹配结束。
```
my $txt="1234ab56";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/g;       # 字母a匹配失败，后移一位，字母b匹配失败，后移位，数值5匹配成功
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d/g;       # 数值6匹配成功
print "matched $&: ",pos $txt,"\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched 12: 2
matched 34: 4
matched 5: 7
matched 6: 8
```

接下来看\G的功能：
* 加\G使得本次匹配强制从位移处进行匹配，不允许跳过任何匹配失败的字符。
* 如果本次\G全局匹配成功，位移指针自然会后移
* 如果本次\G全局匹配失败，且没有加上c修饰符，那么位移指针将重置
* 如果本次\G全局匹配失败，且加上了c修饰符，那么位移指针将卡在那不动

```
my $txt="1234ab56";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\G\d/g;     # 强制从位移4开始匹配，无法匹配字母a，但又不允许跳过
                     # 所以本次\G全局匹配失败，由于没有修饰符c，指针重置
print "matched $&: ",pos $txt,"\n";
$txt =~ /\G\d/g;     # 指针回到0，强制从0处开始匹配，数值1能匹配成功
print "matched $&: ",pos $txt,"\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched 12: 2
matched 34: 4
matched 34: 
matched 1: 1
```
如果将上面第三个匹配语句加上修饰符c，甚至后面的语句也都加上\G和c修饰符，那么位移指针将卡在那个位置：
```
my $txt="1234ab56";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\d\d/g;
print "matched $&: ",pos $txt,"\n";
$txt =~ /\G\d/gc;        # 匹配失败，指针卡在原地
print "matched $&: ",pos $txt,"\n";
$txt =~ /\G\d/gc;        # 匹配失败，指针继续卡在原地
print "matched $&: ",pos $txt,"\n";
$txt =~ /\G\d/gc;        # 匹配失败，指针继续卡在原地
print "matched $&: ",pos $txt,"\n";


[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched 12: 2
matched 34: 4
matched 34: 4
matched 34: 4
matched 34: 4
```
通常全局匹配都会用循环去多次迭代，先看下带与不带\G的差异
```
加了\G当匹配失败时候会退出循环

my $txt="1234ab56";
while($txt =~ m/\G\d\d/gc){
    print "matched: $&, ",pos $txt,"\n";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched: 12, 2
matched: 34, 4
----------------------------------
没有\G可以全匹配出来

my $txt="1234ab56";
while($txt =~ m/\d\d/gc){
    print "matched: $&, ",pos $txt,"\n";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched: 12, 2
matched: 34, 4
matched: 56, 8
```
上面使用c与否是无关紧要的，但如果这个while循环的后面后还需要继续进行匹配的话那c修饰符就有用了。
```
my $txt="1234ab56";
while($txt =~ m/\G\d\d/gc){   # 使用c修饰符
    print "matched: $&, ",pos $txt,"\n";
}
$txt =~ m/\G\d\d/gc;
print "matched: $&, ",pos $txt,"\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched: 12, 2
matched: 34, 4
matched: , 4

---------------------------
my $txt="1234ab56";
while($txt =~ m/\G\d\d/g){   # 不使用c修饰符
    print "matched: $&, ",pos $txt,"\n";
}
$txt =~ m/\G\d\d/gc;
print "matched: $&, ",pos $txt,"\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
matched: 12, 2
matched: 34, 4
matched: 12, 2
```

### 多行模式 m
正则表达式一般都只用来匹配单行数据，但有时候却需要一次性匹配多行。比如匹配跨行单词、匹配跨行词组，匹配跨行的对称分隔符(如一对括号)。
```
my $txt="ab\ncd";

下面3句都可以匹配。关于多行匹配，需要注意的是"."默认情况下无法匹配换行符。可以使用[\d\D]代替点，也可以开启s修饰符使"."能匹配换行符。

$txt =~ /a.*\nc/m;
$txt =~ /a.*c/ms;
$txt =~ /a[\d\D]*c/m;

say "===match start==="
say $&;
say "===match end===";

执行，将输出：

===match start===
ab
c
===match end===
```
### “.”可以匹配换行 s
默认情况下"."是不能匹配换行符\n的，开启了s修饰符功能后，可以让"."匹配换行符。案例见多行匹配模式

### 忽略空白 x
perl正则支持表达式的分隔，甚至支持注释，只需加上x修饰符即可。这时候正则表达式中出现的所有空白符号都不会当作正则的匹配对象，而是直接被忽略。如果想要匹配空白符号，可以使用\s表示，或者将空格使用\Q...\E包围。  
例如，以下4个匹配操作是完全等价的。
```
my $ans="cat sheep tiger";
$ans =~ /(\w) *(\w) *(\w)/;       # 正常情况下的匹配表达式
$ans =~ /(\w)\s*   (\w)\s*   (\w)/x;
$ans = ~ /
        (\w)\s*      # 本行是注释：匹配第一个单词
        (\w)\s*      # 本行是注释：匹配第二个单词
        (\w)         # 本行是注释：匹配第三个单词
        /x;
$ans =~ /
         (\w)\Q \E   # \Q \E强制将中间的空格当作字面符号被匹配
         (\w)\Q \E
         (\w)
        /x;
```
### 代替特殊变量 p
前面的3个特殊变量$`、$&和$'可以保存匹配内容之前的内容，匹配内容以及匹配内容之后的内容。另外perl提供了一个p修饰符，能实现完全相同的功能
* ${^PREMATCH}    <=>   $`
* ${^MATCH}       <=>   $&
* ${^POSTMATCH}   <=>   $'
```
my $ans="cat sheep tiger";
$ans =~ /sheep/p;
print "${^PREMATCH}\n";     # 输出"cat "
print "${^MATCH}\n";        # 输出"sheep"
print "${^POSTMATCH}\n";    # 输出" tiger"
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
cat 
sheep
 tiger
```
###	仅编译一次 o
在较老的perl版本中，如果使用同一个正则表达式做多次匹配，正则引擎将只多次编译正则表达式。很多时候正则表达式并不会改变，比如循环匹配文件中的行，这样的多次编译导致性能下降很明显，于是可以使用o修饰符让正则引擎对同一个正则表达式不重复编译。  
在perl5.6中，默认情况下对同一正则表达式只编译一次，但同样可以指定o修饰符，使得即使正则表达式变化了也不要重新编译。一般情况下，可以无视这个修饰符。
### 匹配范围 (?imsx-imsx:pattern)
前文介绍的修饰符adluoimsxpngc都是放在m//{FLAG}的flag处的，放在这个位置会对整个正则表达式产生影响，所以它的作用范围有点广。

例如m/pattern1 pattern2/i的i修饰符会影响pattern1和pattern2。

perl允许我们定义只在一定范围内生效的修饰符，方式是(?imsx:pattern)或(?-imsx:pattern)或(?imsx-imsx:pattern)，其中加上-表示去除这个修饰符的影响。这里只列出了imsx，因为这几个最常用，其他的修饰符也一样有效。

例如，对于待匹配字符串"Hello world gaoxiaofang"，使用以下几种模式去匹配的话：

* /(?i:hello) world/   表示匹配hello时，可忽略大小写，但匹配world时仍然区分大小写。所以匹配成功

* /(?ims:hello.)world/   表示可以跨行匹配helloworld，也可以匹配单行的hellosworld，且hello部分忽略大小写。所以匹配成功

* /(?i:hello (?-i:world) gaoxiaoFANG)/  表示在第二个括号之前，可用忽略大小写进行匹配，但因为第二个括号里指明了去除i的影响，所以对world的匹配会区分大小写，但是对gaoxiaofang部分的匹配又不区分大小写。所以匹配成功

* /(?i:hello (?-i:world) gaoxiao)FANG/   和前面的类似，但是将"FANG"放到了括号外，意味着这部分要区分大小写。所以匹配失败

## 反斜线序列
### 锚定类
所谓锚定，是指它匹配的是位置，而非字符，比如锚定行首的意思是匹配第一个字母前的空字符。也就是很多人说的"零宽断言(zero-width assertions)"。

    \b：匹配单词边界处的空字符
    \B：匹配非单词边界处的空字符
    \<：匹配单词开头处的空字符
    \>：匹配单词结尾处的空字
    \A：匹配绝对行首，换句话说，就是输入内容的开头
    \z：匹配绝对行尾，换句话说，就是输入内容的绝对尾部
    \Z：匹配绝对行尾或绝对行尾换行符前的位置，换句话说，就是输入内容的尾部
    \G：强制从位移指针处进行匹配，详细内容见g和c修饰符以及\G
```
```
# 函数

## 简单定义与调用
```
# 定义子程序
sub roll{
  print 1 + int(rand(6));
}

# 不同调用方式
roll;		其次
roll();		首选
&roll;
&roll();
```
## 返回值
如果没有使用 return 语句，则子程序的最后一行语句将作为返回值

* 有return且带参数：return的参数(即要指定的返回值)是一个列表上下文
	```
	# 返回单个值
	return 1
	return 0
	return ""

	# 返回多值组成的列表
	return 1,2,3;
	return (1,2,3);

	return无法直接返回数组和hash，因为它们会先转换成列表，再以列表的方式返回。如果确实要返回数组或列表，应当返回它们的引用。
	return $array_ref;
	return $hash_ref;

	也可以返回匿名数组或匿名hash
	return [qw(a b c)];   # 返回匿名数组
	return {(name=>"junmajinlong", age=>23,)};   # 返回匿名hash
	```
* 有return无参数：
	- 列表上下文，返回空列表
	- 标量上下文，返回未定义值(可当作undef或0或空字	- 串使用)
	- 空上下文，不返回任何数据，只用于终止子程序
* 无return：以最后被执行的语句计算结果作为返回值
	```
	下面是两个等价的子程序定义方式：
	sub roll{
		1+int(rand(6));
	}

	sub roll(){
		return 1+int(rand(6));
	}

	最后一条被执行的语句是print语句，print函数的返回值为1，因此roll()的返回值为1。
	sub roll{
		print 123;
	}
	say roll();  # 打印1231，其中123是函数内的打印。最后的1是print的运行结果
	```
	
## wantarray
## 传递标量参数
```
Perl 子程序可以和其他编程一样接受多个参数，子程序参数使用特殊数组@_标明。因此子程序第一个参数为 $_[0], 第二个参数为 $_[1], 以此类推。不论参数是标量型还是数组型的，用户把参数传给子程序时，perl默认按引用的方式调用它们。用户可以通过改变 @_ 数组中的值来改变相应实际参数的值。


# 定义求平均值函数
sub Average{
   # 获取所有传入的参数
   $n = scalar(@_);
   $sum = 0;
 
   foreach $item (@_){
      $sum += $item;
   }
   $average = $sum / $n;
   print '传入的参数为 : ',"@_\n";           # 打印整个数组
   print "第一个参数值为 : $_[0]\n";         # 打印第一个参数
   print "传入参数的平均值为 : $average\n";  # 打印平均值
}
 
# 调用函数
Average(10, 20, 30);

执行以上程序，输出结果为：
传入的参数为 : 10 20 30
第一个参数值为 : 10
传入参数的平均值为 : 20
```

## 传递列表参数
```
由于 @_ 变量是一个数组，所以它可以向子程序中传递列表。但如果我们需要传入标量和数组参数时，需要把列表放在最后一个参数上

# 定义函数
sub PrintList{
   my @list = @_;
   print "列表为 : @list\n";
}
$a = 10;
@b = (1, 2, 3, 4);
 
# 列表参数
PrintList($a, @b);

以上程序将标量和数组合并了，输出结果为：
列表为 : 10 1 2 3 4


可以向子程序传入多个数组和哈希，但是在传入多个数组和哈希时，会导致丢失独立的标识。所以我们需要使用引用（下一章节会介绍）来传递
```
## 传递哈希参数
```
传递哈希表时，它将复制到 @_ 中，哈希表将被展开为键/值组合的列表。

# 方法定义
sub PrintHash{
   my (%hash) = @_;
 
   foreach my $key ( keys %hash ){
      my $value = $hash{$key};
      print "$key : $value\n";
   }
}
%hash = ('name' => 'runoob', 'age' => 3);
 
# 传递哈希
PrintHash(%hash);

以上程序执行输出结果为：

age : 3
name : runoob


---------------------------


sub subname {
  my %hash = @_;
  say keys %hash;
  say values %hash;
}

subname(
  "name", "junmajinlong",
  "age", 23,
);
```

## shift弹出参数
使用shift来弹出参数列表中的第一个，shell里也有类似功能
```
在子程序内部，下面两个操作是等价的：
shift @_;
shift;
```
## 传递引用与匿名对象

传递数组或hash，建议的方式是传递它们的引用。也可以传递匿名数组、匿名hash，它们本质上仍然是引用。
```
sub subname{
  my $arr_ref = shift;
  say $$arr_ref[0];
}

my @arr = qw(a b c);
subname(\@arr);

----------------------------------
sub subname{
  my $arr_ref = shift;
  my $hash_ref = shift;
  my $scalar = shift;
  
  say $$arr_ref[0];
  say $$hash_ref{name};
  say $scalar;
}

subname([qw(a b c)], {name=>'junma', age=>23}, 3333);
```
调用子程序时传递的参数，放入@_中的都是原始数据的别名引用：修改@_各元素的值，也将影响原始数据。即里面改了外面也变了
```
@_中的数据和原始数据是完全相同的，它们是别名关系。这种别名关系也存在于容器的元素中。

sub subname{
  $_[0]++;
  say $_[0];
}

my $a=33;
subname($a);
say $a;


sub subname{
  $_[0]++;
  say $_[0];
}

my @arr = (11,22,33);
subname(@arr);
say $arr[0];

```
如果想要让子程序内部的修改不影响原始数据，则应该将参数数据保存到变量中。
```
sub subname{
  my $a = shift;
  $a++;
  say $a;
}

my $x=33;
subname $x;  # 34
say $x;      # 33

数组也类似：

sub subname{
  my @arr = @_;
  $arr[0]++;
  say $arr[0];
}

my @arr=(11,22,33);
subname @arr;  # 12
say $arr[0];   # 11
```
## 检测原型

## 静态变量 state
state操作符功能类似于C里面的static修饰符，state关键字将局部变量变得持久。state也是词法变量，所以只在定义该变量的词法作用域中有效
```
use feature 'state';
 
sub PrintCount{
   state $count = 0; # 初始化变量
 
   print "counter 值为：$count\n";
   $count++;
}
 
for (1..5){
   PrintCount();
}

以上程序执行输出结果为：

counter 值为：0
counter 值为：1
counter 值为：2
counter 值为：3
counter 值为：4
```
* 注1：state仅能创建闭合作用域为子程序内部的变量
* 注2：state是从Perl 5.9.4开始引入的，所以使用前必须加上 use
* 注3：state可以声明标量、数组、哈希。但在声明数组和哈希时，不能对其初始化（至少Perl 5.14不支持）


# 匿名对象
可有构建匿名的对象，这样就没必要去为只用一两次的数组、hash去取名字，有时候取名是很烦的事。
* 使用中括号 [ ] 构建匿名数组
* 使用大括号 { } 构建匿名hash
* 不包含任何元素的 [ ] 和 { } 分别是匿名空数组、匿名空hash
## 构造匿名对象
在数组、hash中构建匿名数组：
```
my @name=('fairy',['longshuai','wugui','xiaofang']);
# 也可使用qw方式 my @name=('fairy',[qw(longshuai wugui xiaofang)]);
my %hash=('longshuai' => ['male',18,'jiangxi'],
        'wugui'     => ['male',20,'zhejiang'],
        'xiaofang'  => ['female',19,'fujian'],
       );

say "$name[1]";          # 输出ARRAY(0x...)
say "$hash{wugui}";      # 输出ARRAY(0x...)

say "$name[1][2]";
say "$hash{wugui}[1]";
```
在数组、hash中构建匿名hash：
```

my @name=(         # 匿名hash作为数组的元素
       {    # 第一个匿名hash
        'name'=>'longshuai',
        'age'=>18,
        'prov'=>'jiangxi',
       },
       {    # 第二个匿名hash
        'name'=>'wugui',
        'age'=>20,
        'prov'=>'zhejiang',
       },
       {    # 第三个匿名hash
        'name'=>'xiaofang',
        'age'=>19,
        'prov'=>'fujian',
       },
      );

my %hash=(          # 匿名hash作为hash的value
        'longshuai'=>{   # 第一个匿名hash
                      'gender'=>'male',
                      'age'   =>18,
                      'prov'  =>'jiangxi',
                     },
        'wugui'=>{    # 第二个匿名hash
                  'gender'=>'male',
                  'age'   =>20,
                  'prov'  =>'zhejiang',
                 },
        'xiaofang'=>{    # 第三个匿名hash
                     'gender'=>'female',
                     'age'   =>19,
                     'prov'  =>'fujian',
                    },
      );

say $name[2];       # 输出HASH(0x...)
say $hash{"wugui"};   # 输出HASH(0x...)
say "$name[2]{age}";
say "$hash{wugui}{prov}";
```
## 匿名对象的的引用
```
$aref = [ 1, "foo", undef, 13 ];  	   # $aref 保存了这个数组的'引用'

下面2句与上面类似，只是没有创建一个多余的数组变量
@array = (1, 2, 3);
$aref = \@array;

$href = { APR =>4, AUG =>8 };   	   # $href 保存了这个哈希的'引用'

可以创建空的
$aref2 = [];
$href2 = {};
```
## 解除匿名对象的引用
当输出匿名对象时，其实输出的是个引用。
```
可以始终用一个带有大括号的对象'引用'，来替换一个数组的名字

对数组@a操作    对'引用'$aref所指向的数组操作
@a              @{$aref}                		一个数组
reverse @a      reverse @{$aref}        		对一个数组做倒序排序
$a[3]           ${$aref}[3]             		数组中的一个成员
				$$aref[3]
$a[3] = 17;     ${$aref}[3] = 17        		对一个成员赋值

对哈希%a操作    对'引用'$href所指向的哈希操作
%h              %{$href}             			一个哈希
keys %h         keys %{$href}         			从哈希中将键取出来
$h{'red'}       ${$href}{'red'}       			哈希中的一个成员
$h{'red'} = 17  ${$href}{'red'} = 17  			对一个成员赋值

下面未测试：
遍历原始数组
		for my $element (@array) {
				...
		}
接着用'引用'替代数组名@array：
        for my $element (@{$aref}) {
           say $element;
        }
遍历原始哈希
        for my $key (keys %hash) {
          print "$key =>; $hash{$key}\n";
        }
然后用'引用'代替那个哈希的名字：
        for my $key (keys %{$href}) {
          print "$key =>; ${$href}{$key}\n";
        }
```
解除匿名数组对象，并获取匿名数组中的元素
```
my $aref=[ 
           'longshuai',
           'wugui'    ,
           'xiaofang' ,     
];
print "${$aref}[1]\n";

my $href={
           'longshuai'=> ['male',18,'jiangxi'],
           'wugui'    => ['male',20,'zhejiang'],
           'xiaofang' => ['female',19,'fujian'],      
          };
print "${$href}{wugui}[0]\n";



say "@{ ['longshuai','xiaofang','wugui'] }";   # 解除匿名对象的引用
say "@{ [qw(longshuai xiaofang wugui)] }";     # 解除匿名对象的引用
say "@{ [qw(longshuai xiaofang wugui)] }[1]";  # 获取匿名对象中的第二个元素
```
解除匿名hash对象，并获取匿名hash中的元素
```
my $ref_hash={   # 构造匿名hash，赋值给引用变量
           'longshuai'=> ['male',18,'jiangxi'],
           'wugui'    => ['male',20,'zhejiang'],
           'xiaofang' => ['female',19,'fujian'],      
          };

my @mykeys=keys %{ $ref_hash };  # 通过引用还原到匿名hash
say "@mykeys";                # 输出匿名hash中的键
say $ref_hash->{wugui}[2];    # 输出匿名hash中匿名数组的某个元素

-----------------------------

my @mykeys=keys %{   # 解除匿名hash
                 { # 构造匿名hash
                  'longshuai'=> ['male',18,'jiangxi'],
                  'wugui'    => ['male',20,'zhejiang'],
                  'xiaofang' => ['female',19,'fujian'],
                 }
               };
say @mykeys;

say %{   # 解除匿名hash
       { # 构造匿名hash
        'longshuai'=> ['male',18,'jiangxi'],
        'wugui'    => ['male',20,'zhejiang'],
        'xiaofang' => ['female',19,'fujian'],      
       },
     };
```
## 区分匿名{}与作用域{}
* 大括号前面加上+符号，即+{...}，表示这个大括号是用来构造匿名hash的
* 大括号内部第一个语句前，多使用一个;，即{;...}，表示这个大括号是一次性语句块
```
@{ +[qw(longshuai wugui)]}   # 匿名数组中括号前
@{ +$ref_arr }           # 数组引用变量前
%{ +$ref_hash }          # hash引用变量前
```
## autovivification特性
解除引用时，如果解除目标不存在，会自动补全结构中的元素内容  
https://www.cnblogs.com/f-ck-need-u/p/9718238.html?utm_medium=referral&utm_source=itdadao
```
use 5.010;
push  @{ $config{path} },'/usr/bin/perl';

say keys %config;        # 输出：path
say $config{path};       # 输出：ARRAY(0x...)
say $config{path}[0];    # 输出：/usr/bin/perl

perl在解除引用时 1.自建一个hash对象；2.自建hash对象中的一个元素；3.自建hash对象中某个元素的value部分。
-------------------------

push @name,"longshuai";
$person{name}="longshuai";

print "$name[0]\n";
my @keys = keys %person;
print "keys: @keys\n"; 

@{ $config{path} }[2]='/usr/bin/perl';
print "$config{path}\n";       # 输出：ARRAY(0x5571664403c0)
print "$config{path}[0]\n";    # 输出：空
print "$config{path}[1]\n";    # 输出：空
print "$config{path}[2]\n";    # 输出：/usr/bin/perl
print scalar @{$config{path}};   # 输出元素个数：3
```
# 特殊变量
```
Perl 语言中定义了一些特殊的变量，通常以 $, @, 或 % 作为前缀，例如：$_。
很多特殊的变量有一个很长的英文名，操作系统变量 $! 可以写为 $OS_ERROR。
如果你想使用英文名的特殊变量需要在程序头部添加 use English;。这样就可以使用具有描述性的英文特殊变量。
```
https://www.runoob.com/perl/perl-special-variables.html
## 默认参数变量 $_
对于需要参数的函数或表达式，但却没有给参数，这是将会使用perl的默认参数变量$_
```
$_="abcde";
print ;
```

# 进程管理
* 特殊变量 $$ 或 $PROCESS_ID 来获取进程 ID。
* %ENV 哈希存放了父进程，也就是shell中的环境变量，在Perl中可以修改这些变量。
## 反引号运算符
使用反引号运算符可以很容易的执行 Unix 命令。你可以在反引号中插入一些简单的命令。命令执行后将返回结果：
```
@files = `ls -l`;
foreach $file (@files){
   print $file;
}

输出
total 12
-rw-rw-r-- 1 huawei huawei 139 Dec  2 17:28 1.pl
-rw-rw-r-- 1 huawei huawei   0 Nov 29 08:49 23,
-rw-rw-r-- 1 huawei huawei  35 Dec  2 16:38 2.pl
-rw-rw-r-- 1 huawei huawei   0 Nov 29 08:49 junmajinlong,
-rw-rw-r-- 1 huawei huawei   0 Nov 29 08:49 male
-rw-rw-r-- 1 huawei huawei  13 Dec  2 16:56 tempCodeRunnerFile.pl
```
## system
可以使用 system() 函数执行 Unix 命令, 执行该命令将直接输出结果。默认情况下会送到目前Perl的STDOUT指向的地方，一般是屏幕。你也可以使用重定向运算符 > 输出到指定文件
```
$PDDATA = "我是 Perl 的变量";
system('echo $PDDATA');  # $PATH 作为 shell 环境变量
system("echo $PDDATA");  # $PATH 作为 Perl 的变量
system("echo \$PDDATA"); # 转义 $

输出
/home/huawei/pgdata/new
我是 Perl 的变量
/home/huawei/pgdata/new
```
## fork
fork() 函数用于创建一个新进程。在父进程中返回子进程的PID，在子进程中返回0。如果发生错误（比如，内存不足）返回undef，并将$!设为对应的错误信息。fork 可以和 exec 配合使用。exec 函数执行完引号中的命令后进程即结束。
```
if(!defined($pid = fork())) {
   # fork 发生错误返回 undef
   die "无法创建子进程: $!";
}elsif ($pid == 0) {
   print "通过子进程输出\n";
   exec("date") || die "无法输出日期: $!";
  
} else {
   # 在父进程中
   print "通过父进程输出\n";
   $ret = waitpid($pid, 0);
   print "完成的进程ID: $ret\n";
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
通过父进程输出
通过子进程输出
Thu Dec  2 17:34:35 CST 2021
完成的进程ID: 34217
```
使用发CHLD的信号，待细致研究
```
local $SIG{CHLD} = "IGNORE";
 
if(!defined($pid = fork())) {
   # fork 发生错误返回 undef
   die "无法创建子进程: $!";
}elsif ($pid == 0) {
   print "通过子进程输出\n";
   exec("date") || die "无法输出日期: $!";
  
} else {
   # 在父进程中
   print "通过父进程输出\n";
   $ret = waitpid($pid, 0);
   print "完成的进程ID: $ret\n";
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
通过父进程输出
通过子进程输出
Thu Dec  2 17:37:09 CST 2021
完成的进程ID: -1
```
## Kill
kill('signal', (Process List))给一组进程发送信号。signal是发送的数字信号，9为杀掉进程。  
kill('KILL', 进程号1, 进程号2...);
# 目录操作
```
use Cwd;
$dir = cwd();	# 效果同pwd
```
# 文件操作
```
say $0			# 全路径自身文件名
```
## 读取文件每一行
将文件作为perl命令行的参数，perl会使用<>去读取这些文件中的内容。
```
由于<>和<STDIN>读取文件、读取标准输入的时候总是自带换行符，很多时候这个自带的换行符都会带来格式问题。所以，有必要在每次读取数据时将行尾的换行符去掉，使用chomp即可

脚本内容：
foreach (<>){
    chomp;
    say "$_";
}

[huawei@n148 perl]$ perl 1.pl /etc/passwd
```
# 模块操作
```
[huawei@n148 perl]$ cpan -i Term::ANSIScreen	# 安装模块
```
# 命令行
其实就是一行式。perl命令行加上"-e"选项，就能在perl命令行中直接写perl表达式。如
```
echo "malongshuai" | perl -e '$name=<STDIN>;print $name;'
强烈建议"-e"后表达式使用单引号包围，而不是双引号。
```
