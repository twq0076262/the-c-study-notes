# 第一部分: 语言  

示例基于 GCC 32bit...

# 1. 数据类型

## 1.1 整数

以下是基本整数关键词：  
- char: 有符号8位整数。  
- short: 有符号16位整数。  
- int: 有符号32位整数。  
- long: 在32位系统是32整数 (long int)，在64位系统则是64位整数。  
- long long: 有符号64位整数 (long long int)。  
- bool: _Bool 类型，8位整数，在 stdbool.h 中定义了 bool / true / false 宏便于使用。 

由于在不同系统上 char 可能代表有符号或无符号8位整数，因此建议使用 unsigned char /signed char 来表示具体的类型。

在 stdint.h 中定义了一些看上去更明确的整数类型。

```
typedef signed char int8_t;
typedef short int int16_t;
typedef int int32_t;

typedef unsigned char uint8_t;
typedef unsigned short int uint16_t;
typedef unsigned int uint32_t;

#if __WORDSIZE == 64
	typedef long int int64_t;
	typedef unsigned long int uint64_t;
#else
	__extension__
	typedef long long int int64_t;
	typedef unsigned long long int uint64_t;
#endif
```

还有各种整数类型的大小限制。

```
# define INT8_MIN (-128)
# define INT16_MIN (-32767-1)
# define INT32_MIN (-2147483647-1)
# define INT64_MIN (-__INT64_C(9223372036854775807)-1)

# define INT8_MAX (127)
# define INT16_MAX (32767)
# define INT32_MAX (2147483647)
# define INT64_MAX (__INT64_C(9223372036854775807))

# define UINT8_MAX (255)
# define UINT16_MAX (65535)
# define UINT32_MAX (4294967295U)
# define UINT64_MAX (__UINT64_C(18446744073709551615))
```

字符常量默认是一个 int 整数，但编译器可以自行决定将其解释为 char 或 int。

```
char c = 'a';
printf("%c, size(char)=%d, size('a')=%d;\n", c, sizeof(c), sizeof('a'));
```

输出:

```
a, size(char)=1, size('a')=4;
```

指针是个有特殊用途的整数，在 stdint.h 中同样给出了其类型定义。

```
/* Types for `void *' pointers. */
#if __WORDSIZE == 64
	typedef unsigned long int uintptr_t;
#else
	typedef unsigned int uintptr_t;
#endif
```

不过在代码中我们通常用 sizeof(char*) 这样的用法，省得去处理32位和64位的区别。

我们可以用不同的后缀来表示整数常量类型。

```
printf("int size=%d;\n", sizeof(1));
printf("unsigned int size=%d;\n", sizeof(1U));
printf("long size=%d;\n", sizeof(1L));
printf("unsigned long size=%d;\n", sizeof(1UL));
printf("long long size=%d;\n", sizeof(1LL));
printf("unsigned long long size=%d;\n", sizeof(1ULL));
```

输出:

```
int size=4;
unsigned int size=4;
long size=4;
unsigned long size=4;
long long size=8;
unsigned long long size=8;
```

stdint.h 中定义了一些辅助宏。

```
# if __WORDSIZE == 64
# 	define __INT64_C(c) c ## L
# 	define __UINT64_C(c) c ## UL
# else
# 	define __INT64_C(c) c ## LL
# 	define __UINT64_C(c) c ## ULL
# endif
```

注: 宏定义中的 "##" 运算符表示把左和右结合在一起，作为一个符号。

## 1.2 浮点数

C 提供了不同精度的浮点。
- float: 32位4字节浮点数，精确度6。
- double: 64位8字节浮点数，精确度15。
- long double: 80位10字节浮点数，精确度19位。


浮点数默认类型是 double，可以添加后缀 F 来表示 float，L 表示 long double，可以局部省略。

```
printf("float %f size=%d\n", 1.F, sizeof(1.F));
printf("double %f size=%d\n", .123, sizeof(.123));
printf("long double %Lf size=%d\n", 1.234L, sizeof(1.234L));
```

输出:

```
float 1.000000 size=4
double 0.123000 size=8
long double 1.234000 size=12 # 对齐
```

C99 提供了复数支持，用两个相同类型的浮点数分别表示复数的实部和虚部。直接在 float、double、long double 后添加 _Complex 即可表示复数，在 complex.h 中定义了complex 宏使得显示更统一美观。

```
#include <complex.h>

printf("float complex size=%d\n", sizeof((float complex)1.0));
printf("double complex size=%d\n", sizeof((double complex)1.0));
printf("long double complex size=%d\n", sizeof((long double complex)1.0));
```

输出:

```
float complex size=8
double complex size=16
long double complex size=24
```

## 1.3 枚举

和 C# 中我们熟悉的规则类似。

```
enum color { black, red = 5, green, yellow };

enum color b = black;
printf("black = %d\n", b);

enum color r = red;
printf("red = %d\n", r);

enum color g = green;
printf("green = %d\n", g);

enum color y = yellow;
printf("yellow = %d\n", y);
```

输出:

```
black = 0
red = 5
green = 6
yellow = 7
```

枚举成员的值可以相同。

```
enum color { black = 1, red, green = 1, yellow };
```

输出:

```
black = 1
red = 2
green = 1
yellow = 2
```

通常省略枚举小标签用来代替宏定义常量。

```
enum { BLACK = 1, RED, GREEN = 1, YELLOW };

printf("black = %d\n", BLACK);
printf("red = %d\n", RED);
printf("green = %d\n", GREEN);
printf("yellow = %d\n", YELLOW);
```

# 2. 字面值

字面值 (literal) 是源代码中用来描述固定值的记号 (token)，可能是整数、浮点数、字符、字符串。

## 2.1 整数常量

除了常见的十进制整数外，还可以⽤用八进制 (0开头) 或十六进制 (0x/0X)表示法。

```
int x = 010;
int y = 0x0A;
printf("x = %d, y = %d\n", x, y);
```

输出:

```
x = 8, y = 10
```

常量类型很重要，可以通过后缀来区分类型。

```
0x200 -> int
200U -> unsigned int

0L -> long
0xf0f0UL -> unsigned long

0777LL -> long long
0xFFULL -> unsigned long long
```

## 2.2 浮点常量

可以用十进制或十六进制表示浮点数常量。

```
10.0 -> 10
10. -> 10
.123 -> 0.123

