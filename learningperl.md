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

三个也行
($alpha, $beta, $production) = qw(January March August);
# move beta       to alpha,
# move production to beta,
# move alpha      to production
($alpha, $beta, $production) = ($beta, $production, $alpha);
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
## 常量，预定义宏
定义常量
```
use constant COUNT => 100;
print COUNT, "\n";

use constant WEEKDAYS => qw (Sunday Monday Tuesday Wednesday Thursday Friday Saturday);
print "Today is ", (WEEKDAYS)[1], "\n";

use constant {
    AAA => "aaa",
    BBB=> "bbb",
    MIN_TOTAL => 12,
    SCORE_PASS => 90,
    SCORE_RED => 70,
};
print AAA;
print SCORE_PASS;
```
实现预定义宏的效果
```
use constant MYDEBUG => 1;
warn "os: $Config{osname}, warn file: $fname", "\n" if MYDEBUG;
```
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
## 位操作
用下面的案例演示了二进制用于mask标志位的常用方式
```
#!/usr/bin/perl

my $write=0;		# 	00000000
my $red=1;			#	00000001
my $green=1<<1;		#	00000010
my $blue=1<<2;		#	00000100

sub setmask{
	my $val=shift;
	my $col=shift;
	$val |=  $col;    # set bit
	return $val;
}

sub unsetmask{
	my $val=shift;
	my $col=shift;
	$val &=  ~$col;    # clear bit
	return $val;
}

sub show{
	my $val=shift;
	my $Str = sprintf("%08b",$val);
	print $Str, "\n";
	if ($val == $write)  # check bit
	{
		print "is write", "\n";
	}
	if ($val & $red)  # check bit
	{
		print "has red", "\n";
	}
	if ($val & $green)  # check bit
	{
		print "has green", "\n";
	}
	if ($val & $blue)  # check bit
	{
		print "has blue", "\n";
	}
}

my $v1=$write;
$v1 =setmask($v1, $green);
$v1 =setmask($v1, $red);
$v1 =setmask($v1, $blue);
show($v1);

$v1 =unsetmask($v1, $red);
$v1 =unsetmask($v1, $green);
show($v1);

$v1 =unsetmask($v1, $blue);
show($v1);


[huawei@n148 perl]$ perl "/home/huawei/playground/perl/2.pl"
00000111
has red
has green
has blue
00000100
has blue
00000000
is write
```
下面的案例演示了不同等级的log
```
use strict;
use constant WEB => 0x0001; # 1
use constant SQL => 0x0010; # 2
use constant REG => 0x0100; # 4
use constant DEBUG => WEB | REG;

sub dbgprt
{
	my $type = shift;
	warn "***DEBUG***\t", shift if DEBUG & $type;
}

dbgprt(WEB, "WEB");
dbgprt(SQL, "SQL");
dbgprt(REG, "REG");
dbgprt(DEBUG, "DEBUG");


[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
***DEBUG***     WEB at /home/huawei/playground/perl/0.pl line 11.
***DEBUG***     REG at /home/huawei/playground/perl/0.pl line 11.
***DEBUG***     DEBUG at /home/huawei/playground/perl/0.pl line 11.
```

## 除法保留小数位
```
my $totle1=sprintf("%.2f", $a/$b*100);
print "$totle1 %";
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
## q,qw,qr,qx,qq
- q 为字符串添加单引号 q{abcd} 结果为 'abcd'
```
my $v=aaaaa;
my $someword = q~i 've some $v money~; 
print $someword, "\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
i 've some $v money
```
- qq 为字符串添加双引号 qq{abcd} 结果为 "abcd"
```
my $v=aaaaa;
my $someword = qq/i 've some $v money/; 
print $someword, "\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
i 've some aaaaa money
```
- qw 代表用空格来分隔元素,得到列表
```
my @list = qw/perl Regular network web/;
$,=',';
print @list ,"\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
perl,Regular,network,web,
```
- qx 为字符串添加反引

```
print qx{date};
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Mon Dec 20 01:09:58 CST 2021
```
- qr 用于创建模式字符串（注意不是整个正则）
```
my $str = "Who are Who you?";
my $pa=qr/W\w+/;
my $count=$str =~ s/$pa/What/g; 
print "$str\t$count\n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
What are What you?      2
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
## BEGIN、END
BEGIN是在Perl语言运行最开始运行的块，END是在Perl语言运行最后运行的块，和块所在位置无关
```
#!/usr/bin/perl
print"pid=$$\n";  
print"pname=$0\n";  
print"Startmainrunninghere\n";  
BEGIN{print"BEGIN\n";}  
END{print"END\n";}  

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
BEGIN
pid=30241
pname=/home/huawei/playground/perl/2.pl
Startmainrunninghere
END
```
## our、local、my、state
### 包域全局 our
* our操作符用于显式地创建包作用域变量
* 如果全局变量已存在，则our的作用是声明这个全局变量（类似于C中的extern）
```
# 关键字our  
 our $Scalar =1;                  #全局, 作用域为包  
 sub Subroutine{  
     our $Scalar =2;              #全局, 作用域为包  
     $Scalar +=1;  
     print $Scalar;                   
 }  
 &Subroutine;                     #输出3  
 &Subroutine;                     #输出3   
 print $Scalar;                   #输出3
```
### 临时全局 local
* local将全局变量临时借用为局部
* local操作符需配合our操作符使用（或其他包中的全局变量），用于产生一个局部变量的效果
* local只能声明已定义的全局变量，被my定义的变量是不可以被local声明的，即local本身不能创造变量
* local变量是在运行时起作用，它会将参数的值保存在一个运行栈中，当执行线程离开所在作用域时，原先作用域暂存的变量会被恢复

```
# 关键字local  
 our $Scalar =1;                  #全局, 作用域为包  
 sub Subroutine{  
     local $Scalar =2;            #临时全局变量,  作用域为子程序内部  
     $Scalar +=1;  
     print $Scalar;                   
 }  
 &Subroutine;                     #输出3  
 &Subroutine;                     #输出3   
 print $Scalar;                   #输出1
```
### 私有局部 my
* my操作符用于创建词法作用域变量，通过my创建的变量，存活于声明开始的地方，直到闭合作用域的结尾
*  闭合作用域指的可以是一对花括号中的区域，可以是一个文件，也可以是一个eval字符串
* my是编译时在私有符号表中创建新变量，这个变量在运行时无法使用名字进行独立访问，即它不存在于包符号表中（非全局）
* 当闭合作用域里的my变量与外层变量重名时，当前my变量有效，当退出作用域时，外层变量值不变
```
# 关键字my  
 my $Scalar =1;                   #私有局部变量, 作用域为当前文件  
 sub Subroutine{  
     my $Scalar =2;               #私有局部变量, 作用域为花括号  
     $Scalar +=1;  
     print $Scalar;                   
 }  
 &Subroutine;                     #输出3  
 &Subroutine;                     #输出3   
 print $Scalar;                   #输出1
```
### 持久局部 state
* state操作符功能类似于C里面的static修饰符，它与my不同的是，my变量在退出闭合作用域后其值不存在了，而state变量的值会被保留
* state仅能创建闭合作用域为子程序内部的变量
* state是从Perl 5.10 开始引入的，所以使用前必须加上use 5.010或更高版本指令
* state可以声明标量、数组、哈希。但在声明数组和哈希时，不能对其初始化（至少Perl 5.14 不支持）
```
# 关键字state  
 my $Scalar =1;                   #私有局部变量, 作用域为当前文件  
 sub Subroutine{  
     state $Scalar =2;            #持久局部变量, 作用域为子程序内部  
     $Scalar +=1;  
     print $Scalar;                   
 }  
 &Subroutine;                     #输出3  
 &Subroutine;                     #输出4   
 print $Scalar;                   #输出1
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
## 静态变量 state
state操作符功能类似于C里面的static修饰符，state关键字将局部变量变得持久。state也是词法变量，所以只在定义该变量的词法作用域中有效
* state仅能创建闭合作用域为子程序内部的变量
* state是从Perl 5.9.4开始引入的，所以使用前必须加上 use
* state可以声明标量、数组、哈希。但在声明数组和哈希时，不能对其初始化（至少Perl 5.14不支持）
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

```
use strict;
use warnings;
use feature ":5.10";

main(@ARGV);

sub main
{
    my $i = 5;
    increment($i);
    increment($i);
    increment($i);
    increment($i);
}

sub increment{
    state $n = shift;
    say ++$n;
}

sub error
{
    my $e = shift || 'unkown error';
    my $me = ( split(/[\\\/]/, $0 ) )[-1];
    print("$me: $e\n");
    exit 0;
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
遍历数组的引用
my @fruits = ( "Apple", "Blackberry" );
my $fruit_ref = \@fruits;
foreach my $fruit (@$fruit_ref) {
    print "$fruit\n";
}
for (my $i=0; $i <= $#$fruit_ref; $i++) {
    print "$fruit_ref->[$i]\n";
}


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
* 闭包是封闭于外部词法环境之上的函数
* 计算机科学中的术语--高阶函数，指的就是函数的函数.
* 闭包使用词法变量，并且在超出词法作用域后还可以读取该词法变量。
* 闭包除了抽象结构化细节外还可以做更多事。它允许定制特定的行为。从某种意义上讲它还可以 去掉不必要的泛化。
* 闭包是除使用全局变量外，在函数调用间保证数据持续性的简易、有效且安全的的方法

简单应用
```
# 现在设想你要迭代一个列表，但是又不想自己来管理迭代器，你可以这样做：返回一个函数，并且在调用时，迭代下一个项目。

sub make_iterator
{
	my @items = @_;
	my $count = 0;
	return sub
	{
		return if $count == @items;
		return $items[ $count++ ];
	}
}
my $cousins = make_iterator(qw(Rick Alex Kaycee Eric Corey Mandy Christine Alex));
say $cousins->() for 1 .. 6;

尽管make_iterator()已经结束并返回，但是函数中的匿名函数已经和里面的环境关联起来了，（还记得Perl的内存管理机制，引用计数么），所以仍然能够访问。
每次调用make_iterator()都会产生独立的词法环境，匿名函数创建并保持这个独立的环境。（所以每次产生的匿名函数环境互不影响）

```

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
```

sub gen_fib
{
	my @fibs = (0, 1);
	return sub
	{
		my $item = shift;
		if ($item >= @fibs)
		{
			for my $calc (@fibs .. $item)
			{
				$fibs[$calc] = $fibs[$calc - 2] + $fibs[$calc - 1];
			}
		}
		return $fibs[$item];
	}
}

my $fib = gen_fib();
say $fib->(12);
```
# 流程控制与智能匹配

## if、unless、？：
```
COND可以是任意一个表示布尔值的值或表达式（即可以使用&&进行多条件组合），它是一个标量上下文。Perl中任何一个需要进行条件判断的地方都是标量上下文。

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

# 单行式，使用$_代表循环变量，然后进行say之
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
## given-when
如果在default之前的when语句使用了continue，Per就会继续执行default语句
```
检测单个
use 5.010; # 重要 这句没有可能会编译失败
given( $ARGV[0] ) {
  when( $_ ~~ /fred/i ) { say 'Name has fred in it'; continue }
  when( $_ ~~ /^Fred/ ) { say 'Name starts with Fred'; continue }
  when( $_ ~~ 'Fred' ) { say 'Name is Fred'; break } 
  default { say "I don't see a Fred" } 
}


多个项目的when匹配
use 5.010; # 重要 这句没有可能会编译失败
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
```
use strict;
use warnings;
use feature ":5.10";

main(@ARGV);

sub main
{
    my $s='jimi hendrix';#5
    given($s){
        when(undef){say'$s is undefined'}
        when('jimi'){say'$s is musician'}
        when(/jimi/){say'$s maybe a muscian'}
        when([1,3,5,7,9]){say'$s is odd number'}
        default{say '$s is something else!'}
    }
}

sub error
{
    my $e = shift || 'unkown error';
    my $me = ( split(/[\\\/]/, $0 ) )[-1];
    print("$me: $e\n");
    exit 0;
}
```
## 智能匹配 ~~
对两个操作数进行比较，并在互相匹配时返回真值。定义的模糊恰好反映了此操作符的智能程度，比较操作由操作数两者共同决定。之前的given（Given/When）进行的就是隐式智能匹配。


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

# 常用函数
## main函数
貌似需要5.10特性
```
use strict;
use warnings;
use feature ":5.10";

main(@ARGV);

sub main
{
    say "This is the Perl 5.10 new features exercise file.";
    say "this is another line";
}

sub error
{
    my $e = shift || 'unkown error';
    my $me = ( split(/[\\\/]/, $0 ) )[-1];
    print("$me: $e\n");
    exit 0;
}
```
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


my $X=10;
my $Y=20;
my $random = int( rand( $Y-$X+1 ) ) + $X;
#-----------------------------
$random = int( rand(51)) + 25;
print "$random\n";
#-----------------------------
my @array=qw/a ab abc/;
my $elt = $array[ rand @array ];	随机数组元素
print "$elt\n";
#-----------------------------
my @chars = ( "A" .. "Z", "a" .. "z", 0 .. 9, qw(! @ $ % ^ & *) );
print "@chars\n";
my $password = join("", @chars[ map { rand @chars } ( 1 .. 8 ) ]);	随机字符密码
print "$password\n";


其他随机
use Math::TrulyRandom;
$random = truly_random_value();

use Math::Random;
$random = random_uniform();
```

## 进制转换

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
二进制与十进制的转换
```
sub dec2bin {
    my $str = unpack("B32", pack("N", shift));
    $str =~ s/^0+(?=\d)//;   # otherwise you'll get leading zeros
    return $str;
}
#-----------------------------
sub bin2dec {
    return unpack("N", pack("B32", substr("0" x 32 . shift, -32)));
}
#-----------------------------
my $num = bin2dec('0110110');  # $num is 54
print "$num\n";
my $binstr = dec2bin(54);      # $binstr is 110110
print "$binstr\n";

```

## pack 和 unpack 函数
功能很强大待细致研究，暂只用于ip地址打包
```
my ($a,$b,$c,$d)=split(/\./, '18.157.0.125');
print "$a\t$b\t$c\t$d\n";
my $packaddr=pack('C4',$a,$b,$c,$d);
print "$packaddr\n";
($a,$b,$c,$d)=unpack('C4',$packaddr);
print "$a\t$b\t$c\t$d\n";
my $addr=join('.',$a,$b,$c,$d);
print "$addr\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
18      157     0       125
�}
18      157     0       125
18.157.0.125
```
# 日期时间、计时器与sleep
## 取当前年月日时分秒
```
use Time::localtime;
my $tm = localtime;
printf("Dateline: %02d:%02d:%02d-%04d/%02d/%02d\n", $tm->hour, $tm->min, $tm->sec, $tm->year+1900, $tm->mon+1, $tm->mday);

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
Dateline: 14:56:43-2022/04/12
```
## 各种格式解析
```
use Time::Local;
my $date = "1998-06-03";
my ($yyyy, $mm, $dd) = $date =~ /(\d+)-(\d+)-(\d+)/;
print "Date was $mm/$dd/$yyyy\n";  # Date was 06/03/1998
my $epoch_seconds = timelocal(59, 29, 23, $dd, $mm, $yyyy);
print "$epoch_seconds\n";	# 新纪元秒数
print "Scalar localtime gives: ", scalar(localtime($epoch_seconds)), "\n";	# Fri Jul  3 23:29:59 1998
use POSIX qw(strftime);
print "strftime gives: ", strftime("%A %D", localtime($epoch_seconds)), "\n"; # Friday 07/03/98
my $STRING = localtime($epoch_seconds);
print "$STRING\n";	# Fri Jul  3 23:29:59 1998
$date = ParseDate($STRING);
use Date::Manip qw(ParseDate UnixDate);
my $datestr = UnixDate($date, "%a %b %e %H:%M:%S %z %Y");    # as scalar
print "Date::Manip gives: $datestr\n"; # Fri Jul  3 23:29:59 -0500 1998 这里zone显示了-0500可以不使用

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
Date was 06/03/1998
899479799
Scalar localtime gives: Fri Jul  3 23:29:59 1998
strftime gives: Friday 07/03/98
Fri Jul  3 23:29:59 1998
Date::Manip gives: Fri Jul  3 23:29:59 -0500 1998
```
## 日期加减
```
my $tm = localtime;
use Time::localtime;

use Date::Calc qw(Add_Delta_Days);	# 添加天数
printf("Dateline: %02d:%02d:%02d-%04d/%02d/%02d\n", $tm->hour, $tm->min, $tm->sec, $tm->year+1900, $tm->mon+1, $tm->mday);
my ($y2, $m2, $d2) = Add_Delta_Days($tm->year+1900, $tm->mon+1, $tm->mday, 30);
printf("Dateline: %02d:%02d:%02d-%04d/%02d/%02d\n", $tm->hour, $tm->min, $tm->sec, $y2, $m2, $d2);


my ($year, $month, $day) = Add_Delta_Days(1973, 1, 18, 55);
print "Nat was 55 days old on: $month/$day/$year\n";
# Nat was 55 days old on: 3/14/1973

#-----------------------------
use Date::Calc qw(Add_Delta_DHMS);	# 添加天数、小时、分钟、秒
my ($days_offset, $hour_offset, $minute_offset, $second_offset) = (10, 9, 80, 100);
my ($year2, $month2, $day2, $h2, $mm2, $s2) = Add_Delta_DHMS(
	$tm->year+1900, $tm->mon+1, $tm->mday, $tm->hour, $tm->min, $tm->sec,
   	$days_offset, $hour_offset, $minute_offset, $second_offset);
printf("Dateline: %02d:%02d:%02d-%04d/%02d/%02d\n", $h2, $mm2, $s2, $year2, $month2, $day2);				


my ($year3, $month3, $day3, $hh, $mm, $ss) = Add_Delta_DHMS(
    1973, 1, 18, 3, 45, 50, # 18/Jan/1973, 3:45:50 am
             55, 2, 17, 5); # 55 days, 2 hrs, 17 min, 5 sec
print "To be precise: $hh:$mm:$ss, $month3/$day3/$year3\n";
# To be precise: 6:2:55, 3/14/1973

```
## 两个日期的差
```
use Date::Calc qw(Delta_DHMS);
my @bree = (1981, 6, 16, 4, 35, 25);   # 16 Jun 1981, 4:35:25
my @nat  = (1973, 1, 18, 3, 45, 50);   # 18 Jan 1973, 3:45:50
my @diff = Delta_DHMS(@nat, @bree);
print "Bree came $diff[0] days, $diff[1]:$diff[2]:$diff[3] after Nat\n";
# Bree came 3071 days, 0:49:35 after Nat

my ($days, $hours, $minutes, $seconds) =
  Delta_DHMS( 1981, 6, 16, 4, 35, 25,  # earlier
              1973, 1, 18, 3, 45, 50); # later
print "$days, $hours, $minutes, $seconds\n";
($days, $hours, $minutes, $seconds) =
  Delta_DHMS( 1973, 1, 18, 3, 45, 50, # 先早后晚为正
			   1981, 6, 16, 4, 35, 25, );
print "$days, $hours, $minutes, $seconds\n";


use Date::Calc qw(Delta_Days);
@bree = (1981, 6, 16);      # 16 Jun 1981
@nat  = (1973, 1, 18);      # 18 Jan 1973
my $difference = Delta_Days(@nat, @bree);
print "There were $difference days between Nat and Bree\n";
# There were 3071 days between Nat and Bree

$days = Delta_Days( 1981, 12, 30, 1973, 11, 18);
print  $days, "\n";
$days = Delta_Days(1973, 11, 18, 1981, 12, 30); # 先早后晚为正
print  $days, "\n";
```
## 星期几、第几周、第几天
```
use Date::Calc qw(Day_of_Week Week_Number Day_of_Week_to_Text Day_of_Year);

my $year= 1981;
my $month = 6;         # (June)
my $day   = 16;

my $wday = Day_of_Week($year, $month, $day);
print "$month/$day/$year was a ", Day_of_Week_to_Text($wday), "\n";
## see comment above

my $wnum = Week_Number($year, $month, $day);
print "in the $wnum week.\n";
# 6/16/1981 was a Tuesday
# 
# in week number 25.


# you have $year, $month, and $day
# $day is day of month, by definition.
$wday = Day_of_Week($year, $month, $day);
$wnum = Week_Number($year, $month, $day);
my $dnum = Day_of_Year($year, $month, $day);
print "$wday $wnum $dnum \n";

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
6/16/1981 was a Tuesday
in the 25 week.
2 25 167 
```
## 搞精度计时器
```
use Time::HiRes qw(gettimeofday);
print "Press return when ready: ";
my $before = gettimeofday;
my $line = <>;	# 读取按键输入
my $elapsed = gettimeofday-$before;
print "You took $elapsed seconds.\n";
```
## 休眠 sleep

```
use strict;
use warnings;
use Time::HiRes qw(usleep nanosleep);
# 1 millisecond == 1000 microseconds
usleep(1000);
# 1 microsecond == 1000 nanoseconds
nanosleep(1000000);
```
还可以使用select或是高精度sleep
```
while (<>) {
    select(undef, undef, undef, 0.25);
    print "a\n";
}

use Time::HiRes qw(sleep);
while (<>) {
    sleep(0.25);
    print "a\n";
}
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


$string = '\n'; # two characters, \ and an n 注意这是2个字符而不是换行
$string = "\n"; # a "newline" character 这里是换行
```
双引号的都转义
```
my $a = 10;
say " $a ' \"  \\  \t  \n";
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
 10 ' "  \        

[huawei@n148 perl]$ 
```
## 简单多行字符串
可以使用单引号来输出多行字符串
```
$string = '
菜鸟教程
    —— 学的不仅是技术，更是梦想！
