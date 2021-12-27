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
是变量名的前缀，包括$、@、%，在Perl中称为Sigil。作用是区分类型。
## 标量 $
标量是一种变量，只能保存单个值、单个字符串或单个数字，计算结果也是标量
```
$age = 25;             # 整型
$name = "runoob";      # 字符串
$salary = 1445.50;     # 浮点数
 
print "Age = $age\n";
print "Name = $name\n";
print "Salary = $salary\n";
```
## 数组 @
数组是一组有序排列的标量，索引从 0 开始。
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
__END__;	# 表示脚本的逻辑终止位置，告诉 Perl 忽略其后出现的一切字符。
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
## 交换2个变量
```
($a,$b)=qw/a1 2b/;
($a,$b)=($b,$a);
say "$a,$b";
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
## 关于true与false
Perl没有直接代表布尔值的false值和true值。代表布尔假的值有如下：

    数值0、0.0
    空字符串''、字符串"0"
    undef
    空列表，包括() ((())) ((),())
    空数组
    空hash
除以上代表布尔假的值之外，其余都是布尔真。

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
## 检测undef
```
my $test;
if (defined($test))
{
	print $test;
}
else{
	print "undef";
}
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

## 使用警告
```
use warnings;
```
## 环境变量 %ENV
```
while(my ($k, $v) = each %ENV){
  say "key: $k, v: $v";
}
```
# 引用
* 引用类似C语言的指针，是指向一个内存空间的地址
* 引用是一个标量类型，可以指向变量、数组、哈希甚至子程序，可以应用在程序的任何地方。  
* 在一个变量前加' \ '，就得到了这个变量的'引用'。  
* 可以通过引用查看变量所保存数据的内存地址与具体数据。  
* 根据引用获取其指向的原始数据叫解引用。解引用和定义引用的数据符号一致
* 比较引用时候就跟普通标量一样使用“==”即可
## 变量的引用
my $n = 'junma';  
my $n_ref = \$n;  
n_ref是一个标量变量，它保存变量n的引用，或者说，n_ref保存了变量n所指向字符串数据"junma"的内存地址。  
![](https://perl-book.junmajinlong.com/imgs/2021-01-25_19-55-59.png)
```
my $value = 10;
my $pointer = \$value;
print "Pointer Address $pointer of $value \n";
print "What Pointer *($pointer) points to $$pointer\n"; # 这里有打印地址与解引用

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Pointer Address SCALAR(0x110b8f8) of 10 
What Pointer *(SCALAR(0x110b8f8)) points to 10
```
## 语法糖
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
## 数组的引用
```
my @names =(77,88,99);
my $address = \@names;
print "$#names\t$address\n";	# 打印出最大索引与数组地址
print "@names\t@{$address}\n";	# 打印数组内容
print "$$address[1] # 通过引用来访问具体数据，第一种方式最简单
${$address}[1]
@$address[1]
@{$address}[1]
$address->[1]\n";	

sub listem{
   	my ($list) = shift;
   	print "$list\t@$list\n";	# 打印出地址（和外面的实参地址一致）与数组内容
	print "$$list[1]  # 解引用打印元素值，与上面的方式类似
${$list}[1]
@$list[1]
@{$list}[1]
$list->[1]\n";
}
listem($address);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
2       ARRAY(0x1be3928)
77 88 99        77 88 99
88 # 通过引用来访问具体数据，第一种方式最简单
88
88
88
88
ARRAY(0x1be3928)        77 88 99
88  # 解引用打印元素值，与上面的方式类似
88
88
88
88
```

## 哈希的引用
```
my %names=('key1' => 10, 'key2' => 20, 'key3' => 30);
my $address = \%names;
print $address, "\n"; # 打印地址 
print %$address, "\t", %names, "\n"; # 打印哈希内容，但都连一起了
print "$$address{key1}	
${$address}{key1}
$address->{key1}
";	# 通过引用来访问具体数据，第一种方式最简单

sub listem{
   	my ($hash) = shift;
   	print "$hash\n";	# 打印出地址（和外面的实参地址一致）
  	print "$$hash{key1}	
${$hash}{key1}
$hash->{key1}
";  # 解引用打印元素值，与上面的方式类似
}
listem($address);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
HASH(0x18b68f8)
key220key110key330      key220key110key330
10
10
10
HASH(0x18b68f8)
10
10
10
```
## 复杂结构的引用
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


## 函数的引用

### 简单的使用
```
sub PrintHash{
   my $refhash = shift;
   print Dumper($refhash);
}
my %hash = ('name' => 'runoob', 'age' => 3);
PrintHash(\%hash);

my $cref = \&PrintHash;	# 取函数地址
&{$cref}(\%hash);		# 3种调用函数
&$cref(\%hash);
$cref->(\%hash);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = {
          'name' => 'runoob',
          'age' => 3
        };
$VAR1 = {
          'name' => 'runoob',
          'age' => 3
        };
$VAR1 = {
          'name' => 'runoob',
          'age' => 3
        };
$VAR1 = {
          'name' => 'runoob',
          'age' => 3
        };
```
函数指针放到数组里循环调用
```
sub PrintHash{
   print Dumper(shift);
}
sub PrintHash2{
   print Dumper(shift);
}
my %hash = ('name' => 'runoob', 'age' => 3);
for my $pfun (\&PrintHash, \&PrintHash2){
	$pfun->(\%hash);	# 类似上面的案例3种效果一样
	&$pfun(\%hash);
	&{$pfun}(\%hash);
}
```
函数指针放到哈希里循环调用
```
sub PrintHash{
   print Dumper(shift);
}
sub PrintHash2{
   print Dumper(shift);
}

my %hash = (pfun1=>\&PrintHash, pfun2=>\&PrintHash2);
for my $pf (qw\pfun1 pfun2\){
	$hash{$pf}->('sss');
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = 'sss';
$VAR1 = 'sss';
```
互相打招呼，2层循环，从hash里取元素对应的函数指针进行调用，参数也是个函数指针，会自动转为字符串
```
sub fun1{
   print "this is fun1, param is", shift, "\n";
}
sub fun2{
   print "this is fun2, param is", shift, "\n";
}
sub fun3{
   print "this is fun3, param is", shift, "\n";
}
my %hash = (pfun1=>\&fun1, pfun2=>\&fun2, pfun3=>\&fun3);
my @arr=sort keys %hash;

for my $k1 (@arr){
	for my $k2 (@arr){
		$hash{$k1}->($k2) unless $k1 eq $k2;
	}
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
this is fun1, param ispfun2
this is fun1, param ispfun3
this is fun2, param ispfun1
this is fun2, param ispfun3
this is fun3, param ispfun1
this is fun3, param ispfun2
```
### 指向匿名函数
```
my %hash = (
	pfun1=>sub{print "this is fun1, param is", shift, "\n";}, 
	pfun2=>sub{print "this is fun2, param is", shift, "\n";}, 
	pfun3=>sub{print "this is fun3, param is", shift, "\n";},
);
my @arr=sort keys %hash;

for my $k1 (@arr){
	for my $k2 (@arr){
		$hash{$k1}->($k2) unless $k1 eq $k2;
	}
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
this is fun1, param ispfun2
this is fun1, param ispfun3
this is fun2, param ispfun1
this is fun2, param ispfun3
this is fun3, param ispfun1
this is fun3, param ispfun2
```
### 指向（匿名）回调方式
递归遍历文件夹
```
use File::Find;
my @dir=qw/./;
sub fun{
	print "$File::Find::name found\n";
}
find(\&fun,@dir);


与上面的结果一样，但更现代语言一些
find(sub{
	print "$File::Find::name found\n";
},@dir);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
. found
./.vstags found
./junmajinlong, found
./tempCodeRunnerFile.pl found
./1.pl found
./2.pl found
./err.txt.bak found
./fred found
./rocks.txt found
./logf found
./mulu.txt found
./ok.txt found
./err.txt found
./5, found
./.vscode found
./.vscode/launch.json found
./123 found
./a found
./a/c found
./a/b found
./a/b/1 found
./a/b/aa found
./a/b/c found
```
## 闭包
### 简单应用
```
sub how_many {       # 定义函数
    my $count=2;     # 词法变量$count
    return sub {print ++$count,"\n"};  # 返回一个匿名函数，这是一个匿名闭包
}

my $ref=how_many();    # 将闭包赋值给变量$ref

how_many()->();     # (1)调用匿名闭包：输出3
how_many()->();     # (2)调用匿名闭包：输出3
$ref->();           # (3)调用命名闭包：输出3
$ref->();           # (4)再次调用命名闭包：输出4

```
# 流程控制

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


say $_ foreach (qw\a b c\);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
a
b
c
[huawei@n148 perl]$ 
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
单引号内除了'和\要加\，且只能转这2个，其余都不转义，都原样输出
```
my $a = 10;
say ' $a \' "  \\  \t  \n';    # 单引号
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
 $a ' "  \  \t  \n
[huawei@n148 perl]$ 
```
双引号的都转义
```
my $a = 10;
say " $a ' \"  \\  \t  \n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
 10 ' "  \        

[huawei@n148 perl]$ 
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
Perl使用点.来串联字符串。Perl使用x来重复字符串指定次数，如果x重复次数是小数，则截断为整数，如果x是0，则清空字符串。
```
my $a = 10;
$a.='abc';
say "$a";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
10abc
[huawei@n148 perl]$ 

------------------------------
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
## 去掉行尾换行 chomp
移除尾部换行符，如果尾部没有换行符，则不做任何事。实际上，chomp移除的是$/变量值对应的字符，该变量表示输入记录分隔符，默认为换行符，因此默认移除字符串尾部换行符。注意：
* chomp不能对字符串字面量进行操作
* chomp可以对左值进行操作
* chomp可以操作列表，表示移除列表每项元素的尾部换行符
* chomp也可以操作hash，表示移除每项value的为尾部换行符
* chomp返回成功移除的总次数   
    1 对于字符串，如果有换行符，则返回1，否则返回0，因此可通过该返回值判断字符串是否有结尾换行符  
    2 对于列表，可以通过返回值来判断总共操作了多少行数据
```
chomp(my $test=<STDIN>);
print $test;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
123
123[huawei@n148 perl]$ 


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


## 翻转字符串 reverse
见列表的reverse
## 取子串、替换子串 substr
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

my $str="love your everything";
substr($str,5,4) = "fairy's";
say $str;       # 源字符串已被替换：love fairy's everything

substr($str,0,4) = 'i hate';   # 也可以这样替换原字符串
say $str;

```

## 字符串与数组互换 split、join
使用给定分隔符将字符串划分为列表，分隔符支持使用正则表达式。在列表上下文，返回划分后得到的列表，在标量上下文，返回划分后列表的元素数量。
```
split /PATTERN/,EXPR,LIMIT
split /PATTERN/,EXPR
split /PATTERN/
split

参数说明：
* PATTERN：分隔符，默认为空格。
* EXPR：指定字符串数。
* LIMIT：如果指定该参数，则返回该数组的元素个数。

my @fields = split; #等效于split /\s+/, $_;	
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

my $some_input = "This is a \t     test.\n";
my @args = split /\s+/, $some_input; #得到("This", "is", "a", "test.")

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


## 字符索引 index rindex
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
* 数组是存储列表的变量，列表元素可以是不同类型。数组变量以 @ 开头。访问数组元素使用 $ + 变量名称 + [索引值] 格式来读取
* 数组名指向堆中的连续内存，堆中的连续内存里的每个地址再分别指向具体元素的地址，如下图。数组元素可以异质。  
![](https://perl-book.junmajinlong.com/imgs/1610678693613.png)  
* 数组的常见操作还有：each、pop、push、shift、unshift、keys、values、splice。  
## 创建
数组变量以 @ 符号开始，元素放在括号内
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

也支持..构建方式
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

## 长度与最大索引 $#arr
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

还可以使用printf格式化输出，比较帅
my @arr = (11, 22, 33, 44);
printf "the items are :\n" . ("%10s\n" x @arr), @arr;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
the items are :
        11
        22
        33
        44
[huawei@n148 perl]$ 

------------------------------------


use Data::Dumper;
my @arr = (11, 22, 33);
my $address = \@arr;
print Dumper($address);
print Dumper(\@arr);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          11,
          22,
          33
        ];
$VAR1 = [
          11,
          22,
          33
        ];
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
* 数组切片可以内插到字符串中，而列表切片则不能
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
## 遍历与each

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
使用each返回索引与v
```
my @h = qw(k1 v1 k2 v2 k3 v3);
while(my ($idx, $v) = each @h){
  say "idx: $idx, v: $v";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
idx: 0, v: k1
idx: 1, v: v1
idx: 2, v: k2
idx: 3, v: v2
idx: 4, v: k3
idx: 5, v: v3
[huawei@n148 perl]$ 
```
## 默认迭代变量 $_
控制变量是可以省略的，此时将使用Perl的默认标量变量 $ _。Perl的很多操作都允许省略操作目标，此时将使用默认变量$_作为这些操作的操作目标
```
for(11,22,33){ say $_; }
for $_ (11,22,33) { say $_; }
```


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
## 清空数组 ( )
```
my @h = qw(k1 v1 k2 v2 k3 v3);
@h=();
while(my ($idx, $v) = each @h){
  say "idx: $idx, v: $v";
}
```
## 移除替换指定元素 splice
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
## 排序 sort
数组内建支持。案例见列表sort
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
## 查找
```
$,=',';
sub find{
	my ($what,@arr)=@_;
	foreach(0..$#arr) {return $_ if $what == $arr[$_];}
	-1;
}
say find(3, qw/1 2 3 4 5/);
```
## 二维数组
使用引用方式才是正确姿势，见下案例
```
my @array1=("a1","b1","c1","d1");
my @array2=("a2","b2","c2","d2");
my @array3=("a3","b3","c3","d3");
my @array_2d=(@array1,@array2,@array3);	# 将三个数组连起来，还是一维的
print "@array_2d";	# 打印出地址
print "\n";
print "$array_2d[1]\t$array_2d[7]";	# 打印元素具体值，貌似仅能按照一维方式，二维方式见下面
print "\n";
##二维数组的正确使用方式##############
@array_2d=(\@array1,\@array2,\@array3);	# 对array_2d重新赋值，3个元素分别是地址
print "@array_2d";	# 打印数组内容。因为存的是地址，所以打印出3个地址
print "\n";
print "$array_2d[1]";	# 打印元素值，值是个地址
print "\n";
print "$array_2d[0][1]\t$array_2d[1][3]";	# 访问二维数组元素的正确姿势
print "\n";


[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
a1 b1 c1 d1 a2 b2 c2 d2 a3 b3 c3 d3
b1      d2
ARRAY(0xba48f8) ARRAY(0xba48b0) ARRAY(0x11d65d0)
ARRAY(0xba48b0)
b1      d2
```
# 列表
* 是标量的有序集合，列表指的是数据
* Perl中的列表不是数据类型，而是Perl在内部用来临时存放数据的一种方式，只能由Perl自行维护。
* 列表临时保存在栈中，当使用了列表数据后，这些列表数据就会出栈
* 可以将Perl列表看作是一种特殊的底层可迭代对象，它看起来像数组，但不是数组。
```
my @arr = (11,22,33);  # 数组arr的元素来自于列表
```
* push与pop这类仅用于数组，元素排序、元素筛选、迭代遍历等操作，才用于列表
* 列表常见操作包括：grep、join、map、reverse、sort、unpack、x操作符执行列表重复，等等
* 标准库List::Utils中也提供了很多常见的列表操作，如reduce、first、any、sum、uniq、shuffle等。

## 列表直接量
向下面()里的即是
```
my $a=10;
my $b=30;
$, = ", "; 
say (1,2,'xxx',3..$a, $a+$b);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
1, 2, xxx, 3, 4, 5, 6, 7, 8, 9, 10, 40
```
## 列表赋值
注意这里特指等会右边的部分
```
1 给2个标量赋值
($a,$b)=qw/a1 2b/;
say "$a,$b";

2 给数组赋值
my @a;
($a[0],$a[1])=qw/a1 2b/;
$,=',';
say @a;
```
## 构建列表 qw
用qw定义单引号包围的元素列表。qw中使用空格分隔各元素，如果某个元素包含空格，则不要用qw。qw（）可以替换为其他成对的符号，这一点和q或qq的用法完全一致。qw[]、qw!!、qw%%
```
my @array = qw(a b
ccc    11);
$, = ", "; 
say @array;

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
a, b, ccc, 11
```
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
从列表中筛选符合条件的元素，返回列表（结果赋值给数组）或count（结果赋值给标量）。grep会迭代所有，效率并不高。{}里条件为t则将本次迭代的元素放进返回值列表中。
```
my @nums = (11,22,33,44,55,66);
my @odds = grep {$_ % 2} @nums;   # 取奇数
my @evens = grep {$_ % 2 == 0} @nums;  # 取偶数
print Dumper(\@odds);
print Dumper(\@evens);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          11,
          33,
          55
        ];
$VAR1 = [
          22,
          44,
          66
        ];
```
当条件只有一条语句或一个表达式时可以去除{}
```
换一种迭代方式 0..$#arr

my @arr = (1..10);
my @arr2=grep 0==$arr[$_]>5, 0..$#arr;
print Dumper(\@arr2);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          0,
          1,
          2,
          3,
          4
        ];
```
精简版的奇数与偶数
```
print Dumper([grep $_ % 2, (11,22,33,44,55,66)]);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          11,
          33,
          55
        ];

print Dumper([grep $_ % 2 == 0, (11,22,33,44,55,66)]);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          22,
          44,
          66
        ];