2.34E5 -> 2.34 * (10 ** 5)
67e-12 -> 67.0 * (10 ** -12)
```

默认浮点常量是 double，可以用 F 后缀表示 float，用 L 后缀表示 long double 类型。

## 2.3 字符常量

字符常量默认是 int 类型，除非用前置 L 表示 wchar_t 宽字符类型。

```
char c = 0x61;
char c2 = 'a';
char c3 = '\x61';
printf("%c, %c, %c\n", c, c2, c3);
```

输出:

```
a, a, a
```

在 Linux 系统中，默认字符集是 UTF-8，可以用 wctomb 等函数进行转换。wchar_t 默认是4字节长度，足以容纳所有 UCS-4 Unicode 字符。

```
setlocale(LC_CTYPE, "en_US.UTF-8");

wchar_t wc = L'中';
char buf[100] = {};

int len = wctomb(buf, wc);
printf("%d\n", len);

for (int i = 0; i < len; i++)
{
	printf("0x%02X ", (unsigned char)buf[i]);
}
```

输出:

```
3
0xE4 0xB8 0xAD
```

## 2.4 字符串常量

C 语言中的字符串是一个以 NULL (也就是 \0) 结尾的 char 数组。空字符串在内存中占用一个字节，包含一个 NULL 字符，也就是说要表示一个长度为1的字符串最少需要2个字节 (strlen 和 sizeof 表示的含义不同)。

```
char s[] = "Hello, World!";
char* s2 = "Hello, C!";
```

同样可以使用 L 前缀声明一个宽字符串。

```
setlocale(LC_CTYPE, "en_US.UTF-8");

wchar_t* ws = L"中国人";
printf("%ls\n", ws);

char buf[255] = {};
size_t len = wcstombs(buf, ws, 255);

for (int i = 0; i < len; i++)
{
	printf("0x%02X ", (unsigned char)buf[i]);
}
```

输出:

```
中国人
0xE4 0xB8 0xAD 0xE5 0x9B 0xBD 0xE4 0xBA";
```

和 char 字符串类型类似，wchar_t 字符串以一个4字节的 NULL 结束。

```
wchar_t ws[] = L"中国人";
printf("len %d, size %d\n", wcslen(ws), sizeof(ws));

unsigned char* b = (unsigned char*)ws;
int len = sizeof(ws);

for (int i = 0; i < len; i++)
{
	printf("%02X ", b[i]);
}
```

输出:

```
len 3, size 16
2D 4E 00 00 FD 56 00 00 BA 4E 00 00 00 00 00 00
```

编译器会自动连接相邻的字符串，这也便于我们在宏或者代码中更好地处理字符串。

```
#define WORLD "world!"
char* s = "Hello" " " WORLD "\n";
```

对于源代码中超长的字符串，除了使用相邻字符串外，还可以用 "\" 在行尾换行。

```
char* s1 = "Hello"
	" World!";
char* s2 = "Hello \
World!";
```

注意："\" 换行后左侧的空格会被当做字符串的一部分。

# 3. 类型转换

当运算符的几个操作数类型不同时，就需要进行类型转换。通常编译器会做某些自动的隐式转换操作，在不丢失信息的前提下，将位宽 "窄" 的操作数转换为 "宽" 类型。

## 3.1 算术类型转换

编译器默认的隐式转换等级:

```
long double > doulbe > float > long long > long > int > char > _Bool
```

浮点数的等级比任何类型的整数等级都高；有符号整数和其等价的无符号类型等级相同。在表达式中，可能会将 char、short 当做默认 int (unsigned int) 类型操作数，但 float 并不会自动转换为默认的 double 类型。

```
char a = 'a';
char c = 'c';

printf("%d\n", sizeof(c - a));
printf("%d\n", sizeof(1.5F - 1));
```

输出:

```
4
4
```

当包含无符号操作数时，需要注意提升后类型是否能容纳无符号类型的所有值。

```
long a = -1L;
unsigned int b = 100;
printf("%ld\n", a > b ? a : b);
```

输出:

```
-1
```

输出结果让人费解。尽管 long 等级比 unsigned int 高，但在32位系统中，它们都是32位整数，且 long 并不足以容纳 unsigned int 的所有值，因此编译器会将这两个操作数都转换为 unsigned long，也就是高等级的无符号版本，如此 (unsigned long)a 的结果就变成了一个很大的整数。

```
long a = -1L;
unsigned int b = 100;

printf("%lu\n", (unsigned long)a);
printf("%ld\n", a > b ? a : b);
```

输出:

```
4294967295
-1
```

其他隐式转换还包括：
- 赋值和初始化时，右操作数总是被转换成左操作数类型。
- 函数调用时，总是将实参转换为形参类型。
- 将 return 表达式结果转换为函数返回类型。
- 任何类型0值和 NULL 指针都视为 _Bool false，反之为 true。

将宽类型转换为窄类型时，编译器会尝试丢弃高位或者四舍五入等手段返回一个 "近似值"。

## 3.2 非算术类型转换

(1) 数组名或表达式通常被当做指向第一个元素的指针，除非是以下情况：
- 被当做 sizeof 操作数。
- 使用 & 运算符返回 "数组指针"。
- 字符串常量用于初始化 char/wchar_t 数组。

(2) 可以显式将指针转换成任何其他类型指针。

```
int x = 123, *p = &x;
char* c = (char*)x;
```

(3) 任何指针都可以隐式转换为 void 指针，反之亦然。

(4) 任何指针都可以隐式转换为类型更明确的指针 (包含 const、volatile、restrict 等限定符)。

```
int x = 123, *p = &x;
const int* p2 = p;
```

(5) NULL 可以被隐式转换为任何类型指针。

(6) 可以显式将指针转换为整数，反向转换亦可。

```
int x = 123, *p = &x;
int px = (int)p;

printf("%p, %x, %d\n", p, px, *(int*)px);
```

输出:

```
0xbfc1389c, bfc1389c, 123
```

# 4. 运算符

基本的表达式和运算符用法无需多言，仅记录一些特殊的地方。

## 4.1 复合字面值

C99 新增的内容，我们可以直接用该语法声明一个结构或数组指针。

```
(类型名称){ 初始化列表 }
```

演示:

```
int* i = &(int){ 123 }; ! // 整型变量, 指针
int* x = (int[]){ 1, 2, 3, 4 }; ! // 数组, 指针
struct data_t* data = &(struct data_t){ .x = 123 }; ! // 结构, 指针
func(123, &(struct data_t){ .x = 123 }); ! // 函数参数, 结构指针参数
```

如果是静态或全局变量，那么初始化列表必须是编译期常量。

## 4.2 sizeof

返回操作数占用内存空间大小，单位字节 (byte)。sizeof 返回值是 size_t 类型，操作数可以是类型和变量。

```
size_t size;
int x = 1;