';
 
print "$string\n";
```
## 多行字符串 heredoc
当需要声明一个复杂的字符串时，可以用heredoc语法。下例中  <<'END_BLURB'  语法有3部分。
* 二个小于号标志着这里是heredoc语法
* 用单引号引起表示这段字符串不做内插，如果没有使用单引号默认就是双引号可以内插

```
$a = <<"EOF";
This is a multiline here document
terminated by EOF on a line by itself
EOF

上面就是定义了一个2行的字符串，“EOF”仅是个标志没有实际作用，就如同下面的“END_BLURB”


my $v=12345;
my $blurb =<<'END_BLURB';		# 这里模式单引号，下面的$v不会内插
He looked up. $v "Change is the constant on which they all
can agree. We instead, born out of time, remain perfect
and perfectly self-aware. We only suffer change as we
pursue it. It is against our nature. We rebel against
that change. Shall we consider them greater for it $v?"
END_BLURB

my $blurb2 =<<END_BLURB;		# 这里模式双引号，下面的$v会内插
He looked up. "$v Change is the constant on which they all
can agree. We instead, born out of time, remain perfect
and perfectly self-aware. We only suffer change as we
pursue it. It is against our nature. We rebel against
that change. Shall we consider them greater for it?" $v
END_BLURB

say $blurb;
say $blurb2;

sub some_function {
     my $ingredients =<<'END_INGREDIENTS';	# 这里定义了个字符串然后打印出来
     Two eggs
     One cup flour
     Two ounces butter
     One-quarter teaspoon salt
     One cup milk
     One drop vanilla
     Season to taste
END_INGREDIENTS

	say $ingredients;
}
some_function;