```
grep在迭代列表各元素时，$ _ 是各元素的别名引用，在代码块中修改$_，也将影响到源列表，也因此会影响返回值列表。
```
my @nums = (11,22,33,44,55,66);
my @arr = grep {$_++; $_ % 2} @nums;
say "@arr";     # 23 45 67
say "@nums";    # 12 23 34 45 56 67

```
模拟grep的基本功能实现
```
sub grepfile{
	my $file=getfullpathfile(shift);
	my $reg = shift;
	my $counter=0;
	open my $FILE, '<', $file or die "Can't open file: $file"; 
	my $counter = grep /$reg/i, <$FILE>;
	close $FILE or die "Can't close file: $file"; 
	$counter;
}
```
正则过滤出结尾4的数字
```
$, = "; ";
my @arr=grep /4$/, (1..100);
say @arr;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
4; 14; 24; 34; 44; 54; 64; 74; 84; 94
```
回调形式
```
sub fun{
	my $v=shift;
	return $v if $v =~ /4$/;
}

$, = "; ";
my @arr=grep fun($_), (1..100);
say @arr;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
4; 14; 24; 34; 44; 54; 64; 74; 84; 94
```
匿名函数形式
```
$, = "; ";
my @arr=grep {
	$_ if ($_ =~ /4$/);
} (1..100);
say @arr;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
4; 14; 24; 34; 44; 54; 64; 74; 84; 94
```
查找指定元素都在哪些数组里
```
my %h=(
	'k1'=>[qw/a b c d e/],
	'k2'=>[qw/a c e f h/],
	'k3'=>[qw/a b d g x/],
);
my @all=grep{
	my @items=@{$h{$_}};
	grep $_ eq 'b', @items;
}keys %h;
print Dumper(\@all);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          'k1',
          'k3'
        ];
```
## map
map迭代列表的每个元素，并将表达式或语句块中返回的值放进一个列表中，最后返回这个列表。也可生成哈希。也是使用函数或匿名函数作为参数（见grep案例）
```
my @chars = map(chr, (65..70));
say "@chars";  # A B C D E F
my @chars = map($_, (65..70));
say "@chars";	# 65 66 67 68 69 70

my @arr = map { $_ * 2 } (1..5);
say "@arr";  # 2 4 6 8 10
```
也可以使用{}进行判断过滤
```
过滤出数字多于1位的

my @buildnums = ('R010','T230','W11','F56','dd1');
my @nums = map{/(\d{2,})/} @buildnums;
foreach my $num (@nums){
  print "$num \n"
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
010 
230 
11 
56 


过滤出数字大于45的

my @nums = (11,22,33,44,55,66);
my @numbers = map { $_ > 45 ? $_ : ()} @nums;
print Dumper(\@numbers);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          55,
          66
        ];

```
当语句块中只有一条语句时，可使用表达式写法。
```
my @arr = map $_*2, (1..5);
say "@arr";  # 2 4 6 8 10
```
这是个错误的使用方法，会返回undef的内容
```
my @arr = (11,22,33,44,55);
my @evens = map {$_ if $_%2==0} @arr;
print Dumper(\@evens);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          '',
          22,
          '',
          44,
          ''
        ];
```
map允许在一个迭代过程中保存多个元素到返回列表中。
```
my @name=qw(ma long shuai);
my @new_names=map {$_,$_ x 2} @name;
say "@new_names";  # ma mama long longlong shuai shuaishuai
```
一句话的样式
```
print "Some powers of two are:\n", map "\t" . (2 ** $_) . "\n", 0..15;

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Some powers of two are:
        1
        2
        4
        8
        16
        32
        64
        128
        256
        512
        1024
        2048
        4096
        8192
        16384
        32768