size = sizeof(int);

size = sizeof(x);
size = sizeof x;

size = sizeof(&x);
size = sizeof &x;
```

附: 不要用 int 代替 size_t，因为在32位和64位平台 size_t 长度不同。

## 4.3 逗号运算符

逗号是一个二元运算符，确保操作数从左到右被顺序处理，并返回右操作数的值和类型。

```
int i = 1;
long long x = (i++, (long long)i);
printf("%lld\n", x);
```

## 4.4 优先级

C 语言的优先级是个⼤大⿇麻烦，不要吝啬使用 "()" 。

优先级列表 (从高到低):

|类型|符号|结合律|
|:---|:---|:------|
|后置运算符|[]、func()、.、->、(type){init} |从左到右|
|一元运算符|++、--、!、~、+、-、*、&、sizeof |从右到左|
|v转换运算符|(type name) |从右到左|
|乘除运算符|*、/、% |从左到右|
|加减运算符|+、- |从左到右|
|位移运算符|<<、>> |从左到右|
|关系运算符| <、<=、>、>= |从左到右|
|相等运算符| ==、!= |从左到右|
|位运算符|& |从左到右|
|位运算符|^ |从左到右|
|位运算符|/|从左到右|
|逻辑运算符| && |从左到右|
|逻辑运算符|//|从左到右|
|条件运算符| ?: |从右到左|
|赋值运算符| =、+=、-=、*=、/=、%=、&=、^=、/=、<<=、>>= |从右到左|
|逗号运算符 |, |从左到右|

如果表达式中多个操作符具有相同优先级，那么结合律决定了组合方式是从左还是从右开始。如 "a = b = c"，两个 "=" 优先级相同，依结合律顺序 "从右到左"，分解成 "a = (b = c)"。

下面是一些容易引起误解的运算符优先级：

(1) "." 优先级高于 "*"。

```
原型: *p.f
误判: (*p).f
实际: *(p.f)。
```

(2) "[]" 高于 "*"。

```
原型: int *ap[]
误判: int (*ap)[]
实际: int *(ap[])
```

(3) "==" 和 "!=" 高于位操作符。

```
原型: val & mask != 0
误判: (val & mask) != 0
实际: val & (mask != 0)
```

(4) "==" 和 "!=" 高于赋值符。

```
原型: c = getchar() != EOF
误判: (c = getchar()) != EOF
实际: c = (getchar() != EOF)
```

(5) 算术运算符高于位移运算符。

```
原型: msb << 4 + lsb
误判: (msb << 4) + lsb
实际: msb << (4 + lsb)
```

(6) 逗号运算符在所有运算符中优先级最低。

```
原型: i = 1, 2
误判: i = (1, 2)
实际: (i = 1), 2
```

# 5. 语句

## 5.1 语句块

语句块代表了一个作用域，在语句块内声明的自动变量超出范围后立即被释放。除了用 "{...}" 表示一个常规语句块外，还可以直接用于复杂的赋值操作，这在宏中经常使用。

```
int i = ({ char a = 'a'; a++; a; });
printf("%d\n", i);
```

最后一个表达式被当做语句块的返回值。相对应的宏版本如下。

```
#define test() ({ \
	char _a = 'a'; \
	_a++; \
	_a; })

int i = test();
printf("%d\n", i);
```

在宏里使用变量通常会添加下划线前缀，以避免展开后跟上层语句块的同名变量冲突。

## 5.2 循环语句

C 支持 while、for、do...while 几种循环语句。注意下面的例子，循环会导致 get_len 函数被多次调用。

```
size_t get_len(const char* s)
{
	printf("%s\n", __func__);
	return strlen(s);
}

int main(int argc, char* argv[])
{
	char *s = "abcde";
	for (int i = 0; i < get_len(s); i++)
	{
		printf("%c\n", s[i]);
	}

	printf("\n");

	return EXIT_SUCCESS;
}
```

## 5.3 选择语句

除了 if...else if...else... 和 switch { case ... } 还有谁呢。GCC 支持 switch 范围扩展。

```
int x = 1;
switch (x)
{
	case 0 ... 9: printf("0..9\n"); break;
	case 10 ... 99: printf("10..99\n"); break;
	default: printf("default\n"); break;
}

char c = 'C';
switch (c)
{
	case 'a' ... 'z': printf("a..z\n"); break;
	case 'A' ... 'Z': printf("A..Z\n"); break;
	case '0' ... '9': printf("0..9\n"); break;
	default: printf("default\n"); break;
}
```

## 5.4 无条件跳转

无条件跳转: break, continue, goto, return。goto 仅在函数内跳转，常用于跳出嵌套循环。如果在函数外跳转，可使用 longjmp。

### 5.4.1 longjmp

setjmp 将当前位置的相关信息 (堆栈帧、寄存器等) 保存到 jmp_buf 结构中，并返回0。当后续代码执行 longjmp 跳转时，需要提供一个状态码。代码执行绪将返回 setjmp 处，并返回 longjmp 所提供的状态码。

```
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <setjmp.h>

void test(jmp_buf *env)
{
	printf("1....\n");
	longjmp(*env, 10);
}

int main(int argc, char* argv[])
{
	jmp_buf env;
	int ret = setjmp(env); ! // 执⾏行 longjmp 将返回该位置，ret 等于 longjmp 所提供的状态码。

	if (ret == 0)
	{
		test(&env);
	}
	else
	{
		printf("2....(%d)\n", ret);
	}
	
	return EXIT_SUCCESS;
}
```

输出:

```
1....
2....(10)
```

# 6. 函数

函数只能被定义一次，但可以被多次 "声明" 和 "调用"。

## 6.1 嵌套

gcc 支持嵌套函数扩展。

```
typedef void(*func_t)();

func_t test()
{
	void func1()
	{
		printf("%s\n", __func__);
	};

	return func1;
}

int main(int argc, char* argv[])
{
	test()();
	return EXIT_SUCCESS;
}
```

内层函数可以 "读写" 外层函数的参数和变量，外层变量必须在内嵌函数之前定义。

```
#define pp() ({ \
	printf("%s: x = %d(%p), y = %d(%p), s = %s(%p);\n", __func__, x, &x, y, &y, s, s); \
})

void test2(int x, char *s)
{
	int y = 88;
	pp();

	void func1()
	{
		y++;
		x++;
		pp();
	}

	func1();

	x++;
	func1();
	pp();
}