[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
He looked up. $v "Change is the constant on which they all
can agree. We instead, born out of time, remain perfect
and perfectly self-aware. We only suffer change as we
pursue it. It is against our nature. We rebel against
that change. Shall we consider them greater for it $v?"

He looked up. "12345 Change is the constant on which they all
can agree. We instead, born out of time, remain perfect
and perfectly self-aware. We only suffer change as we
pursue it. It is against our nature. We rebel against
that change. Shall we consider them greater for it?" 12345
	
     Two eggs
     One cup flour
     Two ounces butter
     One-quarter teaspoon salt
     One cup milk
     One drop vanilla
     Season to taste

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
演示\Q...\E的效果，看下例的正则表达式即可，2个like都可匹配成功
```
($ret, $stdout, $stderr) = $node->psql('postgres', "select * from pg_audit where adtclass='DDL';", extra_params => [ '-U', "dbaa" ]);
like($stdout, qr@\Qdbda|DDL|CREATE TABLE|TABLE|public.t1|create table t1(a int)\E@, 'test 4 find DDL');
like($stdout, qr@\Qdbda|DDL|DROP TABLE|TABLE|public.t1|drop table t1\E@, 'test 4 find DDL');
```
## ASCII与char的转换
* chr：ASCII码(或unicode码点)转换为对应字符
* ord：字符转换为ASCII码(或unicode码点)
```
my $char = 'E';
my $num  = ord($char);	# char转ascii码值
$char = chr($num);	# ascii码值转char
$char = sprintf("%c", $num); # slower than chr($num)
printf("Number %d is character %c\n", $num, $num);

my @ASCII = unpack("C*", qq/sample/);	# 将char数组转为对应的ascii码值的数组
print "@ASCII\n";
my @ascii_character_numbers = unpack("C*", "sample"); # same
print "@ascii_character_numbers\n";

my $word = pack("C*", qw/115 97 109 112 108 101/);	# 将码值的数组转为对应的char字符串
print "$word\n";
$word = pack("C*", @ascii_character_numbers); # same
print "$word\n";
$word = pack("C*", 115, 97, 109, 112, 108, 101);   # same
print "$word\n";

my $hal = "HAL";
my @ascii = unpack("C*", $hal);
print "@ascii\n";	# 打印每个ascii值
foreach my $val (@ascii) {
    $val++;  # add one to each ASCII value，这里使用的是引用，下面打印的ascii值变了
}
print "@ascii\n";
my $ibm = pack("C*", @ascii);
print "$ibm\n";             # prints "IBM"


[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
Number 69 is character E
115 97 109 112 108 101
115 97 109 112 108 101
sample
sample
sample
72 65 76
73 66 77
IBM
[huawei@n148 perl]$ 
```
## 迭代处理每一个字符
```
my @array = split(//, "sample"); # 拆分为字符数组
print "@array\n"; # s a m p l e
@array = unpack("C*", "sample"); # 转为ascii码数组
print "@array\n";	# 115 97 109 112 108 101
#--统计有多少种不同的字母---------------------------
my %seen = ();
my $string = "an apple a day";
foreach my $byte (split //, $string) {
    $seen{$byte}++;
}
# while ($string =~ /(.)/g) {	使用正则方式，效果一样
    # $seen{$1}++;
# }
print "unique chars are: ", sort(keys %seen), "\n";	#  adelnpy
```
## 控制大小写，实现驼峰
* lc：(lower case)将后面的字母转换为小写，是\L的实现
* uc：(uppercase)将后面的字母转换为大写，是\U的实现
* fc：(foldcase)和lc基本等价，但fc可以处理UTF-8类的字母
* lcfirst：将后面第一个字母转换为小写，是\l的实现
* ucfirst：将后面第一个字母转换为大写，是\u的实现
```
my $little = "bo peep";
my $big = uc($little);          # "bo peep" -> "BO PEEP"
print "$big\n";
$little = lc($big);             # "JOHN"    -> "john"
print "$little\n";
$big = "\U$little";             # "bo peep" -> "BO PEEP"
print "$big\n";
$little = "\L$big";             # "JOHN"    -> "john"
print "$little\n";
#-----------------------------
$big = "\u$little";             # "bo"      -> "Bo"
print "$big\n";
$little = "\l$big";             # "BoPeep"    -> "boPeep" 
print "$little\n";
#-----------------------------
my $beast   = "dromedary";
# capitalize various parts of $beast
my $capit   = ucfirst($beast);         # Dromedary
print "$capit\n";
$capit   = "\u\L$beast";            # (same)
print "$capit\n";
my $capall  = uc($beast);              # DROMEDARY
print "$capall\n";
$capall  = "\U$beast";              # (same)
print "$capall\n";
my $caprest = lcfirst(uc($beast));     # dROMEDARY
print "$caprest\n";
$caprest = "\l\U$beast";            # (same)
print "$caprest\n";

my $text = "thIS is a loNG liNE";
$text =~ s/(\w+)/\u\L$1/g;
print $text; # This Is A Long Line
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
```
my $gnirts   = reverse("sample");       # 翻转字符串	
print "$gnirts\n"; # elpmas
my @words = qw/one two three/;
my @sdrow    = reverse(@words);        # reverse elements in @words 翻转数组元素顺序
print "@sdrow\n"; #   three two one
my $confused = reverse(@words);        # reverse letters in join("", @words) 上面的结果连起来
print "$confused\n"; #   eerhtowteno
```
## 翻转一句话
```
my $string = 'Yoda said, "can you see this?"';
my @allwords    = split(" ", $string);
my $revwords    = join(" ", reverse @allwords);
print $revwords, "\n"; # this?" see you "can said, Yoda


my $revwords = join(" ", reverse split(" ", $string));	# 效果一样
$revwords = join("", reverse split(/(\s+)/, $string));	# 效果一样

```
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
my @bigarray = ();空得
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
```
print "标量上下文中的数组返回数组中元素数量以及最大下标\n";
my @foo = qw(water pepsi coke lemonade);
my $a = @foo;
my $b = $#foo;
print "$a\n";  #4
print "$b\n";  #3

print "输出@foo中元素的数量\n";
print scalar(@foo), "\n"; # scalar这个特殊的伪函数强制@foo在一个标量上下文中进行计算

print "标量上下文中的数组返回数组中元素的数量\n";
my @mydata = qw(oats peas beans barley);
if (@mydata){ #4 is true
 print "The array has elements!\n";
}

```
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

## 切片[]
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
print "遍历数组\n";
my @flavors = qw(chocolate vanilla strawberry mint sherbert );
for(my $index=0; $index<@flavors; $index++) {  # @flavors是在标量上下文中计算的，计算的结果是5
 print "$flavors[$index]\n";
}


print "遍历数组，迭代器\n";
foreach my $cone (@flavors) {  # $cone设置为@flavors中的各个值
 print "$cone\n";
}

可以同时遍历多个数组
my @a = ( .5, 3 ); 
my @b =( 0, 1, -100 );
foreach my $item (@a, @b) {
    $item *= 7;
}
print "@a\n@b\n";
```

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

## 调整size
grow or shrink @ARRAY  
```
$#ARRAY = $NEW_LAST_ELEMENT_INDEX_NUMBER;
$ARRAY[$NEW_LAST_ELEMENT_INDEX_NUMBER] = $VALUE;
```
```
my @people = qw(Crosby Stills Nash Young);
sub what_about_that_array {
    print "The array now has ", scalar(@people), " elements.\n";
    print "The index of the last element is $#people.\n";
    print "Element #3 is `$people[$#people]'.\n" if defined($people[$#people]);
	print "未定义\n" unless defined($people[$#people]);
};
what_about_that_array();
$#people--;
what_about_that_array();
$#people = 10_000;
what_about_that_array();
$#people = 4;
what_about_that_array();

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
The array now has 4 elements.
The index of the last element is 3.
Element #3 is `Young'.
The array now has 3 elements.
The index of the last element is 2.
Element #3 is `Nash'.
The array now has 10001 elements.
The index of the last element is 10000.
未定义
The array now has 5 elements.
The index of the last element is 4.
未定义
```
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
## 交集，并集与补集
取2个数组只属于各自的元素，以及2个数组的共同元素  
![](https://img-blog.csdn.net/20140507111726234)
```
use strict;
use warnings;
use Data::Dumper;

my @a = (1,2,3,4,5,6,7,8,);
my @b = (1,9,0,4,15,6,12,8);
my %hash_a = map{$_=>1} @a;
my %hash_b = map{$_=>1} @b;
my %merge_all = map {$_ => 1} @a,@b;
my @a_only = grep {!$hash_b{$_}} @a;
my @b_only = grep {!$hash_a{$_}} @b;
my @common = grep {$hash_a{$_}} @b;
my @merge = keys (%merge_all);
print "A only :\n";
print Dumper(\@a_only);
print "B only :\n";
print Dumper(\@b_only);
print "Common :\n";
print Dumper(\@common);
print "Merge :\n";
print Dumper(\@merge);
```

## 查找
返回index
```
$,=',';
sub findindex{
	my ($what,@arr)=@_;
	foreach(0..$#arr) {return $_ if $what == $arr[$_];}
	-1;
}
say findindex(3, qw/1 2 3 4 5/);
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
* 列表是一个由逗号分隔、包含一个或多个表达式的组
* 是标量的有序集合，列表指的是数据
* Perl中的列表不是数据类型，而是Perl在内部用来临时存放数据的一种方式，只能由Perl自行维护。
* 列表临时保存在栈中，当使用了列表数据后，这些列表数据就会出栈
* 列表和数组概念之间不可以交换。列表是值而数组是容器。
* 可以将Perl列表看作是一种特殊的底层可迭代对象，它看起来像数组，但不是数组。
```
my @arr = (11,22,33);  # 数组arr的元素来自于列表
```
* push与pop这类仅用于数组，元素排序、元素筛选、迭代遍历等操作，才用于列表
* 列表常见操作包括：grep、join、map、reverse、sort、unpack、x操作符执行列表重复，等等
* 标准库List::Utils中也提供了很多常见的列表操作，如reduce、first、any、sum、uniq、shuffle等。

## 列表直接量
下面()里的即是
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
大案例
```
my @lists = (
    [ 'just one thing' ],
    [ qw(Mutt Jeff) ],
    [ qw(Peter Paul Mary) ],
    [ 'To our parents', 'Mother Theresa', 'God' ],
    [ 'pastrami', 'ham and cheese', 'peanut butter and jelly', 'tuna' ],
    [ 'recycle tired, old phrases', 'ponder big, happy thoughts' ],
    [ 'recycle tired, old phrases', 
	'ponder big, happy thoughts', 
	'sleep and dream peacefully' ],
    );

foreach my $aref (@lists) {
    # 元素是列表，打印元素地址,元素内size，列表内容
	print "The item addr: " . $aref . ", size: " . @$aref . ", and value: @$aref\n";	
	commify_series(@$aref);
} 

sub commify_series {
    my $sepchar = grep(/,/ => @_) ? ";" : ","; # 遍历传入的列表，判断列表的元素中是否有带逗号的，有则返回；无则返回，
	print $sepchar, "元素个数：" , scalar(@_), "\n";
	# 下面这个是嵌套？：用于判断传入的列表的元素数量
    (@_ == 0) ? ''                                      :
    (@_ == 1) ? $_[0]                                   :
    (@_ == 2) ? join(" and ", @_)                       :  # 2个元素用and连接
                join("$sepchar ", @_[0 .. ($#_-1)], "and $_[-1]"); # 再多就用上面得到的分隔符连接所有元素
}

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
The item addr: ARRAY(0x2186a68), size: 1, and value: just one thing
,元素个数：1
The item addr: ARRAY(0x21a38d0), size: 2, and value: Mutt Jeff
,元素个数：2
The item addr: ARRAY(0x21a39a8), size: 3, and value: Peter Paul Mary
,元素个数：3
The item addr: ARRAY(0x21a39f0), size: 3, and value: To our parents Mother Theresa God
,元素个数：3
The item addr: ARRAY(0x21a3b58), size: 4, and value: pastrami ham and cheese peanut butter and jelly tuna
,元素个数：4
The item addr: ARRAY(0x2386190), size: 2, and value: recycle tired, old phrases ponder big, happy thoughts
;元素个数：2
The item addr: ARRAY(0x23f7e60), size: 3, and value: recycle tired, old phrases ponder big, happy thoughts sleep and dream peacefully
;元素个数：3
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
## 列表，数组和哈希的嵌套
http://blog.sina.com.cn/s/blog_48a587210101fwkd.html
```
Perl的数据结构中最常用到的两种类型数组和哈希。
1. 什么是数组（Perl语言入门（第五版）的解释）？
如果说Perl的标量代表的是单数(singular)，那么在Perl里代表复数(plural)的就是列表和数组。
列表(list)指的是标量的有序集合，而数组(array)则是存储列表的变量。在Perl里，这两个术语常常混用。不过更精确地说，列表指的是数据，而数组指的是变量。列表的值不一定要放在数组里，但每个数组变量都一定包含一个列表（即使该列表可能是空的）。

2. 什么是哈希（Perl语言入门（第五版）的解释）？
所谓的哈希就是一种数据结构，和数组的相同之处在于：可以容纳很多值（没有上限），并能随机存取。而区别在于：不像数组是以数字来检索，哈希是以名字来检索。也就是说检索用的键不是数字，而是保证唯一的字符串。
另一种看待哈希的方法，是将它想象成一大桶的数据，其中每个数据都有关联的标签。可以伸手到桶里任意取出一张标签，查看附着的数据是什么。但是桶里没有所谓的第一个元素，只有一堆数据。然而数组里，第一个元素为元素0.然后是元素1、2，等等。但是哈希里没有顺序，因此也没有第一，有的只是一些键/值对。这些键和值都是任意的标量，但是键总是会被转换成字符串。假如你以数字表达式50/20为键，那么它会被转换为一个含有三个字符的字符串"2.5"。


了解完了什么是数组、列表和哈希，下面我们开始分析它们是如何嵌套使用的：

Ⅰ. 列表嵌套列表（列表嵌套列表即为二维列表）
a. 列表的声明
方法一：
@list = (
["banana","apple","orange","pear"],
["cauliflower","lettuce","tomato","cucumber"],
["orange juice","apple juice","red tea","red wine"]
)

方法二：
@fruit = ("banana","apple","orange","pear")
@vegetable = ("cauliflower","lettuce","tomato","cucumber")
@drink = ("orange juice","apple juice","red tea","red wine")

正确方法：
@list = ([@fruit],[@vegetable],[@drink])
错误方法：
@list = (@fruit, @vegetable, @drink)
错误剖析：如果不用[]把每个子列表括起来，那么这个列表@list就相当于@fruit, @vegetable, @drink这三个列表拼接而成的一个一维列表相当于@list  = ("banana","apple","orange","pear","cauliflower","lettuce","tomato","cucumber","orange juice","apple juice","red tea","red wine")

方法三：
正确方法：
$list[0] = ["banana","apple","orange","pear"]
错误方法：
$list[0] = ("banana","apple","orange","pear")
错误剖析：这样的结果是$list[0] = "pear" 并且会报警告

b. 列表的访问
$list[0][0] = "banana"
$list[0] and @list[0] 两种表示都可以，但是 @list[0] 会报警告


Ⅱ. 列表嵌套哈希
a. 列表的声明
方法一：
@list = (
{
'fruit' => 'banana',
'vegetable' => 'cauliflower',
'drink' => 'orange juice'
},
{
'fruit' => 'apple',
'vegetable' => 'lettuce',
'drink' => 'apple juice'
},
{
'fruit' => 'orange',
'vegetable' => 'tomato',
'drink' => 'red tea'
},
{
'fruit' => 'pear',
'vegetable' => 'cucumber',
'drink' => 'red wine'
}
)

方法二：
%group1 = (
'fruit' => 'banana',
'vegetable' => 'cauliflower',
'drink' => 'orange juice'
)
%group2 = (
'fruit' => 'apple',
'vegetable' => 'lettuce',
'drink' => 'apple juice'
)
%group3 = (
'fruit' => 'orange',
'vegetable' => 'tomato',
'drink' => 'red tea'
)
%group4 = (
'fruit' => 'pear',
'vegetable' => 'cucumber',
'drink' => 'red wine'
)

正确方法：
@list = ({%group1}, {%group2}, {%group3}, {%group4})
错误方法：
@list = (%group1, %group2, %group3, %group4)
错误剖析：如果不用{}把每个子列表括起来，那么这个列表@list就相当于%group1, %group2, %group3, %group4 这四个哈希拼接而成的一个一维列表，相当于@list = ( 'fruit', 'banana', 'drink', 'orange juice', 'vegetable', 'cauliflower', 'fruit', 'apple', 'drink', 'apple juice', 'vegetable', 'lettuce', 'fruit', 'orange', 'drink', 'red tea', 'vegetable', 'tomato', 'fruit', 'pear', 'drink', 'red wine', 'vegetable', 'cucumber')

方法三：
正确方法：
$list[0] = {'fruit' => 'banana', 'vegetable' => 'cauliflower','drink' => 'orange juice'}
$list[0] = {%group1};
错误方法：
$list[0] = ('fruit' => 'banana', 'vegetable' => 'cauliflower','drink' => 'orange juice')
错误剖析：这样的结果是$list[0] = 'orange juice' 并且有警告
$list[0] = ['fruit' => 'banana', 'vegetable' => 'cauliflower','drink' => 'orange juice'] 
错误剖析：这样的结果是$list[0] = ['fruit','banana','vegetable','cauliflower','drink','orange juice']

b. 列表的访问
$list[0]{'fruit'} = 'banana' and $list[0]{fruit} are the same.
$list[0] = {'fruit' => 'banana','drink' => 'orange juice','vegetable' => 'cauliflower'}

Ⅲ. 哈希嵌套哈希
a. 哈希的声明
方法一：
my %hash = (
group1 => {
'fruit' => 'banana',
'vegetable' => 'cauliflower',
'drink' => 'orange juice'
},
group2 => {
fruit => 'apple',
vegetable => 'lettuce',
drink => 'apple juice'
},
group3 => {
'fruit' => 'orange',
'vegetable' => 'tomato',
'drink' => 'red tea'
},
group4 => {
fruit => 'pear',
vegetable => 'cucumber',
drink => 'red wine'
}
);

%hash = (
'group1', {'fruit', 'banana', 'drink', 'orange juice', 'vegetable', 'cauliflower'},
'group2', {'fruit', 'apple', 'drink', 'apple juice', 'vegetable', 'lettuce'},
'group3', {'fruit', 'orange', 'drink', 'red tea', 'vegetable', 'tomato'},
'group4', {'fruit', 'pear', 'drink', 'red wine', 'vegetable', 'cucumber'}
);

方法二：
%group1 = (
'fruit' => 'banana',
'vegetable' => 'cauliflower',
'drink' => 'orange juice'
)
%group2 = (
'fruit' => 'apple',
'vegetable' => 'lettuce',
'drink' => 'apple juice'
)
%group3 = (
'fruit' => 'orange',
'vegetable' => 'tomato',
'drink' => 'red tea'
)
%group4 = (
'fruit' => 'pear',
'vegetable' => 'cucumber',
'drink' => 'red wine'
)

正确方法：
%hash = (
'group1' => {%group1},
'group2' => {%group2},
'group3' => {%group3},
'group4' => {%group4}
);
错误方法：
%hash = (%group1, %group2, %group3, %group4); 
错误剖析：这个哈希相当于%hash = %group4
%hash = (
'group1' => %group1,
'group2' => %group2,
'group3' => %group3,
'group4' => %group4
); 
错误剖析：这个哈希的结果相当于 %hash = ('tomato','group4','orange juice','vegetable','cauliflower','group2','orange','drink','fruit','pear','group1','fruit','banana','drink','drink','red wine','vegetable','cucumber','group3','fruit','red tea','vegetable')

方法三：
$hash{group1} = {%group1};
$hash{'group1'} = {
'fruit' => 'banana',
'vegetable' => 'cauliflower',
'drink' => 'orange juice'
};
$hash{'group1'} = {'fruit', 'banana', 'vegetable', 'cauliflower', 'drink', 'orange juice'};
b. 哈希的访问
$hash{'group1'}{'fruit'} = 'banana' and  $hash{group1}{fruit} are the same.
$list{group1) = {'fruit' => 'banana','drink' => 'orange juice','vegetable' => 'cauliflower'}

Ⅳ. 哈希嵌套列表
a. 哈希的声明
方法一：
%hash = (
'fruit' => ["banana","apple","orange","pear"],
'vegetable' => ["cauliflower","lettuce","tomato","cucumber"],
'drink' => ["orange juice","apple juice","red tea","red wine"]
);

%hash = (
'fruit', ["banana","apple","orange","pear"],
'vegetable', ["cauliflower","lettuce","tomato","cucumber"],
'drink', ["orange juice","apple juice","red tea","red wine"]
);

方法二：
@fruit = ("banana","apple","orange","pear")
@vegetable = ("cauliflower","lettuce","tomato","cucumber")
@drink = ("orange juice","apple juice","red tea","red wine")

%hash = (
'fruit', [@fruit],
'vegetable', [@vegetable],
'drink', [@vegetable]
);

方法三：
$hash{fruit} = [@fruit]
$hash{'fruit'} = ["banana","apple","orange","pear"]

b. 哈希的访问
$hash{'fruit'}[0] = 'banana'
$hash{'fruit'} = ['banana','apple','orange','pear']
```
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

# 异常处理与告警
## warn
warn会将列表值输出到STDERR句柄（标准错误输出）。除非列表的最后一个元素以换行符结尾，否则Perl还会在告警中带上文件名和warn所在行号来帮助定位问题。
```
use Modern::Perl;
warn qq@Something went wrong!\n@;
warn qq@Something went wrong!@;

[huawei@n148 perl]$ perl 2.pl 
Something went wrong!
Something went wrong! at 2.pl line 3.
```
## confess
confess与die类似，但提供了从产生错误处的栈回溯追踪
```
sub withdraw {
    my ($self, $amount) = @_;
    my $current_balance = $self->balance();
	# 检测要取的钱要少于自己有的，否则就透支了，
    ($current_balance >= $amount) || confess "Account overdrawn";
    $self->balance($current_balance - $amount);
}

huawei@n148 perl]$ perl 2.pl 
Account overdrawn at 2.pl line 21.
        BankAccount::withdraw('BankAccount=HASH(0x2203580)', 500) called at 2.pl line 46
```
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
## caller()
使用内置函数caller可获取该函数被调用的情况。
* 无参数caller返回一个列表，包含有调用者的包名，调用者的文件名，和调用发生的位置
* caller还接受一个整型参数n，返回n层嵌套外调用的情况。
* caller(0)会上溯到在my_call中被调用的信息；
* caller(1) 会上溯到在程序中被调用的信息；

```
use v5.12;
my_call();
sub my_call
{
	show_call_information();
}
#额外的会返回一个函数名
sub show_call_information
{
my ($package, $file, $line, $func) = caller(1);
say "Called $func from $package in $file:$line";	# 打印出调用堆栈信息
}

my_call;

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
Called main::my_call from main in /home/huawei/playground/perl/2.pl:4
Called main::my_call from main in /home/huawei/playground/perl/2.pl:19
```
## carp、croak
核心模块 Carp 提供了其他产生警告的机制。
* Perl自带的die和warn仅报告代码出错的行，内容不够完善
* Carp模块提供的croak和carp函数提供了更细致的错误追踪功能，用法分别对应die和warn
* Carp模块就使用caller来报告错误和警告信息的。croak()从调用者的角度抛出异常，carp()报告位置。
```
package Newlib;
use 5.006;	
use Carp;
our $VERSION = '0.01';
sub function {
   croak "模块错误！";	# 这里出错，但打印出的错误信息却是调用此方法的那行
}
1;


#!/usr/bin/perl -I/home/huawei/playground/perl/123
use v5.12;
use newlib;
Newlib::function();

[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
模块错误！ at /home/huawei/playground/perl/2.pl line 4.
```
## eval
使用die或Carp的croak，都将报错退出程序，如果不想退出程序，可以使用eval来处理错误：当eval指定的代码出错时，它将返回undef而不是退出。eval有两种用法: 

```
eval 'say "hello world"';	字符串作为参数
eval {say "hello world"};	语句块作为参数
```

无论是字符串参数还是语句块参数，参数部分都会被当作代码执行一次。
```
* 如果eval参数部分执行时没有产生错误，则eval的返回值是最后一条被执行语句的计算结果，并且特殊变量$@为空。
* 如果eval参数部分执行时产生了错误，则eval的返回值为undef或空列表，同时将错误信息设置到$@中。
* 之所以将$@保存起来，是因为handle_error可能也有eval，这时handle_error的eval将覆盖之前的$@。
```
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
编写非常安全和合理的异常处理器非常困难。来自 CPAN Try::Tiny 发行模块非常简短，易于安装， 易于理解，提供了try、catch、finally的语法。同时也易于使用：
```
use Try::Tiny;

my $fh = try { open_log_file( 'monkeytown.log' ) }
catch { log_exception( $_ ) };
```
用try 代替了 eval，try捕获了异常后，catch语句块就会执行。异常会放在$_里面。
## AUTOLOAD
Perl 提供了一种机制，可以截获不存在方法的调用。这样就可以只定义所需的函数或提供有趣的错误信息和警告。
```
use Modern::Perl;

say bake_pie( filling => 'apple' );
sub AUTOLOAD { 
	my ($name) = our $AUTOLOAD =~ /::(\w+)$/;
	# 将参数美化输出
	local $" = ', ';
	say "In AUTOLOAD(@_) for $name!";
	say "In AUTOLOAD(@_) for $AUTOLOAD!";
	return "mmm";
}

[huawei@n148 perl]$ perl 2.pl 
In AUTOLOAD(filling, apple) for bake_pie!
In AUTOLOAD(filling, apple) for main::bake_pie!
mmm
```
# 正则
默认搜索对象是$_，Perl的正则表达式的三种形式：
* 匹配：m//
* 替换：s///
* 转化：tr///
	
这三种形式一般都和 =~ 或 !~ 搭配使用， =~ 表示相匹配，!~ 表示不匹配。
## 元字符 metacharacter
https://www.cnblogs.com/dancheblog/p/3528000.html

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
### 替换CRLF到LF
将文件行尾的CRLF换为LF
- 方法1 使用-i进行备份 
- 方法2 不用-i还能使用重定向流

	my $cmd = qq@perl -pe 's/\\r\$//' < $fname > $fname.$ext@;  
	这句要改一下才行，否则相当于从fname读入后替换的结果保存为fname.ext了  
	即得把这2个文件交互一些内容才对。。。
```
sub crlf2lf
{
	my $fname = $_;
	my ($s, $usec) = gettimeofday();
	my $ext = "$s$usec";	
	my $cmd = qq@perl -i.$ext -pe 's/\\r\$//' $fname@;
	say $cmd;
	system($cmd);
}

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
	
## 语境感知 wantarray
Perl的内置函数wantarray具有感知函数调用语境的功能。  
* 1 wantarray在空语境下返回undef；
* 2 标量语境返回假；
* 3 列表语境返回真。
```
use Modern::Perl;
use Test::More;

sub context_sensitive
{
my $context = wantarray();
return qw( List context ) if $context;
say 'Void context' unless defined $context;
return 'Scalar context' unless $context;
}

context_sensitive(); # 不需要返回内容符合上面1，返回空，打印Void context
say my $scalar = context_sensitive(); # 需要一个标量符合上面2，返回Scalar context
say context_sensitive();	# say的参数是列表需要返回list符合上面3，返回Listcontext

[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
Void context
Scalar context
Listcontext



第二个案例
--------------------
 sub context
 {
	my $context = wantarray();
	my $b=defined $context;
	say "\n---------------context=$context\tdefined context=$b";
	if ($b){
		if ($context)
		{
			say "list";
		}
		else{
			say "scale";
		}
	} else {
		say "void";
	}
	return 0;
 }
 my @list_slice = (1, 2, 3)[context()];	# context处需要列表，wantarray返回t，defined $context以及$context都是t，最终是say list
 my @array_slice = @list_slice[context()]; # 同上
 my $array_index = $array_slice[context()]; # 对array_slice取元素，context处需要标量，wantarray为f，defined $context却是t，最终say scale
 say context(); # say需要列表，wantarray返回t，defined $context以及$context都是t，最终是say list，再出来say return的0
 context() # 空语境wantarray返回undef，say void

 [huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"

---------------context=1        defined context=1
list

---------------context=1        defined context=1
list

---------------context= defined context=1
scale

---------------context=1        defined context=1
list
0
Use of uninitialized value $context in concatenation (.) or string at /home/huawei/playground/perl/2.pl line 13.

---------------context= defined context=
void
```
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
## 匿名函数
使用 sub 关键字 不用 函数名称也可以使得函数正常编译，但是它不会被安装到当前 的名称
空间中。访问此函数的唯一方法就是通过引用。
```
use Modern::Perl;
use Test::More;
sub bake_cake { say 'Baking a wonderful cake!' };
my $cake_ref = \&bake_cake;
my $pie_ref = sub { say 'Making a delicious pie!' };
$cake_ref->();
$pie_ref->();

[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
Baking a wonderful cake!
Making a delicious pie!
```
## 匿名函数的名称
存在可以鉴别一个引用是指向具名函数还是匿名函数的特例。__ANON__ 展示了匿名函数没有 Perl 可以识别的名称
```
sub show_caller
{
	my ($package, $filename, $line, $sub) = caller(1);
 	say "Called from $sub in $package at $filename : $line";
}
sub main
{
 	my $anon_sub = sub { show_caller() };
 	show_caller();
 	$anon_sub->();
}
main();

[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
Called from main::main in main at /home/huawei/playground/perl/2.pl : 16
Called from main::__ANON__ in main at /home/huawei/playground/perl/2.pl : 14
```
还使用Sub库进行解决
```
use Modern::Perl;
use Test::More;
use Sub::Name;
use Sub::Identify 'sub_name';
my $anon = sub {};
say sub_name( $anon );
my $named = subname( 'pseudo-anonymous', $anon );
say sub_name( $named );
say sub_name( $anon );
say sub_name( sub {} );

[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
__ANON__
pseudo-anonymous
pseudo-anonymous
__ANON__
```
## 区分匿名{}与作用域{}
* 大括号前面加上+符号，即+{...}，表示这个大括号是用来构造匿名hash的
* 大括号内部第一个语句前，多使用一个;，即{;...}，表示这个大括号是一次性语句块
```
@{ +[qw(longshuai wugui)]}   # 匿名数组中括号前
@{ +$ref_arr }           # 数组引用变量前
%{ +$ref_hash }          # hash引用变量前
```
## 自生 autovivification
CPAN 上的 autovivification 编译命令（编译命令）让你可以在词法作用域内对某 特定类
型操作禁用自生行为。

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

## 默认标量变量 $_
对于需要参数的函数或表达式，但却没有给参数则默认是变量$_
```
$_="abcde";
print ;

foreach(1..10){
	print $_;
}
```
## 默认数组变量 @_
Perl通过一个名为@_的数组向函数传递参数
```
$,=',';
sub findindex{
	my ($what,@arr)=@_;
	foreach(0..$#arr) {return $_ if $what == $arr[$_];}
	-1;
}
say findindex(3, qw/1 2 3 4 5/);
```
## 命令行参数 @ARGV
```
命令行参数保存在数组@ARGV中，数组下标从0开始
ARGV[0]是第一个参数（非程序名称），以此类推  
$#ARGV保存数组最后一个元素的索引（非数组元素数量），当无参数时$#ARGV等于-1（不是零）
命令行参数的总数是 $#ARGV+1
<ARGV>也可作为文件句柄数组进行迭代，见下面的案例
```
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
## 命令行参数 Getopt::Long
见ftp案例
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
# 信号处理
概念 http://blog.chinaunix.net/uid-30497107-id-5757738.html  
看完上面的内容，基本上linux的信号概念已经完备，下面的描述有可能不准确或仅针对perl
## linux中的信号signal
* 信号就是编程里俗称的中断，它使监视与控制其他进程变为有可能。
* 用来通知进程发生了异步事件。进程之间可以互相通过系统调用kill发送软中断信号
* 发送进程没有任何办法获取任何形式的返回，它只能知道该信号已经合法发送出去了。发送者也接收不到任何接收进程对该信号做的处理的信息
* 内核也可以因为内部事件而给进程发送信号，通知进程发生了某个事件。
* 信号只是用来通知某进程发生了什么事件，并不给该进程传递任何数据。
* 要想查看这些信号和编码的对应关系，可使用命令：kill -l
* 信号处理回调中应尽量少逻辑，如仅改变一个全局变量之类操作，避免io，内存操作
* 信号处理逻辑有可能也无法完全做到可移植，不同系统有可能未实现
* 全部信号看这里 https://blog.csdn.net/tennysonsky/article/details/46010505	也可参考perl网络编程p40
## Perl信号的处理
* Perl 提供了%SIG 这个特殊的默认HASH，包含指向用户定义信号句柄的引用
* 使用’$SIG{信号名}’截取信号,在perl程序中出现这个信号时,执行对应的回调
* 可以设置为DEFAULT为恢复默认处理，以及IGNORE为忽略
```
my$i=1;
while(1){
    $i=$i+1;
    print$i."\n";
}

sub yoursub{
    print" exit ... \n";
    exit 0;
}

BEGIN{
    $SIG{TERM}=$SIG{INT}=\&yoursub;
}  

启动程序，BEGIN中注册回调
（此处使用取地址赋值，也可以使用匿名方式如
$SIG{INT} = sub { ... };）
ctrl+c会导致INT信号，随即触发回调
```
注册die的回调
```
BEGIN{
    $SIG{__DIE__}=$SIG{__WARN__}=\&handler_fatal;
}  

sub handler_fatal {
    print"Content-type: text/html\n";
    print"@_&", "\n";
}

die ".x.x.x.";

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/2.pl"
Content-type: text/html
.x.x.x. at /home/huawei/playground/perl/2.pl line 14.
&
.x.x.x. at /home/huawei/playground/perl/2.pl line 14.
```
## 发送信号 kill
kill('signal', @Processes))给一组进程发送信号。signal是发送的数字信号
* 超级用户可以给任何进程发信号
* 普通用户只能同级别进程发信号
* "kill(0, $pid) 是判断进程是否正在运行，t则返回1，f则是0
* 信号值负数则是杀掉此号绝对值所属组的所有进程  
* kill('INT', $$)是自杀

kill('KILL', 进程号1, 进程号2...);

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
默认情况下输出到STDOUT指向的地方，也可以使用重定向运算符 > 输出到指定文件。返回0则正常，-1是失败。会创建子进程执行命令，父进程等待子进程完毕再继续
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
print "PID=$$\n";
my $child=fork();
die $! unless defined $child;
if ($child>0){
	print "parent process: PID=$$, child=$child\n"; # 父进程
} else {
	my $ppid=getppid();
	print "child process: PID=$$, parent=$ppid\n";	# 子进程
}

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
PID=112138
parent process: PID=112138, child=112139
child process: PID=112139, parent=112138
---------------------------------
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
使用发CHLD的信号
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
## 守护进程与setsid
概念：http://blog.chinaunix.net/uid-30497107-id-5757733.html  


通常，一个会话开始于用户登录，而终止于用户退出，在此期间该用户运行的所有进程都属于这个会话期。在创建守护进程的时候，子进程与父进程同属于相同的会话组与进程组。

守护进程的特点
* 在后台运行
* 脱离控制终端，登录会话和进程组
* 禁止进程重新打开控制终端
* 关闭打开的文件描述符
* 改变当前工作目录
* 重设文件创建掩模
* 处理SIGCHLD信号


setsid函数概念与作用  
https://blog.csdn.net/akunshouyoudou/article/details/50828826  
* 让进程摆脱原会话的控制
* 让进程摆脱原进程组的控制
* 让进程摆脱原控制终端的控制
 
创建守护进程的步骤
* 创建父进程，并且让其推出
* 调用 setsid（）函数
* 改变当前的目录为根目录 chdir（"/"）
* 重设文件权限掩码 umask(0);
* 关闭文件描述符 .常用的方式为 for(i = 0 ; i < MAXFILE;  i ++){close(i);}

案例见 守护进程版聊天服务器
## CHLD信号
CHLD的作用是子进程结束时候会自动发送此信号给父进程，父进程对此信号进行捕获处理即可  
案例来自perl网络编程，作用是与ftp服务器进行交互，

```
use Modern::Perl;
use IO::Socket qw(:DEFAULT :crlf);

my $socket = IO::Socket::INET->new("ftp.cesca.es:ftp") or die $@;
my $child = fork();
die "Can't fork: $!" unless  defined $child;

if ($child) {	# 父进程的作用是读取stdin手敲的命令发往ftp服务器
  $SIG{CHLD} = sub { exit 0 };	# 这里设置来自子进程结束时发来的CHLD信号的回调，即结束程序
  user_to_host($socket);
  $socket->shutdown(1);	# 关闭socket
  sleep;	# 进入无休止休眠，等待来自子进程的CHLD信号

} else {	# 子进程的作用是从ftp服务器读取返回的多行内容，ftp服务器
  host_to_user($socket);
  warn "Connection closed by foreign host.\n";
}

sub user_to_host {
  my $s = shift;
  while (<>) {	# 当标准输入关闭时候退出(ctrl+c)（父进程进入sleep），此时ftp服务器也收到了，然后再会将退出发往子进程的host_to_user，此时子进程的host_to_user也退出了，导致进程结束发送CHLD信号给父进程，进而完美结束
    chomp;
    print $s $_,CRLF;
  }
}

sub host_to_user {
  my $s = shift;
  $/ = CRLF;
  while (<$s>) {	# 读取结束退出
    chomp;
    print $_,"\n";
  }
}


[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/5.pl"
220 Welcome to Anella Cientifica FTP service.
USER anonymous	这行手敲
331 Please specify the password.
PASS ok@better.now	这行手敲
230 Login successful.
HELP	这行手敲
214-The following commands are recognized.
 ABOR ACCT ALLO APPE CDUP CWD  DELE EPRT EPSV FEAT HELP LIST MDTM MKD
 MODE NLST NOOP OPTS PASS PASV PORT PWD  QUIT REIN REST RETR RMD  RNFR
 RNTO SITE SIZE SMNT STAT STOR STOU STRU SYST TYPE USER XCUP XCWD XMKD
 XPWD XRMD
214 Help OK.
quit	这行手敲
221 Goodbye.
Connection closed by foreign host.
```
## wait、waitpid
* 这2个函数可以理解为收尸的作用
* 如果子进程在父进程之前over则可以在父进程中使用wait或waitpid获取子进程的终止状态码
* 如果子进程一直未被wait或waitpid则父进程在资源耗尽之后最终会fork失败
* 子进程终止或挂起都会发CHLD信号给父进程，此时在父进程的回调中wait最合适
* wait会阻塞直到子进程终止，并返回pid，$?是状态码，0则正常其它异常
* waitpid则可以通过参数WNOHANG设置为非阻塞

```
使用类似这样的代码逻辑，详细案例见聊天机器人server

use POSIX 'WNOHANG';
$SIG{CHLD} = sub 
{
	while ((my $kid= waitpid(-1,WNOHANG)) > 0 ) 
	{ 
		warn "pid: $kid\n";
	} 
};
```
## IPC::System::Simple
system、systemx、capture、capturex
## 通过文件句柄执行外部进程
# 多线程
https://www.cnblogs.com/f-ck-need-u/p/10420910.html  
使用threads包。线程一旦被成功创建，它就立刻开始运行了，这个时候你面临两种选择，分别是 join 或者 detach 这个新建线程。当然你也可以什么都不做，不过这可不是一个好习惯。 
## create、new、tid
create、new两个名字效果抑郁一使用线程回调或是匿名回调均可以，还可带有线程参数
```
BEGIN{
    use Config;
    if ($Config{usethreads}) {print "support old thread\n";}
    if ($Config{useithreads}) {print "support interpreter threads\n";}
}

my $tid = threads->tid();
printf("Hello main thread $tid!\n"); 
sub say_hello { 
	my $tid = threads->tid();
    printf("Hello thread $tid! @_.\n"); 
    return( rand(10) ); 
} 
 
my $t1 = threads->create( \&say_hello, "param1", "param2" ); 
my $t2 = threads->new( "say_hello", "param3", "param4" ); 
my $t3 = threads->create( 
sub { 
	my $tid = threads->tid();
    printf("Hello thread $tid! @_\n"); 
    return( rand(10) ); 
}, "param5", "param6" );

support old thread
support interpreter threads
Hello main thread 0!
Hello thread 1! param1 param2.
Hello thread 2! param3 param4.
Hello thread 3! param5 param6
Perl exited with active threads:
        1 running and unjoined
        2 finished and unjoined
        0 running and detached
``` 
## join
join()做三件事：
* 等待子线程退出，等待过程中父线程一直阻塞
* 子线程退出后，为子线程收尸(OS clean up)
* 如果子线程有返回值，则收集返回值

案例：父进程等待、收尸、收集返回值
```
my ($thr) = threads->create(\&sub1);
my @returnData = $thr->join();
print 'thread returned: ', join('@', @returnData), "\n";
sub sub1 {
    # 返回值是列表
    return ('fifty-six', 'foo', 2);
}
[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
thread returned: fifty-six@foo@2
```
普通案例
```
use threads; 
 
sub func { 
 sleep(1); 
 return(rand(10)); 
} 
 
my $t1 = threads->create( \&func ); 
my $t2 = threads->create( \&func ); 
 
printf("do something in the main thread\n"); 
 
my $t1_res = $t1->join(); 
my $t2_res = $t2->join(); 
 
printf("t1_res = $t1_res\nt2_res = $t2_res\n");

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
do something in the main thread
t1_res = 8.35062209823267
t2_res = 4.04523008343862

```
下面2中种方法，第一种会有没join全的。。。
```
# my $running_threads = $totalthreads;
# while ($running_threads) {
# 	for my $thread (threads->list(threads::joinable)) {
# 		$thread->join();
# 		$running_threads--;
# 		$log->debug("$running_threads threads remaining");
# 	}
# }
$_->join() for threads->list();
```
## detach
detach 就是把新创建的线程与当前的主线程剥离开来，让它从此和主线程无关。当你使用 detach 方法的时候，表明主线程并不关心新建线程执行以后返回的结果，新建线程执行完毕后 Perl 会自动释放它所占用的资源。  
一个新建线程一旦被 detach 以后，就无法再 join 了。当你使用 detach 方法剥离线程的时候，有一点需要特别注意，那就是你需要保证被创建的线程先于主线程结束，否则你创建的线程会被迫结束，除非这种结果正是你想要的，否则这也许会造成异常情况的出现，并增加程序调试的难度。
```
sub mysub {
    #alarm 10;
    for (1..10){
        print "I am detached thread\n";
        sleep 1;
    }
}

my $thr1 = threads->new(\&mysub)->detach();

print "main thread will exit in 2 seconds\n";
sleep 2

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
main thread will exit in 2 seconds
I am detached thread
I am detached thread
I am detached thread
```
## 线程的退出
正常情况并且大多情况下，线程都应该通过子程序return的方式退出线程。但是也有其它可能。


* threads->exit()
线程自身可以调用threads->exit()以便在任何时间点退出。这会使得线程在标量上下文返回undef，在列表上下文返回空列表。如果是在main线程中调用`threads->exit()`，则等价于exit(0)

* threads->exit(status)
在线程中调用时，等价于threads->exit()，退出状态码status会被忽略。在main线程中调用时，等价于exit(status)

* die()
直接调用die函数会让线程直接退出，如果设置了 $SIG{__DIE__} 的信号处理机制，则调用该处理方法，像一般情况下的die一样

* exit(status)
在线程内部调用exit()函数会导致整个程序终止(进程中断)，所以不建议在线程内部调用exit()。但是可以改变exit()终止整个程序的行为，见下面几个设置

* use threads 'exit'=>'threads_only'
全局设置，使得在线程内部调用exit()时不会导致整个程序终止，而是只让线程终止。由于这是全局设置，所以不是很建议设置。另外，该设置对main线程无效

* threads->create({'exit'=>'thread_only},\&sub1)
在创建线程的时候，就设置该线程中的exit()只退出当前线程

* $thr->set_thread_exit_only(bool)
修改当前线程中的exit()效果。如果给了true值，则线程内部调用exit()将只退出该线程，给false值，则终止整个程序。对main线程无效

* threads->set_thread_exit_only(bool)
类方法，给true值表示当前线程中的exit()只退出当前线程。对main线程无效

最可能需要的退出方式是threads->exit()或threads->exit(status)，如果对于线程中严重错误的问题，则可能需要的是die或exit()来终止整个程序。

下面是部分演示案例
```
use threads;  
sub say_hello { 
 printf("Hello thread! @_.\n"); 
 sleep(3); 
 printf("Bye\n"); 
} 
 
sub quick_exit { 
 printf("I will be exit in no time\n"); 
 exit(1); 
} 
 
my $t1 = threads->create( \&say_hello, "param1", "param2" ); 
my $t2 = threads->create( {'exit'=>'thread_only'}, \&quick_exit ); 
 
$t1->join(); 
$t2->join();

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
Hello thread! param1 param2.
I will be exit in no time
Bye
```
如果你希望每个线程的 exit 方法都只对自己有效，那么在每次创建一个新线程的时候都去要显式设置’ exit ’ => ’ thread_only ’属性显然有些麻烦，你也可以在引入 threads 包的时候设置这个属性在全局范围内有效
```
use threads ('exit' => 'threads_only'); 
 
sub func { 
 if(shift) { 
 exit(1); 
 } 
} 
 
my $t1 = threads->create( \&func, 1 ); 
my $t2 = threads->create( \&func, 1 ); 
 
$t1->join(); 
$t2->join();
```
## yield
短暂地放弃CPU转交给其它线程
threads->yield();
yield()和操作系统平台有关，不一定真的有效，且出让CPU的时间也一定能保证。

## 线程的信号处理
https://www.cnblogs.com/f-ck-need-u/p/10420910.html
## threads::shared
https://www.cnblogs.com/f-ck-need-u/p/10422445.html
Perl解释器线程在被创建出来的时候，将从父线程中拷贝数据到子线程中，使得数据是线程私有的，并且数据是线程隔离的。如果真的想要在线程间共享数据，需要显式使用threads::shared模块来扩展threads模块的功能。这个模块必须在先导入了threads模块的情况下使用，否则threads::shared模块里的功能都将没效果。要共享数据，只需使用threads::shared模块的share方法即可，也可以直接将数据标记上:shared属性的方式来共享
### 两种声明方式
使用share()和:shared的区别在于后面这种标记的方式是在编译期间完成的。另外，使用:shared属性标记的方式可以直接共享引用类型的数据，但是share()不允许，它使用prototype限制了只允许接收变量类型的参数，可以使用&share()调用子程序的方式禁用prototype：
```
my $answer = 43;
my @arr = qw(1 2 3);
share($answer);
share(@arr);

my $answer :shared = 43;
my @arr :shared = qw(1 2 3);
my %hash :shared = (one=>1, two=>2, three=>3);

my $aref = &share([1 2 3]);
```
简单案例
```
my $foo :shared = 1;
my $bar = 1;
threads->create(
    sub {
        $foo++;
        $bar++;
        say "new thread: \$foo=$foo";   # 2
        say "new thread: \$bar=$bar";   # 2 
    }
)->join();

say "main thread: \$foo=$foo";   # 2
say "main thread: \$bar=$bar";   # 1


[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
new thread: $foo=2
new thread: $bar=2
main thread: $foo=2
main thread: $bar=1
```
### 可以共享的类型
并非所有数据都可以共享，只有普通变量、数组、hash以及已共享数据的引用可以共享
* Ordinary scalars
* Array refs
* Hash refs
* Scalar refs
* Objects based on the above

如果共享hash或array类型，那么里面的所有元素都对外可见，但并不意味着里面的元素是共享的。共享hash/array和共享它们里面的元素是独立的，共享hash/array只意味着共享它们自身，但里面的元素会暴露。反之，可以直接共享某个元素，但hash/array自身不共享。(经测试，数组的元素无法共享，hash的元素可正常共享)
```
my $var = 1;     #  未共享数据
my $svar :shared = 2;    # 共享标量
my @arr :shared = qw(perl python shell);   # 共享数组
my %hash :shared;       # 共享hash

my $thr = threads->new(\&mysub);

sub mysub {
    $hash{a} = 1;       # 成功
    $hash{b} = $var;    # 成功：$var是普通变量
    $hash{c} = \$svar;  # 成功：\$svar是已共享标量
    $hash{d} = @arr;    # 成功：普通数组
    $hash{e} = \@arr;   # 成功：已共享数组的引用

    # $hash{f} = \$var; # 失败并die：$var未共享标量的引用
    # $hash{g} = [];    # 失败：未共享数组的引用
    # $hash{h} = {a=>1};# 失败：未共享hash的引用
}

$thr->join();     # join后文解释
while( my ($key, $value) = each %hash ){
    say "$key => $value";
}

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
e => ARRAY(0x29c6e28)
b => 1
d => 3
a => 1
c => SCALAR(0x29c6df8)
```


即在不同线程中可以操作同一变量，但此处暂未加锁处理，加锁见后面
```
use threads;
use threads::shared;

my $var=100;
share($var); #### 共享变量进程和个线程之间
my $t1 = threads -> create (\&thread_child, "T1");
$t1 -> detach();
sleep 1;
my $t2 = threads -> create (\&thread_child, "T2");
$t2 -> detach();
print "do something in the main thread.......\n";
sleep 1;
thread_child("main_thread");
sub thread_child
{
   my @name =@_;
   print "@name var is $var......\n";
   $var ++;  
}

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
T1 var is 100......
do something in the main thread.......
T2 var is 101......
main_thread var is 102......
```
共享数组与哈希的案例
```
use threads;
use threads::shared;

my $var   :shared  = 0;       	# use :share tag to define 
my @array :shared = (); 		# use :share tag to define 
my %hash = (); 
share(%hash);                  	# use share() funtion to define 


sub start { 
	$var = 100; 

	@array[0] = 200; 
	@array[1] = 201; 

	$hash{'1'} = 301; 
	$hash{'2'} = 302; 
} 

sub verify { 
   sleep(1);                      	# make sure thread t1 execute firstly 
   printf("var = $var\n");     		# var=100 

	for(my $i = 0; $i < scalar(@array); $i++) { 
		printf("array[$i] = $array[$i]\n");    	# array[0]=200; array[1]=201 
	} 

	foreach my $key ( sort( keys(%hash) ) ) { 
		printf("hash{$key} = $hash{$key}\n"); 	# hash{1}=301; hash{2}=302 
	} 
} 

my $t1 = threads->create( \&start ); 
my $t2 = threads->create( \&verify ); 

$t1->join(); 
$t2->join();

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
var = 100
array[0] = 200
array[1] = 201
hash{1} = 301
hash{2} = 302
```

## lock与死锁
threads::shared模块中提供了一个lock()方法，用来将共享数据进行独占锁定，被锁定的数据无法被其它线程修改，直到释放锁其它线程才可以获取锁并修改数据。没有unlock()这样直接释放锁的方法，而是在退出当前作用域的时候自动释放锁。另外，锁住hash和数组的时候，仅仅只是锁住它们自身，但lock()无法去锁hash/array中的元素。lock()是可以递归的，在退出最外层lock()的作用域时释放锁。且递归时重复锁定同一个变量是幂等的。

下面是肯定死锁的案例
```
my $x :shared = 4;
my $y :shared = 'foo';

my $thr1 = threads->create(
    sub {
        lock($x);
        sleep 3;
        lock($y);
    }
);

my $thr2 = threads->create(
    sub {
        lock($y);
        sleep 3;
        lock($x);
    }
);

sleep 10;
```
解决死锁最简单且最佳的方式是保证所有线程以相同的顺序去锁住每一个数据。另一个避免死锁的解决方案是尽可能让锁住共享数据的时间段变短，这样出现僵局的几率就会小很多。但是这两种方式很多时候都派不上用场，因为需要用到锁的情况可能会比较复杂。
## 线程队列 Thread::Queue
(Thread::Queue)队列数据结构(FIFO)是线程安全的，它保证了某些线程从一端写入数据，另一些线程从另一端读取数据。只要队列已经满了，写入操作就自动被阻塞直到有空间支持写操作，只要队列空了，读取操作就会自动阻塞直到队列中有数据可读。这种模式自身就保证了线程安全性。
https://www.cnblogs.com/f-ck-need-u/p/10422293.html
```
use Thread::Queue;
my $q = Thread::Queue->new();
# 放非引用数据到队列
$q->enqueue(1, 2, 3);
# 放引用数据到队列
$q->enqueue(['a', 'b', 'c']);
my $thr = threads->new(
    sub {
        sleep 1;
        while(my $item = $q->dequeue()){
            # 如果是引用，要小心数据被其它线程修改
            if (ref $item){
                foreach (@$item){
                    print "ele: $_\n";
                }
            } else {
                print "item: $item\n";
            }
        }
    }
);

my $num = $q->peek();
say $num;
$num = 11;     # 不影响，因为是非引用数据
my $list = $q->peek(3);
$$list[1] = 'bb';   # 将影响队列中对应元素
$q->end();   # 关闭队列
$thr->join;

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
1
item: 1
item: 2
item: 3
ele: a
ele: bb
ele: c
```

## 线程信号量 Thread::Semaphore
up()表示增加信号量的值，down()表示减信号量的值。只要减法操作后信号量的值为负数，这次减法操作就会被阻塞。
new()方法来创建一个信号量，如果不给任何参数，则默认创建一个信号量值为1的信号量。如果给new()一个整数值N，则表示创建一个信号量值为N的信号量。
new、up、down也可以每次指定数量，而非每次都是操作+1或-1
https://www.cnblogs.com/f-ck-need-u/p/10422445.html#threadsemaphore

案例是3个线程分别对一个全共享变量累加
```
use Thread::Semaphore;


# 新建一个信号量
my $sem = Thread::Semaphore->new();

# 全局共享变量
my $gbvar :shared = 0;

my $thr1 = threads->create(\&mysub, 1);
my $thr2 = threads->create(\&mysub, 2);
my $thr3 = threads->create(\&mysub, 3);

# 每个线程给全局共享变量依次加10
sub mysub {
    my $thr_id = shift;
    my $try_left = 10;
    my $local_value;
    sleep 1;
    while($try_left--){
        # 相当于获取锁
        $sem->down();

        $local_value = $gbvar;
        say "$try_left tries left for sub $thr_id "."(\$gbvar is $gbvar)";
        sleep 1;
        $local_value++;
        $gbvar = $local_value;
        
        # 相当于释放锁
        $sem->up();
    }
}

$thr1->join();
$thr2->join();
$thr3->join();

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl"
9 tries left for sub 1 ($gbvar is 0)
8 tries left for sub 1 ($gbvar is 1)
7 tries left for sub 1 ($gbvar is 2)
6 tries left for sub 1 ($gbvar is 3)
5 tries left for sub 1 ($gbvar is 4)
4 tries left for sub 1 ($gbvar is 5)
9 tries left for sub 2 ($gbvar is 6)
8 tries left for sub 2 ($gbvar is 7)
7 tries left for sub 2 ($gbvar is 8)
6 tries left for sub 2 ($gbvar is 9)
5 tries left for sub 2 ($gbvar is 10)
4 tries left for sub 2 ($gbvar is 11)
3 tries left for sub 2 ($gbvar is 12)
2 tries left for sub 2 ($gbvar is 13)
1 tries left for sub 2 ($gbvar is 14)
0 tries left for sub 2 ($gbvar is 15)
9 tries left for sub 3 ($gbvar is 16)
8 tries left for sub 3 ($gbvar is 17)
7 tries left for sub 3 ($gbvar is 18)
6 tries left for sub 3 ($gbvar is 19)
5 tries left for sub 3 ($gbvar is 20)
4 tries left for sub 3 ($gbvar is 21)
3 tries left for sub 3 ($gbvar is 22)
2 tries left for sub 3 ($gbvar is 23)
1 tries left for sub 3 ($gbvar is 24)
0 tries left for sub 3 ($gbvar is 25)
3 tries left for sub 1 ($gbvar is 26)
2 tries left for sub 1 ($gbvar is 27)
1 tries left for sub 1 ($gbvar is 28)
0 tries left for sub 1 ($gbvar is 29)
```
# 目录操作
## 获取当前路径
```
use Cwd;
$dir = cwd();	# 效果同pwd
```
## 创建与删除
还可以使用File::Path的mkpath()一次创建多级目录，mkdir仅可一级。rmdir必须要求目录为空，而rmtree则任意。总之，File::Path为我们提供了另一种创建和删除目录的机制，由用户自己选用。
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
	-p HANDLE	检测文件句柄是个管道，见管道案例
	-S HANDLE   检测文件句柄是socket
	-b $file 	如果 $file 是块特殊文件，则为真
	-c $file 	如果 $file 是字符特殊文件，则为真
	-u $file 	如果 $file 具有 setuid 位设置，则为真
	-g $file 	如果 $file 具有 setgid 位设置，则为真
	-k $file 	如果 $file 具有 sticky 位设置，则为真
	-t HANDLE 	检测文件句柄是打开的终端
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
print "File is a directory\n" if -d $file; 判断是否为文件夹
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
## 文件信息 stat
perl下的stat函数和shell下的stat命令的功能基本一致，也是取得文件的各类具体信息：stat()函数返回一个数组，下面是数组各个元素的含义：
* dev     ：文件所属文件系统的设备ID
* inode   ：文件inode号码
* mode    ：文件类型和文件权限(两者都是数值表示)
* nlink   ：文件硬链接数
* uid     ：文件所有者的uid
* gid     ：文件所属组的gid
* rdev    ：文件的设备ID(只对特殊文件有效，即设备文件)
* size    ：文件大小，单位字节
* atime   ：文件atime的时间戳(从1970-01-01开始计算的秒数)
* mtime   ：文件mtime的时间戳(从1970-01-01开始计算的秒数)
* ctime   ：文件ctime的时间戳(从1970-01-01开始计算的秒数)
* blksize ：文件所属文件系统的block大小
* blocks  ：文件占用block数量(一般是512字节的块大小，可通过unix的stat -c "%B"获取块的字节)

```
my $filename="1.pl";
my @arr = stat($filename);

say '$dev     :',$arr[0];
say '$inode   :',$arr[1];
say '$mode    :',$arr[2];
say '$nlink   :',$arr[3];
say '$uid     :',$arr[4];
say '$gid     :',$arr[5];
say '$rdev    :',$arr[6];
say '$size    :',$arr[7];
say '$atime   :',$arr[8];
say '$mtime   :',$arr[9];
say '$ctime   :',$arr[10];
say '$blksize :',$arr[11];
say '$blocks  :',$arr[12];

[huawei@n148 perl]$ perl "/home/huawei/playground/perl/1.pl"
$dev     :64770
$inode   :235958415
$mode    :33204
$nlink   :1
$uid     :1004
$gid     :1004
$rdev    :0
$size    :436
$atime   :1645768651
$mtime   :1645768651
$ctime   :1645768651
$blksize :4096
$blocks  :8
```
返回的是文件类型和文件权限的结合体，且文件权限并非直接的8进制权限值，要计算出Unix系统中直观的权限数值(如0755、0644)，需要和0777做位运算，如下两种方式都可
```
printf "perm     :%04o\n",$mode & 0777;  # 将返回0644、0755类型的权限值
my $mode = (stat($filename))[2];
```

以下是stat函数的其它一些注意事项：
* stat函数返回布尔值表示是否成功，如t则立即设置这些属性变量，并缓存到特殊文件句柄 “ _ ” 里（你没没错，就是一个下划线），如f则返回空列表
* 如果stat函数测试的是特殊文件句柄_，它将不会重新测试缓存文件，而是直接返回缓存的属性信息
* 如果省略stat函数的参数，则默认测试$_
* 对于软链接文件，stat会追踪到链接的目标。如果不想追踪，则使用lstat函数替代stat。lstat如果测试的目标不是软链接，则返回空列表。
## 删除文件 unlink
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
## 链接 link,symlink
# 模块
模块有两种类型
* 传统模块  
  定义子例程和变量，供调用者导入和使用
* 面向对象模块  
  相当于类定义，可以通过方法调用来访问
## 换国内源
https://www.shuzhiduo.com/A/pRdBOKZnzn/

	1 在 http://mirrors.cpan.org/ 搜china，找到合适的镜像地址。
	2 vi $HOME/.cpan/CPAN/MyConfig.pm
	3 将'urllist' => [q[http://mirrors.neusoft.edu.cn/cpan/]], 中的地址换为新地址
	4 保存完毕


也可以交互方式，这样更简单，只不过每次都是添加一个，删除还得上面方法
$ perl -MCPAN -e shell
cpan[1]> o conf urllist push http://mirror.lzu.edu.cn/CPAN/
cpan[2]> o conf commit
cpan[3]> q

## cpan

yum install perl-CPAN  这个是安装老板的，新版需要编译，参考安装perl的记录


cpan命令是随perl一起安装的一个perl脚本
```
-a：创建CPAN.pm的autobundle
-D module：查看模块的详细属性信息。例如是否安装，安装的版本号，最新的版本号，对应的模块路径，对应的源码包文件路径，谁维护的
-g module：下载最新版本的模块到当前目录
-i module：安装指定的模块
-j Config.pm：指定CPAN配置数据的文件
-J：以CPAN.pm相同的格式dump当前的配置文件
-O：列出过期的模块
-v：输出cpan脚本的版本号以及CPAN.pm的版本号
```
## 安装模块
```
[huawei@n148 perl]$ cpan -a						# 查看已安装模块
[huawei@n148 perl]$ cpan -i Term::ANSIScreen	# 安装模块
[huawei@n148 perl]$ perldoc Term::ANSIScreen	# 查看模块说明

cpan -i IPC::Run
cpan -i Test::More
cpan -i Time::HiRes
```
## 查看模块的信息
```
[root@redisa-b ~]# cpan -D File::Utils
Reading '/root/.cpan/Metadata'
  Database was generated on Wed, 19 Sep 2018 20:17:03 GMT
File::Utils
-------------------------------------------------------------------------
        (no description)
        P/PE/PEKINGSAM/File-Utils-0.0.5.tar.gz     # 模块的distribution id
        /usr/local/share/perl5/File/Utils.pm       # 模块的安装路径
        Installed: 0.0.5                           # 已安装的模块版本号
        CPAN:      0.000005  up to date            # CPAN中最新的模块版本号
        Yan Xueqing (PEKINGSAM)                    # 作者名称及CPAN中的ID
        yanxueqing10@163.com
```
## 查询一个模块是否已安装
perldoc 即可，如果安装了就能正常输出对应的文档，如果没有安装，则报错
## 查看所有已安装的模块
cpan -a可能会有点慢。。。
```
cpan -a
cpan -a | grep Moose
```
## CPANMinus
```
https://www.cnblogs.com/wq242424/p/8037447.html

yum install perl-App-cpanminus.noarch
默认安装位置在/home/huawei/perl5/
```


这个是真正的完全一键安装，无需任何配置。而且，它没有交互式模式。
cpanm 其实只是一个可执行文件而已。将它下载到 bin 目录，然后添加执行权限就可以用了
```
安装
[huawei@n148 ~]$ curl -L http://cpanmin.us | perl - App::cpanminus
[huawei@n148 ~]$ whereis cpanm
cpanm: /usr/bin/cpanm /usr/local/bin/cpanm /home/huawei/perl5/bin/cpanm

添加与删除模块
[huawei@n148 ~]$ cpanm Perl::LanguageServer
[huawei@n148 ~]$ cpanm -U Perl::LanguageServer
```
## 手动安装模块
```
从网上下载好模块源码包，然后解压，进入源码包目录
wget https://cpan.metacpan.org/authors/id/X/XS/XSAWYERX/Data-Dumper-2.172.tar.gz
tar xf Data-Dumper-2.172.tar.gz
cd Data-Dumper-2.172/

共有2种build方式，Makefile.PL与Build.PL，依据哪个文件存在则使用哪个
（1）Makefile.PL
perl Makefile.PL --bootstrap
make test && make install

如果想要指定安装路径，则加上INSTALL_BASE即可：
perl Makefile.PL INSTALL_BASE=/home/perlapps



（2）Build.PL
perl Build.PL
如果想要指定安装路径，则
perl Build.PL −−install_base /home/perlapps

./Build
./Build install


如果是手动指定的安装路径，还需要设置模块查找路径环境变量：
export PERL5LIB=/home/perlapps 此值可能不同
```
## 查看文档与包的安装路径
安装模块后，都会有对应的文档，可以通过perldoc MODULE_NAME来获取模块的使用帮助。
```
[huawei@n148 perl]$ perldoc File::Basename

查找包安装路径
[huawei@n148 ~]$ perldoc -l Moose
/home/huawei/perl5/lib/perl5/x86_64-linux-thread-multi/Moose.pm

```
## 导入模块 use
* 当使用关键字use加载一个模块时，Perl就会自动调用一个叫import()的方法
	```
	use strict;
	#这句的意思就是加载strict.pm模块，
	#然后调用strict->import()方法（没有参数）。


	use strict 'refs';
	use strict qw( subs vars );
	#加载strict.pm模块，
	#然后调用strict->import( 'refs' ), 
	#再调用 strict->import( 'subs', vars' )。

	你也可以直接显式调用import()方法。和上面的例子等价：
	BEGIN
	{
		require strict;
		strict->import( 'refs' );
		strict->import( qw( subs vars ) );
	}
	```
* use语句是程序编译期间执行的
* 通常use写在程序的开头，但非必须
* use模块后其内的属性就会导入到当前程序的名称空间供当前程序使用
* 可以全导入、全不导入或导入一部分
* 有些模块比较复杂，会提供标签分组功能（其实是一堆函数集合）。可以导入时指定标签
```
全导入
use File::Basename;

导入一部分
use File::Basename qw(basename dirname);
say File::Basename::basename __FILE__;

全不导入
use File::Basename qw//;
use File::Basename ();

指定标签
use CGI qw(:all);
```

## 模块查找路径 @INC
默认情况下，perl将从@INC指定的路径中查找模块，它就像shell的PATH环境变量一样。
```
[huawei@n148 perl]$  perl -e 'foreach (@INC){print "$_\n"};'
/home/huawei/perl5/lib/perl5/5.16.3/x86_64-linux-thread-multi
/home/huawei/perl5/lib/perl5/5.16.3
/home/huawei/perl5/lib/perl5/x86_64-linux-thread-multi
/home/huawei/perl5/lib/perl5
/home/huawei/perl5/lib/perl5/5.16.3/x86_64-linux-thread-multi
/home/huawei/perl5/lib/perl5/5.16.3
/home/huawei/perl5/lib/perl5/x86_64-linux-thread-multi
/home/huawei/perl5/lib/perl5
/home/huawei/perl5/lib/perl5/5.16.3/x86_64-linux-thread-multi
/home/huawei/perl5/lib/perl5/5.16.3
/home/huawei/perl5/lib/perl5/x86_64-linux-thread-multi
/home/huawei/perl5/lib/perl5
/usr/local/lib64/perl5
/usr/local/share/perl5
/usr/lib64/perl5/vendor_perl
/usr/share/perl5/vendor_perl
/usr/lib64/perl5
/usr/share/perl5
.
```
如果是手动安装的包，或者安装到了一个非默认的查找路径下(例如不同用户安装到了不同家目录下)，这时可以通过在shell中设置PERL5LIB环境变量，perl会临时从这个环境变量中去查找模块。
```
[huawei@n148 perl]$ echo $PERL5LIB
/home/huawei/perl5/lib/perl5:/home/huawei/perl5/lib/perl5:/home/huawei/perl5/lib/perl5
```
## 检查模块是否被加载
如果知道模块名字，就可以通过查看%INC来检测模块是否被加载。当使用use或require加载代码时，Perl会在%INC中保存记录：键是模块的文件路径，值是该模块的在磁盘上的全路径。  
```
举个例子，加载 Modern::Perl时等效于:
$INC{'Modern/Perl.pm'} => '/path/to/perl/lib/site_perl/5.12.1/Modern/Perl.pm';
```
要测试是否加载一个模块。就是看是否存在该键：
```
use Modern::Perl;
sub module_loaded
{
	(my $modname = shift) =~ s!::!/!g;
	return exists $INC{ $modname . '.pm' };
}
say module_loaded('Modern::Perl');
```
## 检测一个包是否存在
任何继承UNIVERSAL的包都会提供一个can()方法。如果包不存在，Perl会抛出异常（无效的调用者），可以使用eval来包装下：
```
use Animal;
my $pkg='Animal';
say "$pkg exists" if eval { $pkg->can( 'can' ) };
```
## 检查模块的版本
模块并非一定要提供版本号，但是每一个包都会从UNIVERSAL类继承VERSION()方法.VERSION() 返回模块的版本号（如果有）；否则返回undef；如果模块不存在也返回undef。
```
use Animal;
say Animal->VERSION;
```
## 手动指定use路径
```
库文件
[huawei@n148 perl]$ cat lib/newlib.pm 
package Newlib;
use 5.006;
use strict;
use warnings;
our $VERSION = '0.01';
sub myfun {
	my ($a,$b)=(shift,shift);
	return $a+$b;
}
1;
```
* 在代码中手动指定加载的lib路径地址
	```
	#!/usr/bin/perl
	use lib "./lib";	# 可以相对或绝对路径，要写在文件最上面
	use newlib;
	print Newlib::myfun(10,20), "\n";
	[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
	30
	```
* 运行perl命令使用参数-I
	```
	use newlib;	 # 脚本中不设置库文件位置
	print Newlib::myfun(10,20), "\n";

	[huawei@n148 perl]$ perl -Ilib test.pl  # 手动运行perl命令参数I指定路径./lib/
	30
	```
* 将-I写入脚本中
	```
	#!/usr/bin/perl -Ilib 在shebang里写入-I参数，效果与上一致
	use newlib;
	print Newlib::myfun(10,20), "\n";
	[huawei@n148 perl]$ /usr/bin/perl -Ilib  "/home/huawei/playground/perl/2.pl"
	30
	```
* 在BEGIN块中加入@INC
	```
	#!/usr/bin/perl
	BEGIN {
		unshift(@INC, "lib");
	}
	use newlib;
	print Newlib::myfun(10,20), "\n";
	
	[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
	30
	```	
* 添加路径到PERL5LIB，但结束session即失效（export特点导致）
	```
	[huawei@n148 perl]$ echo $PERL5LIB  # 查看$PERL5LIB库路径变量
	/home/huawei/perl5/lib/perl5:/home/huawei/perl5/lib/perl5:/home/huawei/perl5/lib/perl5
	[huawei@n148 perl]$ export PERL5LIB=$PERL5LIB:lib	# 添加路径，可以相对但最好绝对
	[huawei@n148 perl]$ echo $PERL5LIB
	/home/huawei/perl5/lib/perl5:/home/huawei/perl5/lib/perl5:/home/huawei/perl5/lib/perl5:lib	# 相对就这样的了，且只能在固定位置运行才能找到相对lib位置

	#!/usr/bin/perl
	use newlib;
	print Newlib::myfun(10,20), "\n";

	[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
	30
	```
## 管理分发包的CPAN工具
Perl核心包含了几个用于管理分发包的工具:
* CPAN.pm  
是正式的CPAN客户端。默认的，客户端会从公共库CPAN来安装发行包，当然你也可以配置成指定的库来代替或补充公共库（CPAN）。
* ExtUtils::MakeMaker  
是个复杂但好用的工具，用于打包、构建、测试和安装分发包，它和Makefile.PL一起工作。
* Test::More  
用于编写自动化测试，使用最广泛的基础模块。
* TAP::Harness和prove  
用来运行测试，解析和报告测试结果。
* App::cpanminus  
是一个无需配置的CPAN客户端，它可以处理最常见任务，使用很少的内存并且运行得很快。
* App::perlbrew  
可以帮助管理多个Perl版本。
* CPAN::Mini和cpanmini命令  
允许创建自己的CPAN镜像。可以将自己的分发包放到私有镜像库里面，还可以进行版本管理。（有些情形下非常有用，比如针对自己公司指定哪些版本可用）
* Dist::Zilla  
可自动执行一些分发包相关任务。利用Module::Build或 ExtUtils::MakeMaker进行工作，避免直接使用它们。这里有相关的教程：http://dzil.org/ 。
* Test::Reporter  
报告分发包自动化测试的结果，给作者提供更详细的故障细节。
* Carton和Pinto  
是2个新项目，它们旨在帮助你管理和安装代码的依赖。尚未被广泛使用，处在积极的开发中。
* Module::Build  
功能和ExtUtils::MakeMaker一样，但停止维护了。

## 创建模块 Module::Starter
Module::Starter模块提供了一个命令行程序module-starter，它可以用来构建模块。先安装之
```
[huawei@n148 perl]$ cpan -i Module::Starter
```
常用参数选项如下：
```
--module=module  项目的主模块名 (required, repeatable)
--distro=name    项目名 (optional)
--dir=dirname    新的项目会放到哪个目录中 (optional)

--builder=module 使用哪个模块进行构建，可用的值有： 'ExtUtils::MakeMaker' 和 'Module::Build'
--eumm           和 --builder=ExtUtils::MakeMaker 的功能相同
--mb             和 --builder=Module::Build 的功能相同
--mi             和 --builder=Module::Install 的功能相同

--author=name    作者是名字 (taken from getpwuid if not provided)
--email=email    作者的电子邮件 (taken from EMAIL if not provided)

--ignores=type   需要忽略的文件类型 (repeatable)
--license=type   开源许可证
                 (default is artistic2)
--minperl=ver    支持的最小的Perl版本 (optional  default is 5.006)

--fatalize       生成warnings代码,指定所有警告都会引发致命错误（use warnings FATAL => 'all'）

--verbose        打印详细的工作日志
--force          强制执行，覆盖已经存在的文件和文件夹

--help           显示帮助信息
```
安装后构建一个名为My::Number::Utilities的模块(模块可以不用存在，它会自动创建一些文件结构)

	

	[huawei@n148 perl]$ module-starter --module "My::Number::Utilities" --author="ma long shuai" --email="79123@qq.com"
	Added to MANIFEST: Changes
	Added to MANIFEST: ignore.txt
	Added to MANIFEST: lib/My/Number/Utilities.pm
	Added to MANIFEST: Makefile.PL
	Added to MANIFEST: MANIFEST
	Added to MANIFEST: README
	Added to MANIFEST: t/00-load.t
	Added to MANIFEST: t/manifest.t
	Added to MANIFEST: t/pod-coverage.t
	Added to MANIFEST: t/pod.t
	Added to MANIFEST: xt/boilerplate.t
	Created starter directories and files

	[huawei@n148 perl]$ tree My-Number-Utilities/
	My-Number-Utilities/
	├── Changes
	├── ignore.txt
	├── lib
	│   └── My
	│       └── Number
	│           └── Utilities.pm
	├── Makefile.PL
	├── MANIFEST
	├── README
	├── t
	│   ├── 00-load.t
	│   ├── manifest.t
	│   ├── pod-coverage.t
	│   └── pod.t
	└── xt
		└── boilerplate.t

	5 directories, 11 files	

项目文件说明：

    Changes：模块变更的日志。
    MANIFEST：列出发布的模块版本中必须包含的每个文件。发行模块中包含的所有文件的列表。		这助于打包工具生成完整的 tar 压缩包并帮助验证压缩包使用者得到所有必需的文件；
    Makefile.PL：是Perl程序创建的Makefile，Makefile文件中包含了如何编译和构建程序的说明。如果不是非常熟悉Makefile格式，千万不能修改，一个tab和空格的不同可能都会导致编译出错。
	Build.PL 或是 Makefile.PL，这些程序用于配置、构建、测试、捆绑及 安装模块；
	META.yml 和/或 META.json：一个包含有关发行模块及其依赖的元数据文件；
    README：对发行模块的描述、意图、版权和许可信息；包含了如何构建和安装程序的文档，一般来说，程序的说明文档会嵌入在此文件中。
    ignore.txt：是类似于git/svn版本控制系统的文件忽略模板，你可能会经常拷贝该文件到你的版本控制系统中。
    lib/：该目录包含了实际要安装的模块。含有 Perl 模块的目录；
    t/：代码测试目录，该目录包含了模块测试相关的文件。
	xt/：作者测试目录。

	上面的文件可能已经包含了一些内容，例如lib/My/Number/Utilities.pm文件中已经自动包含了一些编译指示，还包含了默认的pod文档。

默认情况下，Module::Starter构建的模式是Makefile.PL，如果想要构建Build.PL，可以加上--builder=Module::Build或者无需选项参数的选项--mb：
	
	两种效果一样，一个简写一个全称。。。

	[huawei@n148 perl]$ module-starter --mb --module "My::Number::Utilities" --author="ma long shuai" --email="79123@qq.com" --force

	[huawei@n148 perl]$ module-starter --builder=Module::Build --module "My::Number::Utilities" --author="ma long shuai" --email="79123@qq.com" --force
## 默认项配置文件
创建如下文件后可以省略部分选项。再执行module-stater时，只需一个--module选项即可
```
[huawei@n148 perl]$ cat $HOME/.module-starter/config
author: ma long shuai
email: 79123@qq.com
builder: Module::Build
verbose: 1

[huawei@n148 perl]$ module-starter --module=My::Module
```
## 添加pm到现有模块
先安装必备包，然后再配置文件中加入一行
```
[huawei@n148 perl]$ cpan -i Module::Starter::AddModule
[huawei@n148 perl]$ cat $HOME/.module-starter/config
author: ma long shuai
email: 79123@qq.com
builder: Module::Build
verbose: 1
plugins: Module::Starter::AddModule	# 新加此行


如下--dist指定要添加到的模块目录，如果已经在Animal-Rule目录下，则使用相对路径--dist=.即可。
[huawei@n148 perl]$ module-starter --module=Sheep --dist=Animal-Rule
```


## make、test、install
分为两种不同的形式处理
* Makefile.PL  
```
	perl Makefile.PL	# 用于生成真正的makefile
	make	# 编译，并cp对应的pm文件与自动创建的doc到blib子目录（此目录也是自动创建）
	make test	# 测试
	make disttest	# 在新备份的子目录里测试
	make install
	make dist	# 测试打包到压缩文件，为了接下来的分发
```
* Build.PL
```
	perl Build.PL
	./Build
	./Build test
	./Build install
	./Build dist
```
## 改写Makefile.PL与Build.PL
虽然Makefile.PL和Build.PL不建议修改，但小心翼翼点也可以修改。可以通过此2文件添加依赖包与测试项，见  
https://blog.csdn.net/weixin_34124577/article/details/86133529
## 构建多个模块
会以第一个模块名称作为顶级目录，然后在lib中创建各模块文件
```
[huawei@n148 perl]$ module-starter --module "Animal,Cow,Horse,Mouse" --author="ma long shuai" --email="79123@qq.com" --force
Animal/
├── Changes
├── lib
│   ├── Animal.pm
│   ├── Cow.pm
│   ├── Horse.pm
│   └── Mouse.pm

[huawei@n148 perl]$ module-starter --module "Animal::Rule,Cow::Rule,Horse,Mouse" --author="ma long shuai" --email="79123@qq.com" --force
Animal-Rule/
├── Changes
├── lib
│   ├── Animal
│   │   └── Rule.pm
│   ├── Cow
│   │   └── Rule.pm
│   ├── Horse.pm
│   └── Mouse.pm
```
## 编辑POD
- 可以直接再生成的pm文件中对应的段落插入说明性的文字内容
- 可以使用字体信息
- 编辑后可以使用podchecker Animal/lib/Animal.pm进行检查
## TAP测试
* 使用工具创建的模块目录中的t与xt为测试脚本目录
* 测试脚本均为.t文件
* 使用Test::More模块进行测试
* 启动方式如下
	```
	测试 oo-load.t
	[huawei@n148 testclass]$ ./Build  先编译
	[huawei@n148 testclass]$ perl -Iblib/lib -T t/00-load.t
	ok 1 - use Cow;
	ok 2 - use Horse;
	ok 3 - use Sheep;
	1..3
	# Testing Cow 0.01, Perl 5.016003, /usr/bin/perl
	```
* 如后期又添加pm文件，则需要手动修改oo-load.t用于测试所有的use逻辑
* pod测试用于检测pod是否完整无误，需要安装Test::Pod与Test::Pod::Coverage模块，t目录下的pod.t与pod-coverage.t用于此测试


# 多文件
## 导出变量与函数
有这个就行了，后面几种方法都可以不用看了。。。
m2.pm
```
package m2;

use Modern::Perl;
use Cwd;
use Mojo::File;


use Exporter;
our @ISA = 'Exporter';
our @EXPORT = qw($htmlpath myadd);

sub myadd
{
	my $a=shift;
	my $b=shift;
	print "mod1\n";
	$a+$b;
}

my $rootpath = cwd();
our $htmlpath = Mojo::File->new($rootpath, 'html');


1
```
m3.pm
```
package m3; 
use strict;
use warnings;

use Exporter;
our @ISA = 'Exporter';
our @EXPORT = qw(@a @b);

our (@a, @b);

@a = 1..3;
@b = "a".."c";
```
test.pl
```
use Modern::Perl;
use Cwd;

use m3;     # Only imports $a 
use m2;
say @a;
say @b;
say $htmlpath;
say myadd(10,20);
```
运行如下，确保在PERL5LIB里包含了".",即当前目录，否则会找不到。。。
另外确保你的终端当前路径在此pl文件夹内，否则还是找不到pm。。。
```
perl "/home/huawei/hwwork/postdb_doc/test.pl"
123
abc
/home/huawei/hwwork/postdb_doc/html
mod1
30
```
## eval方式插入代码
能用的方式，但不够漂亮。执行多次会出现代码重定义（未测试）
```
a.pm内定义函数
sub myadd
{
	my $a=shift;
	my $b=shift;
	$a+$b;
}


b.pl内加载并调用函数
sub loadmyadd{
	open my $more_fh, '<', '1.pl';
	undef $/;
	my $more_code=<$more_fh>;		# $more_code里是全部代码内容
	close $more_fh;
	eval $more_code;				# 相当于把代码都添加进来
	die $@ if $@;					# 断言代码没问题
}
loadmyadd;
print myadd(10,30);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
40
```
## do方式导入
执行多次会出现代码重定义（未测试）
```
do q/mod.pm/;
die $@ if $@;	# 断言代码没问题
print myadd(10,30);

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
40
```
## require方式导入
require方式最佳，不会出现代码重定义
```
mod.pm内定义函数，注意被require的文件末尾要有1（是个真值即可）否则报错
sub myadd
{
	my $a=shift;
	my $b=shift;
	$a+$b;
}
1

方式1：pm文件直接带路径
require q/mod.pm/;

方式2：将路径临时放入@INC
unshift (@INC, "$ATL3PARAM{'MAINPATH'}");
require "util.pm";

调用函数:
print myadd(10,30);
[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
40
```
## 命名空间解决同名问题
* require多个文件里具有同名函数时候，pl文件中默认使用靠下的require文件里的函数
* 当require和pl文件里具有同名函数时候，默认使用的是require里的函数
* 靠包名来解决此问题，包名建议大写。如下
```
mod1.pm文件，注意要写包名package m1;
#!/usr/bin/perl
package m1;
sub myadd
{
	my $a=shift;
	my $b=shift;
	print "mod1\n";
	$a+$b;
}

1



mod2.pm文件
#!/usr/bin/perl
package m2;
sub myadd
{
	my $a=shift;
	my $b=shift;
	print "mod2\n";
	$a+$b;
}

1


调用包的pl文件，调用自身可以再加作用域main（其实可以不用）
sub myadd
{
	my $a=shift;
	my $b=shift;
	print "pl\n";
	$a+$b;
}
require q/mod2.pm/;
require q/mod1.pm/;

print m2::myadd(10,30), "\n";
print m1::myadd(10,30), "\n";
print myadd(10,30), "\n";
print main::myadd(10,30), "\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
mod2
40
mod1
40
pl
40
pl
40
```

# 包
## 名称空间和包
* 名称空间是一种机制，它将若干具名实体关联并封装于某具名分类之下
* 包是单一名称空间下代码的集合。
* 在某种意义上，包和名称空间是等价的。包代表源代码而名称空间代表当Perl分析这段代码时创建的实体。
* 包是独立于文件的，一个文件中可以有多个包，一个包也可能跨多个文件。所以一个.pm不一定只有一个命名空间。
* 包提供了基本结构模块，基于这些构造模块，可以构建更高层的模块和类概念。
* 当未明确声明一个包，无论在命令行或在Perl程序、甚至是pm文件默认包都是main包。
* 名称空间无继承关系
* 可以使用导出的方式在A包里使用B包的函数且调用时无需包名前缀和::
```
Perl5.12引入了一种简化版本号的定义package新语法一行即可  
package MyCode 1.2.1;
```


# 符号表、类型团、GLOB
## 概念


* 符号表 symbol table
  * 包的内容称为符号表。一个包就是一个符号表。其中包含了这个包中的所有变量及子程序的名字
  * 符号表存储在一个散列中，这个散列与包同名，并且要在后面追加 : : 。
  * main符号表就是%main::，由于main是默认的包，所以可以写作%::。
  * 符号表的键是符号标识符，值是对应的类型团。
* 类型团 typeglob
  * 类型团typeglob是Perl中的一个特殊类型，类型前缀是*，表示所有类型。
  * *foo包含$foo @foo %foo &foo，以及foo的多种其他表示对应的值。
  * 类型团是可理解成一个散列结构的数据结构，其键是固定的，包含SCALAR ARRAY HASH CODE GLOB IO NAME PACKAGE。
* GLOB
  * 类型团数据的数据类型就是GLOB。
  * 输出类型团的数据类型 our a = 1; print "type of typeglob: ", ref \*main::a; # type of typeglob: GLOB
  * 访问类型团，证明其实散列结构 our a = 1; print ${*main::a{SCALAR}}, ${*a{SCALAR}}; # 11
* 变量和函数
  * 命名空间中的变量和函数，可以通过符号表的类型团访问。	
  * 但是符号表机制对词法作用域变量不可见。（my state是词法作用域变量）
```
our $val = 1;
my %hash = qw(a aa b bb c cc);

sub phash {
    print "\n" . $hash{ +shift } . "\n";
}

## 打印符号表
print "打印符号表:\n";
foreach my $key (keys %main::) {
    no strict;
    local *sys = $main::{$key};
    print "KEY\t$key\t\tVALUE:\t @{[*sys]} \n";
}

## 类型团的数据类型就是GLOB
print "\n类型团的数据类型就是GLOB: ";
print ref \*main::val;  ## GLOB
## 从符号表中读取变量
print "\n从符号表中读取变量:  ";
print ${$main::{val}}, ${*main::val{SCALAR}},$main::val; ## 111
## 类型团是一个散列，存储的值是引用
print "\n类型团是一个散列，存储的值是引用:  ";
print *main::phash{CODE}, "\n"; ## CODE(0x6b4718)


[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/2.pl"
打印符号表:
KEY     version::               VALUE:   *main::version:: 
KEY     /               VALUE:   *main::/ 
KEY     stderr          VALUE:   *main::stderr 
KEY     SIG             VALUE:   *main::SIG 
.
.
.
.
.
.
KEY     @               VALUE:   *main::@ 
KEY     STDOUT          VALUE:   *main::STDOUT 
KEY     ]               VALUE:   *main::] 
KEY                     VALUE:   *main:: 
KEY     threads::               VALUE:   *main::threads:: 
KEY     Dumper          VALUE:   *main::Dumper 
KEY     bytes::         VALUE:   *main::bytes:: 
KEY     STDERR          VALUE:   *main::STDERR 
KEY     _<Dumper.c              VALUE:   *main::_<Dumper.c 

类型团的数据类型就是GLOB: GLOB
从符号表中读取变量:  111
类型团是一个散列，存储的值是引用:  CODE(0x27212a0)
```
# Debug Output
## Data::Dumper
可以输出结构，案例此文档中搜索即可
## Data::Printer
可以输出对象且有颜色
```
use Data::Printer;
use LWP::UserAgent;

my $ua = LWP::UserAgent->new;
p $ua;
```
## Data::Show
类似Dumper，省掉了print
```
use strict;
use warnings;
use Data::Show;

my @array = (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

my %hash  = ( foo => 1, bar => { baz => 10, qux => 20 } );

my $href = \%hash;
show @array;
show %hash;
show $href;
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

## 读取文件第一行
```
use IO::File;
my $file=shift;
my $fn=IO::File->new($file);
my $line=<$fn>;
print $line;

通过命令行参数指定文件名
[huawei@n148 perl]$ /usr/bin/perl  2.pl err.txt
```
## 读取文本文件
将文件作为perl命令行的参数，perl会使用<>去读取这些文件中的内容。
```
open my $fh, '<', "/etc/services";
while (<$fh>)
{
	print $_;
}
close $fh;


------------------
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
---------
my $data=<STDIN>;
chomp($data);
print STDOUT "you said: $data\n";
---------
chomp(my $data=<>);
print STDOUT "you said: $data\n";


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
* print 不带\n； 默认输出到STDOUT，可以使用select指定目标句柄  
  如 print OUTFILE ("Hello, there!\n"); 则是输出到文件
* say 自带\n，必须结合use 5.10才能使用；
* printf 格式化输出字符串；默认输出到STDOUT，也可以指定句柄
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
## uft8编码文本文件写入与读取
```
Write to a file：
use open qw( :encoding(UTF-8) :std ); # Make UTF-8 default encoding


Opening UTF8 Text Files：
open my $filehandle, '<:raw:encoding(utf-8)', 'path/to/file' 
   or die "Can't open $name_of_file, $!";
```
## 设定当前输出句柄 select
默认输出句柄是STDOUT，可以使用select指定默认句柄
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
带有管道操作的 Perl 脚本是不能直接在不同系统间互相移植的。pipe增强了open函数，能返回对方的pid，可用于信号的通讯
* 管道符号在命令前面  
	在代码里被open的句柄用于写入数据，被写入的数据用作管道符号后面要执行的命令的标准输入内容
```
使用wc计算字符串中的字符数目并输出

my $pid=open(MYPIPE, "| wc -w");
print MYPIPE "apples pears peaches";
print "pipe handle\n" if -p MYPIPE;		# 检测句柄类型，见文件测试一节
close(MYPIPE);
print "$pid\n";

[huawei@n148 perl]$ /usr/bin/perl "/home/huawei/playground/perl/1.pl"
3
114360
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
* 命令后面是管道符合  
	在代码里被open的句柄用于读取数据，被读取的内容来自管道符合前面的外部命令的标准输出
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

更高级的还有pipe()函数，详见perl网络编程p35
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
## 命令行参数
use Getopt::Std; 参考perl调试技术5.4


# List::Util模块
# Regexp::Common
# File::Basename
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

# 面向对象
## bless
http://blog.chinaunix.net/uid-28246152-id-3399154.html
## UNIVERSAL包
Perl内部的UNIVERSAL包是其他所有包的祖先，以面向对象的视角来看那就是终极父类。UNIVERSAL提供了一些方法可供使用、继承和重写。
### VERSION()方法
返回调用者（包或类）里的变量$VERSION的值。如果你提供了版本号作为参数且比调用者里$VERSION的值还要大，那就会产生一个异常。
```
use Modern::Perl;
use Test::More;
unshift (@INC, "./");
require "Duck.pm";
my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);
say Duck->VERSION; # prints 1.23
say $duck_object->VERSION; # prints 1.23
say $duck_object->VERSION( 0.0 ); # prints 1.23
say $duck_object->VERSION( 1.23 ); # prints 1.23
say $duck_object->VERSION( 2.0 ); # exception!

[huawei@n148 perl]$ perl 2.pl 
1.23
1.23
1.23
1.23
Duck version 2 required--this is only version 1.23 at 2.pl line 13.
```
### DOES()方法
是用来进行角色机制判断的，参数是调用者和角色名，如果调用者实现了该角色就返回真值（角色实现的机制可以有很多，如继承、委托、组成、角色应用或其他机制）。  
默认DOES()的实现机制就是通过isa()方法，因为继承（isa()用于继承）就是这样一个机制：一个类实现了一个角色。
```
此案例仅演示调用，但不是很合适，待将来替换

use Modern::Perl;
use Test::More;
use Test::Class::Load;

unshift (@INC, "./");
require "Duck.pm";
my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);

say $duck_object->DOES( 'Duck' ); # prints 1
say $duck_object->DOES( 'Animal' ); # prints 1

[huawei@n148 perl]$ perl 2.pl 
1
1
```
### can()方法
用来判断调用者是否具有某方法。参数为方法名字符串，如果调用者具有该方法则返回该方法引用，否则就返回假值。你可以在类、对象、包上调用can()方法。
```
use Modern::Perl;
use Test::More;

unshift (@INC, "./");
require "Duck.pm";
my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);

if (my $meth = $duck_object->can('fly'))
{
	$duck_object->$meth();
}
if (my $meth = Duck->can('fly'))
{
	$duck_object->$meth();
}

[huawei@n148 perl]$ perl 2.pl 
It can fly
It can fly

```
### isa()方法
isa()方法接受一个字符串，可以是类名或类型名（SCALAR, ARRAY, HASH, Regexp, IO,CODE）。可以把它当成类方法或实例方法来使用，如果调用者派生于或就是给定类（或调用者就是指定类型的bless过的引用）则返回真。  
任何类都可以重写isa()方法。
```
use Modern::Perl;
use Test::More;
use Test::Class::Load;

unshift (@INC, "./");
require "Duck.pm";
my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);

say $duck_object->isa( 'Duck' ); # prints 1
say $duck_object->isa( 'Animal' ); # prints 1
```
## Moose
Moose是一个完整的面向对象系统，功能齐全而且易于使用。它不是Perl核心模块的一部分，所以需要从CPAN上下载、安装才能使用
### 类、方法、属性
```
use Modern::Perl;
{
	package Cat;
	use Moose;

	# 定义成员方法
	sub meow
	{
		# 指向自身this
		my $self = shift;
		say 'Meow!';
	}

	# 定义属性，ro 可读但不可写read-only
	has 'name1', is => 'ro', isa => 'Str';
	has 'name2' => ( is => 'ro', isa => 'Str' );
	has( 'name3', 'is', 'ro', 'isa', 'Str' );
 	has( qw( name4 is ro isa Str ) );
	has 'name5' => (
		is => 'ro',
		isa => 'Str',
		# 高级 Moose 选项；perldoc Moose
		init_arg => undef,
		lazy_build => 1,
	);

	# 属性的类型可以缺省
	has 'age1', is => 'ro';
	has 'age2', is => 'ro', isa => 'Int';
	# rw 可读且可写read/write 
	has 'diet', is => 'rw';
}

# 创建对象，调用方法
my $alarm = Cat->new();
$alarm->meow();
$alarm->meow();
$alarm->meow();

# 通过类名调用
Cat->meow() for 1 .. 3;

for my $name (qw( Tuxie Petunia Daisy ))
{
	# 构造传参
	my $cat = Cat->new( name1 => $name );
	# 调用自动生成的访问器
	say "Created a cat for ", $cat->name1();
}

# 使用缺省类型的age1
my $invalid = Cat->new( name1 => 'bizarre', age1 => 'purple' );
say "invalid cat for ", $invalid->age1();
my $invalid2 = Cat->new( name1 => 'bizarre', age1 => 12 );
say "invalid2 cat for ", $invalid2->age1();


my $fat = Cat->new( name1 => 'Fatty', age1 => 8, diet => 'Sea Treats' );
say $fat->name1(), ' eats ', $fat->diet();
# 对可写的属性重新赋值
$fat->diet( 'Low Sodium Kitty Lo Mein' );
say $fat->name1(), ' now eats ', $fat->diet();


[huawei@n148 perl]$ perl 2.pl 
Meow!
Meow!
Meow!
Meow!
Meow!
Meow!
Created a cat for Tuxie
Created a cat for Petunia
Created a cat for Daisy
invalid cat for purple
invalid2 cat for 12
Fatty eats Sea Treats
Fatty now eats Low Sodium Kitty Lo Mein

```
### 属性的默认值
```
use Modern::Perl;
{
	package Cat;
	use Moose;
	has 'name', is => 'ro', isa => 'Str';
	has 'diet', is => 'rw';
	# 普通样式
	# has 'birth_year', is => 'ro', isa => 'Int';

	# 默认值样式
	has 'birth_year', is => 'ro', isa => 'Int', default => 2021;

	# 使用提供的方法计算默认值
	# has 'birth_year', is => 'ro', isa => 'Int', default => sub { (localtime)[5] + 1900 };

	sub age
	{
		my $self = shift;
		my $year = (localtime)[5] + 1900;
		return $year - $self->birth_year();
	}	 
}

my $fat = Cat->new( name => 'Fatty', birth_year => 2020, diet => 'Sea Treats' );
say $fat->age();
my $fat2 = Cat->new( name => 'Fatty2', diet2 => 'Sea Treats' );
say $fat2->age();

[huawei@n148 perl]$ perl 2.pl 
2
1
```
### 继承
* 关键字extends用来表示要继承父类，可以多继承
```
演示最基本的继承，但是分为了多个文件


定义基类
[huawei@n148 perl]$ cat Animal.pm 
package Animal;
use Moose;
has 'name'=> (
        is => 'rw',
        isa => 'Str',
);
sub whatamI {
        my $self = shift;
        print "This is a $self->{name}\n";
}


定义子类
[huawei@n148 perl]$ cat Duck.pm 
package Duck;
require "Animal.pm";

our $VERSION = '1.23';
use Moose;
extends 'Animal';
has 'flyable' => (
  is => 'rw',
  isa => 'Bool',
  default => 1,
);

sub fly {
  my $self = shift;
  if ($self->flyable){
    print "It can fly\n";
  }
  else {
    print "Wow it can't fly\n";
  }
}


调用
[huawei@n148 perl]$ cat 2.pl 
use Modern::Perl;
unshift (@INC, "./");
require "Duck.pm";

my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);

$duck_object->whatamI();


运行结果
[huawei@n148 perl]$ perl 2.pl 
This is a duck
```
### 覆盖与重载
* 子类的属性名字前面使用加号，表示在子类中会对这个继承来的属性做一些特别的事情，下例里是重写了默认值
* 子类还可以重写方法（是覆盖），见下例的Glowstick::extinguish
* super()指明去最近的父类调度当前的方法（原始的老式样，moose使用新的after与before），见下例的Cranky
```
演示子类覆盖父类的属性默认值

use Modern::Perl;
package LightSource 
{
	use Moose;

	has 'candle_power', is => 'ro',
	isa => 'Int',
	default => 1;

	has 'enabled', is => 'ro',
	isa => 'Bool',
	default => 0,
	# enabled的writer选项创建了一个私有访问器来设置该值，下面2个方法内使用
	writer => '_set_enabled';

	sub light {
		my $self = shift;
		$self->_set_enabled( 1 );
		say "set enable 1";
	}

	sub extinguish {
		my $self = shift;
		$self->_set_enabled( 0 );
		say "set enable 0";
	}
}

package SuperCandle
{
	use Moose;
	extends 'LightSource';
	has '+candle_power', default => 100;
}

package Glowstick
{
	use Moose;
	extends 'LightSource';
	sub extinguish {}
}

package Cranky
{
	use Carp 'carp';
	use Moose;
	extends 'LightSource';
	override light => sub
	{
		my $self = shift;
		carp "Can't light a lit light source!" if $self->enabled;
		super();
	};

	override extinguish => sub
	{
		my $self = shift;
		carp "Can't extinguish unlit light source!" unless $self->enabled;
		super();
	};
}

# 创建基类对象并调用方法
my $v1 = LightSource->new();
say $v1->candle_power();

# 创建子类对象传参并调用方法
my $v2 = SuperCandle->new(
  candle_power => 50
);
say $v2->candle_power();

# 创建子类对象使用默认参数并调用方法
my $v3 = SuperCandle->new();
say $v3->candle_power();

# 调用了子类的空实现，不再调用父类对应方法了
my $v4 = Glowstick->new();
say $v4->extinguish();

# 创建带有参数的子类对象，调用子类重新实现的方法
my $v5 = Cranky->new(
	candle_power => 50,
	enabled => 1
);
$v5->light();

my $v6 = Cranky->new(
	candle_power => 50,
);
$v6->extinguish();


[huawei@n148 perl]$ perl 2.pl 
1
50
100

Can't light a lit light source! at /home/huawei/perl5/lib/perl5/x86_64-linux-thread-multi/Moose/Meta/Method/Overridden.pm line 38.
set enable 1
Can't extinguish unlit light source! at /home/huawei/perl5/lib/perl5/x86_64-linux-thread-multi/Moose/Meta/Method/Overridden.pm line 38.
set enable 0
```
### 新式的重载：after与before
* after是先调用父类实现，然后再调用子类实现
* before则反之
```
use Modern::Perl;
use Test::More;

package Point;
use Moose;     
has 'x' => (isa => 'Int', is => 'ro');
has 'y' => (isa => 'Int', is => 'rw');

sub clear {
    my $self = shift;
    $self->{x} = 0;
    $self->y(0);    
}
sub say {
	my $self = shift;
	say "x: $self->{x}, y: $self->{y}";
}

package Point3D;
use Moose;
extends 'Point';
has 'z' => (isa => 'Int', is => 'ro');
   
after 'clear' => sub {	# after 先调用父类实现，然后调用子类实现
    my $self = shift;
    $self->{z} = 0;
};
after 'say' => sub {
	my $self = shift;
	say "z: $self->{z}";
};

my $point = Point->new(x => 1, y => 2);
$point->say();
$point->clear();
my $point3d = Point3D->new(x => 1, y => 2, z => 3);
$point3d->say();
$point3d->clear();
$point3d->say();


[huawei@n148 perl]$ perl 2.pl 
x: 1, y: 2
x: 1, y: 2
z: 3
x: 0, y: 0
z: 0
```
关于before的案例，此案例仅演示语法，感觉不是选的不好
```
use Modern::Perl;
use Test::More;

# 基类普通账号
package BankAccount;
use Moose;
# 余额
has 'balance' => (isa => 'Int', is => 'rw', default => 0);

# 存钱
sub deposit {
    my ($self, $amount) = @_;
    $self->balance($self->balance + $amount);
}

# 取钱
sub withdraw {
    my ($self, $amount) = @_;
    my $current_balance = $self->balance();
	# 检测要取的钱要少于自己有的，否则就透支了
    ($current_balance >= $amount) || confess "Account overdrawn";
    $self->balance($current_balance - $amount);
}
sub say{
	my $self = shift;
	say "balance: $self->{balance}";
}

# 子类，包含一个自身类型的成员，用于透支的操作
package CheckingAccount;
use Moose;
extends 'BankAccount';
has 'overdraft_account' => (isa => 'BankAccount', is => 'rw'); # 这里有个自身类型的属性

before 'withdraw' => sub { # 先执行自己再执行父类
    my ($self, $amount) = @_;
    my $overdraft_amount = $amount - $self->balance(); # 要取的钱大于自己有的
    if ($self->overdraft_account && $overdraft_amount > 0) { # overdraft_amount>0则代表自己钱不够
        $self->overdraft_account->withdraw($overdraft_amount);	# 从别人那先减去自己差的那部分钱，如果别人那也不够则confess崩了
        $self->deposit($overdraft_amount); # 从自己账户里减去要取的
    }
};

my $savings_account  = BankAccount->new(balance => 250);
$savings_account->deposit(100);
$savings_account->withdraw(200);
$savings_account->say();
my $checking_account = CheckingAccount->new(balance => 100,overdraft_account => $savings_account);
$checking_account->deposit(1000);
$checking_account->say();
$checking_account->withdraw(200);
$checking_account->say();
$checking_account->withdraw(1000); # 此处后自己没钱了，且透支账户也减了自己差的那部分
$savings_account->say();
$checking_account->say();

[huawei@n148 perl]$ perl 2.pl 
balance: 150
balance: 1100
balance: 900
balance: 50
balance: 0
```
### moose待完善内容
```
Recipe3：weak_ref predicate
Recipe4：ArrayRef subtype
Recipe5：coerce via
```

### with、DOES、isa、class、method、role
* Moose和它的MOP（meta-object protocol元对象协议）提供了更优雅的语法来使用类和对象
* MooseX::Declare模块增加了class，role，和method关键字，这些关键字可以使代码更加简洁。
* 还有一个选择就是Moops模块
* Moose非常强大也非常大，还有个小型版的Moose叫Moo，它更快，资源占用更低。
### 反射
* Class::MOP（Class::MOP）简化了许多对象系统中的反射任务
## 从定义类到简单基类
下面的命令使用了缺省参数，见$HOME/.module-starter/config内的配置
```
1 建立工程,会生成3个pm
[huawei@n148 perl]$ module-starter --module=Cow,Horse,Sheep  --dir=testclass    

2 每个pm中添加speak方法，打印对应的名称speak
sub speak {
	print "cow speak\n";
}

3 库目录同层添加一个pl文件，通过包名调用方法

#!/usr/bin/perl
use Cow;
use Horse;
use Sheep;

Cow::speak();
Horse::speak();
Sheep::speak();

4 只能终端手动执行才可运行，使用-I指定库（-I的作用是将目录加到@INC数组中），然后再运行pl，还得是全路径
[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
cow speak
horse speak
sheep speak

5 奇技淫巧的取地址方式调用
use Cow;
use Horse;
use Sheep;
my @pasture=qw/Cow Horse Sheep/;
foreach my $beast(@pasture){
	no strict 'refs';
	&{$beast."::speak"};
}

[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
cow speak
horse speak
sheep speak


6 通过类名方式调用
use Cow;
use Horse;
use Sheep;
Cow->speak();
Horse->speak();
Sheep->speak();

[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
cow speak
horse speak
sheep speak

7 基于类名形式循环调用
use Cow;
use Horse;
use Sheep;

my @pasture=qw/Cow Horse Sheep/;
foreach (@pasture){
	$_->speak();
}
[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
cow speak
horse speak
sheep speak

8 向OO的形式再推进一步，修改每个类的函数实现如下，各自仅sound里的实现不同。调用处不变，结果依然相同。此时speak可以单独提取出来了。。。

sub sound {
	'cow sound';
}
sub speak {
	my $class=shift;
	print "$class: ", $class->sound, "\n";
}

[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
Cow: cow sound
Horse: horse sound
Sheep: sheep sound

9 提取出基类，通过基类提供的抽象方法传递不同的参数实现初级多态
先添加基类到工程
[huawei@n148 perl]$ module-starter --module=Animal --dist=testclass

基类animal添加函数如下
sub sound {
	die 'you must rewrite';
}
sub speak {
	my $class=shift;
	print "$class: ", $class->sound, "\n";
}

每个子类在文件顶部分别修改为如下
use Animal;
our @ISA=qw/Animal/;  可以再升级为use parent qw/Animal/;代替定义@ISA
sub sound {	'cow sound';}

主调用逻辑如下
use Cow;
use Horse;
use Sheep;
my @pasture=qw/Cow Horse Sheep/;
foreach (@pasture){
	Animal::speak($_);
}
Animal::speak('Cow');
Animal::speak('Horse');
Animal::speak('Sheep');

[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
Cow: cow sound
Horse: horse sound
Sheep: sheep sound
Cow: cow sound
Horse: horse sound
Sheep: sheep sound
```
## 覆盖基类方法
```
添加mouse类
[huawei@n148 perl]$ module-starter --module=Mouse --dist=testclass
在mouse代码中不继承animal（即使继承下面两句也正常运行），即使与animal拥有相同的方法名称也不会有关系,使用的时候如下即可：
use Mouse;
Mouse->speak();
```
## 子类调用父类方法 SUPER
上例中的mouse代码修改为如下
```
use Animal;
use parent qw/Animal/;
sub sound {	'mouse sound';}
sub speak {
	my $class=shift;
	$class->SUPER::speak;
	print "...mouse.. \n";
}


use Mouse;
Mouse->speak();

此处先调用了父类的方法，后又执行了自己的逻辑
[huawei@n148 perl]$ perl -I/home/huawei/playground/perl/testclass/lib /home/huawei/playground/perl/2.pl
Mouse: mouse sound
...mouse.. 
```

# 单元测试
随着敏捷开发模式的流行，如何快速高效地适应不确定或经常性变化的需求显得越来越重要。要做到这一点，需要在开发过程的各个阶段引入足够的测试。而其中单元测试则是保证代码质量的第一个重要关卡。  
创建基于 Test::Simple、Test::More 和 Test::Class 的 Perl 单元测试框架
```
[huawei@n148 testclass]$ cpan -i Test::Simple
[huawei@n148 testclass]$ cpan -i Test::More 
[huawei@n148 testclass]$ cpan -i Test::Class

Hello.pm 用于下面的各类，内容如下
use strict; 
use warnings; 
package Hello; 
$Hello::VERSION = '0.1'; 
sub hello { 
my ($you)=@_; 
return "Hello, $you!"; 
} 
sub bye { 
my ($you)=@_; 
return "Goodbye, $you!"; 
} 
1;
```
## Test::Simple
这个模块只有一个ok()，语法如下：
		
	ok(Arg1, Arg2)
	Arg1: 布尔表达式，如果这个表达式为真， 这个 testcase passed，否则 failed；
	Arg2: 这个参数是可选的，用来设置 testcase name。
```
use Test::Simple tests => 2; # 设置case数量
use Hello;	# 要测试的模块

# 调用模块内的方法，然后ok断言
my $hellostr=Hello::hello('guys'); 
my $byestr=Hello::bye('guys'); 
ok($hellostr eq 'Hello, guys!', 'hello() works'); 
ok($byestr eq 'Goodbye, guys!', 'bye() works'); 

运行测试
[huawei@n148 testclass]$ perl "/home/huawei/playground/perl/test_simple.pl"
1..2
ok 1 - hello() works
ok 2 - bye() works
```
## Test::More
提供了更多更广泛的对 testcase 是否成功的支持  
```
ok 			判断布尔值为t	
			ok(布尔值, “描述此条测试目的的字符串”);
			只会提供失败测试条目的行号
			ok( 1, 'the number one should be true' );
			ok( !'', 'the empty string should be false' );

is/isnt 	is()函数相当于使用Perl的eq操作符来比较2个值，如果值相等就认为测试通过
			s() and isnt()都是进行字符串的比较（相当于使用的是eq，ne操作符）
			is($got,$expected,$test_name);
  	  		isnt($got,$expected,$test_name);
			is() 显示未能匹配的值。
			is( 4, 2 + 2, 'addition should work' );
			isnt( 'pancake', 100, 'pancakes are numeric' );

like/unlike 正则表达式比较 	
			like( $got, qr/expected/, $test_name );
			nlike( $got, qr/expected/, $test_name );
cmp_ok 		可以指定操作符地比较 	
			cmp_ok($got,$op,$expected,$test_name);

			cmp_ok($auth_nologin_times, ">=" ,1, "$auth_nologin_times >= 1"); 
			cmp_ok($ret, 'eq','r4',  'r4');
			
can_ok 		用来验证一个类或对象是否具有某方法	
			can_ok($module,@methods)		
```
```
use Modern::Perl;
use Test::More;
unshift (@INC, "./");
require "Duck.pm";
my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);
can_ok( $duck_object, 'whatamI' );
can_ok( $duck_object, 'fly' );
done_testing();

[huawei@n148 perl]$ perl 2.pl 
ok 1 - Duck->can('whatamI')
ok 2 - Duck->can('fly')
1..2
[huawei@n148 perl]$ prove 2.pl 
2.pl .. ok   
All tests successful.
Files=1, Tests=2,  0 wallclock secs ( 0.02 usr  0.00 sys +  0.14 cusr  0.02 csys =  0.18 CPU)
Result: PASS
```
```	
isa_ok 		测试一个类或对象是否继承了其他类：
			isa_ok($object,$class,$object_name);
			isa_ok($subclass,$class,$object_name);	
```
```
use Modern::Perl;
use Test::More;
unshift (@INC, "./");
require "Duck.pm";
my $duck_object = Duck->new(
  name => 'duck',
  flyable => 1,
);
isa_ok( $duck_object, 'Duck' );
isa_ok( $duck_object, 'Animal' );
done_testing();

[huawei@n148 perl]$ prove 2.pl 
2.pl .. ok   
All tests successful.
Files=1, Tests=2,  0 wallclock secs ( 0.01 usr  0.00 sys +  0.15 cusr  0.02 csys =  0.18 CPU)
Result: PASS
[huawei@n148 perl]$ perl 2.pl 
ok 1 - An object of class 'Duck' isa 'Duck'
ok 2 - An object of class 'Duck' isa 'Animal'
1..2
```
```
subtest 	测试子集 	
			subtest $name=>\&code;
pass/fail 	直接给出通过 / 不通过 	
			pass($test_name);
  	  		fail($test_name);				
use_ok 		测试加载模块并导入相应符号是否成功 	
			BEGIN \{use_ok($module);}
			BEGIN \{use_ok($module,@imports);}
is_deeply 	用来比较2个引用的内容是否一样
			is_deeply($got,$expected,$test_name);	
```
```
use Modern::Perl;
use Clone;
use Test::More;
my $numbers = [ 4, 8, 15, 16, 23, 42 ];
my $clonenums = Clone::clone( $numbers );
is_deeply( $numbers, $clonenums, 'clone() should produce identical items' );
done_testing();

[huawei@n148 perl]$ prove 2.pl 
2.pl .. ok   
All tests successful.
Files=1, Tests=1,  0 wallclock secs ( 0.01 usr  0.01 sys +  0.03 cusr  0.00 csys =  0.05 CPU)
Result: PASS
```
```				
new_ok 		判断创建的对象是否 ok 	
			my $obj=new_ok($class);
			my $obj=new_ok($class=>@args);
			my $obj=new_ok($class=>@args,$object_name);					
```
介绍其中的几个:
* is(Arg1, Arg2, Arg3)    
  类似于 ok(), 用 eq 操作符比较 Arg1 和 Arg2 的值来决定 testcase 成功还是失败。Arg3 是指 testcase 的名字。
* like( Arg1, Arg2, Arg3 )  
  Arg2 是一个正则表达式 , 比较 Arg1 是否 匹配 Arg2 正则表达式。Arg3 是指 testcase 的名字。
* cmp_ok( Arg1, Arg2, Arg3, Arg4 )  
  cmp_ok() 允许用任何二元操作符（Arg2）比较 Arg1,Arg3. 同样，Arg4 是指 testcase 的名字。另外 cmp_ok() 有个好处，如果 testcase failed, 结果中会报告 Arg1 和 Arg2 在运行中的实际值。
* can_ok($ module, @methods); can_ok($object, @methods)  
  can_ok() 判断模块 $module 或对象 $object 能否调用方法 @methods。

```
use Modern::Perl;
use Test::More;
say "Both true!" if ok(1, 'first subexpression') && ok(1, 'second subexpression');
done_testing();
[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
ok 1 - first subexpression
ok 2 - second subexpression
Both true!
1..2


-----------------

use strict; 
use warnings; 
use Test::More tests => 4; # case数量
use Hello;	# 待测包

my $hellostr=Hello::hello('guys'); 
my $byestr=Hello::bye('guys'); 
is($hellostr, 'Hello, guys!', 'hello() works'); 
like($byestr, "/Goodbye/", 'bye() works'); 
cmp_ok($hellostr, 'eq', 'Hello, guys!', 'bye() works'); 
can_ok('Hello', qw(hello bye));

[huawei@n148 testclass]$ perl "/home/huawei/playground/perl/test_simple.pl"
1..4
ok 1 - hello() works
ok 2 - bye() works
ok 3 - bye() works
ok 4 - Hello->can(...)





use Modern::Perl;
use Test::More;
my $some_a = qr/ca+t/;
like( 'cat', $some_a, "'cat' matches /ca+t/" );
like( 'caat', $some_a, "'caat' matches/" );
like( 'caaat', $some_a, "'caaat' matches" );
like( 'caaaat', $some_a, "'caaaat' matches" );
unlike( 'ct', $some_a, "'ct' does not match" );
done_testing();
[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
ok 1 - 'cat' matches /ca+t/
ok 2 - 'caat' matches/
ok 3 - 'caaat' matches
ok 4 - 'caaaat' matches
ok 5 - 'ct' does not match
1..5
```
## prove
* 核心模块TAP::Harness，提供了prove命令来显示你最关心的信息
* perldoc prove可查看更多的测试选项，比如并发运行测试，自动增加包含路径，递归运行所有t目录下的测试条目，优先运行慢测试。
```
use Modern::Perl;
use Test::More;
ok( 1, 'the number one should be true' );
ok( !0, '... and the number zero should not' );
ok( !'', 'the empty string should be false' );
ok( '!', '... and a non-empty string should not' );
done_testing();

[huawei@n148 perl]$ prove 2.pl 
2.pl .. ok   
All tests successful.
Files=1, Tests=4,  0 wallclock secs ( 0.02 usr  0.00 sys +  0.03 cusr  0.00 csys =  0.05 CPU)
Result: PASS
[huawei@n148 perl]$ perl 2.pl 
ok 1 - the number one should be true
ok 2 - ... and the number zero should not
ok 3 - the empty string should be false
ok 4 - ... and a non-empty string should not
1..4
```
## Test::Exception
提供了保证代码正确,不抛出异常的函数
```
use Modern::Perl;
use Test::More tests => 6;
use Test::Exception;
throws_ok { die "I croak!" }
qr/I croak/, 'die() should throw an exception';
lives_ok { 1 + 1 }
'simple addition should not';

say "---------下面的是自定义匿名回调--------";
throws_ok( sub { die "I croak!" },
qr/I croak/, 'die() should throw an exception' );
lives_ok( sub { 1 + 1 },
'simple addition should not' );

say "---------下面的是自定义具名回调--------";
sub croak { die 'I croak!' }
sub add { 1 + 1 }
throws_ok \&croak,
qr/I croak/, 'die() should throw an exception';
lives_ok \&add,
'simple addition should not';

[huawei@n148 perl]$ /usr/bin/perl -I/home/huawei/playground/perl/123 "/home/huawei/playground/perl/2.pl"
1..6
ok 1 - die() should throw an exception
ok 2 - simple addition should not
---------下面的是自定义匿名回调--------
ok 3 - die() should throw an exception
ok 4 - simple addition should not
---------下面的是自定义具名回调--------
ok 5 - die() should throw an exception
ok 6 - simple addition should not
```
## Test::Class
属性是一个附加于变量或函数声明上的前置冒号标识符。
		
		my $fortress :hidden;
 		sub erupt_volcano :ScienceProject { ... }

属性可以包括一个参数列表；Perl 将它们作为一个常量字符串列表，即使它们可能会类似于其他值，如，数字或变量。来自 CPAN 的 Test::Class 模块很好地利用了这一参数形式详见下面的代码

Test::Class 的常用方法包括
* Test  
  参数N代表Test内case数，默认为1，也可为no_plan或空代表任意个
* setup、teardown  
  分别在每个普通测试方法之前和之后调用
* startup、shutdown  
  在所有测试方法执行之前和之后调用
* Runtests 方法  
  通过调用 Test::Class->runtests(); 执行所有从 Test::Class 派生的子类中定义的测试 case

```
use strict; 
use File::Util; 
use Test::More; 
use base qw(Test::Class); 

my $file; 
sub init : Test(startup){ 
 print "开始测试####################################################\n"; 
 $file = File::Util->new(); 
} 
sub shutdown : Test(shutdown){ 
 print "结束测试####################################################\n"; 
} 
sub initial : Test(setup) { 
 print "----------------------------------------------------开始case\n"; 
} 
sub end : Test(teardown) { 
 print "----------------------------------------------------结束case\n"; 
} 
sub test_methods : Test(no_plan) { 
 can_ok('File::Util', qw(existent line_count list_dir)); 
} 
sub test_existent_true : Test(no_plan) { 
 open(FILE, ">test.txt"); 
 print FILE "This is a test."; 
 close FILE; 
 cmp_ok($file -> existent('test.txt'), "==" ,1, 'test_existent_file_exists'); 
 unlink "test.txt"; 
 is($file -> existent('test.txt'), undef, 'test_existent_file_not_exists'); 
} 
sub test_line_count : Test { 
 open(FILE, ">test.txt"); 
 print FILE "This is a test.\n"; 
 close FILE; 
 cmp_ok($file -> line_count('test.txt'), "==" ,1, 'test_line_count = 1'); 
 open(FILE, ">test.txt"); 
 print FILE ""; 
 close FILE; 
 cmp_ok($file -> line_count('test.txt'), "==" ,0, 'test_line_count = 0'); 
} 
Test::Class->runtests();


[huawei@n148 perl]$ perl "/home/huawei/playground/perl/test_simple.pl"
开始测试####################################################
----------------------------------------------------开始case
ok 1 - test_existent_file_exists
ok 2 - test_existent_file_not_exists
----------------------------------------------------结束case
----------------------------------------------------开始case
ok 3 - test_line_count = 1
ok 4 - test_line_count = 0
# expected 1 test(s) in main::test_line_count, 2 completed
----------------------------------------------------结束case
----------------------------------------------------开始case
ok 5 - File::Util->can(...)
----------------------------------------------------结束case
结束测试####################################################
1..5
```

## 测试覆盖率
Devel::Cover模块可用于对函数、语句、分支、条件各自进行统计，分析测试套件的执行情况并报告经由实际测试代码的数量。一般说来，覆盖率越高越好────虽然不是总能达到 100% 覆盖率，但 95% 要比 80% 好上不少。

## 其他测试模块
* Test::Fatal帮助你测试代码抛出异常是否合理，类似Test::Exception。
* Test::MockObject和Test::MockModule帮助你进行仿真
* Test::WWW::Mechanize用来测试WEB程序。Plack::Test，Plack::Test::Agent和子类* Test::WWW::Mechanize::PSGI甚至允许你在测试时不使用额外的WEB服务器
* Test::Database提供函数来测试数据库的使用和滥用。DBICx::TestDatabase帮助你测试数据库* schemas。
* Test::Class提供了另一个组织测试套件的机制。它允许你创建特定测试方法的类，而且可以继承。* 这是一个很好重用测试套件的方法。
* Test::Differences测试字符串和数据结构是否一样，并且在诊断信息中显示差异信息。 * Test::LongString增加了类似的断言。
* Test::Deep用来测试嵌套数据。
* Devel::Cover会分析测试套件的执行情况，报告你的代码数量，报告覆盖率。
* Test::Most集成了几个有用的测试模块。
# 调试
使用perl -dw xxx.pl进行启动，效果类似gdb
# 安装perl

20220630:
1 最好的方法是先去除已有的perl，使用下面的卸载的方法，并且手动删除perl5那个目录
2 使用下面的安装方法安装新的perl版本，但没有cpan，更没有cpanm

不要使用下面的2句，因为默认又把perl回到5.16了
sudo yum install perl-CPAN
sudo yum install perl-App-cpanminus.noarch


## 卸载系统自带的perl
```
yum remove perl
然后再 whereis perl 看哪里还有然后手动删除
```
## 安装perl,cpan
```
wget https://www.cpan.org/src/5.0/perl-5.36.0.tar.gz
tar zxvf perl-5.36.0.tar.gz
cd perl-5.36.0
./Configure -des -Dprefix=/usr/local/perl -Dusethreads -Uversiononly
make -j 3
make -j 3 install


cd /usr/bin
mv perl perl.old
ln -s /usr/local/perl/bin/perl /usr/bin/perl 
perl -version 


wget https://cpan.metacpan.org/authors/id/A/AN/ANDK/CPAN-2.34.tar.gz
tar zxvf CPAN-2.34.tar.gz
cd CPAN-2.34
perl Makefile.PL
make && make test
make install

```
## 配置cpan，设置PERL5LIB
然后最好设置一下环境变量，避免cpan安装模块时候无权限。  
通过设置环境变量，可在无root权限的情况下，用CPAN安装Perl模块到个人目录。  
编辑~/.bashrc文件，末尾添加如下设置。$LOCAL_APP其实是放下载的包的位置的自定义目录
```

LOCAL_APP=$HOME/perl5/

# local perl edition
LOCAL_PERL_EDITION=$LOCAL_APP/perl
export PERL5LIB=$LOCAL_PERL_EDITION/lib
export PATH=$LOCAL_PERL_EDITION/bin:$PATH

# local perl lib
LOCAL_PERL_LIB=$LOCAL_APP/perl5
export PERL_LOCAL_LIB_ROOT="$PERL_LOCAL_LIB_ROOT:$LOCAL_PERL_LIB"
export PERL_MB_OPT="--install_base $LOCAL_PERL_LIB"
export PERL_MM_OPT="INSTALL_BASE=$LOCAL_PERL_LIB"
export PERL5LIB=$LOCAL_PERL_LIB/lib/perl5:.:$PERL5LIB
# 注意上面里的.是代表当前路径，很重要，这样就能导入当前目录的自定义pm文件了，不会再找不到了，也不需要再在添unshift (@INC, cwd());
export PATH=$LOCAL_PERL_LIB/bin:$PATH


save后再使配置文件生效，执行
source ~/.bashrc

以后就可以capn -i xxx进行安装模块了
```

# 升级perl与LanguageServer
```
1：查询perl的真实安装路径
whereis perl
真实安装路径为：/usr/bin/perl

2 wget https://www.cpan.org/src/5.0/perl-5.28.1.tar.gz
tar -xzf perl-5.28.1.tar.gz  会解压到perl528目录中
cd perl-5.28.1
./Configure -des -Dprefix=/usr/local/perl -Dusethreads -Uversiononly
make
make test
make install


去除老版本perl
cd /usr/bin
mv perl perl.old # 换个名字
建立软连接，指向usr/bin/中
[huawei@n148 perl-5.28.1]$ sudo ln -s /home/huawei/perl528/perl-5.28.1/perl  /usr/bin/perl

再perl -v即可


使用此方法是个简陋的多个perl版本并存的方式，需要哪个是当前就改哪个文件为perl即可。更高级的多版本有perlbrew，未研究。。。
```
接着安装语言服务，用于vscode的perl支持，如果此模块已有，可以先去除后再重装，因为此模块还有很多依赖会自动安装，否则可能需要每个都手动了
```
[huawei@n148 usr]$ cpanm Perl::LanguageServer
[huawei@n148 usr]$ cpanm Perl::LanguageServer::DebuggerInterface
```
完成上面两步骤后启动vsc会在输出中看到找到Perl::LanguageServer这类的信息。说明安装正确了。下一步更重要，在vsc中添加断点后debug运行，出现找不到podwalker与dump错误则可以手动添加库路径到DebuggerInterface.pm中，确保此2个pm文件在添加的路径中即可。
```
[huawei@n148 ~]$ head /usr/local/perl/lib/5.30.0/Perl/LanguageServer/DebuggerInterface.pm
#
# We include DB package from perl core here, to be able to modify it...
#
use lib '/usr/local/perl/lib/perl5/x86_64-linux-thread-multi/';
use lib '/home/huawei/perl5/lib/perl5';
```
# perlbrew
https://perlbrew.pl/Perlbrew-%E4%B8%AD%E6%96%87%E7%B0%A1%E4%BB%8B.html
```
[huawei@n148 bin]$ sudo cpan App::perlbrew
[huawei@n148 bin]$ perlbrew init
[huawei@n148 bin]$ source ~/perl5/perlbrew/etc/bashrc

让perlbrew使用本地镜像：
[huawei@n148 bin]$ export PERLBREW_CPAN_MIRROR="http://mirrors.163.com/cpan"

[huawei@n148 bin]$ perlbrew install 5.24.0
安装会有几分钟。。。
```

# 网络编程
## 基于berkeley的tcp简单echo实现
* 伯克利风格c接口，虽然精简了一点，但简直与c的代码一样。。。
* 繁琐的c库api映射方式
* 底层要转为c参数很不便

先启动服务器，然后再启动客户端，双方各自打印一句话后客户端断开连接  

![](https://www.runoob.com/wp-content/uploads/2016/06/1466064198-6811-perl-socket.jpg)

server
```
use Modern::Perl;
use Socket;

my $port = 7890;
my $server = "localhost";  # 设置本地地址

# 创建 socket, 端口可重复使用，创建多个连接
socket(SOCKET, PF_INET, SOCK_STREAM, getprotobyname('tcp')) or die "无法打开 socket $!\n";
setsockopt(SOCKET, SOL_SOCKET, SO_REUSEADDR, 1) or die "无法设置 SO_REUSEADDR $!\n";

# 绑定端口并监听
bind(SOCKET, pack_sockaddr_in($port, inet_aton($server)))  or die "无法绑定端口 $port! \n";
listen(SOCKET, 5) or die "listen: $!";
# 最大5个连接

# 接收请求
my $client_addr;
while ($client_addr = accept(NEW_SOCKET, SOCKET)) {
	print NEW_SOCKET "你已连接到服务器，但又关闭了。。。";
	my ($port,$hisaddr) = sockaddr_in($client_addr);
  	print "Connection from [",inet_ntoa($hisaddr),",$port]\n";
	close NEW_SOCKET;
}
```
client
```
use Modern::Perl;
use Socket;

# socket,connect,close都有返回值（t为成功），但通常不使用
socket(SOCK, AF_INET, SOCK_STREAM, scalar getprotobyname('tcp')) or die "无法创建socket $!\n";
my $port = 7890;
my $server_ip_address = 'localhost';
connect(SOCK, pack_sockaddr_in($port, inet_aton($server_ip_address))) or die "无法连接port $!\n";
# pack_sockaddr_in将地址转换为二进制格式
my $line= <SOCK>;
print "$line\n";
close SOCK or die "close: $!";
```
## 使用IO::Socket::INET重新实现echo
* IO::Socket::INET是面向对象的方式
* 简短了一些但也没太多。。。

server
```
use Modern::Perl;
use IO::Handle;
use IO::Socket::INET;

my $port = 7890;
my $sock = IO::Socket::INET->new( Listen    => 20, 
                                  LocalPort => $port,
                                  Timeout   => 60*60,
                                  Reuse     => 1) or die "无法打开 socket $!\n";
while (1) {
	my $session = $sock->accept;
	print $session "你已连接到服务器，但又关闭了。。。";
	my $peer = $session->peerhost;
	my $port = $session->peerport;
	print "Connection from [$peer,$port]\n";
	close $session;
}					  
close $sock;
```

client  
```
use Modern::Perl;
use IO::Handle;
use IO::Socket::INET;

my $port = 7890;
my $server_ip_address = 'localhost';
my $socket = IO::Socket::INET->new("$server_ip_address:$port") or die $@;
# IO::Socket::INET->new方法还有其它参数，这里仅使用了最简单的方式
my $msg_in = <$socket>;
print $msg_in, "\n";
$socket->close or warn $@;
```
## 简单ftp获取文件
使用Net::FTP（出自libnet包，本类是IO::Socket与Net::Cmd的后代）实现的获取远程文件，本例$ftp对象还有很多方法未被使用，使用了默认用户名
```
use Modern::Perl;
use Net::FTP;

use constant HOST => 'ftp.perl.org';
use constant DIR  => 'pub/mozilla/firefox/releases/latest';
use constant FILE => 'KEY';

my $ftp = Net::FTP->new(HOST) or die "Couldn't connect: $@\n";
$ftp->login('anonymous')      or die $ftp->message;
$ftp->cwd(DIR)                or die $ftp->message;
$ftp->get(FILE)               or die $ftp->message;
$ftp->quit;

warn "File retrieved successfully.\n";
```
## ftp获取指定路径
可以指定获取文件或是指定目录（会递归）
```
use Net::FTP;
use File::Path;
use Getopt::Long;

use constant USAGEMSG => <<USAGE;
Usage: ftp_mirror.pl [options] host:/path/to/directory
Options: 
        --user  <user>  Login name
        --pass  <pass>  Password
        --hash          Progress reports
        --verbose       Verbose messages
USAGE

my ($USERNAME,$PASS,$VERBOSE,$HASH);
die USAGEMSG unless GetOptions('user=s'  => \$USERNAME,
                               'pass=s'  => \$PASS,
                               'hash'    => \$HASH,	# 传输过程打印hash标记
                               'verbose' => \$VERBOSE);	# 获取详细状态报告
die USAGEMSG unless my ($HOST,$PATH) = $ARGV[0]=~/(.+):(.+)/; # 解析参数

my $ftp = Net::FTP->new($HOST) or die "Can't connect: $@\n";
$ftp->login($USERNAME,$PASS)   or die "Can't login: ",$ftp->message; # 使用指定的参数登录，否则使用anonymous
$ftp->binary;	# 设置二进制模式，适用于传输二进制文件，与其对应的还有ascii
$ftp->hash(1) if $HASH;	# 来自命令行参数，用于传输过程是否打印hash标记

do_mirror($PATH);

$ftp->quit;
exit 0;

# top-level entry point for mirroring.
sub do_mirror {
  my $path = shift;

  return unless my $type = find_type($path); # 判断参数是目录还是文件

  my ($prefix,$leaf) = $path =~ m!^(.*?)([^/]+)/?$!; # 路径拆分，分为最末层与其余
#   print "left: $leaf\n";
  $ftp->cwd($prefix) if $prefix; # 进入指定路径的上一层
  # 根据类型进行备份，下面备份时候直接使用left即可，当前路径已在最末层位置
  get_file($leaf)  if $type eq '-';  # ordinary file
  get_dir($leaf)   if $type eq 'd';  # directory

  warn "Don't know what to do with a file of type $type. Skipping.";
}

# subroutine to determine whether a path is a directory or a file
# 判断类型是文件还是目录，使用cwd方法检测是否可以进入来判断是否为目录
sub find_type {
  my $path = shift;
  my $pwd = $ftp->pwd; # 备份ftp当前所在路径
  my $type = '-';  # assume plain file 用-代表文件
  if ($ftp->cwd($path)) {	# 能进入此path则为目录
    $ftp->cwd($pwd);	# 这里又返回了上面的备份
    $type = 'd';	# 用d代表目录
  }
  return $type;
}


# mirror a file
sub get_file {
  my ($path,$mode) = @_;	# 如仅备份单文件则mode为空，path为文件名，当前已在最末层目录中
  my $rtime = $ftp->mdtm($path); 	# 获取文件修改时间
  my $rsize = $ftp->size($path);	# 获取文件size
#   my $msg=sprintf("path:%s, file:%s, time:%s, size:%s\n", $ftp->pwd, $path, $rtime, $rsize);
	# print $msg;
  # dir方法如参数是文件则返回此文件的信息，是路径则返回所有文件的各自信息，返回的是arr
  my @fileinfo = $ftp->dir($path);
  # print $fileinfo[0];
  $mode=(parse_listing($fileinfo[0]))[2] unless defined $mode; # 如有mode则赋值
	# print $mode,"\n";
#   $mode = (parse_listing($ftp->dir($path)))[2] unless defined $mode; # 未定义mode则使用dir获取目录列表

  my ($lsize,$ltime) = stat($path) ? (stat(_))[7,9] : (0,0); # 获取文件信息成功则去第七第九下标的值（代表size与修改时间），否则0，stat(_)代表？前面的计算结果（详见stat）
  if ( defined($rtime) and defined($rsize)  # 如果有值，且本地数值不比远程老，则不用重新备份并返回
       and ($ltime >= $rtime) 
       and ($lsize == $rsize) ) {
    warn "Getting file $path: not newer than local copy.\n" if $VERBOSE;
    return;
  }

  warn "Getting file $path\n" if $VERBOSE;
  $ftp->get($path) or (warn $ftp->message,"\n" and return); # 否则获取文件，chmod文件之
  chmod $mode,$path if $mode;
}

# mirror a directory, recursively 递归的过去文件夹内所有内容
sub get_dir {
  my ($path,$mode) = @_;
  my $localpath = $path; # path为当前的相对路径
  -d $localpath or mkpath $localpath or die "mkpath failed: $!"; # 如果本地没有文件夹则创建（此处支持多层目录），如果创建失败则die
  chdir $localpath                   or die "can't chdir to $localpath: $!"; # 进入本地文件夹
  chmod $mode,'.' if $mode; # 如果有mode这chmod之

  my $cwd = $ftp->pwd                or die "can't pwd: ",$ftp->message;	# 记录当前ftp目录
  $ftp->cwd($path)                   or die "can't cwd: ",$ftp->message;	# 进入要获取的ftp指定路径

  warn "Getting directory $path/\n" if $VERBOSE;

  foreach ($ftp->dir) {	# 获取当前fpt目录列表
    next unless my ($type,$name,$mode) = parse_listing($_);	# 获取每一行的3个属性
    next if $name =~ /^(\.|\.\.)$/;  # skip . and .. 跳过.和..
    get_dir ($name,$mode)    if $type eq 'd';	# 递归处理目录
    get_file($name,$mode)    if $type eq '-'; 	# 处理单文件
    make_link($name)         if $type eq 'l';	# 处理链接
  }

  $ftp->cwd($cwd)     or die "can't cwd: ",$ftp->message;	# 恢复ftp的当前路径
  chdir '..';	# 再向上一层
}


# Attempt to mirror a link.  Only works on relative targets. 仅对已经下载的目录内的文件进行链接
sub make_link {
  my $entry = shift;
  my ($link,$target) = split /\s+->\s+/,$entry;
  return if $target =~ m!^/!;
  warn "Symlinking $link -> $target\n" if $VERBOSE;
  return symlink $target,$link;
}

# parse directory listings 
# -rw-r--r--   1 root     root          312 Aug  1  1994 welcome.msg
# 从一行的文件信息里匹配出3个数据(使用小括号的具名捕获)
sub parse_listing {
  my $listing = shift;
  return unless my ($type,$mode,$name) =
    $listing =~ /^([a-z-])([a-z-]{9})  # -rw-r--r-- 判断第一个字符是-则是文件,d则是目录,同linux
                 \s+\d*                # 1
                 (?:\s+\w+){2}         # root root
                 \s+\d+                # 312
                 \s+\w+\s+\d+\s+[\d:]+ # Aug 1 1994
                 \s+(.+)               # welcome.msg
                 $/x;  
  
  return ($type,$name,filemode($mode));
}

# turn symbolic modes into octal 将rwxr之类的转为数字形式
sub filemode {
  my $symbolic = shift;
  my (@modes) = $symbolic =~ /(...)(...)(...)$/g;
  my $result;
  my $multiplier = 1;
  while (my $mode = pop @modes) {
    my $m = 0;
    $m += 1 if $mode =~ /[xsS]/;
    $m += 2 if $mode =~ /w/;
    $m += 4 if $mode =~ /r/;
    $result += $m * $multiplier if $m > 0;
    $multiplier *= 8;
  }
  $result;
}


[huawei@n148 perl]$ perl "/home/huawei/playground/perl/0.pl" --verbose ftp.cesca.es:/centos/build/
Getting directory build/
Symlinking HEADER.html -> ../HEADER.html
Symlinking HEADER.images -> ../HEADER.images
Getting file build.sh
.
.
.
```
## 安装telnet
```
centos安装服务器
https://blog.51cto.com/wangfeiyu/2173268
测试使用本地登录成功了，但使用win登录到linux就失败了，总是端口23连接失败，没有再研究。。。

windows安装客户端
https://jingyan.baidu.com/article/02027811592cbf1bcc9ce58a.html
```
## 简单telnet登录测试
运行后会列出ps结果
```
use strict;
use Net::Telnet;

warn "Change the constants to match a machine you have login access to.\n";

use constant HOST => '127.0.0.1';
use constant USER => 'huawei';
use constant PASS => '111111';

my $telnet = Net::Telnet->new(HOST);
$telnet->login(USER,PASS);
my @lines = $telnet->cmd('ps -ef');
print @lines;
```
## 更加复杂的telnet案例
perl网络编程第六章中的change_passwd.pl（明文密码）与change_passwd_ssh.pl（ssh方式）分别演示了使用telnet方式修改用户密码，未进行测试
## 发邮件
perl网络编程第七章第八章有介绍，未做测试。且貌似有更新的库可以替代书上的案例

## 解析web网页
perl网络编程第十章，未做测试。
## 简单聊天机器人
使用bye退出聊天
```
use Modern::Perl;
use Chatbot::Eliza;
$| = 1;
my $bot = Chatbot::Eliza->new;
$bot->command_interface();
```
## 多进程版聊天服务器
```
# file: remoteps1.pl

use strict;
use Chatbot::Eliza;
use IO::Socket;
use POSIX 'WNOHANG';

use constant PORT => 12000;
my $quit = 0; # 用于退出程序的循环变量

# signal handler for child die events 子进程中止的回调，配合下面INT的回调为fork处理的最佳组合
$SIG{CHLD} = sub { while ( waitpid(-1,WNOHANG)>0 ) { } };
# signal handler for interrupt key and TERM signal
$SIG{INT} = sub { $quit++ };

my $listen_socket = IO::Socket::INET->new(LocalPort => PORT,
                                          Listen    => 20,
                                          Proto     => 'tcp',
                                          Reuse     => 1,
                                          Timeout   => 60*60,
                                         );
die "Can't create a listening socket: $@" unless $listen_socket;
warn "Server ready.  Waiting for connections...\n";   

while (!$quit) {

  next unless my $connection = $listen_socket->accept;

  defined (my $child = fork()) or die "Can't fork: $!";
  if ($child == 0) {
    $listen_socket->close;	# 子进程则无需再监听，交互结束可以直接结束进程
    interact($connection);
    exit 0;
  }

  $connection->close; # 父进程则关闭此连接的拷贝（可以节省资源）
}

sub interact {
  my $sock = shift;
  STDIN->fdopen($sock,"<")  or die "Can't reopen STDIN: $!"; # 使用STD开启socket来进行交互
  STDOUT->fdopen($sock,">") or die "Can't reopen STDOUT: $!";
  STDERR->fdopen($sock,">") or die "Can't reopen STDERR: $!";
  $|=1;
  my $bot = Chatbot::Eliza->new;
  $bot->command_interface();
}
```
启动server后，使用[huawei@n148 ~]$ telnet localhost 12000 开启连接，bye退出交互
## 守护进程版聊天服务器
deamon版本仅可以用于linux进程模型，windows参考srvany.exe将perl脚本设为后台服务进行实现。  
本案例的代码逻辑是deamon固有套路，如使用则仿制即可。  
运行本服务器代码后，依然使用[huawei@n148 ~]$ telnet localhost 12000 开启连接，bye退出交互
```
use strict;
use Chatbot::Eliza;
use IO::Socket;
use IO::File;
use POSIX qw(WNOHANG setsid);

use constant PORT      => 12000;
use constant PID_FILE  => '/var/tmp/eliza.pid'; # pid文件存放路径
my $quit = 0;

# signal handler for child die events 注册这2个回调是为了END能删除pid文件
$SIG{CHLD} = sub { while ( waitpid(-1,WNOHANG)>0 ) { } };
$SIG{TERM} = $SIG{INT} = sub { $quit++ };

my $fh = open_pid_file(PID_FILE); # 创建pid文件
my $listen_socket = IO::Socket::INET->new(LocalPort => shift || PORT,
                                          Listen    => 20,
                                          Proto     => 'tcp',
                                          Reuse     => 1,
                                          Timeout   => 60*60,
                                         );
die "Can't create a listening socket: $@" unless $listen_socket;

warn "$0 starting...\n";
my $pid = become_daemon();	# 得到deamon的pid
print $fh $pid;	# 写入pid，关闭句柄
close $fh;

while (!$quit) {

  next unless my $connection = $listen_socket->accept;

  die "Can't fork: $!" unless defined (my $child = fork());
  if ($child == 0) {
    $listen_socket->close;
    interact($connection);
    exit 0;
  }

  $connection->close;
}

sub interact {
  my $sock = shift;
  STDIN->fdopen($sock,"<")  or die "Can't reopen STDIN: $!";
  STDOUT->fdopen($sock,">") or die "Can't reopen STDOUT: $!";
  STDERR->fdopen($sock,">") or die "Can't reopen STDERR: $!";
  $| = 1;
  my $bot = Chatbot::Eliza->new;
  $bot->command_interface;
}

sub become_daemon {
  die "Can't fork" unless defined (my $child = fork);
  exit 0 if $child;    # 父进程结束仅保留子进程
  setsid();     # 该子进程成为当前会话组的leader
  open(STDIN, "</dev/null");	# 关闭输入
  open(STDOUT,">/dev/null");	# 关闭输出
  open(STDERR,">&STDOUT");	# 将err输出到stdout
  chdir '/';           # change working directory到根目录
  umask(0);            # forget file mode creation mask，消除继承的文件mask
  $ENV{PATH} = '/bin:/sbin:/usr/bin:/usr/sbin'; # 设置PATH
  return $$;	# 返回子进程的pid
}

sub open_pid_file { # 作用是写入pid到磁盘便于别人给其发送信号
  my $file = shift;
  if (-e $file) {  # oops.  pid file already exists
    my $fh = IO::File->new($file) || return;	# 打开，获取pid（此文件内容仅一个数字）
    my $pid = <$fh>;
    die "Server already running with PID $pid" if kill(0, $pid); # kill 0返回1则deamon仍在运行然后die，否则返回0
    warn "Removing PID file for defunct server process $pid.\n";	# 上面返回0则说明未清理pid文件进程就挂了
    die "Can't unlink PID file $file" unless -w $file && unlink $file;	# 检测pid文件可写并清除，失败则die
  }
  return IO::File->new($file,O_WRONLY|O_CREAT|O_EXCL,0644)	# 创建pid文件并返回句柄
    or die "Can't create $file: $!\n";
}

sub Chatbot::Eliza::_testquit { 
  my ($self,$string) = @_; 
  return 1 unless defined $string;  # test for EOF 
  foreach (@{$self->{quit}}) { return 1 if $string =~ /\b$_\b/i };
} 

END { unlink PID_FILE if $$ == $pid; }	# 如当前是deamon（非其它子进程）则退出前删除pid文件
```
## 多线程版本聊天服务器
本代码基于perl网络编程，书内容较老，也许现今已不适用，仅参考。  
对聊天机器人类进行了继承并重写了入口函数
```
package Chatbot::Eliza::Server;
use Chatbot::Eliza;
# file: Chatbot/Eliza/Server.pm
# Figure 11.2: The Chatbot::Eliza::Server class

@ISA = 'Chatbot::Eliza';	# 基础子此类

sub command_interface {	# 重新定义此方法
  my $self = shift;	# 类似this，非外界调用时候的参数
  my $in   = shift || \*STDIN;	# in与out是外面传进来的参数，此处因为是同一进程所以不能使用相同的STDOUT和STDIN，需要分开
  my $out  = shift || \*STDOUT;	# 即每个线程都有各自的，如没有设置则还使用STD的
  my ($user_input, $previous_user_input, $reply);

  $self->botprompt($self->name . ":\t");  # Set Eliza's prompt 这里以下的逻辑代码是基类的原始实现
  $self->userprompt("you:\t");           # Set user's prompt	原作者照搬过来的

  # Print an initial greeting
  print $out $self->botprompt,
             $self->{initial}->[ int rand scalar @{ $self->{initial} } ],
             "\n";

  while (1) {
    print $out $self->userprompt;
    $previous_user_input = $user_input;
    chomp( $user_input = <$in> ); 
    last unless $user_input;

    # User wants to quit
    if ($self->_testquit($user_input) ) {
      $reply = $self->{final}->[ int rand scalar @{ $self->{final} } ];
      print $out $self->botprompt,$reply,"\n";
      last;
    } 

    # Invoke the transform method to generate a reply.
    $reply = $self->transform( $user_input );

    # Print the actual reply
    print $out $self->botprompt,$reply,"\n";
  }
}

1;
```
下面是server逻辑
```
use strict;
use IO::Socket;
use Thread;
use Chatbot::Eliza::Server;

use constant PORT => 12000;
my $listen_socket = IO::Socket::INET->new(LocalPort => PORT,
                                          Listen    => 20,
                                          Proto     => 'tcp',
                                          Reuse     => 1);
die $@ unless $listen_socket;

warn "Listening for connections...\n";

while (my $connection = $listen_socket->accept) {
  Thread->new(\&interact,$connection);	# 有连接进来就创建线程
}

sub interact {
  my $handle = shift;
  Thread->self->detach;	# 脱离主线程，将sock传入主循环
  Chatbot::Eliza::Server->new->command_interface($handle,$handle);
  $handle->close();	# 线程结束关闭sock
}


这2个文件的目录关系
├── server.pl
├── Chatbot
    └── Eliza
        └── Server.pm
```