[huawei@n148 perl]$ 
```
生成到哈希里
```
my @name=qw(ma long shuai);
my %new_names=map {$_,$_ x 2} @name;
while(my ($k, $v) = each %new_names){
  say "key: $k, v: $v";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
key: shuai, v: shuaishuai
key: ma, v: mama
key: long, v: longlong
```
数组内的每个元素都生成一对kv
```
my %h=(
	'k1'=>[qw/a b c d e/],
	'k2'=>[qw/a c e f h/],
	'k3'=>[qw/a b d g x/],
);

my @pairs=map{
	my $k=$_;
	my @items=@{$h{$k}};
	map[$k=>$_], @items;
}keys %h;
print Dumper(\@pairs);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = [
          [
            'k2',
            'a'
          ],
          [
            'k2',
            'c'
          ],
          [
            'k2',
            'e'
          ],
          [
            'k2',
            'f'
          ],
          [
            'k2',
            'h'
          ],
          [
            'k1',
            'a'
          ],
          [
            'k1',
            'b'
          ],
          [
            'k1',
            'c'
          ],
          [
            'k1',
            'd'
          ],
          [
            'k1',
            'e'
          ],
          [
            'k3',
            'a'
          ],
          [
            'k3',
            'b'
          ],
          [
            'k3',
            'd'
          ],
          [
            'k3',
            'g'
          ],
          [
            'k3',
            'x'
          ]
        ];
```
## 排序 sort
sort用于对列表元素进行排序，返回排序后的列表。算法是可以自定义的，下面的案例里$a<=>$b这样的逻辑可以继续扩展（具体见perl进阶第10章）
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
## 翻转列表 reverse
可以操作列表和字符串，不会覆盖原内容，返回翻转后的结果
```
my @arr1 = qw(aa bb cc dd);
@arr1=reverse @arr1;
say @arr1;

my $s='abc';
my $x=reverse $s;
say $s;
say $x;

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
ddccbbaa
abc
cba
[huawei@n148 perl]$ 
```
# 哈希
* hash结构中的key是唯一的，是字符串，值随意
* hash结构不保证键值对的顺序
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
## 判断空哈希
```
my %h = qw(k1 v1 k2 v2 k3 v3);
if (%h)
{
	say "hash is not empty";
}
else{
	say "hash is empty";
}
```
## 获取哈希大小
先获取 key 或 value 的所有元素数组，再计算数组元素多少来获取哈希的大小
```
%data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
@keys = keys %data;
$size = @keys;
print "1 - 哈希大小: $size\n";
```
## 打印
```
use Data::Dumper;
my %names=('key1' => 10, 'key2' => 20, 'key3' => 30);
my $address = \%names;
print Dumper($address);
print Dumper(\%names);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
$VAR1 = {
          'key2' => 20,
          'key1' => 10,
          'key3' => 30
        };
$VAR1 = {
          'key2' => 20,
          'key1' => 10,
          'key3' => 30
        };
```
## 遍历与each
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
## 检测k是否存在 exists
exists用来判断hash结构中是否存在某个key
```
my %data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
if( exists $data{'facebook'} ){
   print "facebook 的网址为 $data{'facebook'} \n";
}
else
{
   print "facebook 键不存在\n";
}
```
## 检测v是否有值
```
my %data = qw/a 1 b 2 c 3 d/;
foreach my $item (sort keys %data){
	if ($data{$item}){
		print "$item has $data{$item}\n";
	}else
	{
		print "$item not has v\n";
	}
}
```
## 匿名哈希
```
一个数组里有2个元素，每个元素是个哈希的引用

my $ref_data1 = {'google', 'google.com', 'runoob', 'runoob.com', 'taobao', 'taobao.com'};
my $ref_data2 = {k1=>'v1', k2=>'v2', k3=>'v3'};
print Dumper($ref_data1);
print Dumper($ref_data2);
my @arr={$ref_data1, $ref_data2};

下面才是一步到位的姿势
my @arr={
	{'google', 'google.com', 'runoob', 'runoob.com', 'taobao', 'taobao.com'},
	{k1=>'v1', k2=>'v2', k3=>'v3'},
};
```

## 统计单词数量
统计数组内的单词
```
my %count;
$count{$_}++ foreach (qw\a b c a c d e a d\);
while(my ($k, $v) = each %count){
  say "key: $k, v: $v";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
key: e, v: 1
key: c, v: 2
key: a, v: 3
key: b, v: 1
key: d, v: 2
[huawei@n148 perl]$ 
```
## 加载文件统计，剔除了非单词\W
```
my $total;
my $valid;
my %count;
while(<>){
	foreach (split){
		$total++;
		next if /\W/;
		$valid++;
		$count{$_}++;
	}
}
say "total: $total, valid: $valid";
while(my ($k, $v) = each %count){
  say "key: $k, v: $v";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl" err.txt
```
## 案例：文件转目录
mulu.txt 文件内容类似这样
```
第1章 简介 1
	1．1 背景知识 2
	1．2 strict和warnings 2
	1．3 Perl v5．14 3
	1．4 关于这些脚注 4
	1．5 关于后续的练习 4
	1．6 获取帮助的方式 5
	1．7 如果是一个Perl课程讲师 5
	1．8 练习 6
第2章 使用模块 7
	2．1 标准发行版 7
	2．2 探讨CPAN 8
	2．3 使用模块 9
	2．4 功能接口 10
	2．5 面向对象的接口 11
```
读文件转哈希，k是一级标题，v是数组（当前一级下的所有二级标题）
```
use Data::Dumper;

my $bigtitle;
my %contents;
while(<>){
	if(/^(\S.*)/){	# 判断是1级标题
		$bigtitle=$1;
		$contents{$bigtitle}=[] unless $contents{$bigtitle}; # 如不存在此k则创建k，v是匿名数组，准确的说是执行空数组的引用，其实这句也可以不要，perl会自动创建出来。。。
	}elsif(/^\s+(\S.*)/){	# 是二级标题
		die unless defined $bigtitle;	# k不在则die
		push @{$contents{$bigtitle}},$1; # 将二级标题插入到对应的v（是个数组）里
	}else{
		die;
	}
}
print Dumper(\%contents);

[huawei@n148 perl]$ perl 1.pl mulu.txt
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
* $ !变量代表errmsg，但并非所有的错误都有$ !，只有涉及到系统调用且出错时才有。
* die和warn默认会输出程序名称和行号，但如果在错误消息后面加上\n换行符，则不会报告程序名称和行号。

```
程序中变量 $! 返回了错误信息，以及带不带\n的差异

if(chdir("/etc1")){
}else{
   die "Error: 无法打开文件 - $!";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Error: 无法打开文件 - No such file or directory at /home/huawei/playground/perl/1.pl line 25.
[huawei@n148 perl]$

if(chdir("/etc1")){
}else{
   die "Error: 无法打开文件 - $!\n";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Error: 无法打开文件 - No such file or directory
[huawei@n148 perl]$ 

--------------------------------------------------------
chdir("/etc1") || die "Error: 无法打开文件 - $!";



unless(chdir("/etc1")){
   die "Error: 无法打开目录 - $!";
}



die "Error: 无法打开目录!: $!" unless(chdir("/etc1"));



chdir('/etc1') or warn "无法切换目录";
```

## auotdie
导入包后不必每次都or dir那样去写了，大部分io错误都能自动捕获到然后die
```
use autodie;
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
总共有4种类型的错误是eval无法捕获的：为了安全不要在程序里用eval，只有在相当关注安全时才应该使用eval。
* 第一种：是出现在源代码中的语法错误，比如没有匹配的引号，忘写分* 号，漏写操作符，或者非法的正则表达式等。
* 第二种：是让perl解释器本身崩溃的错误，比如内存溢出或者受到无法接* 管的信号。
* 第三种：是eval无法捕获的错误是警告，不管是由用户发出的(通过warn函* 数)，还是perl自己内部发出的(通过打开-W这个命令行选项，或者使用* use warning编译指令。要让eval捕获警告专门的一套机制，请参考perl文* 档中_WARN_伪信号相关的内容)
* 第四种：是exit操作符会立即终止程序运行，就算从eval块内部的子程序* 来调用它
## Try::Tiny
提供了try、catch、finally的语法
# 正则
默认搜索对象是$_，Perl的正则表达式的三种形式：
* 匹配：m//
* 替换：s///
* 转化：tr///
	
这三种形式一般都和 =~ 或 !~ 搭配使用， =~ 表示相匹配，!~ 表示不匹配。
## 元字符 metacharacter
元字符是不代表自身原有含义的字符。前面加上\，元字符就失去了它原有的属性。  
‘.’ 与 ‘\’是元字符，默认‘.’不匹配\n（如需匹配使用s见下文），如需在模式中匹配到‘.’或‘\’只要在前面在加上‘\’即可

	字符类：单字符与数字
		. 匹配除换行符之外的任意字符
		[a-z0-9] 匹配集合中任意单字符
		[^a-z0-9] 匹配不在集合中的任意单字符
		1[0-3] 匹配10到13，不能是[10-13]
		\d 匹配单个数字，即[0-9]
		\D 匹配非数字字符，等效于[^0-9]，即非0-9
		\w 匹配字母数字，等效于[A-Za-z0-9]
		\W 匹配非字母数字，等效于[^A-Za-z0-9]
	字符类：空白字符
		\s 匹配空白字符，如空格、制表符和换行符
		\S 匹配非空白字符
		\n 匹配换行符
		\r 匹配回车符
		\t 匹配制表符
		\f 匹配进纸符
		\b 匹配退格符
		\0 匹配空值字符
	字符类：锚定字符
		\b 匹配字边界（不在 [] 中时）
		\B 匹配非字边界
		^ 匹配行首
		$ 匹配行尾
		\A 匹配字符串开头
		\Z 匹配字符串或行的末尾
		\z 只匹配字符串末尾
		\G 匹配前一次 m//g 离开之处
	字符类：重复字符
		x? 匹配 0 或 1 个 x
		x* 匹配 0 或多个 x
		x+ 匹配 1 或多个 x
		(xyz)+ 匹配 1 或多个模式 xyz
		x(m,n) 匹配 m 到 n 个 x 组成的值
	字符类：替换字符
		was|were|will 匹配 was、were、will 之一
	字符类：记忆字符
		(stirng) 用于反向引用（详见示例 9.38 和 9.39）
		\1 或 $1 匹配第一组括号 a
		\2 或 $2 匹配第二组括号
		\3 或 $3 匹配第三组括号
	字符类：其他字符
		\12 匹配八进制数，直到 \377
		\x811 匹配十六进制数值
		\cX 匹配控制字符。譬如 \cC 指的是 <Ctrl>-C；\cV 指的是 <Ctrl>-V
		\e 匹配 ASCII 编码中的 ESC 符（取消），而非反斜杠
		\E 标识使用 \U、\L 或 \Q 的大小写更改操作的结束位置
		\l 只小写下一个字符
		\L 小写字符，直到字符串末尾或碰到 \E
		\N 匹配已命名的字符，如 \N{greek:Beta}
		\p{PROPERTY} 匹配拥有已命名属性的任意字符，譬如 \p{IsAlpha}/
		\p{PROPERTY} 匹配不带已命名属性的任意字符
		\Q 引用 \E 之前的元字符
		\u 只大写下一个字符
		\U 大写字符，直到字符串末尾或碰到 \E
		\x{NUMBER} 匹配以十六进制形式给出的 Unicode 编码 NUMBER
		\X 匹配 Unicode 编码“组合字符序列”字符串
		\[ 匹配元字符
		\\ 匹配反斜杠

## POSIX 字符类
Perl5.6 引入了 POSIX 字符类。是一种保障程序间跨平台可移植性的工业标准。前者介绍的表示方法只受到 ASCII 字符编码的影响，而这里则可以表示其他语言的相应字符。简单来说就是这个更全面正专业。  

	方括号类 	含 义
	[:alnum:] 	字母数字型字符
	[:alpha:] 	字母型字符
	[:ascii:] 	码值在 0 到 127 之间的字符
	[:cntrl:] 	控制字符
	[:digit:] 	数字字符，即 0-9 或者 \d
	[:graph:] 	除字母数字或标点符号之外的非空字符（即非空格、控制字符等）
	[:lower:] 	小写字母
	[:print:] 	与 [:graph:] 相似，但包含空格字符
	[:punct:] 	标点符号字符
	[:space:] 	所有空白字符（换行符、空格符、制表符等）
	[:upper:] 	大写字母
	[:word:] 	任意字母数字或下划线字符 a
	[:xdigit:] 	十六进制数字中允许的阿拉伯数字（0-9a-fA-F）

* [[:space:]] 表示所有空白的字符
* [^[:space:]] 表示所有非空白的字符
* 注意上面外层要再加[]，以及取反的方法
```
获取名字案例

任意一个大写字母 [[:upper:]]，随后是一到多个（＋）字母字符 [[:alpha:]]，一个空格，后则是一个大写字母与一到多个小写字母。

while(<DATA>){
	print if /[[:upper:]][[:alpha:]]+ [[:upper:]][[:lower:]]+/;
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Betty Boop
Karen Evich
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Betty Boop
Karen Evich
```
## 字符集合与简写形式
* /[0-9]/之类的方式可能会覆盖不全，建议使用简写\d代表数字
* \s匹配n个空白
* [a-zA-Z] [^adg] 匹配其中的一个，或除其中的一个
* \h	匹配水平空白符
* \v	匹配垂直空白符	\h+\v = \p{Space}
* \R	匹配任意一种断行符，如\r\n还是\n
* \w	匹配任意字符0-9a-zA-Z,下划线也是的
* 还有很多...

反义简写:
* [^\d] = [\D]
* [^\w] = [\W]
* [^\s] = [\S]
* [\d\D]	匹配任意字符，包括换行
* [^\d\D]	什么都不匹配

## 重复模式
重复模式匹配元字符，也叫限定符（quantifier）。

### 贪婪匹配
限定符都是贪婪（greedy）的，它们会从左到右搜索能够匹配的最长字符集，并获得最后一个满足条件的字符。

		贪婪元字符		匹配项

		x? 			匹配 0 至 1 个 x 组成的具体值，0或1次
		()			对字符串分组
		(xyz)? 		匹配 0 至 1 个 xyz 模式组成的具体值
		x* 			匹配 0 至多个 x 组成的具体值，>=0次
		(xyz)* 		匹配 0 至多个 xyz 模式组成的具体值
		x+ 			匹配 1 至多个 x 组成的具体值，>=1次
		(xyz)+ 		匹配 1 至多个 xyz 模式组成的具体值，连续>=1个xyz的字符串
		x{m,n} 		匹配多于 m 个且少于 n 个的 x 所组成的具体值，匹配x m到n次，包含m与n
		(fred){3,}	匹配fred 3到无限次
		5{3}		匹配正好三个连续的5，注意是正好，不多不少
		.*			匹配任意字符串 
### 非贪婪匹配
如需关闭贪婪性，只需在贪婪限定符后面加上问号便可。这样就能让搜索操作在碰到第一个匹配时就告一段落，而不是直到最后一个匹配

		关闭贪婪性	匹配项		

		x?? 		匹配由 0 至 1 个 x 组成的具体值
		(xyz)?? 	匹配由 0 至 1 个 xyz 模式组成的具体值
		x*? 		匹配由 0 至多个 x 组成的具体值
		(xyz)*? 	匹配由 0 至多个 xyz 模式组成的具体值
		x+? 		匹配由 1 至多个 x 组成的具体值
		(xyz)+? 	匹配由 1 至多个 xyz 模式组成的具体值
		x{m,n}? 	匹配由多于 m 个且少于 n 个的 x 所组成的具体值
		x{m}? 		匹配由至少 m 个 x 所组成的具体值
		x{m,}? 		匹配至少 m 次

演示如下
```
$_="abcdefghijklmnopqrstuvwxyz";
s/[a-z]+/XXX/;
print $_, "\n";
$_="abcdefghijklmnopqrstuvwxyz";
s/[a-z]+?/XXX/;
print $_, "\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
XXX
XXXbcdefghijklmnopqrstuvwxyz

-----------------------
# A greedy quantifier
my $string="I got a cup of sugar and two cups of flour from the cupboard.";
$string =~ s/cup.*/tablespoon/;
print "$string\n";
# Turning off greed
$string="I got a cup of sugar and two cups of flour from the cupboard.";
$string =~ s/cup.*?/tablespoon/;
print "$string\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
I got a tablespoon
I got a tablespoon of sugar and two cups of flour from the cupboard.
```
## 条件修饰符 if、unless
可以用if、unless进行模式匹配
* Expression2 if Expression1;  
  如果 Expression1 表达式为真，则执行 Expression2 表达式的内容。

* Expression2 unless Expression1;   
  如果 Expression1 表达式为假，则执行 Expression2 表达式的内容。
```
my $x = 5;
say $x if $x == 5;	# 可以这样做普通的if判断

$_ = "xabcy\n";
print if /abc/;		# 可以使用正则的匹配，默认打印$_

$_ = "I lost my gloves in the clover.";
print "Found love in gloves!\n" if /love/;		# 类似上句效果


while(<DATA>){
	print unless /Norma/; 		# 可以使用正则的匹配
}

__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich
```
## 循环修饰符 while、until、foreach
* Expression2 while Expression1;  
  只要第一个表达式为真，while 循环修饰符便会重复执行第二个表达式。
* Expression2 until Expression1;  
  只要第一个表达式值为假，until 循环修饰符便会重复执行第二个表达式。
* foreach   
  foreach修饰符会逐个判断列表中每个元素的值，并通过标量 $_依次引用各个列表元素。

```
my $x=1;
# 下面2种方式结果一样
print $x++,"\n" while $x != 5; # 当 $x 不等于 5 时，Perl 打印 $x 的值。
print $x++,"\n" until $x == 5; # 反复打印 $x 的值，直到$x==5为真退出循环。

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/tempCodeRunnerFile.pl"
1
2
3
4


----------------------------
my @alpha=(a .. z, "\n");	# 将数组 @alpha 赋值为一组小写字。
print foreach @alpha;	# 逐个地将列表每一项赋予 $_变量，直到遍历完列表中所有的项。

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
abcdefghijklmnopqrstuvwxyz
```

## 交替（即或 | ）
交替特性允许正则表达式中出现能从多种可能中选中一种的模式匹配
```
while(<DATA>){
	print if /Steve|Betty|Jon/;
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jonathan DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Steve Blenheim
Betty Boop
Jonathan DeLoach
```
## 分组 grouping
* 如果正则表达式的模式位于括号内，便可以创建一个子模式
* 此后表达式不再通过贪婪元字符去匹配零个或多个字符，而是去匹配前面定义的子模式
* 如果模式位于括号中，它也能控制交替性（还不理解。。。）

常用分组语法
 	 		
	语法			说明

	捕获类型
	(exp) 			匹配exp,并捕获文本到自动命名的组里
	(?<name>exp) 	匹配exp,并捕获文本到名称为name的组里，也可以写成('name'exp)
	(?:exp) 		匹配exp,不捕获匹配的文本，也不给此分组分配组号
	
	零宽断言类型 	
	(?=exp) 		匹配exp前面的位置
	(?<=exp) 		匹配exp后面的位置
	(?!exp) 		匹配后面跟的不是exp的位置
	(?<!exp) 		匹配前面不是exp的位置

	注释类型	
	(?#comment) 	这种类型的分组不对正则表达式的处理产生任何影响，用于提注释让人阅读


```
替换连续一个或多个ma或pa为goo，此处使用了（）进行分组，+则是对分组进行修饰的

$_=qq/The baby says, "Mama, Mama, I can say Papa!"\n/;
print if s/(ma|pa)+/goo/gi;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
The baby says, "goo, goo, I can say goo!"


--------------------
匹配行尾连续正好3个12的行

while(<DATA>){
	print if /\s(12){3}$/; 
}
__DATA__
Steve Blenheim 121212
Betty Boop 123
Igor Chevsky 123444123
Norma Cord 51235
Jonathan DeLoach123456
Karen Evich 121212456

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Steve Blenheim 121212

--------------------

$_="Tom and Dan Savage and Ellie Main are cousins.\n";
print if s/Tom|Ellie Main/Archie/g;		# 匹配Tom或Ellie Main，则将二者都替换成Archie
$_="Tom and Dan Savage and Ellie Main are cousins.\n";
print if s/(Tom|Ellie) Main/Archie/g;	# 匹配Tom Main或Ellie Main替换成Archie

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Archie and Dan Savage and Archie are cousins.
Tom and Dan Savage and Archie are cousins.

--------------------

注意下面的差异

while(<DATA>){
	print if /^(Steve|Boop)/;	匹配Steve或Boop开头的，也能写成 /(^Steve|^boop)/
	print if /^Steve|Boop/;		匹配Steve开头的行，以及任何含有Boop的行
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jonathan DeLoach
Karen Evich
```
## 反向引用
为了避免原内容串中出现\1\2之类的，推荐使用\g{N}表示分组序号，避免歧义
```
使用(.)\1匹配连续2个相同字符
my $_="abba";
say $& if (/.(.)\1/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
abb


匹配两个abba
my $_="yabba dabba doo";
say $& if (/y(....) d\1/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
yabba dabba


多个分组
my $_="yabba dabba doo";
say $& if (/y(.)(.)\2\1/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
yabba


分组的嵌套，最外层()是\1
my $_="yabba dabba doo";
say $& if (/y((.)(.)\3\2) d\1/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
yabba dabba


使用\g{N}表示分组序号
my $_="aa11bb";
say $& if (/(.)\g{1}11/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
aa11


可以在\g{N}中使用相对序号，避免每次修改表达式都要改序号值
my $_="aa11bb";
say $& if (/(.)\g{-1}11/);
my $_="xaa11bb";
say $& if (/(.)(.)\g{-1}11/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
aa11
xaa11
```
## 匹配操作 m
匹配操作符 m// 用于匹配一个字符串语句或者一个正则表达式。当正则表达式本身含有“/”时可换为 m()，m{}，m!!，m%%，只有使用斜线才可省略m
### 模式绑定运算符=~、!=
		计算结果返回真假
		默认情况下模式匹配的操作对象是 $ _，但使用绑定运算符则不再需要$ _
		$name =~/John/ 如果 $name 含有模式则为真。如果是真，返回 1；否* 回空值
		$name !~/John/ 如果 $name 不含有模式，则为真
		$name =~s/John/Sam/ 将匹配 John 的第一个值替换为 Sam
		$name =~s/John/Sam/g 将匹配 John 的所有具体值替换为 Sam
		$name =~tr/a-z/A-Z/ 将所有小写字符翻译为大写字母
		$name =~/$pal/ 在搜索字符串时使用变量

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


----------------------------
$_ = "Hello there, neighbor!";
my($first, $second, $third) = /(\S+) (\S+) (\S+)/;
print "$second is my $third\n";

----------------------------
while(<DATA>){
	print if /Betty/; 
	print $_ if $_ =~ /Betty/; # 与上面效果一样，这里使用了绑定运算符
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Betty Boop
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
### 匹配来自__DATA__的内容
```
while(<DATA>){
	my @line = split(":", $_);
	print $line[0],"\n" if $line[1] =~ /408-/
}
__DATA__
Steve Blenheim:415-444-6677:12 Main St.
Betty Boop:303-223-1234:234 Ethan Ln.
Igor Chevsky:408-567-4444:3456 Mary Way
Norma Cord:555-234-5764:18880 Fiftieth St.
Jon DeLoach:201-444-6556:54 Penny Ln.
Karen Evich:306-333-7654:123 4th Ave.

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Igor Chevsky

---------------------------------------
while(<DATA>){
	my ($name, $phone, $address) = split(":", $_);
	print $name,"\n" if $phone =~ /408-/
}
__DATA__
Steve Blenheim:415-444-6677:12 Main St.
Betty Boop:303-223-1234:234 Ethan Ln.
Igor Chevsky:408-567-4444:3456 Mary Way
Norma Cord:555-234-5764:18880 Fiftieth St.
Jon DeLoach:201-444-6556:54 Penny Ln.
Karen Evich:306-333-7654:123 4th Ave.

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Igor Chevsky
---------------------------------------

while($inputline=<DATA>){

	my ($name, $phone, $address) = split(":", $inputline);
	print $name,"\n" if $phone =~ /^408-/;
	print $inputline,"\n" if $name =~ /^Karen/;
	print if /^Norma/;  # 不会打印，因为使用了变量inputline，而非$_，所以匹配失败
}
__DATA__
Steve Blenheim:415-444-6677:12 Main St.
Betty Boop:303-223-1234:234 Ethan Ln.
Igor Chevsky:408-567-4444:3456 Mary Way
Norma Cord:555-234-5764:18880 Fiftieth St.
Jon DeLoach:201-444-6556:54 Penny Ln.
Karen Evich:306-333-7654:123 4th Ave.

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Igor Chevsky
Karen Evich:306-333-7654:123 4th Ave.

[huawei@n148 perl]$ 
```

### 匹配来自手敲的内容
```
my $name="Tommy Tuttle";
print "Hello Tommy\n" if $name =~ /Tom/;	# 打印Hello Tommy
print "$name\n" if $name !~ /Tom/; 
$name =~ s/T/M/; 
print "$name.\n";	# 打印Mommy Tuttle.
$name="Tommy Tuttle";
print "$name\n" if $name =~ s/T/M/g;	# 打印Mommy Muttle
print "What is Tommy's last name? ";
print "You got it!\n" if <STDIN> =~ /Tuttle/;	# 手敲Tuttle才能匹配成功

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Hello Tommy
Mommy Tuttle.
Mommy Muttle
What is Tommy's last name? Tuttle
You got it!
```

## 匹配操作修饰符
perl总共支持以下几种修饰符，这些修饰符可以连用，连用时顺序可随意
* i：匹配时忽略大小写
* g：全局匹配，默认情况下，正则表达式"abc"匹配"abcdabc"字符串的时候，将之匹* 配左边的abc，使用g将匹配两个"abc"
* c：在开启g的情况下，如果匹配失败，将不重置搜索位置
* m：多行匹配模式
* s：让.可以匹配换行符"\n"，也就是说该修饰符让.真的可以匹配任意字符
* x：允许正则表达式使用空白符号，免得让整个表达式难读难懂，但这样会让原本的空* 白符号失去意义，这是可以使用\s来表示空白
* o：只编译一次正则表达式，用于优化搜索流程
* n：非捕获模式
* p：保存匹配的字符串到 $ {^PREMATCH}、$ {^MATCH}、$ {^POSTMATCH}中，它们在结果* 上对应$`、$& 和 $'，但性能上要更好
* a和u和l：分别表示用ASCII、Unicode和Locale的方式来解释正则表达式，一般不用* 考虑这几个修饰符
* d：使用unicode或原生字符集，就像5.12和之前那样，也不用考虑这个修饰符

### 忽略大小写 i
```
$_ = "I lost my gloves in the clover, Love.";
my @list=/love/gi;
$,=",";
say @list;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
love,love,Love
-------------------------------------

my $name="aAbBcC";
if($name =~ m/ab/i){
    print "pre match: $` \n";     # 输出a
    print "match: $& \n";         # 输出Ab
    print "post match: $' \n";    # 输出BcC
}
```
### 全局匹配 g
* 修饰符 g 用于产生全局匹配效果，即匹配行中所有模式的具体值。如果
不使用 g 的话，模式只会与碰到的第一个值相匹配。m 运算符将返回匹配模式列表。  
* 如果用于数组型上下文语境，则会返回一个列表；如果用于标量型上下文语境，则返回真或假

```
$_ = "I lost my gloves in the clover, Love.";
my @list=/love/g;
$,=",";
say @list;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
love,love

-------------------------------------
my @arr = "abcabc" =~ /ab/g;  # 匹配返回qw(ab ab)
say "@arr";  # ab ab

-------------------------------------
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
* 正则表达式一般都只用来匹配单行数据，但有时候却需要一次性匹配多行。比如匹配跨行单词、匹配跨行词组，匹配跨行的对称分隔符(如一对括号)。  
* 一般匹配结尾常用$，在带有\n的字符串中 $ 只会比对最后的\n ，如希望$能匹配所有\n就要加 /m
* 注意：/m 修饰符对于 \A 和 \z 是无效的，见下例
```
$str = "line123\nline456\nline789";
$str =~ s/\d+$//g;	# 将数字结尾的行替换为空，这里仅替换了最后一处
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
line123
line456
line

注意现在加了m，每个换行符号都视为结尾
$str = "line123\nline456\nline789";
$str =~ s/\d+$//mg;
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
line
line
line

-----------------------------------

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

-----------------------------------
$_ = "I'm much better \nthan Barney is \nat bowling, \nWilma.\n";
print "Found 'wilma' at start of line\n" if /^wilma\b/im;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Found 'wilma' at start of line

-----------------------------------
复杂些的案例，配合了锚点进行行首词首的匹配

$_="Today is history.\nTomorrow will never be here.\n";
print if /^Tomorrow/; # 失败，没有/m即使有\n也当做一行处理，行首是Today，不是Tomorrow
$_="Today is history.\nTomorrow will never be here.\n";
print if /\ATomorrow/; # 失败，$_此字符串首是Today，不是Tomorrow
$_="Today is history.\nTomorrow will never be here.\n";
print if /^Tomorrow/m; # 成功，有/m可匹配多行。第二行行首是Tomorrow，打印出$_
$_="Today is history.\nTomorrow will never be here.\n";
print if /\ATomorrow/m; # 失败，\A匹配字符串首，虽然有/m，但m修饰符对于\A和\z是无效的，字符串首是Today，不是Tomorrow
$_="Today is history.\nTomorrow will never be here.\n";
print if /history\.$/m; # 成功，有/m可匹配多行。第1行行尾是history.\n（看来$可以包含\n），打印出$_

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Today is history.
Tomorrow will never be here.
Today is history.
Tomorrow will never be here.
```
### “.”能匹配\n s
* 字符串中的换行字符 '\n' 被当成是一个字符来处理，所以假设一个具有换行字符的(多行的的字符串，希望比对时忽略那个换行字符，就要加上 s  修饰子
* 默认情况下"."是不能匹配换行符\n的，开启了s修饰符功能后，可以让"."匹配换行符。  
* <font size=3 color=red >注意注意注意：
注意这里含有换行的必须是字符串，即使是文件也得先转数组再join成字符串再匹配才行，不能循环去匹配文件。。。</font>
```
$str = "What a wonder\nful wonderful world.";
$str =~ s/wonder.?ful/www/g;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What a wonder
ful www world.

接下来加上 s  修饰子后，\n就等于是"一个字符"，也等于'\n'；否则未加s的情况则不属于一个字符，也就是和 '.' 比对不会成功：

$str = "What a wonder\nful wonderful world.";
$str =~ s/wonder.?ful/www/sg;
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What a www www world.

如果坚持一定要和换行比对成功，则：注意没加 s
$str = "What a wonder\nful wonderful world.";
$str =~ s/wonder\nful/www/g;
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What a www wonderful world.
----------------------------------------------
print "$str\n";
$_="Sing a song of sixpence\nA pocket full of rye.\n";
print;
print $& if /pence./s;
print $& if /rye\../s;
print if s/sixpence.A/twopence, a/s;


huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Sing a song of sixpence
A pocket full of rye.		# 这两行是打印原内容
pence						# /pence./s匹配到pence\n
rye.						# /rye\../s匹配到rye.\n
Sing a song of twopence, a pocket full of rye.	# 此句是替换后的
```
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
### 字符解释方式
* /\w+/a	#仅仅是A-Z、a-z、0-9、_这些字符
* /\w+/u	#任何Unicode当中定义为单词的字符
* /\w+/l	#类同于ACSII的版本，但单词字符的定义取决于本地化设定
			#所以如果设定为Latin-9的话
## 锚定元字符
能使模式匹配只发生在字符串的开头或末尾位置，这些元字符基于待匹配字符的左侧位置或右侧位置

	元字符 	匹配项
	^ 		匹配行首位置
			/^[JK]/ 	# 匹配行首是J或K的
			/^[^JK]/ 	# 匹配行首不是J也不是K的
	$ 		匹配行尾位置
			/10$/		# 匹配行尾是10的
	\A		只匹配字符串的开头位置，也许也是行首	
	\Z 		匹配字符串或行的末尾位置，它允许后面出现换行符
			while(<STDIN>){ 
				print if /\.png\Z/ 
			};
			/\A\s*\Z/ 	# 匹配一个空行
	\z 		只匹配字符串的末尾位置，后面再没其它东西
			m{\.png\z}i	#匹配以.png结尾的字符串	
	\G 		强制从位移指针处进行匹配，
			匹配前面 m//g 模式的离开位置，
			详细内容见g和c修饰符以及\G
	\b 		匹配词首或词尾
			只匹配符合\w的开头或结尾，即[0-9a-zA-Z]
			/\bhunt/	# 匹配hunt hunting hunter,而不匹配shunt
			/stone\b/	# 匹配standstone flintstone但不包括capstones
			/\bJon\b/ 	# 匹配JonxxxxJon这样的
	\B 		匹配一个非字边界
			/\bsearch\B/会匹配searches、searching与searched，
			但不匹配search或researching
    \<		匹配单词开头处的空字符
    \>		匹配单词结尾处的空字

## 记忆或捕获
如果正则表达式位于括号中，则表示创建子模式。子模式保存在具有特殊编号的标量型变量中，首先是 $1，然后是 $2，依此类推。可在程序中使用这些变量，直到其他成功的匹配将它覆盖为止。
### 捕获变量
$ 1、$ 2、$ 3...保存了各个分组捕获的内容，使用（）进行捕获，$n代表第n个（）捕获到的内容

```
将Jon换为Jonathan，将jon换为jonathan

while(<DATA>){
	s/([Jj]on)/$1athan/;
	print;
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jonathan DeLoach
Karen Evich

------------------------

是反转Steve与Blenhein

while(<DATA>){
	print if s/(Steve) (Blenheim)/$2, $1/
}
__DATA__
Steve Blenheim
Betty Boop

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Blenheim, Steve
------------------------

反转姓名，用了2种方式

查找一个大写字母后面跟随一个或多个小写字母为分组1
空格
查找一个大写字母后面跟随一个或多个小写字母为分组2
然后交换2个分组

while(<DATA>){
	s/([A-Z][a-z]+)\s([A-Z][a-z]+)/$2, $1/;
	s/(\w+)\s(\w+)/$2, $1/;	# 这是第二种方式
	print;
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Blenheim, Steve
Boop, Betty
Chevsky, Igor
Cord, Norma
De, JonLoach
Evich, Karen

------------------------

将整个字符串分成了2部分，分别捕获到2个变量里

my $string="ABCdefghiCxyzwerC YOU!";
$string=~s/(.*C) (.*)/HEY/;
print $1, "\n";
print $2, "\n";
print "$string\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
ABCdefghiCxyzwerC
YOU!
HEY

------------------------
第一次是贪婪的，第二次是非贪婪的，观察结果

my $fruit="apples pears peaches plums";
$fruit =~ /(.*)\s(.*)\s(.*)/;
print "$1\n";
print "$2\n";
print "$3\n";
print "~~~second~~~\n";
$fruit="apples pears peaches plums";
$fruit =~ /(.*?)\s(.*?)\s(.*?)\s/; # Turn off greedy quantifier
print "$1\n";
print "$2\n";
print "$3\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
apples pears
peaches
plums
~~~second~~~
apples
pears
peaches


------------------------
$_="hello there, neighbor";
if (/(\S+) (\S+), (\S+)/){
	say "$1, $2, $3";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
hello, there, neighbor
```
### 命名捕获
即使用具名的符号代替$1,$2之类的变量
```
还是反转姓名
正则表达式含有两个位于括号内的模式。其中 \w+ 代表一个或多个字。正则表达式由两个带括号的子模式（又称反向引用）构成。其返回值是一个由所有反向引用所组成的数组。分别将每个字赋值给变量 $first 与 $last。


while(<DATA>){
	($first, $last)=/(\w+) (\w+)/;
	print "$last, $first\n";
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Blenheim, Steve
Boop, Betty
Chevsky, Igor
Cord, Norma
DeLoach, Jon
Evich, Karen

-----------------------


my $names = 'fred or Barray';
if($names =~ m/(?<name1>\w+) (and|or) (?<name2>\w+)/){
	say "1 I saw $+{name1} and $+{name2}";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
1 I saw fred and Barray

---------------

my $names = "Fred Filnstone and wilma Filnstone";
if($names =~ m/(?<last_name>\w+) and \w+ \g{last_name}/){
	say "2 I saw $+{last_name}"
};
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
2 I saw filntstone
```
### 自动捕获变量
```
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

### 生存期
这些特殊变量是只读的，每次正则匹配时Perl会自动重置这些变量。这些变量只在当前作用域内有效，并且只在本次正则匹配后、下次正则匹配前有效。如下面的第二次say就失败了
```
my $str = "abbc123";
$str=~/a(.*)c/;
say $1;
$str=~/a.*c/;
say $1;   # undef
```
### 常用特殊变量
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
### 捕获到列表
如果模式匹配成功，那么返回的是所有捕获变量的列表；如果匹配失败，则会返回空列表
```
my $str = "Hello there, neighbor!";
my($first, $second, $third) = ($str =~ /(\S+) (\S+) (\S+)/);
print "$second is my $third\n";

用等号，默认匹配的是$_。可以简写为先这样，但注意捕获那句仅是“=”
$_ = "Hello there, neighbor!";
my($first, $second, $third) = /(\S+) (\S+) (\S+)/;
print "$second is my $third\n";


------------------------------
my $text = "Fred dropped a 5 ton granite block on Mr.Slate";
my @words = ($text =~ /([a-z]+)/ig);
$,=',';
say @words;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Fred,dropped,a,ton,granite,block,on,Mr,Slate
```
### 捕获到哈希
```
my $data = "Barney Rubble Fred Flintstone Wilma Fllintstone";
my %last_name = ($data =~ /(\w+)\s+(\w+)/g);
while(my ($k, $v) = each %last_name){
  say "key: $k, v: $v";
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
key: Wilma, v: Fllintstone
key: Barney, v: Rubble
key: Fred, v: Flintstone
```
## 零宽断言 ?!、?<!、?=、?<=
断言（assertion）  
通常情况下指的是在目标字符串的当前位置进行的一种判断测试，但这种测试不会占用目标字符串，这意味着不会移动目标字符串在当前匹配中的位置。
* 零宽断言表示匹配字符的时候再添加一些定位条件，使匹配更精准
* 零宽断言是一种零宽度的匹配，它匹配的内容不会保存到匹配结果中，也不会占用index宽度，最终匹配的结果只是一个位置

零宽断言分为4类
* 零宽正向预测先行  
  (?=exp)  
  如 /x(?=y)/ 当x后面是y时才算成功匹配x
* 零宽负向预测先行  
  (?!exp)  
  如 /x(?!y)/ 只有当x后面不是y时才算成功匹配x
* 零宽正向回顾后发    
  (?<=exp)  
  如 /(?<=x)y/ 只有当y的前面是x时，才算成功匹配y。
* 零宽负向回顾后发  
  (?<!exp)  
  如 /(?<!x)y/ 只有当y的前面不是x时，才算成功匹配y。
```
while(<DATA>){
	# print if /\.min(?!\.js$)/;		# 找.min且后面不是.js的（匹配asd.hs.ja.min..js）
	# print if /\.min(?=\.js$)/;		# 找.min且后面是.js的
	# print if /(?<=\.min)\.js$/;		# 找.js且前面是.min的
	# print if /(?<!\.min)\.js$/;		# 找.js且前面不是.min的
}
__DATA__
a.js
b.js
c.min.js
minmin.js
asd.hs.ja.min..js
```

## 替换操作 s
* s/原来字符串/目的字符串/修饰子  
使用新的字符串替换指定的字符串。返回的结果是其完成的替换数目。也可以将"/"换成下面这样：
```
s(Blenheim){Dobbins};
s#Igor#Boris#;
```
简单的演示
```
基本的替换，默认仅替换第一个，打印替换后的结果
my $str = "Who are Who you?";
$str =~ s/Who/What/; 
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What are Who you?


全部替换，获取替换了几次，并打印
my $str = "Who are Who you?";
my $count=$str =~ s/Who/What/g; 
print "$str\t$count\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What are What you?      2

--------------------------------------
$_ = "green scaly dinosaur";
s/(\w+) (\w+)/$2, $1/; #替换后为"scaly, green dinosaur"
s/^/huge, /; #替换后为"huge, scaly, green dinosaur"，即在上一个结果前插入字符串
s/,.*een//; #空替换：此时为"huge dinosaur"，即把“,”到“een”之间的删掉
s/green/red/; #匹配失败：仍为"huge dinosaur"
s/\w+$/($`!)$&/;	#替换为huge (huge !)dinosaur
s/\s+(!\W+)/$1 /; #替换为"huge (huge!) dinosaur"
s/huge/gigantic/; #替换为"gigantic (huge!) dinosaur"
```
无论是否发生替换，打印每一行内容。
```
while(<DATA>){
	s/Norma/Jane/;
	print;
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Steve Blenheim
Betty Boop
Igor Chevsky
Jane Cord
Jon DeLoach
```
只有在替换成功时才会打印该行内容。

```
while(<DATA>){
	print if s/Igor/Ivan/;
}
__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Ivan Chevsky
```
### 替换操作修饰符
```
修饰符	描述
i	取消大小写敏感性，即"a"和"A" 是一样的。
m	将字符串作为多行处理
o	只编译模式一次。用于优化搜索流程
s	嵌入换行符时，将字符串作为单行处理
x	如允许在正则表达式中提供注释，并忽略空白字符
g	允许在正则表达式中提供注释，并忽略空白字符
e	将替换一侧作为表达式来求值
```
### 全局替换 g
修饰符 g 用于产生全局替换效果，即替换行中所有模式的具体值。如果不使用 g 的话，则每行只会替换模式的第一个具体值。
```
注意这里仅替换了一次

while(<DATA>){
	print if s/Tom/Christian/;
}

__DATA__
Tom Dave Dan Tom
Betty Tom Henry Tom
Igor Norma Tom Tom
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Christian Dave Dan Tom
Betty Christian Henry Tom
Igor Norma Christian Tom
--------------------------

注意这里全都替换了

while(<DATA>){
	print if s/Tom/Christian/g;
}

__DATA__
Tom Dave Dan Tom
Betty Tom Henry Tom
Igor Norma Tom Tom

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Christian Dave Dan Christian
Betty Christian Henry Christian
Igor Norma Christian Christian
--------------------------

$_ = "home, sweet home!";
s/home/cave/g;
print "$_\n"; #打印"cave, sweet cave"

```
### 忽略大小写 i
默认是对大小写敏感的。如果需要关闭大小写敏感性，则应当在匹配运算符的最后一个定界符后面加上 i 修饰符
```
$str = "What a wonderful wonderful world.";
$str =~ s/w/www/ig;
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
wwwhat a wwwonderful wwwonderful wwworld.



while(<DATA>){
	print if s/igor/Daniel/i;
}

__DATA__
Steve Blenheim
Betty Boop
Igor Chevsky
Norma Cord
Jon DeLoach
Karen Evich

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Daniel Chevsky
```
### 求表达式值 e
在进行替换操作时，可能需要去求表达式或者函数的值。本修饰符负责将搜索端内容替换成上述求值的结果。

使用数学计算结果替换原数字的值
```
while(<DATA>){
	s/6/6 * 7.3/eg;		# 将6换为43.8
	print;
}

__DATA__
Steve Blenheim 5
Betty Boop 4
Igor Chevsky 6
Norma Cord 1
Jon DeLoach 3
Karen Evich 66

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Steve Blenheim 5
Betty Boop 4
Igor Chevsky 43.8
Norma Cord 1
Jon DeLoach 3
Karen Evich 43.843.8

--------------------
$_=5;
s/5/6 * 4 - 22/e;		# 将5换为2
print "The result is: $_\n";
$_=1055;
s/5/3*2/eg;				# 将5换为6
print "The result is: $_\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
The result is: 2
The result is: 1066
```
使用字符串计算结果替换原字符串的值
```
$_ = "knock at heaven's door.\n";
s/knock/"knock, " x 2 . "knocking"/ei;
print "He's $_";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
He's knock, knock, knocking at heaven's door.
```
使用$&参与的计算结果替换变量值
```
my $salary=50000;
$salary =~ s/$salary/$& * 1.1/e;
print "\$& is $&\n";
print "The salary is now \$$salary.\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/tempCodeRunnerFile.pl"
$& is 50000
The salary is now $55000.
```
### 加引号
```
$str = "What a wonderful wonderful world.";
$str =~ s/w/'$&'/g;
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What a 'w'onderful 'w'onderful 'w'orld.
```
### 局部大小写转换
s替换里面可以使用函数，例如以下几个常用的函数：
uc($str) 把$str 全转成大写
lc($str) 把$str 全转成小写
ucfirst($str) 把$str 第一码转成大写
```
$str = "What a wonderful wonderful world.";
$str =~ s/w\w+/uc($&)/ge;	# w\w代表w开头的单词，注意要有e参与计算
print "$str\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What a WONDERFUL WONDERFUL WORLD.
```
### 替换空白
```
一个常见的全局替换是缩减空白，也就是将任何连续的空白转换成单一空格：
$_ = "Input data\t may have    extra whitespace";
s/\s+/ /g; #现在它变成了"Input data may have extra whitespace."
say $_;

s/^\s+//; #将开头的空白替换成空字符串
s/\s+$//; #将结尾的空白替换成空字符串
s/^\s+|\s+$//g; #去除开头和结尾的空白符，但运行可能会慢，由于perl的引擎问题
```
### 无损替换
如果需要同时保留原始字符串和替换后的字符串则可以这样
```
my $original = 'Fred ate 1 rib';

方法1，最原始
# my $copy = $original;
# $copy =~ s/\d+ ribs?/10 ribs/;
方法2，合成为一句
# (my $copy = $original) =~ s/\d+ ribs?/10 ribs/;
方法3，新写法
my $copy = $original =~ s/\d+ ribs?/10 ribs/r; #先做替换，再复制

say $copy;
say $original;
```
### 大小写切换
* \U:	使目标变为大写
* \L:	使目标变为小写
* \E: 关闭大小写转换，不然会影响后面的字符
```
$_ = "I saw Barney with Fred.";


s/(fred|barney)/\U$1/gi; 				#$_现在成了"I saw BARNEY with FRED."
s/(fred|barney)/\L$1/gi; 				#$_现在成了"I saw barney with fred."
s/(\w+) with (\w+)/\U$2\E with $1/i; 	#$_现在成了"I saw FRED with barney"
```
## 列表的置换 tr
* tr/原来比对列表/目的比对列表/选项  
perl 的 tr把置换的功能再扩张，虽然 s很强，可是也有做不到的事，例如今天要把大写换成小写，「同时」小写也换成大写，s就一筹莫展了
* tr也可换为y命令，功能一样...

```
$str = "Aine123\nBine789";
$str =~ tr/a-zA-Z/A-Za-z/;
print "$str\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
aINE123
bINE789
```
* tr 有三个选项
```
修饰符	描述
c	列表没写到的就补给他右边列表的最后一个字符
d	对照表中没有的项目就删掉
s	连续重复出现的字压成一个
```
### 去重s与删除d
去重s
```
my $text = 'good cheese';
$text =~ tr/eo/eu/s;	# 对e和o进行去重，并且将e换为e，将o换为u
print "$text\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
gud chese
```
删除d
```
my $big = 'vowels are useful';
$big =~ tr/aeiou/AEI/d;		# aei换为AEI，ou被删除
print "$big\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
vwEls ArE sEfl

----------------------------------------
my $temp="AAAABCDEF";
my $count=$temp=~tr/A/H/d;		# 仅将A换为H，没有要删掉的
print "$temp\t$count\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
HHHHBCDEF       4
```
去重加删除ds
```
my $text = 'good cheese';
$text =~ tr/eogd/eu/ds; 
print "$text\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
u chese
```
### 替换匹配不是的 c
		tr/左列表/右列表/c
规则、左列表没有的，就补右清单的东西。
```
my $text = 'good cheese';
$text =~ tr/eo /123/c;	# 使用“123”里的最末字符3替换除了“eo ”之外的其余字符
print "$text\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
3oo3 33ee3e
----------------------------------------

使用z替换除了字母之外的所有字符

$doc="<78>Nov  3 11:20:01 163.17.44.1 crond[30367]";
$doc =~ y/a-zA-Z/a-z/c;
print "$doc\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
zzzzNovzzzzzzzzzzzzzzzzzzzzzzzzzcrondzzzzzzz

----------------------------------------

只保留字母

my $doc="<78>Nov  3 11:20:01 163.17.44.1 crond[30367]: (root) CMD (LANG=C LC_ALL=C /usr/bin/mrtg /etc/mrtg/mrtg.cfg --lock-file /var/lock/mrtg/mrtg_l --confcache-file /var/lib/mrtg/mrtg.ok)";
$doc =~ y/a-zA-Z/ /sc;
print "$doc\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
 Nov crond root CMD LANG C LC ALL C usr bin mrtg etc mrtg mrtg cfg lock file var lock mrtg mrtg l confcache file var lib mrtg mrtg ok 
```
### 统计指定字符数量
```
my $doc="<78>Nov  3 11:20:01 163.17.44.1 crond[30367]";
my $count=$doc=~tr/o//;	
my $count=$doc=~tr/o/o/;	和上面效果一样
print $count, "\n" ;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
2
```
## unicode
待完善
# 子程序

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
	return (1..3);

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
## 参数列表，传递标量参数
* 子程序可有多个参数，函数内使用 @_ 表示参数列表
* 子程序内也可直接使用 $ _[0], $ _[1]代表第n个参数
* 不论参数是何类型，传给子程序时，perl默认按引用的方式传递
* 可以通过改变 @_ 数组中的值来改变外面实参的值
* 子程序内多使用shift按顺序弹出
```
参数是n个标量
$,=',';
sub f{
	say @_;			整个参数列表
	say $_[0];		单个元素
	say $_[1];
	say $_[2];
}
f(1,'aaa', 123);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
1,aaa,123
1
aaa
123
[huawei@n148 perl]$ 
---------------------------------------
sub add{
	die unless 2==@_;		注意这里还可以这样检查参数个数
	my($a,$b)=@_;
	$a+$b;
}
say add(1, 123);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
124
[huawei@n148 perl]$ 

---------------------------------------



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
## shift弹出参数
使用shift来弹出参数列表中的第一个，shell里也有类似功能
```
在子程序内部，下面两个操作是等价的：
shift @_;
shift;


$,=',';
sub max{
	my $maxval=shift;
	foreach(@_)
	{
		$maxval=$_ if $_>$maxval;
	}
	$maxval;
}
say max(1, 123, '45aaa', 2);
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

-------------------------------

案例：检查在a列表里的每个元素是否在b列表里。典型的传值方式，参数的数据量小还是可以的，本例在“传递引用与匿名对象”中有完善

sub check{
	my $who=shift;
	my %whoitems=map{$_,1}@_;
	my @required=qw/a b c d/;
	for my $item(@required){
		unless ($whoitems{$item}){
			print "$who is missing $item\n";
		}
	}
}
check('tom', qw/a c e f/);
check('dick', qw/b d g h/);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
tom is missing b
tom is missing d
dick is missing a
dick is missing c
```
可以向子程序传入多个数组和哈希，但是在传入多个数组和哈希时，会导致丢失独立的标识。所以我们需要使用引用（下一章节会介绍）来传递
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
----------------------------------

传递数组的引用

sub check{
	my $who=shift;
	my $items=shift;
	my %whoitems=map{$_,1} @{$items};
	my @required=qw/a b c d/;
	for my $item(@required){
		unless ($whoitems{$item}){
			print "$who is missing $item\n";
		}
	}
}

上面的精简版：
sub check{
	my %whoitems=map{$_,1} @{$_[1]};
	my @required=qw/a b c d/;
	for my $item(@required){
		unless ($whoitems{$item}){
			print "$_[0] is missing $item\n";
		}
	}
}

使用grep 2个数组，省略了哈希。上面的再次精简版：
sub check{
	my $who=shift;
	my $items=shift;
	my @required=qw/a b c d/;
	for my $item(@required){
		unless (grep $item eq $_, @$items){
			print "$who is missing $item\n";
		}
	}
}

my @s=qw/a c e f/;
check('tom', \@s);
@s=qw/b d g h/;
check('dick', \@s);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
tom is missing b
tom is missing d
dick is missing a
dick is missing c

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

## 默认参数 $_
对于需要参数的函数或表达式，但却没有给参数则默认是变量$_
```
$_="abcde";
print ;

foreach(1..10){
	print $_;
}
```
## 命令行参数 ARGV
* 命令行参数保存在数组@ARGV中，数组下标从0开始
* ARGV[0]是第一个参数（非程序名称），以此类推
* $ #ARGV保存数组最后一个元素的索引（非数组元素数量），当无参数时$#ARGV等于-1（不是零）
* 命令行参数的总数是 $#ARGV+1
* < ARGV >也可作为文件句柄数组进行迭代，见下面的案例

参数列表案例
```
die "$0 运行需要参数\n" if $#ARGV < 0 ;
print "参数列表=@ARGV\n";
print "第一个=$ARGV[0]\n";
print "第二个=$ARGV[1]\n";
print "参数总个数=", $#ARGV + 1,"\n";
print "最末参数=$ARGV[$#ARGV]\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl" 
/home/huawei/playground/perl/1.pl 运行需要参数

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"  a b c d e
参数列表=a b c d e
第一个=a
第二个=b
参数总个数=5
最末参数=e
```
文件句柄数组案例
```
die "$0 运行需要参数\n" if $#ARGV < 0 ;
while( <ARGV> ) {
	print;
}

运行程序会将多个文件的内容依次打印出来

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl" err.txt  rocks.txt  a.log 
```
在文件中查找匹配正则的行
```
die "$0 运行需要参数\n" if $#ARGV < 1 ;
my $pattern = shift;
while(my $line=<ARGV> ) {
	print "文件名=$ARGV，行号=$.，$line" if $line=~/$pattern/i;
	close(ARGV) if eof;
}

当到达待处理文件的末尾（EOF）时，关闭 ARGV 文件句柄。这样做即可重置变量 $.。如果这里没有显式地关闭 ARGV，变量 $. 就会不断递增下去，并在读取下一个文件时仍无法回到 1。

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl" abc err.txt  
文件名=err.txt，行号=1，bar, World!/nabc
文件名=err.txt，行号=5，bar, World!/nabc
```
展示指定用户所开启的进程列表，综合大案例。。。
```
# 该脚本只接受一个参数。如果 ARGV 为空（即命令行没有提供任何参数）的话，就执行 die
# 函数退出脚本，并产生一个出错消息（请记住，$#ARGV 负责保存最后一个参数的编号；
# @ARGV[0] 是第一个参数，而不是脚本名称 ；脚本名保存在变量 $0 中）。如果用户提供了多
# 个参数，该脚本在执行时也将产生错误信息。
unless ( $#ARGV == 0 ){ die "Usage: $0 <argument>: $!"; }

open(PASSWD, "/etc/passwd") || die "Can't open: $!"; # 通过 PASSWD 文件句柄以读方式打开文件 /etc/passwd。
my $username=shift(@ARGV);	# 从 @ARGV 中移出第一个参数，并赋值给变量 $username。
my $hasuser=0;
while( my $pwline = <PASSWD>){ # 迭代PASSWD 每一行
	if ($pwline =~ /^$username/) # 使用 =~ 检查第一个参数内容是否匹配 $username。
	{
		print "passwd里找到$username\n";
		$hasuser=1;
		last;
	}
}
die "$username is not a user here.\n" if 0 == $hasuser; # 如果找不到匹配，则退出循环。
close PASSWD; # 关闭文件句柄。

open(LOGGEDON, "who |" ) || die "Can't open: $!" ; # 将文件句柄 LOGGEDON 打开为输入过滤器。把 UNIX 中 who 命令的输出内容转发给该文件句柄。
my $logged_on;
while(my $logged = <LOGGEDON> ){ #检查输入过滤器的每一行内容。如果用户已登录的话，就把标量 $logged_on 设置为 1，并退出循环。
	if ($logged =~ /$username/){ 
		print "who里找到$username\n";
		$logged_on = 1; last;
	}
}
close LOGGEDON; # 关闭输入过滤器。
die "$username is not logged on.\n" if ! $logged_on;

print "$username is logged on and running these processes.\n";
open(PROC, "ps -aux|" ) || die "Can't open: $! "; # 将文件句柄 PROC 打开为输入过滤器。把 UNIX 命令的输出内容转发给该文件句柄
my $idx=0;
while(my $line=<PROC>){# 依次通过过滤器读取每一行内容，并将其存入标量 $line。
	if ($line =~ /^$username/){ # 如果 $line 中含有匹配于该用户的项的话，则将该行内容打印到 STDOUT，即终端屏幕。
		print $line;
		if (++$idx >= 10)	# 只看10条就够了
		{
			last;
		}
	}
}
close PROC; # 关闭过滤器。


[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl" huawei
passwd里找到huawei
who里找到huawei
huawei is logged on and running these processes.
huawei     6434  0.0  0.0 163956  2628 ?        S    Dec09   0:13 sshd: huawei@pts/0
huawei     6441  0.0  0.0 117036  3664 pts/0    Ss+  Dec09   0:00 -bash
huawei     6495  0.0  0.0 163960  2536 ?        S    Dec09   0:00 sshd: huawei@notty
huawei     6500  0.0  0.0 115272  3560 ?        Ss   Dec09   0:41 bash -c while [ -d /proc/$PPID ]; do sleep 1;head -v -n 8 /proc/meminfo; head -v -n 2 /proc/stat /proc/version /proc/uptime /proc/loadavg /proc/sys/fs/file-nr /proc/sys/kernel/hostname; tail -v -n 16 /proc/net/dev;echo '==> /proc/df <==';df;echo '==> /proc/who <==';who;echo '==> /proc/end <==';echo '##Moba##'; done
huawei     6508  0.0  0.0  72292  2908 ?        Ss   Dec09   0:00 /usr/libexec/openssh/sftp-server
huawei     9419  0.0  0.0 116864  3452 pts/4    Ss+  00:57   0:00 /usr/bin/bash
huawei    16409  0.0  0.0 116996  3660 pts/5    Ss+  01:30   0:00 /usr/bin/bash
huawei    50751  0.0  0.0 116996  3692 pts/6    Ss   04:13   0:00 /usr/bin/bash
huawei    73004  0.0  0.0 156964  2504 ?        S    05:49   0:00 sshd: huawei@notty
huawei    73011  0.0  0.0 113292  1396 ?        Ss   05:49   0:00 bash
```
## 所有变量
变量对应的名称见 https://www.runoob.com/perl/perl-special-variables.html
```
$0 当前perl脚本文件名称
$! 获取当前错误信息值，常用于 die 命令
$” 列表分隔符
$# 打印数字时默认的数字输出格式
$$ 正在执行本脚本的 Perl 进程号
$% 当前输出通道的当前页号
$& 与上个格式匹配的字符串
$( 当前进程的组ID$) 当前进程的有效组ID
$* 设置1表示处理多行格式.现在多以/s和/m修饰符取代之.
$, 当前输出字段分隔符
$. 代表处理文件时当前迭代的行号
$/ 当前输入记录分隔符,默认情况是新行
$: 字符设置,此后的字符串将被分开,以填充连续的字段.
$; 在仿真多维数组时使用的分隔符.
$? 返回上一个外部命令的状态
$@ 由最近一个 eval() 运算符提供的 Perl 语法报错信息
$[ 数组中第一个元素的索引号
$\ 当前输出记录的分隔符
$] Perl解释器的子版本号
$^ 当前通道最上面的页面输出格式名字
$^A 打印前用于保存格式化数据的变量
$^D 调试标志的值
$^E 在非UNIX环境中的操作系统扩展错误信息
$^F 最大的文件捆述符数值
$^H 由编译器激活的语法检查状态
$^I 内置控制编辑器的值
$^L 发送到输出通道的走纸换页符
$^M 备用内存池的大小
$^O 操作系统名
$^P 指定当前调试值的内部变量
$^R 正则表达式块的上次求值结果
$^S 当前解释器状态
$^T 从新世纪开始算起,脚步本以秒计算的开始运行的时间
$^W 警告开关的当前值
$^X Perl二进制可执行代码的名字
$^V Perl 解释器的版本、子版本和修订版本信息，同$PERL_VERSION
$_ 在执行输入和模式搜索操作时使用的默认空格变量
$| 默认等于0，如果等于非0表示当前的输出不经过缓存立刻输出
$~ 当前报告格式的名字
$` 在上个格式匹配信息前的字符串
$’ 在上个格式匹配信息后的字符串
$+ 与上个正则表达式搜索格式匹配的最后一个括号
$< 当前执行解释器的用户的真实ID
$ 含有与上个匹配正则表达式对应括号结果
$= 当前页面可打印行的数目
$> 当前进程的有效用户ID包含正在执行的脚本的文件名
$ARGV 将<ARGV>当做句柄使用时代表命令行参数所对应的文件名，见命令行参数ARGV案例
ARGV 一个特殊的文件句柄，用于遍历 @ ARGV 中出现的所有文件名
%ENV 环境变量列表
%INC 库文件的搜索路径
%SIG 信号列表及其处理方式
@_ 在子例程中@_含有传给该子例程的参数列表
@ARGV 传给脚本的命令行参数列表
@INC 在导入模块时需要搜索的目录列表
$-[0]和$+[0] 代表当前匹配的正则表达式在被匹配的字符串中的起始和终止的位置 
```

# 进程管理
* 特殊变量 $$ 或 $PROCESS_ID 来获取进程 ID。
* %ENV 哈希存放了父进程，也就是shell中的环境变量，在Perl中可以修改这些变量。
## 反引号运算符
如果不需要获取命令的输出就不用使用反引号。也可使用qx()、qx\\...这类方式
```
foreach (`cat /etc/passwd`){
	my @lst=split /:/, $_;
   	say $lst[0], "\t", $lst[5];
}



----------------------------------
my @files = `ls -l`;
my @files = qx(ls -l);
foreach my $file (@files){
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
默认情况下输出到STDOUT指向的地方，也可以使用重定向运算符 > 输出到指定文件。返回0则正常。会创建子进程执行命令，父进程等待子进程完毕再继续
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
## fork与exec
fork() 函数用于创建一个新进程。在父进程中返回子进程的PID，在子进程中返回0。如果发生错误（比如，内存不足）返回undef，并将$!设为对应的错误信息。fork 可以和 exec 配合使用。exec 函数执行完引号中的命令后进程即结束（不会创建子进程）。
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

------------------------------------
这个更精练
defined(my $pid = fork) or die "Cannot fork:$!";
unless($pid)
{
	#能运行到这里的是子进程
	exec 'date' or die "Cannot exec date:$!";
}
waitpid($pid, 0); #能运行到这里的是父进程
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
## IPC::System::Simple
system、systemx、capture、capturex
## 通过文件句柄执行外部进程
## 发送及接收信号 Kill
kill('signal', (Process List))给一组进程发送信号。signal是发送的数字信号，9为杀掉进程。  
kill('KILL', 进程号1, 进程号2...);
# 目录操作
## 获取当前路径
```
use Cwd;
$dir = cwd();	# 效果同pwd
```
## 创建与删除
```
创建目录
my $dir='/tmp/test123/';
mkdir $dir, 0700 or die;
---------------
my $temp_directory = "/tmp/ATL3Perl.$$";
mkdir $temp_directory, 0700 or die "Cannot create $temp_directory: $!";



清除目录里的文件后删除目录
unlink glob "$dir/* $dir/.*";
rmdir $dir;
```
## 进入指定目录
```
say cwd();
chdir '/etc' or die;
say cwd();
```
## 遍历指定目录 glob
不递归，两句效果一致
```
say $_ foreach(glob '/etc/* /etc/.*');
say $_ foreach(</etc/* /etc/.*>);
```
## 遍历指定目录句柄
不递归
```
排除.和..还有pl文件
opendir DIR, './';
foreach(readdir DIR)
{
	next if $_ eq '.' || $_ eq '..' || $_ =~ /\.pl/;
	say $_;
}
closedir DIR;
```
## 递归遍历指定目录
File::Find::Rule
# 文件操作
## 自身文件名
```
say $0			# 全路径自身文件名
say __FILE__;	# 一样
```
## 文件测试
* Perl 也提供了许多文件测试运算符用于查看各种文件属性
* 其中大部分的运算符都在结果为真时返回 1，在结果为假时返回“”（即 null）
* 如果需要反复多次测试同一个文件，则可在程序中使用单个下划线来代表文件名。这样	便可利用前一次文件测试的 stat 结构。


文件测试运算符
	
	运算符 		含 义
	-r $file 	如果 $file 可读，则为真
	-w $file 	如果 $file 可写，则为真
	-x $file 	如果 $file 可执行，则为真
	-o $file 	如果 $file 的属主是有效的 uid，则为真
	-e $file 	如果 $file 存在，则为真
	-z $file 	如果 $file 大小为 0，则为真
	-s $file 	如果 $file 大小非 0，则为真。返回文件字节大小
	-f $file 	如果 $file 是普通文件，则为真
	-d $file 	如果 $file 是目录，则为真
	-l $file 	如果 $file 是符号链接，则为真
	-p $file 	如果 $file 是命名的管道或 FIFO，则为真
	-S $file 	如果 $file 是套接字，则为真
	-b $file 	如果 $file 是块特殊文件，则为真
	-c $file 	如果 $file 是字符特殊文件，则为真
	-u $file 	如果 $file 具有 setuid 位设置，则为真
	-g $file 	如果 $file 具有 setgid 位设置，则为真
	-k $file 	如果 $file 具有 sticky 位设置，则为真
	-t $file 	如果 $file 文件句柄对 tty 打开，则为真
	-T $file 	如果 $file 是文本文件，则为真
```
是否存在
say -e '/etc/passwd';
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
1
----------------------------
上次修改距今天数
say -M '/etc/passwd';
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
74.6710532407407
----------------------------
更多的测试

my $file="err.txt";
print "File is readable\n" if -r $file;
print "File is writeable\n" if -w $file;
print "File is executable\n" if -x $file;
print "File is a regular file\n" if -f $file;
print "File is a directory\n" if -d $file;
print "File is text file\n" if -T $file;
printf "File was last modified %f days ago.\n", -M $file;
print "File has read, write, and execute set.\n" if -r $file && -w _ && -x _;
stat($file);
print "File is a set user id program.\n" if -u _;
print "File is zero size.\n" if -z_;

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
File is readable
File is writeable
File is a regular file
File is text file
File was last modified 0.009444 days ago.
File is zero size.
```

## 删除文件
返回删除成功数量
```
unlink '23,';
unlink qw/aa bb/;
unlink glob '*.t';
```
## 重定名
```
rename 'oldfile', 'newfile';
```
## pack 和 unpack 函数
用于二进制文件
# 模块操作
## 安装
其实也可以手动安装，也就是下载解压编译安装...待完善
```
[huawei@n148 perl]$ cpan -a						# 查看已安装模块
[huawei@n148 perl]$ cpan -i Term::ANSIScreen	# 安装模块
[huawei@n148 perl]$ preldoc Term::ANSIScreen	# 查看模块说明
```
## 导入模块
```
use File::Basename;
```
## 导入函数
```
use File::Basename qw/basename/;
say File::Basename::basename __FILE__;

还可以不导入函数
use File::Basename qw//;
```
## 查看文档
```
[huawei@n148 perl]$ perldoc File::Basename
```
## File::Basename
```
只取文件名 my $filename_only = basename($filename);
取路径部分 my $path_only = dirname($filename); 

my ($base, $path, $suffix) = File::Basename::fileparse( __FILE__); 
say "$base, $path, $suffix";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
1.pl, /home/huawei/playground/perl/, 

第二个参数可以用于正则过滤
my ($base, $path, $suffix) = File::Basename::fileparse( __FILE__, qr{.pl}); 
say "$base, $path, $suffix";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
1, /home/huawei/playground/perl/, .pl
```


# 句柄
句柄实际上包含文件、管道、进程和套接字的读写。
## open、写入文本、close
```
最简单方式：向文件中追加一段数据后关闭文件。写入可以使用print或say  

use autodie;	# 当捕获到某些错误时，会自动调用die结束程序
open LOG,">>a.log";
print LOG "haha, hello world\n";
close LOG;
```
下面几种常用open方式，实际上perl会在句柄超出范围或程序结束时，自动关闭。
* 读是 <
* 写是 >
* 带有+则是读写都有，具体见下面
```
open(FD,"> filename") 以覆盖写入的方式打开文件
open(FD,">> filename") 以追加写入的方式打开文件
open FD,">>","filename" 同上，也可以不用括号
open(FD,"filename") 读文件
open(FD,"< filename") 同上，默认的模式就是输入
open(FD,"process |") 读进程结果
open(FD,"| to process") 往进程中写数据，不过对WINDOWS系统写会有问题

打开文件进行读写的操作
open(FD,"+<filename") 先读后写
open(FD,"+>filename") 先写后读
open(FD,"+>>filename") 先追加后读
```
```
$line=<FD> 获取起始行
while (defined $line=<FD>) {}	如果要遍历整个文件
@lines=<FD> 将整个文件放入lines数组中

use FileHandle;	可以使用FileHandle包，可以避免变量覆盖的现象。此方法实验未成功
$fileHandleName= new FileHandle("filename");
line=<fileHandleName>;
```
写文本文件
```
open my $rocks_fh, '>>', 'rocks.txt' or die "could not open rocks.txt: $!";
foreach my $rock (qw /s l g/){
	say $rocks_fh $rock;
}
print $rocks_fh "end\n";
close $rocks_fh;
```


## 读取指定字节数 read
可以从指定的文件句柄中将指定数目的字节读入到变量里。如果是从标准输入中读
取的话，相应的文件句柄就是 STDIN。read 函数将返回读到的字节数目。
```
my($buffer) = "";
open(FILE, "/etc/services") or	 # 打开标准输入
     die("Error reading file, stopped");
while(read(FILE, $buffer, 8) )
{
   print("$buffer\n");
}
close(FILE);

运行结果打印时刻一行最多8可字符。
```
## 读一个字符 getc
getc 函数能从键盘或者文件中获得单个字符。如果碰到 EOF，getc 函数会返回空字符串。案例见tell


## 读取文本文件
将文件作为perl命令行的参数，perl会使用<>去读取这些文件中的内容。
```
由于<>和<STDIN>读取文件、读取标准输入的时候总是自带换行符，很多时候这个自带的换行符都会带来格式问题。所以，有必要在每次读取数据时将行尾的换行符去掉，使用chomp即可

脚本内容：
foreach (<>){
    chomp;
    say "$_";
}

[huawei@n148 perl]$ perl 1.pl /etc/passwd

-------------------

打印指定文件的匹配行

open(FILE, "err.txt") || die "Can't open datebook: $!\n";
while(<FILE>) {
	print if /abc/;		# 这里每次迭代的行内容是$_，打印的也是$_
}
close(FILE);

与上面的效果一样，只是更啰嗦
while(my $line = <FILE>) {
	print "$line" if $line =~ /abc$/;
}
-------------------

文本文件转字符串，并正则替换每行前缀内容

my $filename="err.txt";
open FILE, $filename or die "Can't open '$filename':$|";
my $lines = join ' ', <FILE>;
$lines =~ s/^/$filename: /gm;
say $lines;
close FILE

-------------------

文本文件转数组，并打印内容与行数

open(FILE, "err.txt") || die "Can't open datebook: $!\n";
my @lines = <FILE>;
print @lines;
print $#lines + 1, "\n";
close(FILE);
```
## 读取标准输入 STDIN
使用一对尖括号格式的< STDIN>来读取来自非文件的标准输入，例如来自管道的数据，来自输入重定向的数据或者来自键盘的输入，< STDIN>读取的输入会自带换行符，所以print输出的时候不要加上额外的换行符。如需去除行尾的换行，使用chomp
```
读取一行：
my $data=<STDIN>;
say "$data";

循环读取多行：
while (defined($_=<STDIN>)){
	print "I saw $_";
}

这个方式仅能用于命令行，交互时候不好使。。。
foreach (<STDIN>){
    say "$_";
}

[huawei@n148 perl]$ echo "aaa\nbbb"|perl 1.pl 
aaa\nbbb

[huawei@n148 perl]$ 
```
## 输出与格式化 print、printf、sprintf、say
* print 不带\n；
* say 自带\n，必须结合use 5.10才能使用；
* printf 格式化输出字符串；
* sprintf 只格式化，无print功能。

printf sprintf常用格式符含义
* %%         百分号
* %s         字符串
* %d         整型数字
* %f         浮点型数字
* %e         科学计算法

%s %d %f %e可以设置显示字符宽度，补位字符（字符宽度不够时用于补齐的字符），小数位数。
```
print "hah\n";
say "hah1"; #say自带\n，必须使用use 5.010
printf "hah2\n";
printf "%d\n", 3.1415126; #输出 3
printf "%010d\n", 3.1415126; #输出 0000000003，字符宽度为10，向右对齐，宽度不足用0补齐，默认用空格补齐

#%010.2f
#0      设置字符宽度补齐字符
#10     设置字符宽度为10
#.2     设置显示2位小数
#f      输出浮点型
printf "%010.2f\n", 3.1415126;	# 0000003.14

printf "%d%%\n", 3.1415126; #输出 3%
printf "%010.3e\n", 23450000;	# 02.345e+07
printf "%010s\n", "haha";	#输出000000haha，字符宽度为10，向右对齐，宽度不足用0补齐，默认用空格补齐

#sprintf
my $result = sprintf("%010d",3.1415126); #()内方法类似于printf, 内容是  0000000003
print "$result\n";

```

## 二进制读写 binmode
binmode用以指定使用二进制方式进行读写  
演示png复制文件
```
open IN_FD,"1.png";
open OUT_FD, " > copy.png";
binmode(IN_FD);
binmode(OUT_FD);
while(read(IN_FD, my $buffer,1024)){
	 print OUT_FD $buffer;
}
close(IN_FD);
close(OUT_FD);
```
## 设定当前输出句柄 select
在select指定句柄后，随后输出在默认情况下，会输出到指定的句柄
```
open(FD,"> newfile");
select(FD);
print "test"; #将test添加到newfile中
select(STDOUT);
print "ok";  #将ok输出到屏幕
```
## 文件锁 flock
* 如要避免两个程序同时写同一个文件，用户可以先为该文件加锁，然后由一个程序单独访问它，并在使用完毕后再对文件进行解锁。
* flock 函数含有两个参数：文件句柄，以及文件锁操作。
* 在非 UNIX 系统上，这些文件锁操作可能是无效的。

文件锁操作类型：
* 共享锁	lock_sh 1
* 排他锁	lock_ex 2
* 非阻塞锁	lock_nb 4 
* 解除锁	lock_un 8
```
演示在多进程中使用排他锁（LOCK_EX）对数据源进行串行操作。

use Fcntl qw(:flock);	# 使用宏名称，否则就得使用数字。。。
use POSIX qw(strftime);

open (FD, " < err.txt") or die "$!\n";
flock(FD, LOCK_EX);
print "Yeah i get the lock by pid=$$ at ", cur_time(), "\n";
sleep 10;
flock(FD, LOCK_UN);
print "Oops i lose the lock by pid=$$ at ", cur_time(), "\n";
close FD;

sub cur_time {
      strftime "%H:%M:%S", localtime;
}

使用2个终端分别运行，可以看到第二个终端会阻塞直到第一个释放锁后，第二个才能获取锁。
```
使用flock.不会影响其他任何不使用flock的Script. 下面是掌握locks的五个规则:
* Locks只会影响其他locks. 不会阻止进程打开，读，写，删除文件等操作。
* 每个打开的文件只能有一个lock.
* 如果一个进程有LOCK_EX （独占）lock，则其他进程只能等待知道该进程释放后取得。（补充：包括请求LOCK_EX和LOCK_SH lock)
* 如果一个进程有LOCK_SH （共享）lock，则任何试图取得LOCK_EX的进程将失败，但是可以同时取得LOCK_SH lock;
* 对flock请求直到获得之后才会返回，除非lock是用 LOCK_SH| LOCK_NB（非封锁 non-block）取得的，这时一旦其他进程已经有lock，将直接返回错误信息。（关于这段的实验总结：如果是同时读两个文件并都使用flock函数，如果从前一个进程的LOCK_EX lock申请LOCK_EX lock，或者如果前一个进程是LOCK_SH lock,申请LOCK_EX lock都需要等待第一个进程结束。如果第一个进程是LOCK_SH lock，第二个进程申请LOCK_SH lock可以马上得到。）


## 移动文件指针 seek
可以让文件指针指向到指定位置

seek (filevar, distance, relative_to);
* filevar，文件指针
* distance，移动的字节数，正数向前移动，负数往回移动
* reletive_to，值可为
  * 0：SEEK_SET，从文件头开始移动
  * 1：SEEK_CUR，从当前位置移动
  * 2：SEEK_END，从文件末尾移动
* 运行成功返回真（非零值），失败则返回零
* 常与tell函数合用。
```
[huawei@n148 perl]$ cat char.txt 
ABCDEFGHIJKLMNOPQRSTUVWXYZ
-------------------------------------
这个图配合下面的代码逻辑演示SEEK_SET的作用

         +--------------------------  0: Initially.
         |         +---------------- 10: After seek($fh, 10, SEEK_SET).
         |         |    +----------- 15: After reading "KLMNO".
         |         |    |    +------ 20: After seek($fh, 20, SEEK_SET).
         |         |    |    |
         v         v    v    v     
file:    ABCDEFGHIJKLMNOPQRSTUVWXYZ
indexes: 01234567890123456789012345


use Fcntl qw( SEEK_SET );
open IN, "<char.txt";
seek(IN,10,SEEK_SET);
read IN, my $temp, 5;
say $temp;
seek(IN,20,SEEK_SET);
read IN, $temp, 5;
say $temp;
close(IN);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
KLMNO
UVWXY

-------------------------------------
这个图配合下面的代码逻辑演示SEEK_CUR的作用

         +--------------------------  0: Initially.
         |         +---------------- 10: After seek($fh, 10, SEEK_CUR).
         |         |    +----------- 15: After reading "KLMNO".
         |         |    |         +- 25: After seek($fh, 10, SEEK_CUR).
         |         |    |         |
         v         v    v         v 
file:    ABCDEFGHIJKLMNOPQRSTUVWXYZ
indexes: 01234567890123456789012345

use Fcntl qw( SEEK_CUR );
open IN, "<char.txt";
seek(IN,10,SEEK_CUR);
read IN, my $temp, 5;
say $temp;
seek(IN,10,SEEK_CUR);
read IN, $temp, 5;
say $temp;
close(IN);

因为char.txt内尾行有回车，所以这里也读了进来并打印除了回车换行

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
KLMNO
Z

[huawei@n148 perl]$ 
-------------------------------------

获得文件的字节数，可以先偏移到文件末尾，再查看当前偏移位置来查看

open FILE, "char.txt";
seek(FILE, 0, 2);
my $position = tell(FILE);
say $position;

因为文件行尾有回车所以是26+1字节
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
27

-rw-rw-r-- 1 huawei huawei    27 Dec 15 01:48 char.txt
```


## 获取文件指针位置 tell
返回文件指针的当前字节位置
```
open(fh, "<", "char.txt");  
my $position = tell(fh); 	# 新打开的文件位置是0
print("Position of read pointer before reading: $position\n"); 
print("First ten characters are: "); 
for(my $i = 0; $i < 10; $i++) 
{ 
    my $ch = getc(fh);  # 读1个char，读完后指针后移1位
    print" $ch"; 
} 
$position = tell(fh);  # 此时已经读了10个char，所以位置在10
print("\nCurrent Position: $position\n"); 
close(fh); 

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Position of read pointer before reading: 0
First ten characters are:  A B C D E F G H I J
Current Position: 10
```

## 管道 |
带有管道操作的 Perl 脚本是不能直接在不同系统间互相移植的。
* 输出过滤器  
	将perl代码的输出作为外部命令的输入
```
使用wc计算字符串中的字符数目并输出

open(MYPIPE, "| wc -w");
print MYPIPE "apples pears peaches";
close(MYPIPE);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
3

------------------------------------

两次过滤

open(FOO, "| sort| tr '[a-z]' '[A-Z]'");
open(DB, "err.txt"); #
while(<DB>){ print FOO ; }
close FOO;

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
AAA = '$LIBDIR/DBMAC'
AUTH_NOLOGIN_TIMES    = 3
BAR, WORLD!/NABC
BAR, WORLD!/NABC
PUT BEFORE THIRD LINE
PUT BEFORE THIRD LINEMALONGSHUAI GAOXIAOFANG
SHARED_PRELOAD_LIBRARIES = '$LIBDIR/DBMAC, $LIBDIR/PGAUDIT, $LIBDIR/PASSWORDCHECK'     # (CHANGE REQUIRES RESTART)
STRING
WORD MATCHING USING: THETHE
```
* 输入过滤器  
	外部命令的输出作为perl代码的输入
```

将date命令的输出传递给perl

open(INPIPE, "date |"); # Windows (2000/NT) use: date /T
my $today = <INPIPE> ;
print $today;
close(INPIPE);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
Wed Dec 15 04:37:59 CST 2021

--------------------------------

使用find命令查找文件并使用perl输出

open(FINDIT, "find . -name '*.txt' -print |") || die "Couldn't execute find!\n";
while( my $filename = <FINDIT> ){
 print $filename;
}
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
./rocks.txt
./mulu.txt
./ok.txt
./err.txt
./file.txt
./char.txt
```
## 检查文件尾 eof
* eof 函数用于检查是否到达文件末尾
* 如果对文件句柄的下一次读操作是发生在文件末尾，或者文件没有打开的话，函数返回1
* 如果没有提供参数，则 eof 函数将返回上一次文件读操作的eof状态
* 带括号的 eof 函数可用在循环体代码内，负责在读取上一个文件句柄时判断其文件末尾状态
* 如果不带括号的话，该函数则可检查每个已打开文件的末尾状态。

打印成功匹配的行到文件尾
```
open ( DB, "err.txt") || die "Can't open emp.names: $!";
while(<DB>){
	print if (/string/ .. eof); # 打印成功匹配的行到文件尾
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
string
bar, World!/nabc
Put before third linemalongshuai gaoxiaofang
auth_nologin_times    = 3
Put before third line
aaa = '$libdir/dbmac'
```
依次打印多个文件的内容，注意2个文件之间的打印逻辑
```
while(<>){	# 此处其实是迭代命令行参数中的每一个文件的所有行
	print "$.\t$_";  # 打印行号与行内容
	if (eof){	# 如果已到文件末尾位置，则打印一行 30 个短横线。
 		print "-" x 30, "\n";
		close(ARGV); # 关闭文件句柄，把 $. 的值重置为 1，以便下一次还能打开文件。在达到文件 file1 的末尾后，脚本将继续处理下一个参数file2，同样从第 1 行开始。
 	}
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"  err.txt rocks.txt 
1       bar, World!/nabc
2       word matching using: thethe
3       shared_preload_libraries = '$libdir/dbmac, $libdir/pgaudit, $libdir/passwordcheck'     # (change requires restart)
4       string
5       bar, World!/nabc
6       Put before third linemalongshuai gaoxiaofang
7       auth_nologin_times    = 3
8       Put before third line
9       aaa = '$libdir/dbmac'
------------------------------
1       s
2       l
3       g
4       end
------------------------------
```
## 原位编辑 -i
```
while(<ARGV>){ # Open ARGV for reading
	tr/a-z/A-Z/;	
	print; # Output goes to file currently being read in-place
	close ARGV if eof;
}

[huawei@n148 perl]$ perl -i.bak  1.pl  err.txt
执行上面后原err.txt内文件变为大写，且会生成err.txt.bak文件（内容为未修改之前）
```
## _ _ DATA _ _
有太多次写完一个perl程序，需要另外新建一个文件来测试，每次觉得很繁琐，但又不得不这么做。没想到原来perl已经提供了解决方案，这就是DATA。简单的应用案例如下
```
while (<DATA>) {
        print;
}

__DATA__
hello perl
hello world

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
hello perl
hello world
```
```
<IN>可以从打开的句柄IN中获得数据。
<STDIN>可以从标准输入接收数据。
<DATA> 文件句柄可以直接从执行它的脚本中获取数据，而不是从命令行或者从另一个文件里获取。
<DATA> 所读取的数据保存在每个脚本末尾的特殊常量__DATA__之后，同时这个常量也标志着脚本的逻辑结束  

use Data::Dumper;
my %organisms = ();

while(<DATA>){
    chomp;
    if(/^(\S+)\s+(\S+)$/){
        my $u=lc($1);
        my $v=lc($2);
        $u =~ s/ //g;
        $v =~ s/ //g;
        $organisms{$u}=$v;
    }
}
print Dumper(\%organisms);
__DATA__
hsa                 Human
ptr                 Chimp
na                  Orangutan
na                  Rhesus
na                  Marmoset
mmu                 Mouse
rno                 Rat

huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
$VAR1 = {
          'hsa' => 'human',
          'ptr' => 'chimp',
          'rno' => 'rat',
          'mmu' => 'mouse',
          'na' => 'marmoset'
        };
```
# 命令行
其实就是一行式。perl命令行加上"-e"选项，就能在perl命令行中直接写perl表达式。如
```
echo "malongshuai" | perl -e '$name=<STDIN>;print $name;'
强烈建议"-e"后表达式使用单引号包围，而不是双引号。
```
可以在pl代码中使用@ARGV获取命令行参数内容


http://blog.sina.com.cn/s/blog_494bf2bf0100lidf.html

https://blog.csdn.net/lw370481/article/details/17392679

https://blog.csdn.net/alivio/article/details/6898254?spm=1001.2101.3001.6650.7&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7Edefault-7.opensearchhbase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7Edefault-7.opensearchhbase

https://www.cnblogs.com/air-of-code/p/5990436.html



## 执行命令 -e
-e 允许 Perl 从命令行而不是脚本来执行 Perl 语句
```
[huawei@n148 perl]$ perl -e 'print "Hellow world\n" '
Hellow world

```
## 分为多行
```
[huawei@n148 perl]$ perl -e '
my $a="aaaaa";
$a=~s/a/c/g;
print $a,"\n";'
ccccc	# 打印出此行
```
## 处理每一行 -n
遍历文件每一行进行处理，并不是打印每一行，如需打印需自行处理
```
下面2种方式效果一样
[huawei@n148 perl]$ perl -e "while (<>){print }" file.txt
this is file
hello
你好

[huawei@n148 perl]$ perl -ne "print" file.txt
this is file
hello
你好

-----------------------------------

[huawei@n148 perl]$ perl -ne "print if /hello/" file.txt
hello
```
## 打印每一行 -p
先按行处理，然后打印结果，-p时候默认带有-n效果
```
[huawei@n148 perl]$ perl -pe '' file.txt
this is file
hello
你好

[huawei@n148 perl]$ perl -pe 's/hello/xxx/' file.txt
this is file
xxx
你好
```
## 开启warning -w
```
perl -w <scriptname>
```
* 用户亦可在程序的#!行下（若没有 #! 行则在脚本开头）添加下行内容：  
	use warnings;  
	这样就启用了所有可能的警告信息。
* 若要关闭警告信息，只需在脚本中添加下列内容：  
	no warnings；  
	这样便可关闭脚本中所有可能的警告信息。

## 分割本行到数组 -a
默认按照空白分隔符分割行并存储结果到默认数组@F，一般与-ne一起用
```
[huawei@n148 perl]$ perl -pe '' file.txt 
this is file
hello world
你 好
123

打印文件第一列与每行的列数
[huawei@n148 perl]$ perl -ane 'print $F[0],"\t", @F+0,"\n"' file.txt 
this    3
hello   2
你      2
123     1
```
## 指定行分隔符 -F
指定-a选项使用的分隔符，支持正则
```
[huawei@n148 perl]$ perl -pe '' file.txt 
123@456@789
@123@456@789@
@@@
@

打印每行分割后的前2个与总个数，仔细看分割后的总数理解分割原理

[huawei@n148 perl]$ perl -F'@' -ane 'print $F[0],",",$F[1], ",",@F+0,"\n"' file.txt 
123,456,3
,123,5
,,4
,,0
```

## 去除行尾换行 -l
对所有输入的命令进行chomp,即去除\n;同时对所有输出数据自动附件\n
```
这里使用-l替代了-F案例中手动打印行尾\n

[huawei@n148 perl]$ perl -F'@' -lane 'print $F[0],",",$F[1], ",",@F+0' file.txt 
123,456,3
,123,4
,,0
,,0
```
## 更新原文件 -i
启动原文编辑功能， 这一点可以代替sed 操作命令
```
[huawei@n148 perl]$ perl -pe '' file.txt 
123@456@789
@123@456@789@
@@@
@

下面操作后原文件就会被改变了
[huawei@n148 perl]$ perl -i -pe 's/@/*/g' file.txt

下面的会将原文件先备份再更新
[huawei@n148 perl]$ perl -i.bak -pe 's/@/*/g' file.txt 
```
## 导入模块 -M
Perl单行命令可以使用perl的模块，如使用sum函数的模块计算@F求和
```
[huawei@n148 perl]$ perl -pe '' file.txt
1 2 3
4 5 
6

[huawei@n148 perl]$ perl -MList::Util=sum -alne 'print sum @F' file.txt
6
9
6
```
## 类似awk的BEGIN与END
Perl也可以像awk一样使用END命令
```
[huawei@n148 perl]$ perl -pe '' file.txt
1 2 3
4 5 
6

统计单词数量

[huawei@n148 perl]$ perl -alne 'BEGIN{ print "start to run..."}; $t += @F; END { print $t,"\nend"}' file.txt
start to run...
6
end
```
## 查看前n行
```
[huawei@n148 perl]$ perl -pe 'exit if $.>10' /etc/passwd
```
## 一行式
https://github.com/vinian/perl1line.txt/blob/master/perl1line-ch.txt

# 智能匹配 ~~
检测某个元素是否在数组中的代码，使用智能匹配
```
my $x=2;
my @array;
if(@array~~$x)
{
print "$x is in the array";
}
else
{
print "$x is in the array";
}
```
不使用的样子
```
my $x=2;
my @array=(1,2,3);
my $flag=0;
for (@array)
{
if($x==$_)
{
$flag=1;
}
}

if($flag == 1){
print "$x is in the array";
}
else
{
print "$x is not in the array";
}
```
智能匹配操作的处理方式
```
$a      $b        Type of Match Implied    Matching Cod
======  =====     =====================    =============
Hash    Hash      hash keys identical      [sort keys %$a]~~[sort keys %$b]
Hash    Array     hash slice existence     grep {exists $a->{$_}} @$b
Hash    Regex     hash key grep            grep /$b/, keys %$a
Hash    Any       hash entry existence     exists $a->{$b}
Array   Array     arrays are identical[*]
Array   Regex     array grep               grep /$b/, @$a
Array   Num       array contains number    grep $_ == $b, @$a
Array   Any       array contains string    grep $_ eq $b, @$a
Any     undef     undefined                !defined $a
Any     Regex     pattern match            $a =~ /$b/
Code()  Code()    results are equal        $a->() eq $b->()
Any     Code()    simple closure truth     $b->() # ignoring $a
Num     numish[!] numeric equality         $a == $b
Any     Str       string equality          $a eq $b
Any     Num       numeric equality         $a == $b
Any     Any       string equality          $a eq $b

~~两边的操作数可以互换，不影响结果

例子				匹配方式
%a ~~ %b			哈希的键是否一致
%a ~~ @b			至少 %a 中的一个键在列表@b中
%a ~~ /Fred/		至少一个键匹配给定的模式
%a ~~ 'Fred'		哈希中某一指定键$a{Fred}是否存在 $a{Fred}
@a ~~ @b			数组是否相同
@a ~~ /Fred/		有一个元素匹配给定的模式
@a ~~ 123			至少有一个元素转化为数字后是123
@a ~~ 'Fred'		至少有一个元素转化为字符串后是'Fred'
$name ~~ undef		$name确实尚未定义
$name ~~ /Fred/		模式匹配
123 ~~ '123.0'		数字和字符串是否相等
'Fred' ~~ 'Fred'	字符串是否相同
123 ~~ 456			数字是否相等

```


检测哈希是否包含k
```
my %data = ('google'=>'google.com', 'runoob'=>'runoob.com', 'taobao'=>'taobao.com');
say 'find' if %data ~~ /oo/;
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
find
```

# given-when
如果在default之前的when语句使用了continue，Per就会继续执行default语句
```
检测单个
given( $ARGV[0] ) {
  when( $_ ~~ /fred/i ) { say 'Name has fred in it'; continue }
  when( $_ ~~ /^Fred/ ) { say 'Name starts with Fred'; continue }
  when( $_ ~~ 'Fred' ) { say 'Name is Fred'; break } 
  default { say "I don't see a Fred" } 
}


多个项目的when匹配
my @names = ("google", "runoob", "taobao", "fred");
foreach ( @names ) {
  say("\nProcessing $_");
  when( /fred/i ) { say 'Name has fred in it'; continue }
  when( /^Fred/ ) { say 'Name starts with Fred'; continue }
  when( 'Fred' )  { say 'Name is Fred'; }
  say("Moving on to default...");
  default { say "I don't see a Fred" }
}
```
# List::Util模块
# Regexp::Common