int main (int argc, char * argv[])
{
	test2(1234, "abc");
	return EXIT_SUCCESS;
}
```

输出:

```
test2: x = 1234(0xbffff7d4), y = 88(0xbffff7d8), s = abc(0x4ad3);
func1: x = 1235(0xbffff7d4), y = 89(0xbffff7d8), s = abc(0x4ad3);
func1: x = 1237(0xbffff7d4), y = 90(0xbffff7d8), s = abc(0x4ad3);
test2: x = 1237(0xbffff7d4), y = 90(0xbffff7d8), s = abc(0x4ad3);
```

## 6.2 类型

注意区分定义 "函数类型" 和 "函数指针 类型"的区别。函数名是一个指向当前函数的指针。

```
typedef void(func_t)(); // 函数类型
typedef void(*func_ptr_t)(); // 函数指针类型

void test()
{
	printf("%s\n", __func__);
}

int main(int argc, char* argv[])
{
	func_t* func = test; // 声明一个指针
	func_ptr_t func2 = test; // 已经是指针类型

	void (*func3)(); // 声明一个包含函数原型的函数指针变量
	func3 = test;

	func();
	func2();
	func3();

	return EXIT_SUCCESS;
}
```

## 6.3 调用

C 函数默认采用 cdecl 调用约定，参数从右往左入栈，且由调用者负责参数入栈和清理。

```
int main(int argc, char* argv[])
{
	int a()
	{
		printf("a\n");
		return 1;
	}

	char* s()
	{
		printf("s\n");
		return "abc";
	}

	printf("call: %d, %s\n", a(), s());
	return EXIT_SUCCESS;
}
```

输出:

```
s
a
call: 1, abc
```

C 语言中所有对象，包括指针本身都是 "复制传值" 传递，我们可以通过传递 "指针的指针" 来实现传出参数。

```
void test(int** x)
{
	int* p = malloc(sizeof(int));
	*p = 123;
	*x = p;
}

int main(int argc, char* argv[])
{
	int* p;
	test(&p);
	printf("%d\n", *p);
	free(p);

	return EXIT_SUCCESS;
}
```

注意: 别返回 test 中的栈变量。

## 6.4 修饰符

C99 修饰符:
- extern: 默认修饰符，用于函数表示 "具有外部链接的标识符"，这类函数可用于任何程序文件。用于变量声明表示该变量在其他单元中定义。
- static: 使用该修饰符的函数仅在其所在编译单元 (源码文件) 中可用。还可以表示函数类的静态变量。
- inline: 修饰符 inline 建议编译器将函数代码内联到调用处，但编译器可自主决定是否完成。通常包含循环或递归函数不能被定义为 inline 函数。

GNU inline 相关说明:
- static inline: 内链接函数，在当前编译单元内内联。不过 -O0 时依然是 call。
- inline: 外连接函数，当前单元内联，外部单元为普通外连接函数 (头文件中不能添加 inline 关键字)。

附：inline 关键字只能用在函数定义处。

## 6.5 可选性自变量

使用可选性自变量实现变参。
- va_start: 通过可选自变量前一个参数位置来初始化 va_list 自变量类型指针。
- va_arg: 获取当前可选自变量值，并将指针移到下一个可选自变量。
- va_end: 当不再需要自变量指针时调用。
- va_copy: 用现存的自变量指针 (va_list) 来初始化另一指针。

```
#include <stdarg.h>

/* 指定自变量数量 */
void test(int count, ...)
{
	va_list args;
	va_start(args, count);

	for (int i = 0; i < count; i++)
	{
		int value = va_arg(args, int);
		printf("%d\n", value);
	}

	va_end(args);
}

/* 以 NULL 为结束标记 */
void test2(const char* s, ...)
{
	printf("%s\n", s);

	va_list args;
	va_start(args, s);

	char* value;
	do
	{
		value = va_arg(args, char*);
		if (value) printf("%s\n", value);
	}
	while (value != NULL);
	va_end(args);
}

/* 直接将 va_list 传递个其他可选自变量函数 */
void test3(const char* format, ...)
{
	va_list args;
	va_start(args, format);

	vprintf(format, args);
	
	va_end(args);
}

int main(int argc, char* argv[])
{
	test(3, 11, 22, 33);
	test2("hello", "aa", "bb", "cc", "dd", NULL);
	test3("%s, %d\n", "hello, world!", 1234);

	return EXIT_SUCCESS;
}
```

# 7. 数组

## 7.1 可变长度数组

如果数组具有自动生存周期，且没有 static 修饰符，那么可以用非常量表达式来定义数组。

```
void test(int n)
{
	int x[n];
	for (int i = 0; i < n; i++)
	{
		x[i] = i;
	}

	struct data { int x[n]; } d;
	printf("%d\n", sizeof(d));
}

int main(int argc, char* argv[])
{
	int x[] = { 1, 2, 3, 4 };
	printf("%d\n", sizeof(x));
	
	test(2);
	return EXIT_SUCCESS;
}
```

## 7.2 下标存储

x[i] 相当于 *(x + i)，数组名默认为指向第一元素的指针。

```
int x[] = { 1, 2, 3, 4 };

x[1] = 10;
printf("%d\n", *(x + 1));

*(x + 2) = 20;
printf("%d\n", x[2]);
```

C 不会对数组下标索引进行范围检查，编码时需要注意过界检查。数组名默认是指向第一元素指针的常量，而 &x[i] 则返回 int* 类型指针，指向目标序号元素。

## 7.3 初始化

除了使用下标初始化外，还可以直接用初始化器。

```
int x[] = { 1, 2, 3 };
int y[5] = { 1, 2 };
int a[3] = {};

int z[][2] =
{
	{ 1, 1 },
	{ 2, 1 },
	{ 3, 1 },
};
```

初始化规则:
- 如果数组为静态生存周期，那么初始化器必须是常量表达式。
- 如果提供初始化器，那么可以不提供数组长度，由初始化器的最后一个元素决定。
- 如果同时提供长度和初始化器，那么没有提供初始值的元素都被初始化为0或 NULL。

我们还可以在初始化器中初始化特定的元素。

```
int x[] = { 1, 2, [6] = 10, 11 };
int len = sizeof(x) / sizeof(int);

for (int i = 0; i < len; i++)
{
	printf("x[%d] = %d\n", i, x[i]);
}
```

输出:

```
x[0] = 1
x[1] = 2
x[2] = 0
x[3] = 0
x[4] = 0
x[5] = 0
x[6] = 10
x[7] = 11
```

## 7.4 字符串

字符串是以 '\0' 结尾的 char 数组。

```
char s[10] = "abc";
char x[] = "abc";

