# CSAPP:datalab及总结

> **万丈高楼平地起 勿在浮沙筑高台**



***注：本文为datalab及Chapter2内容的个人记录总结，有错误欢迎评论指出***

## datalab代码及分析思路

### Q1 bitXor

#### Source code

```c
/* 
 * bitXor - x^y using only ~ and & 
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 1
 */
int bitXor(int x, int y) {
 return ~(~(~x&y)&~(x&~y));
}
```

#### Analysis

异或定义
$$
x\oplus y=(x\& \lnot y)|(\lnot x\&y)
$$
德摩根定律
$$
\lnot(A\lor B)=\lnot A\land \lnot B\\
\lnot(A\land B)=\lnot A\lor \lnot B
$$
根据异或的定义，运用德摩根定律，考察逻辑运算变换。根据公式变换成只有~和&。

### Q2 tmin

#### Source code

```c
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
 return 0x1<<31;
}
```

#### Analysis

对于补码来说，int32_min为-214748368，十六进制(补码)为0x1000 0000。0x1左移31位即可。

### Q3 isTmax

#### Source code

```c
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x){
 int tm=x+1;
 return !(~(tm^x)|!tm);  //相当于!(~(tm^x))&!!tm(分析得到的)
}
```

#### Analysis

我一开始思路有问题，这里讲一下参考后我的思路。int32_max为214748367，十六进制(补码)为0x7fff ffff。考虑到-1以及2^31-1有这样的特点：**~((x+1)^x)=0**。接着想办法分辨-1和2^31-1。考虑到**x+1**后-1变为0x0，2^31-1变为0x1000 0000。对其取两次非，这样就能让目标取到1，-1只能取到0。

### Q4 allOddBits

#### Source code

```c
/* 
 * allOddBits - return 1 if all odd-numbered bits in word set to 1
 *   where bits are numbered from 0 (least significant) to 31 (most significant)
 *   Examples allOddBits(0xFFFFFFFD) = 0, allOddBits(0xAAAAAAAA) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int allOddBits(int x) {
  int t=0xAA;
  t|=t<<8;
  t|=t<<16;
  return !((t&x)^t);
}
```

#### Analysis

要取到某数所有的奇数位，考虑构造0xAAAA AAAA来按位并上它。拿到奇数位状态后与0xAAAA AAAA异或检查奇数位是否都为1。

### Q5 negate

#### Source code

```c
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x+1;
}
```

#### Analysis

学过计组补码相关知识，应该都知道对补码取反加1得到原补码表示数的相反数。这点CSAPP上也有提到。这里评测应该是要求输入合法的，所以我当时也没做TMin的特判。

### Q6 isAsciiDigit

#### Source code

```c
/* 
 * isAsciiDigit - return 1 if 0x30 <= x <= 0x39 (ASCII codes for characters '0' to '9')
 *   Example: isAsciiDigit(0x35) = 1.
 *            isAsciiDigit(0x3a) = 0.
 *            isAsciiDigit(0x05) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 3
 */
int isAsciiDigit(int x) {
  int h24=(1<<31)>>23;
  return (!(h24&x))&((!((x&0xf8)^0x30))|(!((x&0xfe)^0x38)));
}
```

#### Analysis

这题我一开始弄错题目意思，求的低8位为0x30~0x39，后来参考题解才反应过来就增加了对高24位是否为0的判断。对于位运算，我首先就是考虑从数的特殊点下手，考虑到这个范围都是0x0011xxxx，继续看低4位，发现0~7的形式为0xxx，且恰好对应第三位的八种二进制组合，所以就可以分出第一类情况形如0x0011 0xxx就行。然后考虑剩下的数(8,9)，同样的思路去考虑，分析出第二类情况形如0x0011 100x即可。最后构造出0xFFFF FF00来检查高24位是否为0。

### Q7 conditional

#### Source code

```c
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
  int t=!x;
  return (((!t)<<31)>>31&y)|((t<<31)>>31&z);
}
```

#### Analysis

题目要求就是x为0取z，非0取y。很容易想到对x取奇偶次的！分别得到x为0时为真、x为非0时为真。这样就有(!!x&y)|(!x&z)，这是我一开始的思路，没适应这是二进制。对应情况为真时我们只能取到最低位，考虑扩展0x1为0xFFFF FFFF，这样为真时取到对应情况的数，为假时不取。

