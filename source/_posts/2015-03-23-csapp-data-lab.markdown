---
layout: post
title: "[CSAPP] Data Lab"
date: 2015-03-23 16:34:31 +0800
comments: true
categories: ['C']
---
Data Lab: Manipulating Bits
<!--more-->
``` C bit.c
/* 
 * bitOr - x|y using only ~ and & 
 *   Example: bitOr(6, 5) = 7
 *   Legal ops: ~ &
 *   Max ops: 8
 *   Rating: 1
 */
int bitOr(int x, int y) {
    return ~((~x)&(~y));
}
/* 
 * anyOddBit - return 1 if any odd-numbered bit in word set to 1
 *   Examples anyOddBit(0x5) = 0, anyOddBit(0x7) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 2
 */
int anyOddBit(int x) {
    return (x&1) | ((x&4)>>2) | ((x&16)>>4) | ((x&64)>>6) | ((x&256)>>8) | ((x&1024)>>10) | ((x&1073741824)>>30);
}
/* 
 * copyLSB - set all bits of result to least significant bit of x
 *   Example: copyLSB(5) = 0xFFFFFFFF, copyLSB(6) = 0x00000000
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int copyLSB(int x) {
  return ((x << 31) >> 31);
}
/* 
 * bitMask - Generate a mask consisting of all 1's 
 *   lowbit and highbit
 *   Examples: bitMask(5,3) = 0x38
 *   Assume 0 <= lowbit <= 31, and 0 <= highbit <= 31
 *   If lowbit > highbit, then mask should be all 0's
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int bitMask(int highbit, int lowbit) {
    int mask1 = ~0 << highbit;
    int mask2 = ~(~0 << lowbit);
    int mask3 = ~(1 << highbit);
    mask1 = mask1 & mask3;
    
    return ~(mask1 | mask2);
}
/* 
 * reverseBytes - reverse the bytes of x
 *   Example: reverseBytes(0x01020304) = 0x04030201
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 25
 *   Rating: 3
 */
int reverseBytes(int x) {
    
    
    int newbyte0 = (x >> 24) & 0xff;
    int newbyte1 = (x >> 8) & 0xff00;
    int newbyte2 = (x << 8) & (0xff<<8);
    int newbyte3 = x << 24;
    
    return newbyte0 | newbyte1 | newbyte2 | newbyte3;
}
/* 
 * conditional - same as x ? y : z 
 *   Example: conditional(2,4,5) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 16
 *   Rating: 3
 */
int conditional(int x, int y, int z) {
    int num;
    num = -1;
    (((!x)+num)&y) + ((~((!x)+num))&z); 
}
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {    
    x = ( x >> 16 ) | x;
    x = ( x >> 8 ) | x;
    x = ( x >> 4 ) | x;
    x = ( x >> 2 ) | x;
    x = ( x >> 1) | x;
    
    return ~x & 1;
}
/*
 * isTmax - returns 1 if x is the maximum, two's complement number,
 *     and 0 otherwise 
 *   Legal ops: ! ~ & ^ | +
 *   Max ops: 10
 *   Rating: 1
 */
int isTmax(int x) {
    int m;
    m = (1 << 31) + 1;
    
    return !(x ^ m);
}
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) {
    int mask = x >> 31;
    
    return !(((~x & mask) + (x & ~mask)) >> (n + ~0));
}
/* 
 * isNotEqual - return 0 if x == y, and 1 otherwise 
 *   Examples: isNotEqual(5,5) = 0, isNotEqual(4,5) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int isNotEqual(int x, int y) {
  return (!!(x ^ y));
}
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {
    int one = 0x1 << 31, two, three;
    x = x >> n;
    two = one >> n;
    three = ~(two << 1);
    
    return x & three;
}
/* 
 * rotateLeft - Rotate x to the left by n
 *   Can assume that 0 <= n <= 31
 *   Examples: rotateLeft(0x87654321,4) = 0x76543218
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 25
 *   Rating: 3 
 */
int rotateLeft(int x, int n) {
    return (x << n) | (x >> (32 - n)) & ~(-1 << n);
}
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
  return (!(x<<6))&((x << 5)) & (x << 4) & (~(x << 3) | (~(x << 2) & ~(x << 1)));
}
/* 
 * absVal - absolute value of x
 *   Example: absVal(-1) = 1.
 *   You may assume -TMax <= x <= TMax
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 10
 *   Rating: 4
 */
int absVal(int x) {
    int z; 
    z = x>>31;
    z = z&1;
    z = ~z;
    z = z+1;
    z = x^z;
    int y; 
    y= x>>31;
    y = y&1;
    z = z+y;
    
    return z;
}
/* 
 * isNonZero - Check whether x is nonzero using
 *              the legal operators except !
 *   Examples: isNonZero(3) = 1, isNonZero(0) = 0
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 10
 *   Rating: 4 
 */
int isNonZero(int x) {
    int n = ~x + 1;
    
    return ((n >> 31) | (x >> 31)) & 1;
}
/* 
 * bitAnd - x&y using only ~ and | 
 *   Example: bitAnd(6, 5) = 4
 *   Legal ops: ~ |
 *   Max ops: 8
 *   Rating: 1
 */
int bitAnd(int x, int y) {
  return ~(~x | ~y);  //Using De Morgan's law
}
/* 
 * getByte - Extract byte n from word x
 *   Bytes numbered from 0 (LSB) to 3 (MSB)
 *   Examples: getByte(0x12345678,1) = 0x56
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 2
 */
int getByte(int x, int n) {
  /*
   *First we rightshift x (fine if you like to leftshift :D)
   *Then use &0 to clear the part we don't need
  */
  return (x >> (n << 3)) & (0xFF);
}
/* 
 * logicalShift - shift x to the right by n, using a logical shift
 *   Can assume that 0 <= n <= 31
 *   Examples: logicalShift(0x87654321,4) = 0x08765432
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3 
 */
int logicalShift(int x, int n) {
  /*
   *All we need to do is to clear the sign bit after shifting
   *So we build a mask to make sure that the sign bit of (x>>n) is always 0
  */
  int mask = ~(((1 << 31) >> n) << 1); //set 1 for the hightest digit
  return mask & (x >> n);
}
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
int bitCount(int x) {
  int mask1 = (0x55) | (0x55 << 8) | (0x55 << 16) | (0x55 << 24); //  mask1=0x55555555
  int mask2 = (0x33) | (0x33 << 8) | (0x33 << 16) | (0x33 << 24); //  mask2=0x33333333
  int mask3 = (0x0f) | (0x0f << 8) | (0x0f << 16) | (0x0f << 24); //  mask3=0x0f0f0f0f
  int mask4 = (0xff) | (0xff << 16);  //  mask4=0x00ff00ff
  int mask5 = (0xff) | (0xff << 8);  // mask5=0x0000ffff
  int result; 
  /*
   *result = (mask1 & x) + (mask1 & (x >> 1));  //  add every 2 bits
   *result = (mask2 & result) + (mask2 & (result >> 2));  //  add every 4 bits
   *result = (mask3 & result) + (mask3 & (result >> 4));  //  add every 8 bits
   *result = (mask4 & result) + (mask4 & (result >> 8));  //  add every 16 bits
   *result = (mask5 & result) + (mask5 & (result >> 16)); //  add every 32 bits
  */
  result = (x & mask1) + ((x >> 1) & mask1);
  result = (result & mask2) + ((result >> 2) & mask2); 
  result = (result + (result >> 4)) & mask3;
  result = (result + (result >> 8)) & mask4;
  result = (result + (result >> 16)) & mask5;
  return result;
}
/* 
 * bang - Compute !x without using !
 *   Examples: bang(3) = 0, bang(0) = 1
 *   Legal ops: ~ & ^ | + << >>
 *   Max ops: 12
 *   Rating: 4 
 */
int bang(int x) {
  return (~((x | (~x + 1)) >> 31) & 1);
}
/* 
 * tmin - return minimum two's complement integer 
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 4
 *   Rating: 1
 */
int tmin(void) {
  return 1 << 31;
}
/* 
 * fitsBits - return 1 if x can be represented as an 
 *  n-bit, two's complement integer.
 *   1 <= n <= 32
 *   Examples: fitsBits(5,3) = 0, fitsBits(-4,3) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int fitsBits(int x, int n) {
  /* if x can be represented as an n-bit, two's complement integer,
   * then high 32-n+1 bits are all 0s or 1s.
   */
  x = x >> (n + ~0);  /* x >> n-1, ~0 to get -1 */
  x = x + (x & 1);
  return !x;
}
/* 
 * divpwr2 - Compute x/(2^n), for 0 <= n <= 30
 *  Round toward zero
 *   Examples: divpwr2(15,1) = 7, divpwr2(-33,4) = -2
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 15
 *   Rating: 2
 */
int divpwr2(int x, int n) {
    int sign = (x >> 31) & 1; /* get sign */
    int mask = (1 << n) + ~0;
    int lowbits = x & mask;
    return (x >> n) + (((!lowbits) ^ 1) & sign);
}
/* 
 * negate - return -x 
 *   Example: negate(1) = -1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 5
 *   Rating: 2
 */
int negate(int x) {
  return ~x + 1;
}
/* 
 * isPositive - return 1 if x > 0, return 0 otherwise 
 *   Example: isPositive(-1) = 0.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 8
 *   Rating: 3
 */
int isPositive(int x) {
  return !((x >> 31) | !x);
}
/* 
 * isLessOrEqual - if x <= y  then return 1, else return 0 
 *   Example: isLessOrEqual(4,5) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isLessOrEqual(int x, int y) {
  int z = (~x + 1) + y; /*  z = y - x  */
  int signx = (x >> 31) & 1;
  int signy = (y >> 31) & 1;
  int signz = (z >> 31) & 1;
  /*  (x < 0 and y >= 0)  and  (x * y >= 0 and x <= y)  */
  return ((signx ^ signy) & signx) + ((signx ^ signy ^ 1) & !signz);
}
/*
 * ilog2 - return floor(log base 2 of x), where x > 0
 *   Example: ilog2(16) = 4
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 90
 *   Rating: 4
 */
int ilog2(int x) {
  /*  while (x) x /= 2;
   *  use dichotomy to calculate the sum of 0s
   *  if there're x 0s on the left then shiftleft x bits
   */
  int x1, x2, x3, x4, x5, n;
  x1 = x >> 16;
  n = (16 & ((!x1) << 4));
  x = x << (16 & ((!x1) << 4));
  x2 = x >> 24;
  n = n + (8 & ((!x2) << 3));
  x = x << (8 & ((!x2) << 3));
  x3 = x >> 28;
  n = n + (4 & ((!x3) << 2));
  x = x << (4 & ((!x3) << 2));
  x4 = x >> 30;
  n = n + (2 & ((!x4) << 1));
  x = x << (2 & ((!x4) << 1));
  x5 = x >> 31;
  n = n + (1 & (!x5));
  return 31 + ~n + 1;  /* 32 - (n + 1) */
}
/* 
 * float_neg - Return bit-level equivalent of expression -f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representations of
 *   single-precision floating point values.
 *   When argument is NaN, return argument.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 10
 *   Rating: 2
 */
unsigned float_neg(unsigned uf) {
  unsigned result;  
  unsigned tmp;  
  tmp = uf & 0x7fffffff;  /* remove sign bit */
  result = uf ^ 0x80000000; /* change sign bit */
  if ( tmp > 0x7f800000)  /* NaN */
        result = uf;  
 return result;  
}
/* 
 * float_i2f - Return bit-level equivalent of expression (float) x
 *   Result is returned as unsigned int, but
 *   it is to be interpreted as the bit-level representation of a
 *   single-precision floating point values.
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_i2f(int x) {
  unsigned shiftLeft = 0, afterShift, tmp, flag, abs_x = x, sign = 0;  
  if (x == 0) 
    return 0; /* special case */
  /* get sign and abs for negative x */
  if (x < 0)  
  {  
    sign = 0x80000000;  
    abs_x = -x;  
  }  
  afterShift = abs_x;  
  while (1)  
  {  
    tmp = afterShift;
    afterShift <<= 1;
    shiftLeft++;  /* count shiftleft */
    if (tmp & 0x80000000)
      break;  
  }
  if ((afterShift & 0x01ff) > 0x0100)
    flag = 1;  
  else 
    if ((afterShift & 0x03ff) == 0x0300)  
      flag = 1;  
    else
      flag = 0;
  /* construct the float expression */
  return sign + (afterShift >> 9) + ((159 - shiftLeft) << 23) + flag;
}
/* 
 * float_twice - Return bit-level equivalent of expression 2*f for
 *   floating point argument f.
 *   Both the argument and result are passed as unsigned int's, but
 *   they are to be interpreted as the bit-level representation of
 *   single-precision floating point values.
 *   When argument is NaN, return argument
 *   Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
 *   Max ops: 30
 *   Rating: 4
 */
unsigned float_twice(unsigned uf) {
  unsigned f = uf;  
  /* Compute 2*f. If f is NaN, then return f */  
  if ((f & 0x7F800000) == 0)
    f = ((f & 0x007FFFFF) << 1) | (0x80000000 & f);
  else
    if ((f & 0x7F800000) != 0x7F800000) /* exp can be extended */
      f =f+0x00800000;  /* exp + 1 */
  return f;  
}
/*
 * bitNor - ~(x|y) using only ~ and &
 *   Example: bitNor(0x6, 0x5) = 0xFFFFFFF8
 *   Legal ops: ~ &
 *   Max ops: 8
 *   Rating: 1
 */
int bitNor(int x, int y) {

  return (~x)&(~y);
}
/*
 * bitXor - x^y using only ~ and &
 *   Example: bitXor(4, 5) = 1
 *   Legal ops: ~ &
 *   Max ops: 14
 *   Rating: 2
 */
int bitXor(int x, int y) {
  int a=(~x)&y;
  int b=x&(~y);
  return ~(~a&~b);
}
/*
 * leastBitPos - return a mask that marks the position of the
 *               least significant 1 bit. If x == 0, return 0
 *   Example: leastBitPos(96) = 0x20
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 4
 */
int leastBitPos(int x) {
    return x&(~x+1);
}
/*
 * isNonNegative - return 1 if x >= 0, return 0 otherwise
 *   Example: isNonNegative(-1) = 0.  isNonNegative(0) = 1.
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 6
 *   Rating: 3
 */
int isNonNegative(int x) {
    return !((x>>31)&0x01);
}
/*
 * isGreater - if x > y  then return 1, else return 0
 *   Example: isGreater(4,5) = 0, isGreater(5,4) = 1
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 24
 *   Rating: 3
 */
int isGreater(int x, int y) {
    int signofx=x>>31;
    int signofy=y>>31;
    int signequal=(!(signofx ^ signofy)) & ((~y + x) >> 31);
    int signnequal=signofx & !signofy;
    return !(signequal | signnequal);
}
/*
 * addOK - Determine if can compute x+y without overflow
 *   Example: addOK(0x80000000,0x80000000) = 0,
 *            addOK(0x80000000,0x70000000) = 1,
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 20
 *   Rating: 3
 */
int addOK(int x, int y) {
    int xysum = x + y;
    int signx = x >> 31;
    int signy = y >> 31;
    int signsumxy = xysum >> 31;  
    return !(~(signx ^ signy) & (signx ^ signsumxy));
}
```