printf("s, size=%d, len=%d\n", sizeof(s), strlen(s));
printf("x, size=%d, len=%d\n", sizeof(x), strlen(x));
```

输出:

```
s, size=10, len=3
x, size=4, len=3
```

## 7.5 多维数组

实际上就是 "元素为数组" 的数组，注意元素是数组，并不是数组指针。多维数组的第一个维度下标可以不指定。

```
int x[][2] =
{
	{ 1, 11 },
	{ 2, 22 },
	{ 3, 33 }
};

int col = 2, row = sizeof(x) / sizeof(int) / col;

for (int r = 0; r < row; r++)
{
	for (int c = 0; c < col; c++)
	{
		printf("x[%d][%d] = %d\n", r, c, x[r][c]);
	}
}
```

输出:

```
x[0][0] = 1
x[0][1] = 11
x[1][0] = 2
x[1][1] = 22
x[2][0] = 3
x[2][1] = 33
```

二维数组通常也被称为 "矩阵 (matrix)"，相当于一个 row * column 的表格。比如 x[3][2] 相当于三行二列表格。多维数组的元素是连续排列的，这也是区别指针数组的一个重要特征。

```
int x[][2] =
{
	{ 1, 11 },
	{ 2, 22 },
	{ 3, 33 }
};

int len = sizeof(x) / sizeof(int);
int* p = (int*)x;

for (int i = 0; i < len; i++)
{
	printf("x[%d] = %d\n", i, p[i]);
}
```

输出:

```
x[0] = 1
x[1] = 11
x[2] = 2
x[3] = 22
x[4] = 3
x[5] = 33
```

同样，我们可以初始化特定的元素。

```
int x[][2] =
{
	{ 1, 11 },
	{ 2, 22 },
	{ 3, 33 },
	[4][1] = 100,
	{ 6, 66 },
	[7] = { 9, 99 }
};

int col = 2, row = sizeof(x) / sizeof(int) / col;

for (int r = 0; r < row; r++)
{
	for (int c = 0; c < col; c++)
	{
		printf("x[%d][%d] = %d\n", r, c, x[r][c]);
	}
}
```

输出:

```
x[0][0] = 1
x[0][1] = 11
x[1][0] = 2
x[1][1] = 22
x[2][0] = 0
x[2][1] = 0
x[3][0] = 0
x[3][1] = 0
x[4][0] = 0
x[4][1] = 100
x[5][0] = 6
x[5][1] = 66
x[6][0] = 0
x[6][1] = 0
x[7][0] = 9
x[7][1] = 99
```

## 7.6 数组参数

当数组作为函数参数时，总是被隐式转换为指向数组第一元素的指针，也就是说我们再也无法用 sizeof 获得数组的实际长度了。

```
void test(int x[])
{
	printf("%d\n", sizeof(x));
}

void test2(int* x)
{
	printf("%d\n", sizeof(x));
}

int main(int argc, char* argv[])
{
	int x[] = { 1, 2, 3 };
	printf("%d\n", sizeof(x));

	test(x);
	test2(x);

	return EXIT_SUCCESS;
}
```

输出:

```
12
4
4
```

test 和 test2 中的 sizeof(x) 实际效果是 sizeof(int*)。我们需要显式传递数组长度，或者是一个以特定标记结尾的数组 (NULL)。C99 支持长度可变数组作为函数函数。当我们传递数组参数时，可能的写法包括：

```
/* 数组名默认指向第一元素指针，和 test2 一个意思 */
void test1(int len, int x[])
{
	int i;
	for (i = 0; i < len; i++)
	{
		printf("x[%d] = %d; ", i, x[i]);
	}

	printf("\n");
}

/* 直接传入数组第一元素指针 */
void test2(int len, int* x)
{
	for (int i = 0; i < len; i++)
	{
		printf("x[%d] = %d; ", i, *(x + i));
	}

	printf("\n");
}

/* 数组指针: 数组名默认指向第一元素指针，&array 则是获得整个数组指针 */
void test3(int len, int(*x)[len])
{
	for (int i = 0; i < len; i++)
	{
	printf("x[%d] = %d; ", i, (*x)[i]);
	}

	printf("\n");
}

/* 多维数组: 数组名默认指向第一元素指针，也即是 int(*)[] */
void test4(int r, int c, int y[][c])
{
	for (int a = 0; a < r; a++)
	{
		for (int b = 0; b < c; b++)
		{
			printf("y[%d][%d] = %d; ", a, b, y[a][b]);
		}
	}

	printf("\n");
}

/* 多维数组: 传递第一个元素的指针 */
void test5(int r, int c, int (*y)[c])
{
	for (int a = 0; a < r; a++)
	{
		for (int b = 0; b < c; b++)
		{
			printf("y[%d][%d] = %d; ", a, b, (*y)[b]);
		}

		y++;
	}

	printf("\n");
}

/* 多维数组 */
void test6(int r, int c, int (*y)[][c])
{
	for (int a = 0; a < r; a++)
	{
		for (int b = 0; b < c; b++)
		{
			printf("y[%d][%d] = %d; ", a, b, (*y)[a][b]);
		}
	}

	printf("\n");
}

/* 元素为指针的指针数组，相当于 test8 */
void test7(int count, char** s)
{
	for (int i = 0; i < count; i++)
	{
		printf("%s; ", *(s++));
	}

	printf("\n");
}

void test8(int count, char* s[count])
{
	for (int i = 0; i < count; i++)
	{
		printf("%s; ", s[i]);
	}

	printf("\n");
}

/* 以 NULL 结尾的指针数组 */
void test9(int** x)
{
	int* p;
	while ((p = *x) != NULL)
	{
		printf("%d; ", *p);
		x++;
	}
	
	printf("\n");
}

int main(int argc, char* argv[])
{
	int x[] = { 1, 2, 3 };

	int len = sizeof(x) / sizeof(int);
	test1(len, x);
	test2(len, x);
	test3(len, &x);

	int y[][2] =
	{
		{10, 11},
		{20, 21},
		{30, 31}
	};

	int a = sizeof(y) / (sizeof(int) * 2);
	int b = 2;
	test4(a, b, y);
	test5(a, b, y);
	test6(a, b, &y);

	char* s[] = { "aaa", "bbb", "ccc" };
	test7(sizeof(s) / sizeof(char*), s);
	test8(sizeof(s) / sizeof(char*), s);
	
	int* xx[] = { &(int){111}, &(int){222}, &(int){333}, NULL };
	test9(xx);

	return EXIT_SUCCESS;
}
```

# 8. 指针

## 8.1 void 指针

void* 又被称为万能指针，可以代表任何对象的地址，但没有该对象的类型。也就是说必须转型后才能进行对象操作。void* 指针可以与其他任何类型指针进行隐式转换。

```
void test(void* p, size_t len)
{
	unsigned char* cp = p;

	for (int i = 0; i < len; i++)
	{
		printf("%02x ", *(cp + i));
	}

	printf("\n");
}