### Q8 isLessOrEqual

#### Source code

```c
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int c1=!(x^y);

  int sigx=(x>>31)&1;
  int sigy=(y>>31)&1;
  int c2=!(!sigx&sigy);
  int c3=sigx&!sigy;

  int sum=x+~y+1;
  int c4=sum>>31&1;

  return c2&(c1|c3|c4);
}
```

#### Analysis

第一次写的思路比较乱，有想到一些点但是考虑不全面。这里我参考了题解做一些思路总结。我第一次想到是直接作差，但是得做几个特判，所以直接分情况讨论处理：

**1.x=y，!(x^y)=1**

**2.x为正，y为负，!((x>>31)&1)&((y>>31)&1)**

**3.x为负，y为正，((x>>31)&1)&!((y>>31)&1)**

**4.同号作差，((x+(~y+1))>>31)&1**

按题目要求1，3，4的真假正好对应题目要求，但是2得取反。这里就需要注意一个点了，2的反面对应1，3，4中的一种可能，在2取反之后即情况2为真返回0，为假返回1。位运算中的按位或不同于高级语言中的if..else。由于这里对情况2取反了，那么只有当情况2为假时即返回1时，才会出现1，3，4中的一种可能。所以最后结果应该是c2&(c1|c3|c4)。第一时间我想当然就取的c1|c2|c3|c4就出问题了，这里我提醒一下。~~还是我位运算不熟练~~

### Q9 LogicalNeg

#### Source code

```c
/* 
 * logicalNeg - implement the ! operator, using all of 
 *              the legal operators except !
 *   Examples: logicalNeg(3) = 0, logicalNeg(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int logicalNeg(int x) {
  x|=x>>16;
  x|=x>>8;
  x|=x>>4;
  x|=x>>2;
  x|=x>>1;
  return (x&1)^1;
}
```

#### Analysis

要求实现！运算，即非0取0x0，0取0x1。考虑检测所有位是否全0，根据位运算的特点，这里对32个二进制位对半或取，直到或取上所有位。然后得到最低位，这样非0返回1，0返回0只需要异或上一个1即可对应题目要求。

### Q10 howManyBits

#### Source code

```c
/* howManyBits - return the minimum number of bits required to represent x in
 *             two's complement
 *  Examples: howManyBits(12) = 5
 *            howManyBits(298) = 10
 *            howManyBits(-5) = 4
 *            howManyBits(0)  = 1
 *            howManyBits(-1) = 1
 *            howManyBits(0x80000000) = 32
 *  Legal ops: ! ~ & ^ | + << >>
 *  Max ops: 90
 *  Rating: 4
 */
int howManyBits(int x){
  int isZero=!x;
  int hb16,hb8,hb4,hb2,hb1,hb0;
  int sum;
  int sig=x>>31;
  x=(~sig&x)|(sig&~x);
  hb16=!!(x>>16)<<4;
  x>>=hb16;
  hb8=!!(x>>8)<<3;
  x>>=hb8;
  hb4=!!(x>>4)<<2;
  x>>=hb4;
  hb2=!!(x>>2)<<1;
  x>>=hb2;
  hb1=!!(x>>1);
  x>>=hb1;
  hb0=x;

  sum=hb16+hb8+hb4+hb2+hb1+hb0+1;

  return isZero|(((!isZero)<<31)>>31)&sum;
}
```

#### Analysis

这题还是比较难想的，得抓住二进制数特点以及单调性巧用二分查找，说实话有点**Hamming code**的味道。首先考虑特殊值0，一定只需要一位来表示。非0部分我们不妨简单点先尝试正数部分，正数最重要的就是最高位的1，这个1上面位多的0都没意义，只需要一个0来表示符号就够了。这样就得出正数可以这样表示，一个符号位加上最高位1所在的位数。对于负数部分我们尝试用同样的思路去解决，由于负数符号位可以进行拓展，但实际上一个符号位就可以表示正负了，而且扩展前后补码对应的数是一样的。继续看剩下的位，显然我们要看的就是符号位后的第一个0，这样就和整数对应起来了，只需要对负数取反，和正数一样找最高位1所在位数加上符号位即可。这样就有：

