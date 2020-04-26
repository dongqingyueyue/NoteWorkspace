# 如何设计一个类

1. 数据尽量放到private
2. 成员函数尽量加const
3. 参数尽量传引用,如果不允许修改参数,可以加const 引用;
4. 返回值如果可以传引用的就传引用,关键看传进去的参数存到哪个位置,如果是临时的变量,传引用后临时变量就消失了,引用也就失效了.
5. 同一个类的对象互为友元

## 学习
1. 如何打印出当前源文件的文件名以及源文件的当前行号？

参考答案： 通常使用的就是__FILE__, __LINE__，在调试函数中利用”%s”,”%ld”,打印就好了。

2. main主函数执行完毕后，是否可能会再执行一段代码，给出说明？

参考答案： crt会执行另一些代码，进行处理工作。

如果你需要加入一段在main退出后执行的代码，可以使用atexit()函数，注册一个函数。

语法：

\#include <stdlib.h>

int atexit(void (*function”)(void));

\#include <stdlib.h>

\#include <stdio.h>

void fn1( void );

int main( void )

{

atexit( fn1 );

printf( “This is executed first.\n” );

}