int main(int argc, char* argv[])
{
	int x = 0x00112233;
	test(&x, sizeof(x));

	return EXIT_SUCCESS;
}
```

输出:

```
33 22 11 00
```

## 8.2 初始化指针

可以用初始化器初始化指针。
- 空指针常量 NULL。
- 相同类型的指针，或者指向限定符较少的相同类型指针。
- void 指针。

非自动周期指针变量或静态生存期指针变量必须用编译期常量表达式初始化，比如函数名称等。

```
char s[] = "abc";
char* sp = s;

int x = 5;
int* xp = &x;

void test() {}
typedef void(*test_t)();
int main(int argc, char* argv[])
{
	static int* sx = &x;
	static test_t t = test;

	return EXIT_SUCCESS;
}
```

## 8.3 指针运算

(1) 对指针进行相等或不等运算来判断是否指向同一对象。

```
int x = 1;

int *a, *b;
a = &x;
b = &x;

printf("%d\n", a == b);
```

(2) 对指针进行加法运算获取数组第 n 个元素指针。

```
int x[] = { 1, 2, 3 };
int* p = x;

printf("%d, %d\n", x[1], *(p + 1));
```

(3) 对指针进行减法运算，以获取指针所在元素的数组索引序号。

```
int x[] = { 1, 2, 3 };

int* p = x;
p++; p++;

int index = p - x;

printf("x[%d] = %d\n", index, x[index]);
```

输出:

```
x[2] = 3;
```

(4) 对指针进行大小比较运算，相当于判断数组索引序号大小。

```
int x[] = { 1, 2, 3 };
int* p1 = x;
int* p2 = x;
p1++; p2++; p2++;

printf("p1 < p2? %s\n", p1 < p2 ? "Y" : "N");
```

输出:

```
p1 < p2? Y
```

(5) 我们可以直接用 &x[i] 获取指定序号元素的指针。

```
int x[] = { 1, 2, 3 };

int* p = &x[1];
*p += 10;

printf("%d\n", x[1]);
```

注: [] 优先级比 & 高，* 运算符优先级比算术运算符高。

## 8.4 限定符

限定符 const 可以声明 "类型为指针的常量" 和 "指向常量的指针" 。

```
int x[] = { 1, 2, 3 };

// 指针常量: 指针本身为常量，不可修改，但可修改目标对象。
int* const p1 = x;
*(p1 + 1) = 22;
printf("%d\n", x[1]);

// 常量指针: 目标对象为常量，不可修改，但可修改指针。
int const *p2 = x;
p2++;
printf("%d\n", *p2);
```

区别在于 const 是修饰 p 还是 *p。具有 restrict 限定符的指针被称为限定指针。告诉编译器在指针生存周期内，只能通过该指针修改对象，但编译器可自主决定是否采纳该建议。

## 8.5 数组指针

指向数组本身的指针，而非指向第一元素的指针。

```
int x[] = { 1, 2, 3 };
int(*p)[] = &x;

for (int i = 0; i < 3; i++)
{
	printf("x[%d] = %d\n", i, (*p)[i]);
	printf("x[%d] = %d\n", i, *(*p + i));
}
```

&x 返回数组指针，*p 获取和 x 相同的指针，也就是指向第一元素的指针，然后可以用下标或指针运算存储元素。

## 8.6 指针数组

元素是指针的数组，通常用于表示字符串数组或交错数组。数组元素是目标对象 (可以是数组或其他对象) 的指针，而非实际嵌入内容。

```
int* x[3] = {};

x[0] = (int[]){ 1 };
x[1] = (int[]){ 2, 22 };
x[2] = (int[]){ 3, 33, 33 };

int* x1 = *(x + 1);
for (int i = 0; i < 2; i++)
{
	printf("%d\n", x1[i]);
	printf("%d\n", *(*(x + 1) + i));
}
```
输出:

```
2
2
22
22
```

指针数组 x 是三个指向目标对象(数组)的指针，*(x + 1) 获取目标对象，也就是 x[1]。

# 9. 结构

## 9.1 不完整结构

结构类型无法把自己作为成员类型，但可以包含 "指向自己类型" 的指针成员。

```
struct list_node
{
	struct list_node* prev;
	struct list_node* next;
	void* value;
};
```

定义不完整结构类型，只能使用小标签，像下面这样的 typedef 类型名称是不行的。

```
typedef struct
{
	list_node* prev;
	list_node* next;
	void* value;
} list_node;
```

编译出错:

```
$ make

gcc -Wall -g -c -std=c99 -o main.o main.c
main.c:15: error: expected specifier-qualifier-list before ‘list_node’
```
结合起来用吧。

```
typedef struct node_t
{
	struct node_t* prev;
	struct node_t* next;
	void* value;
} list_node;
```

小标签可以和 typedef 定义的类型名相同。

```
typedef struct node_t
{
	struct node_t* prev;
	struct node_t* next;
	void* value;
} node_t;
```

## 9.2 匿名结构

在结构体内部使用匿名结构体成员，也是一种很常见的做法。

```
typedef struct
{
	struct
	{
		int length;
		char chars[100];
	} s;

	int x;
} data_t;

int main(int argc, char * argv[])
{
	data_t d = { .s.length = 100, .s.chars = "abcd", .x = 1234 };
	printf("%d\n%s\n%d\n", d.s.length, d.s.chars, d.x);

	return EXIT_SUCCESS;
}
```

或者直接定义一个匿名变量。

```
int main(int argc, char * argv[])
{
	struct { int a; char b[100]; } d = { .a = 100, .b = "abcd" };
	printf("%d\n%s\n", d.a, d.b);

	return EXIT_SUCCESS;
}
```

## 9.3 成员偏移量

利用 stddef.h 中的 offsetof 宏可以获取结构成员的偏移量。

```
typedef struct
{
	int x;
	short y[3];
	long long z;
} data_t;

