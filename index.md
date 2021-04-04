# 第一周lab实验周记

## 1、bitXor
实现异或的功能  
数电里学的摩根定理派上用场了  

```
int bitXor(int x, int y){
    return  ~( ~((~x) & y ) & ~( x & (~y) ) );
}
```
## 2、tmin
要找出最小的32位整数补码
```
int tmin(void) {
  return 1 << 31;
}
```
## 3、isTmax
判断传参是否为32位的最大值，是返回1，不是返回0
```
int isTmax(int x){
    return !( (x+1+x+1) | !(~x) );
}
```
最大值0x7fffffff加一就是最小值，最小值加自己本身也是0，也就是说除了最小值和0其他的数返回的都是1，但是0xffffffff(-1)也满足第一个条件，所以第二个限制条件先对0xffffffff按位取反全部变0，然后再把值取反变1，1或0为1，再取值的反为0。
## 4、allOddBits
判断传参数（32位）的奇数位是否都为1，如果是返回1，不是则为0
```
int allOddBits(int x){
    int odd = 0xAA << 24;
	odd = odd | 0xaa << 16;
	odd = odd | 0xaa << 8;
	odd = odd | 0xaa;
	return !((x & odd) ^ odd);
}
```
通过位移构造0xaaaaaaaa，然后将输入的参数与odd进行与运算（0xffffffff），然后与odd异或，相同返回!0，不同返回!1
## 5、negate
要求返回的是参数的负数形式，负数的补码是取反+1，也就是说~x + 1就行了
```
int negate(int x) {
  return ~x + 1;
}
```
## 6、isAsciiDigit
判断输入的数是否在规定范围内，在的话返回1，不在返回0
```
int isAsciiDigit(int x) {
    int m = x + (~(0x30) + 1);
	int n = 0x39 + (~(x) + 1);
	return !((m&(1<<31))|(n&(1<<31)));
}
```
## 7、conditional
模拟 x ? y : z 
```
int conditional(int x, int y, int z) {
  return ((((!!x)<<31)>>31) & y )|(~(((!!x)<<31)>>31) & z);  
}
```
## 8、isLessOrEqual
实现<=的功能
```
int isLessOrEqual(int x, int y) {
  int sub = y + (~x + 1);
  x = !!(x&(1<<31));
  y = !!(y&(1<<31));
  return ((x^y)&(((~(x)+1))|(~y)))|((~(x^y))&(!(sub&(1<<31))));
}
```
## 9、logicalNeg
实现 ！的功能，0则返回1，否则返回0
```
int logicalNeg(int x) {
  return ((x|(~x+1))>>31) + 1;
}
```
这题冥思苦想了半天没有很好的思路，于是从网上看别的师傅的博客，这题的解法其实是用的对除了0和最小数以外的所有数取相反数的时候，其值都是 -(x) ，也就是最高位为1，而最小数的最高为同样也是1，所以除了0以外的所有数进行>>31后都为 0xffffffff，最后再+1的话0xffffffff变为0，0变为1，完成本题目的要求。
## 10、howManyBits
实现将一个数的补码最少有几位打印出来
```
int howManyBits(int x) {
  int sign = x>>31;
  int b16, b8, b4, b2, b1, b0;
  x = (~sign&x)|(sign&~x);
  b16 = !!(x>>16)<<4;
  x >>= b16;
  b8 = !!(x>>8)<<3;
  x >>= b8;
  b4 = !!(x>>4)<<2;
  x >>= b4;
  b2 = !!(x>>2)<<1;
  x >>= b2;
  b1 = !!(x>>1);
  x >>= b1;
  b0 = !!(x);
  return b16 + b8 + b4 + b2 + b1 + b0 + 1;
}
```
这题的思路其实挺明确的，就是判断最左端1的位置在哪里，但是实现的时候发现思路根本不通，于是去看的网上师傅们的做法，了解到了这题可以通过二分法做出来。
首先是符号位的问题，整数不变负数取反，然后整数共32位先判断前16位中是否有1，如果有那就将x右移16位，否则不变，再然后就是重复上述步骤了，不断缩小判断的范围直到最后一位，最后将所有位移数加起来就是左边第一个1的位置了，最后+1是因为还有个符号位，over。
## 11、floatScale2
计算出uf * 2的值
```
unsigned floatScale2(unsigned uf) {
  int sign = uf&(1<<31);
  int exp = (uf&0x7f800000)>>23;
  int frac = uf&0x7fffff;
  if(exp==0)
    return sign|(uf<<1);
  if(exp==255) 
    return uf;
  exp++;
  if(exp==255) 
    return sign | 0x7f800000;
  return sign|(exp<<23)|frac;
}
```
首先将uf分为三部分（符号位，指数位，尾数位），然后对指数位进行判断，当exp为全零的时候（0或无穷小）对uf乘二再加符号位， 或全为1的时候（无穷大或NaN）则直接输出本身。若都不是则就是规范式，将exp++（数值乘二），需要特别注意的是当exp为11111110（即254）时，如果乘以2会造成数值变为255，需要特别设置返回值，若都不是则将符号位指数位尾数位组合即可
## 12、floatFloat2Int
将浮点数转化为整数
```
int floatFloat2Int(unsigned uf) {
  int sign = uf>>31;
  int exp = ((uf&0x7f800000)>>23)-127;
  int frac = (uf&(0x7fffff))|0x800000;

  if(exp>31)
    return 0x80000000;
  if(exp<0)
    return 0;

  if(exp>23)
    frac = frac<<(exp-23);
  else
    frac = frac>>(23-exp);
  if(!(frac>>31 ^ sign))
    return frac;
  if(frac>>31)
    return 0x80000000;
  return ~frac + 1;
}
```
首先将uf分为三部分（符号位，指数位，尾数位），这里因为是要转成int型，所以将原先隐藏的首位1表示出来（即1.frac），再然后进行判断exp是否溢出，若溢出则返回0x80000000，若小于0则返回0。后面将frac转为整数，并判断符号是否发生改变，没改直接返回，如果改了则要判断是正是负，是负数则说明溢出直接返回0x80000000，整数则进行取反操作。
## 13、floatPower2
返回2的x幂
```
unsigned floatPower2(int x) {
  int exp = x + 127;
  if(exp <= 0) 
    return 0;
  if(exp >= 255) 
    return 0x7f800000;
  return exp << 23;
}
```
需要考虑两个特殊情况，当exp>128时返回0x7f800000，当exp小于-127时返回0
### 小结
一整个data lab做下来收获还是不小的，对位操作有了更深层次的理解，但是本章最后的浮点数那块掌握的不是很好，最后的三个lab实验都是去网上看了很多别的师傅的做法然后照着做的，等着一轮学习结束以后再回过头来好好把这块补上。

