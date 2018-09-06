title: 一个接口性能测试脚本开发案例
date: 2015-03-08 22:03:54
author: 赵空暖
tags: 性能测试
categories: 性能测试
---
#课堂笔记#
开发人员开发出来的接口，提供给测试人员详细的接口使用说明书,然后测试人员根据文档的描述在LoadRunner书写相应的接口测试脚本。

#课堂作业#
<b>1、在互联网上找到LoadRunner使用md5加密的函数并放入脚本中，拼接以下字符串并使用md5加密：待加密的字符串是 = 自己名字的全拼 + 当前时间到1970年的秒数 + 当前主机的hostname将要加密的字符串打印出来。</b>
当前时间到1970年的秒数,用到的一个函数：
```bash
time_t time( time_t *timeptr ); 
timeptr  Pointer to long integer that will store the time.  
The time function returns the number of seconds elapsed since midnight (00:00:00), January 1, 1970, coordinated universal time (UTC), according to the system clock. The return value is stored in the location given by timeptr. If timeptr is NULL the value is not stored. 
```
网上找的MD5算法代码,将以下代码保存为md5.h文件导入到loadrunner中，然后在`globals.h`中引入头文件`#include "md5.h"`
```c
#ifndef MD5_H
#define MD5_H
#ifdef __alpha
typedef unsigned int uint32;
#else
typedef unsigned long uint32;
#endif
struct MD5Context {
        uint32 buf[4];
        uint32 bits[2];
        unsigned char in[64];
};
extern void MD5Init();
extern void MD5Update();
extern void MD5Final();
extern void MD5Transform();
typedef struct MD5Context MD5_CTX;
#endif
#ifdef sgi
#define HIGHFIRST
#endif
#ifdef sun
#define HIGHFIRST
#endif
#ifndef HIGHFIRST
#define byteReverse(buf, len)    /* Nothing */
#else
void byteReverse(buf, longs)unsigned char *buf; unsigned longs;
{
    uint32 t;
    do {
    t = (uint32) ((unsigned) buf[3] << 8 | buf[2]) << 16 |((unsigned) buf[1] << 8 | buf[0]);
    *(uint32 *) buf = t;
    buf += 4;
    } while (--longs);
}
#endif
void MD5Init(ctx)struct MD5Context *ctx;
{
    ctx->buf[0] = 0x67452301;
    ctx->buf[1] = 0xefcdab89;
    ctx->buf[2] = 0x98badcfe;
    ctx->buf[3] = 0x10325476;
    ctx->bits[0] = 0;
    ctx->bits[1] = 0;
}
void MD5Update(ctx, buf, len) struct MD5Context *ctx; unsigned char *buf; unsigned len;
{
    uint32 t;
    t = ctx->bits[0];
    if ((ctx->bits[0] = t + ((uint32) len << 3)) < t)
    ctx->bits[1]++;
    ctx->bits[1] += len >> 29;
    t = (t >> 3) & 0x3f;
    if (t) {
    unsigned char *p = (unsigned char *) ctx->in + t;
    t = 64 - t;
    if (len < t) {
        memcpy(p, buf, len);
        return;
    }
    memcpy(p, buf, t);
    byteReverse(ctx->in, 16);
    MD5Transform(ctx->buf, (uint32 *) ctx->in);
    buf += t;
    len -= t;
    }
    while (len >= 64) {
    memcpy(ctx->in, buf, 64);
    byteReverse(ctx->in, 16);
    MD5Transform(ctx->buf, (uint32 *) ctx->in);
    buf += 64;
    len -= 64;
    }
    memcpy(ctx->in, buf, len);
}
void MD5Final(digest, ctx)
    unsigned char digest[16]; struct MD5Context *ctx;
{
    unsigned count;
    unsigned char *p;
    count = (ctx->bits[0] >> 3) & 0x3F;
    p = ctx->in + count;
    *p++ = 0x80;
    count = 64 - 1 - count;
    if (count < 8) {
    memset(p, 0, count);
    byteReverse(ctx->in, 16);
    MD5Transform(ctx->buf, (uint32 *) ctx->in);
    memset(ctx->in, 0, 56);
    } else {
    memset(p, 0, count - 8);
    }
    byteReverse(ctx->in, 14);
    ((uint32 *) ctx->in)[14] = ctx->bits[0];
    ((uint32 *) ctx->in)[15] = ctx->bits[1];
    MD5Transform(ctx->buf, (uint32 *) ctx->in);
    byteReverse((unsigned char *) ctx->buf, 4);
    memcpy(digest, ctx->buf, 16);
    memset(ctx, 0, sizeof(ctx));
}
#define F1(x, y, z) (z ^ (x & (y ^ z)))
#define F2(x, y, z) F1(z, x, y)
#define F3(x, y, z) (x ^ y ^ z)
#define F4(x, y, z) (y ^ (x | ~z))
#define MD5STEP(f, w, x, y, z, data, s) ( w += f(x, y, z) + data,  w = w<<s | w>>(32-s),  w += x )
void MD5Transform(buf, in)
    uint32 buf[4]; uint32 in[16];
{
    register uint32 a, b, c, d;
    a = buf[0];
    b = buf[1];
    c = buf[2];
    d = buf[3];
    MD5STEP(F1, a, b, c, d, in[0] + 0xd76aa478, 7);
    MD5STEP(F1, d, a, b, c, in[1] + 0xe8c7b756, 12);
    MD5STEP(F1, c, d, a, b, in[2] + 0x242070db, 17);
    MD5STEP(F1, b, c, d, a, in[3] + 0xc1bdceee, 22);
    MD5STEP(F1, a, b, c, d, in[4] + 0xf57c0faf, 7);
    MD5STEP(F1, d, a, b, c, in[5] + 0x4787c62a, 12);
    MD5STEP(F1, c, d, a, b, in[6] + 0xa8304613, 17);
    MD5STEP(F1, b, c, d, a, in[7] + 0xfd469501, 22);
    MD5STEP(F1, a, b, c, d, in[8] + 0x698098d8, 7);
    MD5STEP(F1, d, a, b, c, in[9] + 0x8b44f7af, 12);
    MD5STEP(F1, c, d, a, b, in[10] + 0xffff5bb1, 17);
    MD5STEP(F1, b, c, d, a, in[11] + 0x895cd7be, 22);
    MD5STEP(F1, a, b, c, d, in[12] + 0x6b901122, 7);
    MD5STEP(F1, d, a, b, c, in[13] + 0xfd987193, 12);
    MD5STEP(F1, c, d, a, b, in[14] + 0xa679438e, 17);
    MD5STEP(F1, b, c, d, a, in[15] + 0x49b40821, 22);
    MD5STEP(F2, a, b, c, d, in[1] + 0xf61e2562, 5);
    MD5STEP(F2, d, a, b, c, in[6] + 0xc040b340, 9);
    MD5STEP(F2, c, d, a, b, in[11] + 0x265e5a51, 14);
    MD5STEP(F2, b, c, d, a, in[0] + 0xe9b6c7aa, 20);
    MD5STEP(F2, a, b, c, d, in[5] + 0xd62f105d, 5);
    MD5STEP(F2, d, a, b, c, in[10] + 0x02441453, 9);
    MD5STEP(F2, c, d, a, b, in[15] + 0xd8a1e681, 14);
    MD5STEP(F2, b, c, d, a, in[4] + 0xe7d3fbc8, 20);
    MD5STEP(F2, a, b, c, d, in[9] + 0x21e1cde6, 5);
    MD5STEP(F2, d, a, b, c, in[14] + 0xc33707d6, 9);
    MD5STEP(F2, c, d, a, b, in[3] + 0xf4d50d87, 14);
    MD5STEP(F2, b, c, d, a, in[8] + 0x455a14ed, 20);
    MD5STEP(F2, a, b, c, d, in[13] + 0xa9e3e905, 5);
    MD5STEP(F2, d, a, b, c, in[2] + 0xfcefa3f8, 9);
    MD5STEP(F2, c, d, a, b, in[7] + 0x676f02d9, 14);
    MD5STEP(F2, b, c, d, a, in[12] + 0x8d2a4c8a, 20);
    MD5STEP(F3, a, b, c, d, in[5] + 0xfffa3942, 4);
    MD5STEP(F3, d, a, b, c, in[8] + 0x8771f681, 11);
    MD5STEP(F3, c, d, a, b, in[11] + 0x6d9d6122, 16);
    MD5STEP(F3, b, c, d, a, in[14] + 0xfde5380c, 23);
    MD5STEP(F3, a, b, c, d, in[1] + 0xa4beea44, 4);
    MD5STEP(F3, d, a, b, c, in[4] + 0x4bdecfa9, 11);
    MD5STEP(F3, c, d, a, b, in[7] + 0xf6bb4b60, 16);
    MD5STEP(F3, b, c, d, a, in[10] + 0xbebfbc70, 23);
    MD5STEP(F3, a, b, c, d, in[13] + 0x289b7ec6, 4);
    MD5STEP(F3, d, a, b, c, in[0] + 0xeaa127fa, 11);
    MD5STEP(F3, c, d, a, b, in[3] + 0xd4ef3085, 16);
    MD5STEP(F3, b, c, d, a, in[6] + 0x04881d05, 23);
    MD5STEP(F3, a, b, c, d, in[9] + 0xd9d4d039, 4);
    MD5STEP(F3, d, a, b, c, in[12] + 0xe6db99e5, 11);
    MD5STEP(F3, c, d, a, b, in[15] + 0x1fa27cf8, 16);
    MD5STEP(F3, b, c, d, a, in[2] + 0xc4ac5665, 23);
    MD5STEP(F4, a, b, c, d, in[0] + 0xf4292244, 6);
    MD5STEP(F4, d, a, b, c, in[7] + 0x432aff97, 10);
    MD5STEP(F4, c, d, a, b, in[14] + 0xab9423a7, 15);
    MD5STEP(F4, b, c, d, a, in[5] + 0xfc93a039, 21);
    MD5STEP(F4, a, b, c, d, in[12] + 0x655b59c3, 6);
    MD5STEP(F4, d, a, b, c, in[3] + 0x8f0ccc92, 10);
    MD5STEP(F4, c, d, a, b, in[10] + 0xffeff47d, 15);
    MD5STEP(F4, b, c, d, a, in[1] + 0x85845dd1, 21);
    MD5STEP(F4, a, b, c, d, in[8] + 0x6fa87e4f, 6);
    MD5STEP(F4, d, a, b, c, in[15] + 0xfe2ce6e0, 10);
    MD5STEP(F4, c, d, a, b, in[6] + 0xa3014314, 15);
    MD5STEP(F4, b, c, d, a, in[13] + 0x4e0811a1, 21);
    MD5STEP(F4, a, b, c, d, in[4] + 0xf7537e82, 6);
    MD5STEP(F4, d, a, b, c, in[11] + 0xbd3af235, 10);
    MD5STEP(F4, c, d, a, b, in[2] + 0x2ad7d2bb, 15);
    MD5STEP(F4, b, c, d, a, in[9] + 0xeb86d391, 21);
    buf[0] += a;
    buf[1] += b;
    buf[2] += c;
    buf[3] += d;
}
char* CMd5(const char* s)
{
     struct MD5Context md5c;
     unsigned char ss[16];
     char subStr[3],resStr[33];
     int i;
     MD5Init( &md5c );
     MD5Update( &md5c, s, strlen(s) );
     MD5Final( ss, &md5c );
     strcpy(resStr,"");
     for( i=0; i<16; i++ )
     {
         sprintf(subStr, "%02x", ss[i] );
         itoa(ss[i],subStr,16);
         if (strlen(subStr)==1) {
             strcat(resStr,"0");
         }
         strcat(resStr,subStr);
     }
     strcat(resStr,"\0");
     return resStr;
}

```
Action主要代码
```c
Action()
{
char *timen;
char * my_host;
char * str[11];
typedef long time_t; 
time_t t;  
lr_message ("Time in seconds since 1/1/70: %ld\n", time(&t));
//lr_message ("Formatted time and date: %s", ctime(&t)); 

my_host=lr_get_host_name();
//lr_output_message("%s",my_host);

strcat(my_host,"zhaokongnuan");
//lr_output_message("%s",my_host);
itoa(t,str,10);
//lr_output_message("%s",str);
strcat(my_host,str);
lr_output_message("%s",my_host);

lr_output_message("%s",CMd5(my_host));

return 0;
}

```
运行结果如下：
```bash
Virtual User Script started at : 2015-03-09 03:28:02
Starting action vuser_init.
Web Turbo Replay of LoadRunner 11.0.0 for Windows 7; build 9409 (Jan 11 2012 11:34:16)  	[MsgId: MMSG-27143]
Run Mode: HTML  	[MsgId: MMSG-26000]
Run-Time Settings file: "E:\HP\mytest\MD5\\default.cfg"  	[MsgId: MMSG-27141]
Ending action vuser_init.
Running Vuser...
Starting iteration 1.
Starting action Action.
Time in seconds since 1/1/70: 1425842882
Action.c(19): PC-20131002YAUMzhaokongnuan1425842882
Action.c(21): 4183157065b2d476d7d81f88308791f8
Ending action Action.
Ending iteration 1.
Ending Vuser...
Starting action vuser_end.
Ending action vuser_end.
Vuser Terminated.

```