int main(int argc, char* argv[])
{
	printf("x %d\n", offsetof(data_t, x));
	printf("y %d\n", offsetof(data_t, y));
	printf("y[1] %d\n", offsetof(data_t, y[1]));
	printf("z %d\n", offsetof(data_t, z));

	return EXIT_SUCCESS;
}
```

注意：输出结果有字节对齐。

## 9.4 定义

定义结构类型有多种灵活的⽅方式。

```
int main(int argc, char* argv[])
{
	/* 直接定义结构类型和变量 */
	struct { int x; short y; } a = { 1, 2 }, a2 = {};
	printf("a.x = %d, a.y = %d\n", a.x, a.y);

	/* 函数内部也可以定义结构类型 */
	struct data { int x; short y; };

	struct data b = { .y = 3 };
	printf("b.x = %d, b.y = %d\n", b.x, b.y);

	/* 复合字面值 */
	struct data* c = &(struct data){ 1, 2 };
	printf("c.x = %d, c.y = %d\n", c->x, c->y);

	/* 也可以直接将结构体类型定义放在复合字面值中 */
	void* p = &(struct data2 { int x; short y; }){ 11, 22 };

	/* 相同内存布局的结构体可以直接转换 */
	struct data* d = (struct data*)p;
	printf("d.x = %d, d.y = %d\n", d->x, d->y);

	return EXIT_SUCCESS;
}
```

输出:

```
a.x = 1, a.y = 2
b.x = 0, b.y = 3
c.x = 1, c.y = 2
d.x = 11, d.y = 22
```

## 9.5 初始化

结构体的初始化和数组一样简洁方便，包括使用初始化器初始化特定的某些成员。未被初始化器初始化的成员将被设置为0。

```
typedef struct
{
	int x;
	short y[3];
	long long z;
} data_t;

int main(int argc, char* argv[])
{
	data_t d = {};
	data_t d1 = { 1, { 11, 22, 33 }, 2LL };
	data_t d2 = { .z = 3LL, .y[2] = 2 };

	return EXIT_SUCCESS;
}
```

结果:

```
d = {x = 0, y = {0, 0, 0}, z = 0}
d1 = {x = 1, y = {11, 22, 33}, z = 2}
d2 = {x = 0, y = {0, 0, 2}, z = 3}
```

## 9.6 弹性结构成员

通常又称作 “不定长结构”，就是在结构体尾部声明一个未指定长度的数组。用 sizeof 运算符时，该数组未计入结果。

```
typedef struct string
{
	int length;
	char chars[];
} string;

int main(int argc, char * argv[])
{
	int len = sizeof(string) + 10; // 计算存储一个 10 字节长度的字符串（包括 \0）所需的长度。
	char buf[len]; // 从栈上分配所需的内存空间。

	string *s = (string*)buf; // 转换成 struct string 指针。
	s->length = 9;
	strcpy(s->chars, "123456789");

	printf("%d\n%s\n", s->length, s->chars);

	return EXIT_SUCCESS;
}
```

考虑到不同编译器和 ANSI C 标准的问题，也用 char chars[1] 或 char chars[0] 来代替。对这类结构体进行拷贝的时候，尾部结构成员不会被复制。

```
int main(int argc, char * argv[])
{
	int len = sizeof(string) + 10;
	char buf[len];

	string *s = (string*)buf;
	s->length = 10;
	strcpy(s->chars, "123456789");

	string s2 = *s; ! ! ! ! ! // 复制 struct string s。
	printf("%d\n%s\n", s2.length, s2.chars); ! // s2.length 正常，s2.chars 就悲剧了。

	return EXIT_SUCCESS;
}
```

而且不能直接对弹性成员进行初始化。

# 10. 联合

联合和结构的区别在于：联合每次只能存储一个成员，联合的长度由最宽成员类型决定。

```
typedef struct
{
	int type;
	union
	{
		int ivalue;
		long long lvalue;
	} value;
} data_t;

data_t d = { 0x8899, .value.lvalue = 0x1234LL };
data_t d2;
memcpy(&d2, &d, sizeof(d));

printf("type:%d, value:%lld\n", d2.type, d2.value.lvalue);
```

当然也可以用指针来实现上例功能，但 union 会将数据内嵌在结构体中，这对于进行 memcpy 等操作更加方便快捷，而且无需进行指针类型转换。

可以使用初始化器初始化联合，如果没有指定成员修饰符，则默认是第一个成员。

```
union value_t
{
	int ivalue;
	long long lvalue;
};

union value_t v1 = { 10 };
printf("%d\n", v1.ivalue);

union value_t v2 = { .lvalue = 20LL };
printf("%lld\n", v2.lvalue);

union value2_t { char c; int x; } v3 = { .x = 100 };
printf("%d\n", v3.x);
```

一个常用的联合用法。

```
union { int x; struct {char a, b, c, d;} bytes; } n = { 0x12345678 };
printf("%#x => %x, %x, %x, %x\n", n.x, n.bytes.a, n.bytes.b, n.bytes.c, n.bytes.d);
```

输出:

```
0x12345678 => 78, 56, 34, 12
```

# 11. 位字段

可以把结构或联合的多个成员 "压缩存储" 在一个字段中，以节约内存。

```
struct
{
	unsigned int year : 22;
	unsigned int month : 4;
	unsigned int day : 5;
} d = { 2010, 4, 30 };