**1.x=0，返回1**

**2.x为正不变，为负按位取反，返回1(符号位)+最高位的1所在位数。**

现在问题就到了如何求最高位1上，采用位运算中的二分查找。根据单调性，位次从左到右依次降低一个单位，先看高32/2位是否有1，!!(x>>32/2)有1返回1意味着x至少有32/2位(记录下二进制数!!(x>>32/2)>>log2(32/2))才能表示，否则返回0意味着最高位1可能在低32/2位。同样的方法对半看16、8、4、2、1位。每次二分查找过程将检测结果记录下来，然后将记录结果加起来就可以得到最高位后面至少有多少个位。最后记录下最后依次迭代的x也就是最高位数值(是为了排除像0xFFFF FFFF这样的)。将记录的hb16、hb8、hb4、hb2、hb1、hb0加起来然后加上一个符号位的位数就可以得到答案了。代码实现逻辑参考上面就行了。

### Q11 floatScale2

#### Knowledge points

*这个点后面两题也要用到，先弄清楚IEEE754单精度浮点数表示的类型(规格化、非规格化或特殊值)及其对应的范围，参考下面两张图，建议根据第一张图手推一下范围及二进制形式*

![QQ截图20231007151313](D:\CSAPP总结\assets\QQ截图20231007151313.png)

![QQ截图20231007151408](D:\CSAPP总结\assets\QQ截图20231007151408.png)

#### Source code

```c
/* 
 * floatScale2 - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned floatScale2(unsigned uf) {
  int sig=(1<<31)&uf;
  int exp=(uf>>23)&0xff;
  int frac=uf&0x7fffff;

  if(exp==0xff||(exp==0&&frac==0))return uf;
  if(exp==0){
   frac<<=1;
   return sig|frac;
}
  if(exp==0xff-1){
   exp=0xff;
   frac=0;
   return sig|exp<<23;
}

   exp+=1;
   return sig|exp<<23|frac;
}
```

#### Analysis

在熟悉上面那个知识点后我们可以很容易分类讨论出四种情况：

**1.特判NaN(e=0xFF,frac!=0),0(e=0,frac=0),∞(e=0xFF,frac=0)，return uf**

**2.非规格化显然不会溢出(exp=0)，frac<<=1**

**3.规格化乘2溢出(exp=0xFF-1)，return ∞(e=0xFF,frac=0)**

**4.规格化乘2未溢出，exp+=1**

这里测评应该是没给到第三种情况的特殊值，没有第三种情况判断也能过

### Q12 *floatFloat2Int*

#### Source code

```c
/* 
 * floatFloat2Int - Return bit-level equivalent of expression (int) f
 *   for floating point argument f.
 *   Argument is passed as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point value.
 *   Anything out of range (including NaN and infinity) should return
 *   0x80000000u.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
int floatFloat2Int(unsigned uf) {
  int sig=uf>>31;
  int exp=(uf>>23)&0xff;
  int frac=uf&0x7fffff;
  int E;

  if((exp==0)||(exp>=1&&exp<=126))return 0;
  else if(exp>=127&&exp<=157){
   E=exp-127;
   frac|=0x800000;
   if(E>23){
	frac<<=(E-23);
   }
   else if(E<=23){
	frac>>=(23-E);
   }
}
  else if(exp>=158&&exp<=255)return 0x80000000u;
  
  if(sig)return ~frac+1;
  return frac;
}
```

#### Analysis

考虑int最大为2^31-1，最小为0，E=e-bias，bias=2^7-1=127，e∈(0,255)，则e-bias∈(-127,128)。这样我们可以分情况讨论下溢、未溢出、上溢情况：

**1.下溢，返回的数本身小于1**

i.为非规格化形式，exp=0

ii.为规格化形式，E∈[-126,-1]，e∈[1,126]

**2.未溢出，返回的数大于等于1但不溢出，E∈[0,31)，e∈[127,158)**

对frac补上小数点的前一位1，再根据E调整数位

i.E>23，(frac|=0x800 000)<<=(E-23)

ii.E<=23，(frac|=0x800 000)>>=(23-E)

默认向0舍入，E<=23时，低于23位的frac直接丢掉

**3.上溢，返回的数超过int所能表示的范围，E∈[31,128]，e∈[158,255]**

i.E∈[31,127]，e∈[158,254]，return 0x8000 0000

ii.为NaN，e=255，return 0x8000 0000

### Q13 floatPower2

#### Source code

```c
/* 
 * floatPower2 - Return bit-level equivalent of the expression 2.0^x
 *   (2.0 raised to the power x) for any 32-bit integer x.
 *
 *   The unsigned value that is returned should have the identical bit
 *   representation as the single-precision floating-point number 2.0^x.
 *   If the result is too small to be represented as a denorm, return
 *   0. If too large, return +INF.
 * 
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while 
 *   Max ops: 30 
 *   Rating: 4
 */
unsigned floatPower2(int x) {
    unsigned exp=0,frac=0;
    if(x<-149){
    }
    else if(x<-126)
	frac=1<<(x+149);
    else if(x<128)
	exp=x+127;
    else 
	exp=0xff;
    return exp<<23|frac;
}
```

#### Analysis

根据题目有
$$
2^x=(-1)^s*2^E*M\\
e∈(0,255)\\
E∈(-127,128)\\
M=1.fff...=0.000...1*2^{23}
$$
和上题类似已知情况讨论范围：

**1.下溢**

E+23<-127，E<-149小到e无法表示，return 0

**2.非规格化形式**

E+23>=-126,E>=-149且E<=1-bias=-127

推导x过程
$$
e=0\\
M*2^E=2^x\\
M=2^x/2^E=2^{x-E}\\
\because\ x\  is\ Denormalize\\
\therefore M=f=2^{x-E}=0.fff...\\
\therefore x=2^{x-E}<<23=2^{x-E+23}
$$
frac=1<<(x-(1-bias)+23)

**3.规格化形式**

E>1-bias=-127且e<255，E<e-bias<128

2^x=M*2^E，说明M小数部分为0，则e=E+bias=x+bias=x+127

exp=x+127，frac=0

**4.上溢**

e>=255

exp=0xFF，frac=0

## datalab实验结果判分

![QQ截图20231007174337](D:\CSAPP总结\assets\QQ截图20231007174337.png)

## CSAPPChapter2学习总结

### 零散的值得记录的一些Knowledge Points

 1.如何区分机器字节顺序(大小端)

```c
#include<stdio.h>

typedef unsigned char *byty_pointer

void show_bytes(byte_pointer start,size_t len){
    size_t i;
    for(i=0;i<len;i++)
        printf("%.2x",start[i]);
    printf("\n");
}

void show_int(int x){
    show_bytes((byte_pointer)&x,sizeof(int));
}

void show_float(float x){
    show_bytes((byte_pointer)&x,sizeof(float));
}

void show_pointer(void *x){
    show_bytes((byte_pointer)&x,sizeof(void *));
}

void test_show_bytes(int val){
    int ival=val;
    float fval=(float)val;
    int *pval=&ival;
    show_int(ival);
    show_float(fval);
    whow_pointer(pval);
}

int main(){
    int val=12345;
    test_show_bytes(val);
    return 0;
}
```

2.整数乘法、加法遵循交换律和结合律，但溢出时为模运算

3.浮点数加法不遵循交换律和结合律，乘法遵循交换律不遵循结合律

4.有符号数(以int_32为例)位模式及补码范围

<img src="D:\CSAPP总结\assets\IMG_0741(20231007-170026).JPG" alt="IMG_0741(20231007-170026)"  />

5.浮点数符号位单独考虑，其正负性只受符号位影响

6.浮点数非规格化数的E=1-bias，M=0.fff...

7.始终注意有符号数到无符号数的隐式强制类型转换

8.清楚向偶舍入、向零舍入、向下舍入、向上舍入的概念

9.IEEE754浮点表示位模式及各类范围(Q11中的KP)

10.各种位运算的奇淫巧计

### 对于学习过程的反思总结

一章内容作为一个模块，内容相对紧凑，尽可能地集中时间学习某一章的内容并且做完相关lab，这样才能提高学习效率，缩短学习周期，做到高质高效。个人感觉CMU15-213的内容没有书上详细，Open course教授也说以书本为讲义内容，最好还是以书本为主。