printf("size: %d\n", sizeof(d));
printf("year = %u, month = %u, day = %u\n", d.year, d.month, d.day);
```

输出:

```
size: 4
year = 2010, month = 4, day = 30
```

用来做标志位也挺好的，比用位移运算符更直观，更节省内存。

```
int main(int argc, char * argv[])
{
	struct
	{
		bool a: 1;
		bool b: 1;
		bool c: 1;
	} flags = { .b = true };

	printf("%s\n", flags.b ? "b.T" : "b.F");
	printf("%s\n", flags.c ? "c.T" : "c.F");
	
	return EXIT_SUCCESS;
}
```

不能对位字段成员使用 offsetof。

# 12. 声明

声明 (declaration) 表示目标样式，可以在多处声明同一个目标，但只能有一个定义(definition)。定义将创建对象实体，为其分配存储空间 (内存)，而声明不会。

声明通常包括：
- 声明结构、联合或枚举等用户自定义类型 (UDT)。
- 声明函数。
- 声明并定义一个全局变量。
- 声明一个外部变量。
- 用 typedef 为已有类型声明一个新名字。

如果声明函数时同时出现函数体，则此函数的声明同时也是定义。如果声明对象时给此对象分配内存 (比如定义变量)，那么此对象声明的同时也是定义。

## 12.1 类型修饰符

C99 定义的类型修饰符:
- const: 常量修饰符，定义后无法修改。
- volatile: 目标可能被其他线程或事件修改，使用该变量前，都须从主存重新获取。
- restrict: 修饰指针。除了该指针，不能用其他任何方式修改目标对象。
- 
## 12.2 链接类型

|元素|存储类型|作用域|生存周期|链接类型|
|:---|:------|:------|:--------|:------|
|全局UDT|-| 文件| -|内链接|
|嵌套UDT| -| 类| -|内链接|
|局部UDT| -| 程序块| -|无链接|
|全局函数、变量|extern| 文件|永久|外连接|
|静态全局函数和变量|static| 文件|永久|内链接|
|局部变量、常量|auto |程序块|临时|无链接|
|局部静态变量、常量|static| 程序块|永久|无链接|
|全局常量| -|文件|永久|内链接|
|静态全局常量|static |文件|永久|内链接|
|宏定义| -|文件| -|内链接|

## 12.3 隐式初始化

具有静态生存周期的对象，会被初始化位默认值0（指针为NULL）。

# 13. 预处理

预处理指令以 # 开始 (其前面可以有 space 或 tab)，通常独立一行，但可以用 "\" 换行。

##13.1 常量

编译器会展开替换掉宏。

```
#define SIZE 10

int main(int argc, char* argv[])
{
	int x[SIZE] = {};
	return EXIT_SUCCESS;
}
```

展开:

```
$ gcc -E main.c

int main(int argc, char* argv[])
{
	int x[10] = {};
	return 0;
}
```

## 13.2 宏函数

利用宏可以定义伪函数，通常用 ({ ... }) 来组织多行语句，最后一个表达式作为返回值 (无 return，且有个 ";" 结束)。

```
#define test(x, y) ({ \
	int _z = x + y; \
	_z; })

int main(int argc, char* argv[])
{
	printf("%d\n", test(1, 2));
	return EXIT_SUCCESS;
}
```

展开:

```
int main(int argc, char* argv[])
{
	printf("%d\n", ({ int _z = 1 + 2; _z; }));
	return 0;
}
```

## 13.3 可选性变量

__VA_ARGS__ 标识符用来表示一组可选性自变量。

```
#define println(format, ...) ({ \
	printf(format "\n", __VA_ARGS__); })

int main(int argc, char* argv[])
{
	println("%s, %d", "string", 1234);
	return EXIT_SUCCESS;
}
```

展开:

```
int main(int argc, char* argv[])
{
	({ printf("%s, %d" "\n", "string", 1234); });
	return 0;
}
```

## 13.4 字符串化运算符

单元运算符 # 将一个宏参数转换为字符串。

```
#define test(name) ({ \
	printf("%s\n", #name); })

int main(int argc, char* argv[])
{
	test(main);
	test("\"main");
	return EXIT_SUCCESS;
}
```

展开:

```
int main(int argc, char* argv[])
{
	({ printf("%s\n", "main"); });
	({ printf("%s\n", "\"\\\"main\""); });
	return 0;
}
```

这个不错，会自动进行转义操作。

## 13.5 粘贴记号运算符

二元运算符 ## 将左和右操作数结合成一个记号。

```
#define test(name, index) ({ \
	int i, len = sizeof(name ## index) / sizeof(int); \
	for (i = 0; i < len; i++) \
	{ \
		printf("%d\n", name ## index[i]); \
	}})

int main(int argc, char* argv[])
{
	int x1[] = { 1, 2, 3 };
	int x2[] = { 11, 22, 33, 44, 55 };

	test(x, 1);
	test(x, 2);

	return EXIT_SUCCESS;
}
```

展开:

```
int main(int argc, char* argv[])
{
	int x1[] = { 1, 2, 3 };
	int x2[] = { 11, 22, 33, 44, 55 };

	({ int i, len = sizeof(x1) / sizeof(int); for (i = 0; i < len; i++) { printf("%d\n",
x1[i]); }});
	({ int i, len = sizeof(x2) / sizeof(int); for (i = 0; i < len; i++) { printf("%d\n",
x2[i]); }});

	return 0;
}
```

## 13.6 条件编译

可以使用 "#if ... #elif ... #else ... #endif"、#define、#undef 进行条件编译。

```
#define V1

#if defined(V1) || defined(V2)
	printf("Old\n");
#else
	printf("New\n");
#endif

#undef V1
```

展开:

```
int main(int argc, char* argv[])
{
	printf("Old\n");
	return 0;
}
```

也可以用 #ifdef、#ifndef 代替 #if。

```
#define V1

#ifdef V1
	printf("Old\n");
#else
	printf("New\n");
#endif

#undef A
```

展开:

```
int main(int argc, char* argv[])
{
	printf("Old\n");
	return 0;
}
```

## 13.7 typeof

使用 GCC 扩展 typeof 可以获知参数的类型。

```
#define test(x) ({ \
	typeof(x) _x = (x); \
	_x += 1; \
	_x; \
})

int main(int argc, char* argv[])
{
	float f = 0.5F;
	float f2 = test(f);
	printf("%f\n", f2);

	return EXIT_SUCCESS;
}
```

## 13.8 其他

一些常用的特殊常量。
- error "message" : 定义一个编译器错误信息。
- __DATE__ : 编译日期字符串。
- __TIME__ : 编译时间字符串。
- __FILE__ : 当前源码文件名。
- __LINE__ : 当前源码行号。
- __func__ : 当前函数名称。

# 14. 调试

要习惯使用 assert 宏进行函数参数和执行条件判断，这可以省却很多麻烦。

```
#include <assert.h>

void test(int x)
{
	assert(x > 0);
	printf("%d\n", x);
}

int main(int argc, char* argv[])
{
	test(-1);
	return EXIT_SUCCESS;
}
```

展开效果：

```
$ gcc -E main.c

void test(int x)
{
	((x > 0) ? (void) (0) : __assert_fail ("x > 0", "main.c", 16, __PRETTY_FUNCTION__));
	printf("%d\n", x);
}
```

如果 assert 条件表达式不为 true，则出错并终止进程。

```
$ ./test

test: main.c:16: test: Assertion `x > 0' failed.
Aborted
```

不过呢在编译 Release 版本时，记得加上 -DNDEBUG 参数。

```
$ gcc -E -DNDEBUG main.c

void test(int x)
{
	((void) (0));
	printf("%d\n", x);
}
```